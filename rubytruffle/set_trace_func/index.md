---
layout: article
title: Tracing With Zero Overhead in JRuby+Truffle
author: Chris Seaton
date: 15 June 2014
image: trace.png
image_alt: Tracing
copyright: Copyright © 2014 Chris Seaton.
---

## `set_trace_func`

The new [Truffle backend in JRuby](https://github.com/jruby/jruby/wiki/Truffle) gives us an extremely fast implementation of Ruby, but as well as being fast its architecture allows us to tackle some of the features of Ruby that are traditionally seen as being very hard to implement, often in a refreshingly simple way.

Ruby's core library has a method [`Kernel#set_trace_func`](http://ruby-doc.org/core-2.1.2/Kernel.html#method-i-set_trace_func) that allows you to install a method to be called as the interpreter traces through your program. It's called on every source code line, every time you call or return from a method, and on some other events. The method is passed some information about the program state, including a [`Binding`](http://ruby-doc.org/core-2.1.2/Binding.html) object - an object like a hash that holds all the local variables in scope at the point the method was called.

{% highlight ruby %}
set_trace_func proc { |event, file, line, id, binding, classname|
  puts binding.local_variable_get[:x]
}
{% endhighlight %}

It's not hard to understand why this is often a very expensive feature to implement. Every single source code line gains an extra method call, and we have to construct an object with a copy of all the local variables in scope, both locally and lexically, on each line to pass to the call. Charles Nutter identified `set_trace_func` as one of the [features in Ruby you should implement before making any claims about how fast your Ruby is](http://blog.headius.com/2012/10/so-you-want-to-optimize-ruby.html), so we made sure we tackled it in the first few months of development of the Truffle backend.

Most of the other implementations of Ruby support `set_trace_func` to some degree. MRI always supports it, of course. JRuby supports it only if you use the `--debug` flag, which unfortunately also disables JIT compilation, so any program using `set_trace_func` is going to run very slowly in the interpreter. Rubinius doesn't support `set_trace_func`, but it does have a debug API on top of which some people have [built support](https://github.com/rocky/rbx-tracer). Unfortunately Rubinius runs debugger code in a special debug interpreter loop instead of using the JIT so performance is minimal. Topaz supports `set_trace_func` without any flag, using a technique similar to the one we're about to describe, but in our opinion it's a little more complicated and doesn't achieve the same performance.

## The Truffle Implementation

Truffle is an AST interpreter framework. That means we parse Ruby to a tree of expressions, and we execute it by walking the tree as in the visitor pattern. For example the `if` node executes the condition as a child node, then depending on the value that node returns it will either execute the `then` node or the `else` node. We achieve high performance by dynamically compiling a partial evaluation of this interpreter - partial evaluation is a kind of very aggressive inlining and constant folding.

To implement `set_trace_func` we added a new kind of node - a wrapper node. It has a single child node that it always executes, but only after calling a callback.

The `set_trace_func` wrapper node's callback simply calls the trace method if one is installed. But then if the `set_trace_func` node is constantly checking if there is a trace method, doesn't that slow things down? The trick is that the node has an inactive state and an active state. The inactive state does nothing and just executes the child, so it has no overhead. In fact exactly the same machine code is generated as if the node was never there. The active node calls the trace method. We use the same [method dispatch mechanism](how-method-dispatch-works-in-jruby-truffle) as we've discussed before, so we can inline the trace method.

To switch between the two nodes, we use the dynamic deoptimisation functionality that is already in the JVM, and that we are already using for method dispatch. When we want to install a `set_trace_func` method we stop all running Ruby methods, whether they have been compiled or not, and replace the inactive nodes with active nodes. We can remove the trace method by again replacing the active nodes with inactive versions.

Adding support for `set_trace_func` was not much more work than adding the [trace node](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/truffle/nodes/debug/TraceNode.java) and the code to [install or remove a trace method](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/truffle/runtime/subsystems/TraceManager.java).

The implementation isn't quite complete - it calls the trace method for every line, but not for method calls yet. It also doesn't pass in all the parameters the method expects, but it does construct the full binding, which is the expensive bit.

## Other Applications

By this stage you may have realised that what we really have is a general mechanism for installing and removing arbitrary code at any location in the program source code, with no overhead for enabling this feature, and still very low overhead when it's actually used. That's an exciting tool to have, and we can think of lots of ideas where we can apply it. Hot code patching, profiling, coverage tools and aspect oriented programming are all applications we thought of, but we decided to look at writing a debugger first.

When `set_trace_func` is used, a method is installed on every line of the source program. If we modify it so that you can specify which line to install it on, we can build a line breakpoint. It could look something like this:

{% highlight ruby %}
set_trace_func_on_file_line file, line, proc { |event, file, line, id, binding, classname|
  Debug.break
}
{% endhighlight %}

We can then make that into a conditional breakpoint, such as one that breaks if `x > 14`:

{% highlight ruby %}
set_trace_func_on_file_line file, line, proc { |event, file, line, id, binding, classname|
  Debug.break if binding.local_variable_get(:x) > 14
}
{% endhighlight %}

The body of the breakpoint will be inlined in the same way as a normal `set_trace_func` method is, so although it looks like we're doing a really expensive allocation of a `Binding`, it's actually all in the same compilation unit, and through partial evaluation it becomes just a normal access to the local variable. It's the same as if you were to manually write the breakpoint in the source code. However you can install these breakpoints in optimised code that is already running, and they will run as fast as before.

## How Fast Is It?

We compared the performance of `set_trace_func` across different implementations of Ruby. We looked at the overhead of enabling `set_trace_func`, and of actually using it, by looking at how many times slower a simple benchmark ran under these conditions. This data is just a simple summary - we'll show you where to find all the detail such as our methodology and statistical error later. For MRI and Topaz we had to manually disable `set_trace_func` by modifying the source code, as there isn't a flag for it.

|                                    | MRI      | JRuby    | Topaz  | JRuby+Truffle |
|------------------------------------|----------|----------|--------|---------------|
| Cost of enabling `set_trace_func`  | 0.1x     | 3.9x     | 0.0x   | 0.0x          |
| Cost of using `set_trace_func`     | 24.6x    | 199.4x   | 661.1x | 4.0x          |

Table: Slowdown using trace functionality

MRI shows a low overhead for `set_trace_func`, but that's because it's already so slow that an extra check on each line doesn't make a difference. JRuby has a large overhead because we have to disable the JIT with the `--debug` flag to use tracing. The overhead of actually calling a trace method is huge as it has to construct a binding. JRuby+Truffle shows zero overhead for enableing `set_trace_func`. It also shows a very reasonable overhead for actually using it - we can very efficiently allocate a binding. Topaz doesn't seem to be optimised for the case of a trace method being installed, but to be fair to them they might just not have looked at this code path yet.

We also evaluated our prototype debugger implementation against `ruby-debug`, `jruby-debug` (the two native C and Java extensions for debugging) and the Rubinius native debugger. Topaz doesn't yet have a debugger. We looked at the cost of enabling debugging, placing a breakpoint on a line that is never taken (so the cost is for having the breakpoint at all), a breakpoint with a constant condition that tested in the inner loop of the benchmark (so it should be able to be optimised away), and a breakpoint with a non-constant but simple condition. Again this is just a summary, see at the end of the post for the detail.

|                                    | MRI      | Rubinius | JRuby  | JRuby+Truffle |
|------------------------------------|----------|----------|--------|---------------|
| Enabling debugging                 | 4.9x     | 4.6x     | 4.6x   | 0.0x          |
| Breakpoint on a line never taken   | 5.1x     | 9.3x     | 46.5x  | 0.0x          |
| Breakpoint with constant condition | 30.6x    | 425.1x   | 44.6x  | 4.9x          |
| Breakpoint with simple condition   | 41.2x    | 423.7x   | 95.8x  | 9.0x          |

Table: Slowdown using debugging functionality

Here we can really see the scale of the differences - multiple orders of magnitude in terms of run time.

Keep in mind this is all on top of the significant performance lead JRuby+Truffle already has on all other implementations of Ruby.

## More Details

A more formal description of how we implemented and evaluated `set_trace_func` and our prototype debugger was the subject of the paper [Debugging at Full Speed](http://www.lifl.fr/dyla14/papers/dyla14-3-Debugging_at_Full_Speed.pdf) by [Chris Seaton](http://www.chrisseaton.com/), [Michael Van de Vanter](http://vandevanter.net/mlvdv/) and [Michael Haupt](https://labs.oracle.com/pls/apex/f?p=labs:bio:0:44), all of Oracle Labs, presented at the 2014 Workshop on Dynamic Languages and Applications in Edinburgh. The paper includes a full evaluation and a link to the experimental harness we built so you can reproduce the results.

If you'd like to know any more, please get in touch!

{% include jrubytrufflelinks.html %}
