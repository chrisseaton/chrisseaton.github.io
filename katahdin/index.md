---
layout: article
title: Katahdin
author: Chris Seaton
date: 2007
image: katahdin.png
image_alt: Katahdin
copyright: Copyright © 2007-2012 Chris Seaton
---

Katahdin is a programming language where the syntax and semantics are mutable at
runtime. It was the 2007 master’s project of [Chris
Seaton](http://www.chrisseaton.com/) at the University of Bristol [Department of
Computer Science](http://www.cs.bris.ac.uk/), under the supervision of Dr Henk
Muller.

Katahdin employs the theory of [parsing expression grammars and packrat
parsing](http://bford.info/packrat/). Unlike other contemporary work, Katahdin
applies these techniques at runtime to allow the grammar to be modified by a
running program. New constructs such as expressions and statements can be
defined, or a new language can be implemented from scratch. It is built as an
interpreter on the Mono implementation of the .NET framework.

----

# Introduction

In most programming languages you can define new functions and types. In the
same way, Katahdin allows you to define new expressions, statements and other
language constructs.

For example, most programming languages have a modulo, or remainder operator. If
your language does not have one, and there is not one that you can overload,
creating one would involve learning about the internals of the implementation,
modifying the grammar and adding semantic actions throughout the code base. You
would then have to recompile and install your new implementation. Anyone else
wanting to use it would have to be given a set of patches for your changes
against the official implementation. In Katahdin, the complete syntax and
semantics of the modulo operator can be defined from scratch in just a few
lines:

    class ModExpression : Expression {
        pattern {
            option leftRecursive;
            a:Expression "%" b:Expression
        }

        method Get() {
            a = this.a.Get...();
            b = this.a.Get...();
            return a - (b * (a / b));
        }
    }

In Katahdin this is a runtime operation, so immediately after defining
ModExpression the modulo operator becomes part of the language in line with all
the other operators. You can define a new construct on one line and use it on
the next.

There are no special constructs in Katahdin that cannot be modified, including
white-space and basic tokens. If all of the constructs in a language are defined
as above, Katahdin can be turned into an interpreter for that language. In this
way, Katahdin can be used as a generic interpreter for any language, or one
language can be used from within another by composing the two or more
definitions. My implementation includes proof-of-concept language definitions
for FORTRAN and Python, which can be composed to write a single program with a
mix of the two languages.

    import "fortran.kat";
    import "python.kat";

    fortran {
        SUBROUTINE RANDOM(SEED, RANDX)
            INTEGER SEED
            REAL RANDX
            SEED = 2045*SEED + 1
            SEED = SEED - (SEED/1048576)*1048576
            RANDX = REAL(SEED + 1)/1048577.0
            RETURN
        END
    }

    python {
        seed = 128
        randx = 0
        for n in range(5):
            RANDOM(seed, randx)
            print randx
    }

There are real world scenarios where one would want to use one language from
within another. People use SQL from within other languages all the time by
writing the SQL in strings. In Katahdin, the definition of SQL can be composed
with the main language and used as if it were all part of one language. There
are also good examples of why one would want to define new constructs in a
running program. For example, Java 1.5 included a new for-each-statement.
Programmers wanting to use the statement had to wait for Sun’s implementation to
be drafted, implemented, tested and distributed. In Katahdin programmers can
create new statements on their own as they need them, without modifying the
implementation. If you want a new for-each-statement in your language you can
add one in just a minute.

----

# Literature

*   [A Programming Language Where the Syntax and Semantics Are Mutable at
    Runtime](katahdin.pdf)

    My master’s thesis describing the background, theory, implementation and
    current status of Katahdin. Also published as a technical report at the
    University of Bristol.

*   [Thesis defence slides](katahdin-slides.pdf)

*   [Katahdin: Mutating a Programming Language’s Syntax and Semantics at
    Runtime](katahdin-paper.pdf)

    An unpublished paper.

*   [Katahdin Consulting Business Plan](katahdin-business-plan.pdf)

    A business plan commercialising Katahdin, written for an associated course
    unit.

# Source Code

The Katahdin [source code](http://github.com/chrisseaton/katahdin/) is
available from GitHub, or as [katahdin-0.2.tar.gz](katahdin-0.2.tar.gz). It is
available as public domain, or BSD licence if that causes problems for you.
