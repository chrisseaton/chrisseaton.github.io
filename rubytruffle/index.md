---
layout: section
title: JRuby+Truffle
copyright: Copyright © 2016 Chris Seaton
---

<p style="text-align: center">
<img alt="Ruby Logo (Copyright (c) 2006, Yukihiro Matsumoto. Licensed
under the terms of Creative Commons Attribution-ShareAlike 2.5.)" src="ruby.png" width="96" height="96">
<img alt="JRuby Logo (Copyright (c) 2011, Tony Price. Licensed
under the terms of Creative Commons Attribution-NoDerivs 3.0 Unported (CC BY-ND 3.0))." src="jruby.png" width="96" height="96">
</p>

JRuby+Truffle started as [my](..) internship project at [Oracle
Labs](http://labs.oracle.com/) in early 2013. It is an implementation of the
[Ruby](https://www.ruby-lang.org/) programming language on the JVM, using the
[Graal dynamic compiler and the Truffle AST interpreter
framework](http://openjdk.java.net/projects/graal/). JRuby+Truffle can achieve
peak performance well beyond that possible in JRuby at the same time as being a
significantly simpler system. In early 2014 it was open sourced and integrated
into [JRuby](http://jruby.org/).

This page links to the literature and code related to the project. Note that any
views expressed are my own and not those of Oracle.

<i class="icon-trophy"></i> Nominee, Ruby Prize, 2016

----

# Blog Posts and Articles

*   [Flip-Flops &mdash; the 1-in-10-million operator](flip-flops/). Do people actually use flip-flops?

*   [Deoptimizing Ruby](deoptimizing/). What deoptimization means for Ruby and how JRuby+Truffle implements and applies it.

*   [Very High Performance C Extensions For JRuby+Truffle](cext/). How JRuby+Truffle supports C extensions.

*   [Optimising Small Data Structures in JRuby+Truffle](small-data-structures/). Specialised optimisations for small arrays and hashes.

*   [Pushing Pixels with JRuby+Truffle](pushing-pixels). Running real-world Ruby gems.

*   [Tracing With Zero Overhead in JRuby+Truffle](set_trace_func/). How JRuby+Truffle implements `set_trace_func` with zero overhead, and how we use the same technique to implement debugging.

*   [How Method Dispatch Works in JRuby/Truffle](how-method-dispatch-works-in-jruby-truffle/). How method calls work all the way from AST down to machine code.

*   [A Truffle/Graal High Performance Backend for JRuby](announcement/). Blog post announcing the open sourcing.


----

# Research Papers and Thesis

*    B. Daloze, S. Marr, D. Bonetta, H. Mössenböck. [Efficient and Thread-Safe Objects for Dynamically-Typed Languages](http://ssw.jku.at/General/Staff/Daloze/thread-safe-objects.pdf). In Proceedings of the ACM International Conference on Object Oriented Programming Systems Languages and Applications (OOPSLA), 2016.<br>
      <span class="smaller">
        <a href="http://ssw.jku.at/General/Staff/Daloze/thread-safe-objects.pdf">PDF</a>
      </span>

*    C. Seaton. [AST Specialisation and Partial Evaluation for Easy High-Performance Metaprogramming](meta16/meta16-ruby.pdf). In Proceedings of the 1st Workshop on Meta-Programming Techniques and Reflection (META), 2016.<br>
      <span class="smaller">
        <a href="meta16/meta16-ruby.pdf">PDF</a>
      </span>

*    C. Seaton. [Specialising Dynamic Techniques for Implementing the Ruby Programming Langauge](../phd). PhD thesis, University of Manchester, 2015.<br>
        <span class="smaller">
            <a href="../phd">Abstract</a>,
            <a href="../phd/specialising-ruby.pdf">PDF</a>
        </span>

*    M. Grimmer, C. Seaton, R. Schatz, T. Würthinger, H. Mössenböck. [High-Performance Cross-Language Interoperability in a Multi-Language Runtime](dls15-interop/dls15-interop.pdf). In Proceedings of 11th Dynamic Languages Symposium (DLS), 2015.<br>
        <span class="smaller">
            <a href="dls15-interop/dls15-interop.pdf">PDF</a>
        </span>

*    F. Niephaus, M. Springer, T. Felgentreff, T. Pape, R. Hirschfeld. [Call-target-specific Method Arguments](https://github.com/HPI-SWA-Lab/TargetSpecific-ICOOOLPS/raw/gh-pages/call_target_specific_method_arguments.pdf). In Proceedings of the 10th Implementation, Compilation, Optimization of Object-Oriented Languages, Programs and Systems Workshop (ICOOOLPS), 2015.<br>
        <span class="smaller">
            <a href="https://github.com/HPI-SWA-Lab/TargetSpecific-ICOOOLPS/raw/gh-pages/call_target_specific_method_arguments.pdf">PDF</a>
        </span>

*    B. Daloze, C. Seaton, D. Bonetta, H. Mössenböck. [Techniques and Applications for Guest-Language Safepoints](icooolps15-safepoints/safepoints.pdf). In Proceedings of the 10th Implementation, Compilation, Optimization of Object-Oriented Languages, Programs and Systems Workshop (ICOOOLPS), 2015.<br>
        <span class="smaller">
            <a href="icooolps15-safepoints/safepoints.pdf">PDF</a>
        </span>

*    S. Marr, C. Seaton, S. Ducasse. [Zero-Overhead Metaprogramming: Reflection and Metaobject Protocols Fast and without Compromises](pldi15-metaprogramming/pldi15-marr-et-al-zero-overhead-metaprogramming.pdf). In Proceedings of the 36th Conference on Programming Language Design and Implementation (PLDI), 2015.<br>
        <span class="smaller">
            <a href="pldi15-metaprogramming/pldi15-marr-et-al-zero-overhead-metaprogramming.pdf">PDF</a>
        </span>

*    M. Grimmer, C. Seaton, T. Würthinger, H. Mössenböck. [Dynamically Composing Languages in a Modular Way: Supporting C Extensions for Dynamic Languages](modularity15/rubyextensions.pdf). In Proceedings of the 14th International Conference on Modularity, 2015.<br>
        <span class="smaller">
            <a href="modularity15/rubyextensions.pdf">PDF</a>,
        </span>

*    A. Wöß, C. Wirth, D. Bonetta, C. Seaton, C. Humer, and H. Mössenböck. [An object storage model for the Truffle language implementation framework](pppj14-om/ppj14-om.pdf). In Proceedings of the International Conference on Principles and Practices of Programming on the Java Platform (PPPJ), 2014.<br>
        <span class="smaller">
            <a href="pppj14-om/ppj14-om.pdf">PDF</a>
        </span>

*   C. Seaton, M. L. Van De Vanter, and M. Haupt. [Debugging at full speed](http://www.lifl.fr/dyla14/papers/dyla14-3-Debugging_at_Full_Speed.pdf). In Proceedings of the 8th Workshop on Dynamic Languages and Applications (DYLA), 2014.<br>
        <span class="smaller">
            <a href="http://www.lifl.fr/dyla14/papers/dyla14-3-Debugging_at_Full_Speed.pdf">PDF</a>,
            <a href="http://lafo.ssw.uni-linz.ac.at/truffle/debugging/dyla14-debugging-artifact-0557a4f756d4.tar.gz">Code</a>
        </span>

----

# Videos of Talks and Slide Decks

*   Chris Seaton. [Faster Ruby and JavaScript with GraalVM](javaone16/faster-ruby-javascript-graalvm.pdf). At JavaOne. 2016. [Slides](javaone16/faster-ruby-javascript-graalvm.pdf).

*   Chris Seaton. [Using LLVM and Sulong for Language C Extensions](llvm-cauldron-16/llvm-cauldron-sulong.pdf). At the LLVM Cauldron 2016. [Slides](llvm-cauldron-16/llvm-cauldron-sulong.pdf) and [video](https://www.youtube.com/watch?v=bJzMfYX6n9A).

*   Chris Seaton. [JRuby+Truffle: Why it's important to optimise the tricky parts](javaone15/guilt-free-ruby-on-the-jvm.pdf). Virtual Machines Summer School (VMSS) 2016. [Slides](vmss16/vmss16-ruby.pdf) and [video](https://www.youtube.com/watch?v=b1NTaVQPt1E).

*   Chris Seaton. [Guilt Free Ruby on the JVM](javaone15/guilt-free-ruby-on-the-jvm.pdf). At JavaOne 2015. [Slides](javaone15/guilt-free-ruby-on-the-jvm.pdf).

*   Chris Seaton and Benoit Daloze. [A Tour Through a New Ruby Implementation](fosdem15/truffle-tour.pdf). At FOSDEM 2015. [Slides](fosdem15/truffle-tour.pdf).

*   Chris Seaton. [Deoptimizing Ruby](). At RubyConf 2014. [Video](http://confreaks.tv/videos/rubyconf2014-deoptimizing-ruby), [slides](deoptimizing/deoptimizing-ruby.pdf) and [blogpost](deoptimizing/).

*   Chris Seaton. [Implementing Ruby Using Truffle and Graal](ecoop14/ecoop14-ruby-truffle.pdf). At the European Conference on Object Oriented Programming (ECOOP) Summer School. 2014. [Slides](ecoop14/ecoop14-ruby-truffle.pdf).

*   Christian Wimmer and Chris Seaton. [One VM to Rule Them All](../jvmls13-one-vm/jvmls13-one-vm.pdf). At the JVM Language Summit. 2013. [Video](http://medianetwork.oracle.com/video/player/2623645003001) and [slides](../jvmls13-one-vm/jvmls13-one-vm.pdf).

*    Charles Nutter. [Beyond JVM](http://www.slideshare.net/CharlesNutter/yow-sydney-2013-beyond-jvm) talk at Sydney in December 2013 made some references to this work in the context of JRuby (no video yet). Also a [video](http://www.youtube.com/watch?feature=player_detailpage&v=8ZEAwWLwmfQ#t=951) of a talk with earlier details at Baruco 2013.

*    Thomas Würthinger. [Graal and Truffle: One VM To Rule Them All](http://www.slideshare.net/ThomasWuerthinger/graal-truffle-ethdec2013) recent project overview.

----

# Source Code

The RubyTruffle source code is available as an optional backend in the JRuby
repository. Initially it was available in the OpenJDK Graal Repository, but it's
since been removed to focus on JRuby.

*   [JRuby+Truffle code](https://github.com/jruby/jruby/tree/truffle-head/truffle/src/main/java/org/jruby/truffle)

*   [Graal and Truffle code](https://github.com/graalvm)
