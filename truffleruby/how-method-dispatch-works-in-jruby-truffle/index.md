---
layout: article
title: How Method Dispatch Works in JRuby+Truffle
author: Chris Seaton
date: 11 March 2014
copyright: Copyright © 2014 Chris Seaton.
image: tree.png
image_alt: AST
redirect_from: "/rubytruffle/how-method-dispatch-works-in-jruby-truffle/"
---

## Method Calls in Ruby

Method calls are a very important part of any implementation of the Ruby programming language. Unlike a language like Java, Python or JavaScript, all properties, operators and many control structures in Ruby are actually method calls. When you write `x + y`, Ruby interprets that as `x.+(y)`, that is, a call to a method named `+`. When you write `x.y = z`, Ruby interprets that as `x.y=(z)`, that is, a call to a method named `y=`. Also, when you write:

```ruby
for x in y
  puts x
end
```

Ruby will interpret this as:

```ruby
y.each do |x|
  puts x
end
```

That is, a call to a method named `each`.

With almost everything being a method call, the speed at which we can execute these calls becomes very important if you want to have a high performance Ruby implementation.

This article is going to talk about how the new Truffle backend in JRuby implements a very high performance method dispatch system on top the JVM. In fact I'll show you that it's so efficient that with important types like `Fixnum` we entirely optimize away the method call and how a call to something like `Fixnum#+` becomes just a few inline instructions.

You won't need any deep technical understanding of JRuby, Java, the JVM or Truffle to follow this article. I'll explain all the concepts we build on as I go. Hopefully you'll enjoy some of the technical depth, even if you don't understand it all. As method dispatch is such an important part of a JRuby+Truffle, seeing how it works will give you a good overview of how Truffle in general works. If you want some more background on the Truffle backed in JRuby there is an [announcement blog post](http://www.chrisseaton.com/truffleruby/announcement/) and a [wiki page](https://github.com/jruby/jruby/wiki/Truffle) with lots of pointers to more information.

If you want to try out the practical examples, you'll need to install a development version of JRuby 9000 and the Graal VM. If you are on Linux or Mac and use rbenv with the latest ruby-build this is easy:

```bash
rbenv install jruby-9000+graal-dev
rbenv shell jruby-9000+graal-dev
```

If you aren't using those tools, follow the instructions on the [JRuby Truffle wiki page](https://github.com/jruby/jruby/wiki/Truffle) to get started.

## Method Lookup

All Ruby implementations work out which method to call in the same way, using the same method lookup algorithm. For a clear description of how we do this, see the excellent book by Pat Shaughnessy, [Ruby Under a Microscope](http://patshaughnessy.net/ruby-under-a-microscope). For a given receiver object on which we are calling the method, we first look in the object's class. If we don't find the method there, we look in the superclass of that class, then the superclass of the superclass and so on. Metaclasses, included modules and so on fit into the search algorithm as well but we'll keep it simple. At some point we'll either find the method to call, or we'll resort to calling `#method_missing` instead.

## Method Caches

Even based on that simplified description, you can imagine the complexity of the method lookup algorithm. All Ruby implementations store the methods in each class in some kind of hash table, so each level of the lookup is a hash lookup. Hash table lookups can be fast, but if everything you do in your program ended up being a hash lookup, it's still going to make your program very slow.

To solve that we don't do the method lookup each time we call a method. Instead we predict that the method we called last time is likely going to be same method we want the next time we make that call and cache it to use it again. 

### Global Caches

The most basic type of method cache is a global one. For example, MRI has a global hash of pairs of classes and method names to the methods. At a method call it can lookup just once in this global cache and avoid possibly having to make repeated lookups in multiple classes.

### Inline Caches

The problem with a global cache is that it still involves a lookup - a lookup in the cache hash. If we had a single one-entry cache per method call in your program we wouldn't need to lookup in anything, we'd just need to read the value. This is called an inline cache - because the cache is inline as part of the call instruction.

### Cache Invalidation

Ruby is a very dynamic language - methods can be added, removed or redefined at any point in a program. This means that the methods that we cached for a particular class and name might have become wrong. This can happen at absolutely any point in your program. When it does, the caches we've made might become invalid. A global cache is a simple case - we can just clear the whole thing when methods change. Inline caches are more complicated, as we first have to find all the inline caches in the program, and ideally we only want to clear those that are actually now invalid.

MRI does this by storing a version number for each class. When the cache is set we store the version number. To invalidate the cache we increase the version number. Each time MRI reads the cache it checks the version is unchanged. This is the cache check ([source on GitHub](https://github.com/ruby/ruby/blob/e9328e699f/vm_insnhelper.c#L828)):

```c
  if (LIKELY(GET_GLOBAL_METHOD_STATE() == ci->method_state && RCLASS_SERIAL(klass) == ci->class_serial)) {
/* cache hit! */
return;
  }
```

And the invalidation ([source on GitHub](https://github.com/ruby/ruby/blob/e9328e699f/vm_method.c#L59)), which just sets a new version number:

```c
static void
rb_class_clear_method_cache(VALUE klass)
{
    RCLASS_SERIAL(klass) = rb_next_class_serial();
    rb_class_foreach_subclass(klass, rb_class_clear_method_cache);
}
```

## Method Dispatch in JRuby+Truffle

Method lookup works exactly the same in JRuby+Truffle as it does in all the other implementations, using the standard lookup algorithm. However our implementation of caches and dispatch is pretty unique - we implement a system of polymorphic inline caches via abstract syntax tree rewriting with dynamic deoptimization for invalidation. Some of those techniques might sound very advanced but we'll show how Truffle makes them very straightforward.

JRuby+Truffle parses your Ruby code (using JRuby's parser) into a tree of nodes - an abstract syntax tree. Unlike other implementations this is as far as we go - we don't explicitly emit bytecode or machine code. Our implementation is always interpreting the AST, and Truffle and Graal convert this to machine code on our behalf.

When your Ruby is read by JRuby+Truffle we create a `RubyCallNode` for each call site - location where you call a method - in your code. The `RubyCallNode` has the name of the method you want to call, any arguments and maybe a block. It executes these and then calls the first of one or more dispatch nodes. Each `CachedDispatchNode` is designed to handle a single class of receiver. They store the class they're designed to handle and the method they call, making an single inline cache entry.

This is code for a single `CachedDispatchNode` ([GitHub source](https://github.com/jruby/jruby/blob/07d4fa828b92c511ae45f1705929955cd2b15dfe/core/src/main/java/org/jruby/truffle/nodes/call/CachedBoxedDispatchNode.java)):

```java
public Object dispatch(VirtualFrame frame,
        RubyBasicObject receiverObject,
        RubyProc blockObject,
        Object[] argumentsObjects) {
    
    // Check the lookup node is what we expect

    if (receiverObject.getLookupNode() != expectedLookupNode) {
        return next.dispatch(frame,
          receiverObject, blockObject, argumentsObjects);
    }

    // Check the class has not been modified

    try {
        unmodifiedAssumption.check();
    } catch (InvalidAssumptionException e) {
        return respecialize("class modified", frame,
          receiverObject, blockObject, argumentsObjects);
    }

    // Call the method

    return method.call(frame.pack(), receiverObject,
      blockObject, argumentsObjects);
}
```

We've already executed the receiver, the block and the arguments. This `CachedDispatchNode` then just checks that this is the class we were expecting (we talk about `LookupNode` rather than class to support the complex Ruby object model). If it wasn't what we expected it tries the next entry in the cache - the next `DispatchNode` in the chain.

We then check the class hasn't been modified since we created the cache entry. This is like how  MRI checks the version number, but we're using an object rather than a version number. We don't compare numbers, we just see if the object is still marked as valid by calling `check`, which throws an exception if it isn't valid.

We then simply call the method as normal. So the only work we have to do before calling the method is one reference comparison on that `LookupNode`, and a call to `check` on this `Assumption` object. Later on I'll show how both of these checks are entirely removed by the JIT compiler.

If your call site only ever sees one class of object you'll have only one dispatch node, if you have more you'll have a chain of them each handling calls to a different class. That's makes the inline cache polymorphic - it can handle more than one class.

There's a special `DispatchNode` called `UninitializedDispatchNode` that is always at the end of the chain. Instead of expecting a particular class and caching a method, this node runs the lookup algorithm and replaces itself with a new `DispatchNode` that caches the result of that lookup.

Take some simple code like adding two `Fixnum` objects together: `a + b`. That's syntactic sugar for a method call: `a.+(b)`. When it leaves the parser the `RubyCallNode` in that code will look like this:

```
RubyCallNode
    name: "+"
    receiver: local variable a
    arguments: [ local variable b ]
    block: nil
    dispatch:
        UninitializedDispatchNode
```

The first time this code runs we'll go straight to the `UninitializedDispatchNode`. That will do full method lookup. We'll find that the method we want is `Fixnum#+`. The uninitialized node will replace itself with a new dispatch node that remembers that when the receiver is a `Fixnum`, the method we want to call is `Fixnum#+`. The `UninitializedDispatchNode` will follow that new node in the chain, ready to handle different classes.

The code will now look like this:

```
RubyCallNode
    name: "+"
    receiver: local variable a
    arguments: [ local variable b ]
    block: nil
    dispatch:
        CachedDispatchNode
            class: Fixnum
            method: Fixnum#+
            next:
              UninitializedDispatchNode
```

If we called the same code with two `Float` objects, we'd add a new node to the chain, and then we'd have two cache entries able to handle both cases:

```
RubyCallNode
    name: "+"
    receiver: local variable a
    arguments: [ local variable b ]
    block: nil
    dispatch:
        CachedDispatchNode
            class: Fixnum
            method: Fixnum#+
            next:
                CachedDispatchNode
                    class: Float
                    method: Float#+
                    next:
                      UninitializedDispatchNode  
```

There are some other details to handle corner cases. If the dispatch chain gets too long because it's seen too many classes we replace the whole thing with a node that does lookup every time it's called. We also handle primitive Java classes such as `Integer` and `Double` slightly differently than we do full Ruby objects like `Hash`. Finally we have code to handle `#method_missing`. I've ignored these fine details for simplicity. I've also slightly simplified these code trees from the real structures we use. To get the reality you can use this program:

```ruby
def add(a, b)
  a + b
end

loop do
  add(14, 2)
  add(14.5, 2.25)
end
```

And run it with these options:

```bash
ruby -X+T -J-G:+TraceTruffleExpansion test.rb
```

`TraceTruffleExpansion` means to list all the Java methods involved in interpreting the Ruby method.

## Inlining

Those `CachedDispatchNodes` at some point call another Ruby method. Thanks to caching, that can always be the same method, but we're still doing the work of making a call, which means managing arguments and a return value. The other disadvantage is that a lot of compiler optimisations don't work across method boundaries, as methods are compiled separately. Since we already worked out that we're normally going to be calling just one method that we cached, we might as well put the code for the method directly at the call site, especially if the method is something as simple as `Fixnum#+`.

Truffle inlines methods by taking the AST that represents the method we are inlining, and simply copying it and making it a child of the dispatch node.

## All the Way Down to Machine Code

The data structures we've illustrated above look quite complicated. Even a simple call is going to involved multiple Java objects to implement it, and to execute it we're going to have to go through methods in `CallNode` and `DisaptchNode`. So we're trying to create a fast implementation of method calls, but it's already using multiple Java method calls before we actually call the Ruby method.

This is where the Graal compiler comes in. Graal is a JIT compiler for Java, implemented in Java. This isn't an exercise for the sake it - by implementing the JIT in Java we can provide a full interface to the program it is compiling, rather than just the one way interface of sending bytecode. Talking to a compiler on its terms is complicated, so the Truffle framework does most of the work on our behalf, and we just have to specify a few extra commands to control how it compiles our Ruby interpreter.

The first thing to know about Truffle is that it generates a single machine code method for each Ruby method. The technique we use is called *partial evaluation*, but if you want you can just think about it as aggressive inlining. In fact Truffle will continue to inline until we tell it to stop. When we inline, Java method calls disappear and it's as if the code was all written in one big method, so the fact that our Ruby method calls involve multiple Java method calls isn't a problem.

Lets take a look at the assembly generated for a real Ruby method.

```ruby
def add_and_subtract(a, b, c)
    a + b - c
end

loop do
    add_and_subtract(14, 12, 2)
end
```

We run the method in a loop so that the JIT will be run. We're not doing anything with the result, but like all Ruby interpreters, we're not quite strong enough yet to completely elide the computation - Ruby has complicated side effect semantics, so this is non-trivial.

We'll run this using the Truffle backend, and we'll ask the JVM to print out machine code as it's generated. `UnlockDiagnosticVMOptions` turns on some special JVM options that are normally hidden to avoid confusion. `CompileCommand=print` turns on printing machine code, and `executeHelper` refers to an internal method in Graal that is always the starting point of compilation.

```bash
ruby -X+T -J-XX:+UnlockDiagnosticVMOptions -J-XX:CompileCommand=print,*::executeHelper test.rb
```

The output from this command is complicated, as it also involves boxing at the start and unboxing at the end, but I'll just point out what's happened to the `a + b - c`. If you dig through the machine code you'll see something like this somewhere in the middle (the addresses will be different of course):

```
0x000000010f620102: add    %esi,%eax
0x000000010f620104: jo     0x000000010f620208
0x000000010f62010a: mov    0xc(%rbx),%esi
0x000000010f62010d: sub    %esi,%eax
0x000000010f62010f: jo     0x000000010f620201
0x000000010f620115: cmp    $0xffffffffffffff80,%eax
0x000000010f620118: jl     0x000000010f62015b
0x000000010f62011e: cmp    $0x7f,%eax
0x000000010f620121: jg     0x000000010f62015b
```

If you don't know x86_64 machine code, just look at the instruction names. There's an `add` and a `sub` instruction, which directly correspond to the `+` and `-` operations that we wrote. These instructions were in the methods `Fixnum#+` and `Fixnum#-`, but because they're so simple we've decided to inline them at the point where they're called. `add` and `sub` instructions work on 32 bit integers, so we've also detected that we've only called this method with `Fixnum` types and specialised for just that case. After each of the instructions there's a `jo` instruction. That stands for *jump if overflowed* (jump is like `goto`). This is because if a result can't fit in a `Fixnum`, Ruby will automatically convert to a `Bignum` for you. The code that actually handles that is somewhere else that we jump to, because it rarely happens. Following the `add` and `sub` we have `cmp` (compare) and `jl` and `jg` (jump if less, and jump if greater). This checks if the value fits into a `Fixnum`, which is actually slightly smaller than a 32 bit integer, so we need to do a check as well as handling the overflow.

This short sequence of machine code shows everything that we've talked about so far. We've inlined the `Fixnum#+` and `Fixnum#-` methods into the `add_and_subtract` method. We've also gotten rid of all the Java method boundaries. We talked earlier about two checks, and there's no sign of them anymore. We could get rid of the check that the class was what the cache expected, because the local variables have been typed as `Fixnum` from the start of the method. We've also gotten rid of the `check` method call on the `Assumption` object. I'll explain how we did that in the next section.

The result is that we've produced machine code that looks like code that I would write if I were translating Ruby to x86_64 by hand. This is what makes the Truffle backend in JRuby fast.

If you're interested, you also can compare the code we generate against the JRuby `invokedynamic` backend, using the command:

```bash
ruby -J-XX:+UnlockDiagnosticVMOptions -J-XX:+PrintAssembly test.rb
```

Look for the method `test.method__0$RUBY$add_and_subtract`. You can find the `add` and `sub` instructions, but you'll find they're about 40 instructions apart. The instructions between deal with boxing and bookkeeping of the `invokedynamic` system.

We can also look at what Rubinius produces, using the command:

```bash
rbenv shell rbx-2.2.2
ruby -Xjit.dump_code=4 test.rb
```

Look for the method `_X_Object#add_and_subtract`. Again we'll find `add` and `sub`, and in this case they're a lot closer together. Around them is code for handling tagging - that is, packing a `Fixnum` value into a pointer.

## Cache Invalidation

We've explained how method caches are created, how we get rid of the expensive Java method calls involved in all these data structures, and how we've generated minimal machine code, but what happens when methods are added, removed or redefined?

The approach we use in the Truffle backend looks similar to MRI. Truffle provides a utility for doing this kind of thing, called the `Assumption` class. An `Assumption` object is a bit like a version number, but it's binary - either it's valid or not. Instead of storing a number in the inline cache, and increasing it when the cache is invalidated, we store the `Assumption` object when we create the cache, and we make it as invalid and create a new object when the cache is invalidated.

However when I showed the machine code above, there was no mention of an `Assumption` object, so where has it gone? When your Ruby method is compiled, Truffle removes all references to the `Assumption` objects. Instead, when we want to clear the cache, we mark the `Assumption` objects as being invalid and *deoptimize* all the machine code that used those `Assumption` objects. That means the runtime goes in and stops the machine code from running, and puts us back in the interpreter, where we will update the cache.

So instead of actively checking that a cache is still valid, we stop machine code that has an invalid cache. We go from a lazy check to an active one, and one that has no overhead while the cache is still valid. That's exactly what we need for a fast implementation of method dispatch - remove the cost from the fast path, and put it onto the less common path.

Deoptimization is an incredibly complicated technique, so I won't try to explain it in depth here, but Truffle simplifies using it down to a method `Assumption.invalidate`. You can see the invalidation being triggered being used here ([source on GitHub](https://github.com/jruby/jruby/blob/07d4fa8/core/src/main/java/org/jruby/truffle/runtime/core/RubyModule.java#L318)):

```java
public void newVersion() {
    unmodifiedAssumption.invalidate();

    // Make dependents new versions

    for (RubyModule dependent : dependents) {
        dependent.newVersion();
    }
}
```

Going back to the machine code we showed, those `jo` instructions which deal with uncommon cases like overflow also cause deoptimization. In general our approach to things which rarely happen is to not compile them, and to deoptimize if we actually encounter them.

## Summary

Looking at the simple example of `add_and_subtract` and following it all the way down into the generated machine code we've showed how Truffle and Graal allow us to produce a really efficient implementation of method dispatch for Ruby. We showed how the partial evaluation in Truffle removes Java method call overheads, how the type profiling and inlining allows us to use primitive machine instructions for operations on Ruby objects like `Fixnum`, and how the `Assumption` object removes the need to check that caches are still valid.

There's a lot of work going on behind the scenes in Truffle, Graal and the JVM to enable the techniques we've discussed, but the interfaces that Truffle provides makes using them easy. For example, the `Assumption` object has only two methods that you need to understand - `check` and `invalidate`, and that provides access to one of the most powerful techniques in the JVM, deoptimization.

If you want to get involved in Truffle development in JRuby, or want to know more about what we've talked about, feel free to get in touch with me.

{% include trufflerubylinks.html %}
