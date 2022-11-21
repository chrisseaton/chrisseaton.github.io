---
layout: section
title: TruffleRuby
copyright: Copyright © 2018 Chris Seaton.
footer: <p>TruffleRuby logo Copyright © 2017 Talkdesk, Inc. Licensed under <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>.</p>
redirect_from: "/rubytruffle/"
---

<p style="text-align: center">
<img alt="TruffleRuby" src="truffleruby.png" width="400" height="400">
</p>

TruffleRuby started as [my](..) internship project at [Oracle
Labs](https://labs.oracle.com/) in early 2013. It is an implementation of the
[Ruby](https://www.ruby-lang.org/) programming language on the JVM, using the
[Graal dynamic compiler and the Truffle AST interpreter
framework](https://openjdk.java.net/projects/graal/). TruffleRuby can achieve
peak performance well beyond that possible in JRuby at the same time as being a
significantly simpler system. In early 2014 it was open sourced and integrated
into [JRuby](https://www.jruby.org/) for incubation, then in 2017 it became its
own project, and now it is part of [GraalVM](https://www.graalvm.org/). It was
the subject of my [PhD](../phd/), and since 2019
[Shopify](https://shopify.engineering/) has sponsored development.

This page links to the literature and code related to the project. Note that any
views expressed are my own and not those of Oracle.

<i class="fas fa-trophy"></i> Finalist, Ruby Prize, 2016

----

# Blog Posts and Articles

*   [Stamping Out Overflow Checks in Ruby](stamping-out-overflow-checks/). Can you remove overflow checks on integer arithmetic in Ruby?

*   [The Future Shape of Ruby Objects](rubykaigi21/). How does TruffleRuby represent objects and could MRI do the same?

*   [Seeing Escape Analysis Working](seeing-escape-analysis). Can we see escape analysis working in practice?

*   [Understanding Basic Truffle Graphs](basic-truffle-graphs). How can you make sense of Graal graphs from Truffle?

*   [Context on STM in Ruby](ruby-stm). What is STM and how does it apply to Ruby?

*   [Seeing Register Allocation Working in Java](register-allocation). Can se we the theory of register allocation working in practice?

*   [Understanding Basic Graal Graphs](basic-graal-graphs). How can you make sense of Graal graphs?

*   [Understanding Programs Using Graphs](https://shopify.engineering/blogs/engineering/understanding-programs-using-graphs). What can we learn about a program using graphs?

*   [Low Overhead Polling For Ruby](low-overhead-polling/). How can Ruby check for interruptions without a branch?

*   [Top 10 Things To Do With GraalVM](tenthings/). What does GraalVM actually do?

*   [Ruby Objects as C Structs and Vice Versa](structs/). How you can use Ruby objects as if they were C structs and C structs as if they were Ruby objects.

*   [Understanding How Graal Works](jokerconf17/). A Java JIT Compiler Written in Java.

*   [Flip-Flops &mdash; the 1-in-10-million operator](flip-flops/). Do people actually use flip-flops?

*   [Deoptimizing Ruby](deoptimizing/). What deoptimization means for Ruby and how JRuby+Truffle implements and applies it.

*   [Very High Performance C Extensions For JRuby+Truffle](cext/). How JRuby+Truffle supports C extensions.

*   [Optimising Small Data Structures in JRuby+Truffle](small-data-structures/). Specialised optimisations for small arrays and hashes.

*   [Pushing Pixels with JRuby+Truffle](pushing-pixels/). Running real-world Ruby gems.

*   [Tracing With Zero Overhead in JRuby+Truffle](set_trace_func/). How JRuby+Truffle implements `set_trace_func` with zero overhead, and how we use the same technique to implement debugging.

*   [How Method Dispatch Works in JRuby/Truffle](how-method-dispatch-works-in-jruby-truffle/). How method calls work all the way from AST down to machine code.

*   [A Truffle/Graal High Performance Backend for JRuby](announcement/). Blog post announcing the open sourcing.

----

# Research Papers and Thesis

*    B. Daloze, A. Tal, S. Marr, H. Mössenböck, E. Petrank. [Parallelization of Dynamic Languages: Synchronizing Built-in Collections](http://ssw.jku.at/General/Staff/Daloze/thread-safe-collections.pdf). In Proceedings of the Conference on Object-Oriented Programming, Systems, Languages, and Applications (OOPSLA), 2018.
        <span class="smaller">
            <a href="http://ssw.jku.at/General/Staff/Daloze/thread-safe-collections.pdf">PDF</a>
        </span>

*    J. Kreindl, M. Rigger, and H. Mössenböck. [Debugging Native Extensions of Dynamic Languages](https://arxiv.org/pdf/1808.00823.pdf). In Proceedings of 15th International Conference on Managed Languages & Runtimes (ManLang), 2018.
        <span class="smaller">
            <a href="https://arxiv.org/pdf/1808.00823.pdf">PDF</a>
        </span>

*    K. Menard, C. Seaton, and B. Daloze. [Specializing Ropes for Ruby](ropes-manlang.pdf). In Proceedings of 15th International Conference on Managed Languages & Runtimes (ManLang), 2018.<br>
        <span class="smaller">
            <a href="ropes-manlang.pdf">PDF</a>
        </span>

*    M. Van De Vanter, C. Seaton, M. Haupt, C. Humer, and T. Würthinger. [Fast, Flexible, Polyglot Instrumentation Support for Debuggers and other Tools](fastflexible/fastflexible.pdf). In The Art, Science, and Engineering of Programming, Vol. 2, No. 3, 2018.<br>
        <span class="smaller">
            <a href="fastflexible/fastflexible.pdf">PDF</a>
        </span>

*    M. Grimmer, R. Schatz, C. Seaton, T. Würthinger, M. Luján. [Cross-Language Interoperability in a Multi-Language Runtime](cross-language-interop.pdf). In ACM Transactions on Programming Languages and Systems (TOPLAS), Vol. 40, No. 2, 2018.<br>
    <span class="smaller">
        <a href="cross-language-interop.pdf">PDF</a>
    </span>

*    T. Würthinger, C. Wimmer, C. Humer, A. Wöss, L. Stadler, C. Seaton, G. Duboscq, D. Simon, M. Grimmer. [Practical Partial Evaluation for High-Performance Dynamic Language Runtimes](pldi17-truffle/pldi17-truffle.pdf). In Proceedings of the Conference on Programming Language Design and Implementation (PLDI), 2017.<br>
    <span class="smaller">
        <a href="pldi17-truffle/pldi17-truffle.pdf">PDF</a>
    </span>

*    B. Daloze, S. Marr, D. Bonetta, H. Mössenböck. [Efficient and Thread-Safe Objects for Dynamically-Typed Languages](http://ssw.jku.at/General/Staff/Daloze/thread-safe-objects.pdf). In Proceedings of the ACM International Conference on Object Oriented Programming Systems Languages and Applications (OOPSLA), 2016.<br>
      <span class="smaller">
        <a href="http://ssw.jku.at/General/Staff/Daloze/thread-safe-objects.pdf">PDF</a>
      </span>

*    C. Seaton. [AST Specialisation and Partial Evaluation for Easy High-Performance Metaprogramming](meta16/meta16-ruby.pdf). In Proceedings of the 1st Workshop on Meta-Programming Techniques and Reflection (META), 2016.<br>
      <span class="smaller">
        <a href="meta16/meta16-ruby.pdf">PDF</a>,
        <a href="meta16/meta16-ruby-slides.pdf">Slides</a>
      </span>

*    C. Seaton. [Specialising Dynamic Techniques for Implementing the Ruby Programming Language](../phd). PhD thesis, University of Manchester, 2015.<br>
        <span class="smaller">
            <a href="../phd">Abstract</a>,
            <a href="../phd/specialising-ruby.pdf">PDF</a>
        </span>

*    M. Grimmer, C. Seaton, R. Schatz, T. Würthinger, H. Mössenböck. [High-Performance Cross-Language Interoperability in a Multi-Language Runtime](dls15-interop/dls15-interop.pdf). In Proceedings of 11th Dynamic Languages Symposium (DLS), 2015.<br>
        <span class="smaller">
            <a href="dls15-interop/dls15-interop.pdf">PDF</a>
        </span>

*    F. Niephaus, M. Springer, T. Felgentreff, T. Pape, R. Hirschfeld. [Call-target-specific Method Arguments](https://raw.githubusercontent.com/HPI-SWA-Lab/TargetSpecific-ICOOOLPS/gh-pages/call_target_specific_method_arguments.pdf). In Proceedings of the 10th Implementation, Compilation, Optimization of Object-Oriented Languages, Programs and Systems Workshop (ICOOOLPS), 2015.<br>
        <span class="smaller">
            <a href="https://raw.githubusercontent.com/HPI-SWA-Lab/TargetSpecific-ICOOOLPS/gh-pages/call_target_specific_method_arguments.pdf">PDF</a>
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

*    A. Wöß, C. Wirth, D. Bonetta, C. Seaton, C. Humer, and H. Mössenböck. [An object storage model for the Truffle language implementation framework](pppj14-om/pppj14-om.pdf). In Proceedings of the International Conference on Principles and Practices of Programming on the Java Platform (PPPJ), 2014.<br>
        <span class="smaller">
            <a href="pppj14-om/pppj14-om.pdf">PDF</a>
        </span>

*   C. Seaton, M. L. Van De Vanter, and M. Haupt. [Debugging at full speed](https://www.cristal.univ-lille.fr/dyla14/papers/dyla14-3-Debugging_at_Full_Speed.pdf). In Proceedings of the 8th Workshop on Dynamic Languages and Applications (DYLA), 2014.<br>
        <span class="smaller">
            <a href="https://www.cristal.univ-lille.fr/dyla14/papers/dyla14-3-Debugging_at_Full_Speed.pdf">PDF</a>,
            <a href="https://web.archive.org/web/20150117042919/https://lafo.ssw.uni-linz.ac.at/truffle/debugging/dyla14-debugging-artifact-0557a4f756d4.tar.gz">Code</a>
        </span>

# Videos of Talks and Slide Decks

*   Chris Seaton. [Ruby's Call-Site Behaviour - An Advertisement for Sophie Kaleba's Research](rubys-call-site-behaviour.pdf). Lightning Talk, RubyConf Mini. 2022. [Slides](rubys-call-site-behaviour.pdf).

*   Maple Ong and Chris Seaton. [Call-Target Agnostic Keyword Arguments](graalworkshop22/call-target-agnostic-keyword-arguments.pdf). At the Graal Workshop 2022. [Video](https://www.youtube.com/watch?v=RVqY1FRUm_8) and [slides](graalworkshop22/call-target-agnostic-keyword-arguments.pdf).

*   Stefan Marr, Octave Larose, Sophie Kaleba, and Chris Seaton. [Truffle Interpreter Performance without the Holy Graal](https://kar.kent.ac.uk/93938/1/Truffle_Interpreter_Performance_without_the_Holy_Graal.pdf). At the Graal Workshop 2022. [Slides](https://kar.kent.ac.uk/93938/1/Truffle_Interpreter_Performance_without_the_Holy_Graal.pdf).

*   Chris Seaton. [A History of Compiling Ruby](https://www.youtube.com/watch?v=Zg-1_7ed0hE). At RubyConf 2021. [Video](https://www.youtube.com/watch?v=Zg-1_7ed0hE) and [website](https://ruby-compilers.com/).

*   Chris Seaton. Understanding JIT Optimisations By Decompilation. At QCon Plus Online 2021.

*   Chris Seaton. [The Importance of Optimising Little Languages](https://www.youtube.com/watch?v=EybsDKXGsEc). At VMM 2021. [Video](https://www.youtube.com/watch?v=EybsDKXGsEc).

*   Chris Seaton. [The Future Shape of Ruby Objects](https://www.youtube.com/embed/RqwVEw-Rd5c). Keynote at RubyKaigi 2021. [Video](https://www.youtube.com/embed/RqwVEw-Rd5c) and [blog post](rubykaigi21).

*   Chris Seaton. [Understanding Graal IR](https://www.youtube.com/watch?v=ypETuCHnmxA). At VMIL 2020. [Video](https://www.youtube.com/watch?v=ypETuCHnmxA).

*   Chris Seaton. [Visualizing Graal](graalworkshop20/visualizing-graal.pdf). At Science, Art, Voodoo: Using and Developing The Graal Compiler 2020. [Slides](graalworkshop20/visualizing-graal.pdf) and [video](https://www.pscp.tv/w/1DXxyezOjpbxM).

*   Chris Seaton. [The TruffleRuby Compilation Pipeline](wrocloverb19/truffleruby-compilation-pipeline.pdf). At Wroclove.rb 2019. [Slides](wrocloverb19/truffleruby-compilation-pipeline.pdf) and [video](https://www.youtube.com/watch?v=bf5pQVgux3c).

*   Chris Seaton. [Graal: where it's from and where it's going](graalworkshop19/graal-from-and-going.pdf), keynote. At the Graal Workshop 2019. [Slides](graalworkshop19/graal-from-and-going.pdf).

*   Chris Seaton. [Ten Things To Do With GraalVM](codeone18/ten-things-graal.pdf). At Oracle Code One 2018. [Slides](codeone18/ten-things-graal.pdf).

*   Eric Sedlar and Chris Seaton. [Run Programs Faster with GraalVM](https://www.youtube.com/watch?v=wBegU4d4GRc). At Oracle Code Boston 2018. [Video](https://www.youtube.com/watch?v=wBegU4d4GRc).

*   Chris Seaton. [Understanding How Graal Works - a Java JIT Compiler Written in Java](jokerconf17). At JokerConf 2017. [Slides](jokerconf17/understanding-how-graal-works.pdf), [video](https://www.youtube.com/watch?v=D2IbrPCiupA) and [blog post](jokerconf17/).

*   Chris Seaton. [Polyglot From the Very Old to the Very New](polyconf17/polyglot-old-to-new-seaton.pdf) (keynote). At PolyConf 2017. [Slides](polyconf17/polyglot-old-to-new-seaton.pdf) and [video](https://www.youtube.com/watch?v=FPN4IScbE60).

*   Chris Seaton. [Turning the JVM into a Polyglot VM with Graal](oraclecode17/oraclecode17.pdf). At Oracle Code London 2017. [Slides](oraclecode17/oraclecode17.pdf) and [video](https://www.youtube.com/watch?v=oWX2tpIO4Yc).

*   Chris Seaton. [Ruby's C Extension Problem and How We're Fixing It](rubyconf16/rubyconf16-cexts.pdf). At RubyConf 2016. [Slides](rubyconf16/rubyconf16-cexts.pdf) and [video](https://www.youtube.com/watch?v=YLtjkP9bD_U).

*   Chris Seaton. [Faster Ruby and JavaScript with GraalVM](javaone16/faster-ruby-javascript-graalvm.pdf). At JavaOne 2016. [Slides](javaone16/faster-ruby-javascript-graalvm.pdf).

*   Chris Seaton. [Using LLVM and Sulong for Language C Extensions](llvm-cauldron-16/llvm-cauldron-sulong.pdf). At the LLVM Cauldron 2016. [Slides](llvm-cauldron-16/llvm-cauldron-sulong.pdf) and [video](https://www.youtube.com/watch?v=bJzMfYX6n9A).

*   Chris Seaton. [JRuby+Truffle: Why it's important to optimise the tricky parts](javaone15/guilt-free-ruby-on-the-jvm.pdf). Virtual Machines Summer School (VMSS) 2016. [Slides](vmss16/vmss16-ruby.pdf) and [video](https://www.youtube.com/watch?v=b1NTaVQPt1E).

*   Chris Seaton. [Guilt Free Ruby on the JVM](javaone15/guilt-free-ruby-on-the-jvm.pdf). At JavaOne 2015. [Slides](javaone15/guilt-free-ruby-on-the-jvm.pdf).

*   Chris Seaton and Benoit Daloze. [A Tour Through a New Ruby Implementation](fosdem15/truffle-tour.pdf). At FOSDEM 2015. [Slides](fosdem15/truffle-tour.pdf).

*   Chris Seaton. [Deoptimizing Ruby](). At RubyConf 2014. [Video](https://www.youtube.com/watch?v=z-YVygbDHLE), [slides](deoptimizing/deoptimizing-ruby.pdf) and [blog post](deoptimizing/).

*   Chris Seaton. [Implementing Ruby Using Truffle and Graal](ecoop14/ecoop14-ruby-truffle.pdf). At the European Conference on Object Oriented Programming (ECOOP) Summer School. 2014. [Slides](ecoop14/ecoop14-ruby-truffle.pdf).

*   Christian Wimmer and Chris Seaton. [One VM to Rule Them All](../jvmls13-one-vm/jvmls13-one-vm.pdf). At the JVM Language Summit. 2013. [Video](https://www.youtube.com/watch?v=hDfYRjNynmQ) and [slides](../jvmls13-one-vm/jvmls13-one-vm.pdf).

*    Charles Nutter. [Beyond JVM](https://www.slideshare.net/CharlesNutter/yow-sydney-2013-beyond-jvm) talk at Sydney in December 2013 made some references to this work in the context of JRuby. Also a [video](https://www.youtube.com/watch?v=8ZEAwWLwmfQ#t=951) of a talk with earlier details at Baruco 2013.

---

# Source Code

*   [TruffleRuby code](https://github.com/oracle/truffleruby)

*   [Graal and Truffle code](https://github.com/oracle/graal)
