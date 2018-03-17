---
layout: article
title: Optimising Small Data Structures in JRuby+Truffle
author: Chris Seaton
date: 10 August 2014
copyright: Copyright © 2014 Chris Seaton.
redirect_from: "/rubytruffle/small-data-structures/"
---

Small data structures such as arrays and hashes with just a few elements or key-value pairs are very common in Ruby programs. They're typically used to implement many patterns in idiomatic Ruby code. For example an array with just two or three elements is often used to implement a kind of multiple return, and a hash with just a couple of key-value pairs is often used to implement keyword arguments.

Of course, some of these data structures will grow as the program runs, but it's intuitively a reasonable hypothesis that many arrays and hashes begin small and stay small. If that's the case, then we want to optimise for that.

Our immediate motivation for this work was that we found that small arrays and hashes were very common in the `chunky_png` and `psd.rb` gems that we wanted to experiment with optimising using Truffle. As we explained in a [previous blog post](http://www.chrisseaton.com/truffleruby/pushing-pixels/), both of these gems use many small arrays and hashes in the inner loop of image processing routines. Examples include storing pixel values as a small `{r:, g:, b:}` hash, or creating a small array of `[n, min, max]` and sorting it to clamp a value in bounds.

In this blog post we'll show what [JRuby+Truffle](http://www.chrisseaton.com/truffleruby/) does to optimise these small data structures. It will help to explain how we are able to entirely constant fold the acid test benchmark in the previous post.

## Storage Strategies

As a Ruby programmer you may think that the `Array` and `Hash` data structures that you use every day are implemented using one fixed algorithm no matter what they contain. This is generally the case in other implementations of Ruby. MRI will always use a C `VALUE*` to implement an array, and will always use a full hash table with an array of pointers to a linked list of buckets to implement a hash. JRuby and Rubinius work in a very similar way.

The problem with this is that it's often way over the top for what we actually need. An array that only ever contains `Fixnum` values could be implemented as a `int*` array, saving us the work of tagging and untagging the value. A hash that only contains three key-value pairs doesn't need all of the indirection of a list of buckets and it could be better implemented as a single flat array of keys and values.

We have to meet the contract that the `Array` and `Hash` classes specify, which includes the explicit contract such as the methods they provide, and the implicit contract that is the algorithmic complexity of these methods. But as long as we can do that, we're free to implement these data structures however we like.

JRuby+Truffle uses a novel variant of a technique that is already in use in the various PyPy projects, called strategies, [published by Carl Friedrich Bolz, Lukas Diekmann and Laurence Tratt](http://tratt.net/laurie/research/pubs/html/bolz_diekmann_tratt__storage_strategies_for_collections_in_dynamically_typed_languages/). The idea is that we have a range of different ways of implementing each data structure. Each array and hash references the strategy that they're currently using and you delegate operations to the current strategy. As the program runs, objects can change their strategy if the current one isn't appropriate any more.

In JRuby+Truffle our most simple strategy is the `null` strategy. If a hash or array is empty, we don't allocate any storage at all. Many operations on a `null` strategy are often no-ops returning constant values.

If your array does include elements we have strategies to implement it as Java `int[]`, `long[]` and `double[]`. Storing immediate objects in these arrays requires no boxing or tagging. In some loops this means we can stop allocating objects all together, which is definitely a key part of optimising Ruby. If you have a big heterogeneous array with objects of many different types, you'll eventually end up using the last-resort `Object[]` strategy. We can put anything in here, but we'll have to box any immediate objects, and that may mean allocating them on the heap. Our last-resort strategy is the one that JRuby and MRI use all of the time.

At the moment our implementation of hashes currently includes just two simple specialisations as well as `null` - one that uses a full Java linked hash table, and a simpler case where we represent the hash as a flattened Java `Object[]`. That is, a hash `{a: 1, b: 2, c: 3}` will be represented as `new Object[]{new Symbol('a'), new Integer(1), new Symbol('b'), new Integer(2), new Symbol('c'), new Integer(3)}`. Instead of hashing a key and doing a lookup we can do a simple linear search through this array. As long as we only use this specialisation up to some limited maximum size we can still meet the *O(1)* lookup time implicit contract.

The PyPy paper referenced above describes how they have some 17 storage strategies, so we probably have some more work to do. We'll introduce more as we find applications or benchmarks to exercise them.

## Specialising for Small Arrays and Hashes

JRuby+Truffle includes extra strategies for arrays and hashes that we arbitrarily consider to be 'small'. That's currently less than four array elements, or four key-value pairs. We found that this constant worked well for our benchmarks, but it's configurable and we'll keep experimenting with it as we run other programs.

For these objects, we always allocate enough space for them to contain three elements of key-value pairs, even if they're not using them. When we know that's always going to be the case, we can use simpler and more uniform operations that don't depend on the size of the array.

Consider the case of a `Array#each`. For an array of arbitrary size, this is implemented something like this:

```java
for (int n = 0; n < array.length; n++) {
  block.call(array[n]);
}
```

If the array is using the small strategy, we can make the loop iteration count constant at our maximum size, and include a test for the actual size of the array inside the loop.

```java
for (int n = 0; n < 3; n++) {
  if (n < array.size)
    block.call(array[n]);
}
```

The key difference here is that as this array is of a constant size, it can be easily unrolled by the compiler. In Truffle we call this loop explosion, and it can be forced through an annotation to ensure that it's performed. This means the compiler can automatically turn the loop into a series of loop bodies without any branches or loop variables.

```java
if (0 < array.size) block.call(array[0]);
if (1 < array.size) block.call(array[1]);
if (2 < array.size) block.call(array[2]);
```

Unlike with an array of variable number of iterations, these three statements are now clearly independent and can be pipelined much more effectively by a processor than a loop with branches.

We could probably achieve the same effect by specialising `Array#each` for the number of elements in an array, but the advantage of our current approach is that it's very simple.

Another example is `Array#sort`. Java has a high performance implementation of array sorting routines, which happen to use Tim sort. This is very good code that performs well on a wide range of arrays, but it's also complicated. When compiling code, simple but theoretically slow is often better than complex but theoretically fast. When we're working with a small array we want as simple as possible, so we actually use our own simple implementation of insertion sort. You may not think of that as a fast algorithm, but it's simple enough to inline, which means the compiler can see it all and optimise it alongside the rest of our code. If we used the more complicated Tim sort, we would have to call it as a normal method and the compiler would stop optimising.

The same approach works for hashes as well. The `Hash#merge` method returns a new hash that includes the contents of two hashes, with priority given to the first. Here there are two data structures involved, and they may have different strategies. We have written specialised versions of `Hash#merge` for some specific combinations of two strategies that have obvious optimisations. For example merging a `null` hash with one that uses an array specialisation can be implemented as a very simple array copy of the second hash's array.

```ruby
public RubyHash mergeObjectArrayNull(RubyHash hash, RubyHash other) {
    final Object[] store = (Object[]) hash.getStore();
    final Object[] copy = Arrays.copyOf(
      store, RubyContext.HASHES_SMALL * 2);
    return new RubyHash(getContext().getCoreLibrary().getHashClass(),
      hash.getDefaultBlock(), copy, hash.getStoreSize());
}
```

That's an awful lot simpler than enumerating one Java `HashMap` and looking up in another. In fact, it doesn't even contain any loops, just a memory copy which can be vectorised.

The Truffle framework includes excellent support for declaratively adding new specialisations for particular situations. The annotation for that merge specialisation is:

```ruby
@Specialization(guards = {"isObjectArray", "isOtherNull"}, order = 1)
```

This says that this specialisation applies if the first hash is an object array, and the second is `null`. The ordering argument is about controlling the latice that these specialisations form - we want any switch to other specialisations to be monotonic. Also take a look at the specialisations for [`Array#each`](https://github.com/jruby/jruby/blob/bd12275b137b1daafc28a058b16ea91a361dfd15/core/src/main/java/org/jruby/truffle/nodes/core/ArrayNodes.java#L1323-L1491) and [`Array#sort`](https://github.com/jruby/jruby/blob/bd12275b137b1daafc28a058b16ea91a361dfd15/core/src/main/java/org/jruby/truffle/nodes/core/ArrayNodes.java#L3036-L3162).

The great thing about Truffle is that you just have to specify a list of these specialisations, and it will automatically use the correct one, or will combine multiple specialisations if your method is polymorphic.

## Allocation Removal

We've said that in a compiler it's often better to have simpler code that is inline, rather than more complicated code which you have to call in a library. A key reason for this is allocation removal, or as we talk about in Graal, scalar replacement of aggregates. That means instead of allocating an object on the heap, we allocate it on the stack, or even in registers.

The compiler can do this if it can see where an object is allocated, where it is used, and that it can't be seen by anyone else who might expect it to be in the heap (this test is called escape analysis). We described above how our special strategies are designed to produce simple code for simple cases. This helps the allocation removal as our simpler specialisations are generally much more amenable to allocation removal. They're also faster and use less memory to compile.

## Dataflow Analysis

Dataflow analysis of data structures follows on from allocation removal. If the compiler has determined that a data structures does not escape and it has removed the allocation of it, we can then say that that if we put a value into the data structure and then take it out again, perhaps by a method like `Array#each` or a more complicated method like `Array#sort`, we can replace the code that takes the value out with the value we were going to put in, and entirely skip the part that actually uses the data structure.

This way we can follow values as they pass through various core library methods and in and out of data structures. You may think that you're creating a temporary array and sorting it, but in the final code you really have an inline version of insertion sort that's working on registers and never even allocates an array. If the values you are sorting are constant, or some of them are constant, those constants are also folded through the operations that we've inlined.

We get all of this for free from Truffle and Graal - the only work we need to do in the JRuby backend is to make our code amenable to these optimisations, and that's what the strategies do.

## Summary

Truffle is all about specialisation. Here we've talked about specialisations for the way data structures are implemented. For simpler cases we use simpler strategies, which allow for simpler implementations of operations on them. That simpler code is easier to optimise and produces better machine code, and we end up with a faster program at the end.

The acid test benchmark presented in our [previous blog post](http://www.chrisseaton.com/truffleruby/pushing-pixels/) is fast because of the use of several of these strategies working together.

Of course it's not the case that JRuby and Rubinius haven't implemented strategies because they haven't thought of it. I know that Charles Nutter has been considering more efficient implementations of small hashes for a while, and 9k recently gained [an optimisation for empty hashes](https://github.com/jruby/jruby/pull/1676) that is also in MRI. But in general JRuby and Rubinius will always fully heap-allocate arrays and hashes in their current implementations, and almost always box or tag immediate objects like `Fixum` and `Float`. If you're allocating on the heap, the speedup from these optimisations will be relatively minimal.

{% include trufflerubylinks.html %}
