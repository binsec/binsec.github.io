---
layout: post
title: "PLDI'24: research paper"
paper-title: "Compiling with Abstract Interpretation"
topic: "Compilers; Abstract Interpretation; Static single assignement (SSA)"
pdf: "/assets/publications/papers/2024-pldi.pdf"
redirect_from: /new/publication/1970/01/01/nutshell-pldi-24.html
categories: new publication
date: 2024-04-22
---

## Context

Software analyzers and compilers have a lot in common: they both have to read and
understand source code in order to **prove facts about it** and **transform it** into
an equivalent code. While the goal of an analyzer is to prove facts (correctness, safety...)
about the source, it often transforms the code through rewrites to make the analysis easier.
As an example, rewriting `e + e` into `2*e` makes it obvious that the value is even.
Symmetrically, the goal of a compiler is code transformation, but compilers often
run analyses on the code to perform optimizations. For instance, many compilers
remove variables which are never read by the program: this is possible thanks to
a liveness analysis.

With this in mind, it seems compilers and analyzers could be written using the same
core library of analysis and program transformations.
A problem that creeps up in both case is known as the **phase-ordering problem**:
in what order should we run transformations and analyses? Should we start by analyzing
the code, and use that information to transform it? Or rather should first transform the
code and then run the analysis on the simplified version? Should we alternate between
transformation and analysis passes?

In practice, the best precision is obtained by running transformations and analyses
simultaneously. Fortunately, **abstract interpretation** is well-suited to fuse different analyses together. In abstract interpretation, each analysis is viewed as a domain,
and all domains have a common signature/interface. This allows running multiple analyses
in parallel (using a product of the relevant domains), and have them collaborate
(domains can query other domains to see if they can prove a property).

## Example

Below is an example of compiling a source program to SSA using our technique (you can see the labels in the nodes as just different names for each node). Our analyzer is capable of eliminating the dead else branch inside the loop. Doing so requires simultaneously
performing
**numerical analysis** (to learn that z is even), some **syntactic transformations** (to learn
that F(j + z%2) is F(j)), optimistic **global value numbering** (to learn that i = j), and
**dead code elimination** so that no analysis takes the else branch (which breaks all those properties).

<img src="/assets/publications/pictures/2024-pldi-full-example.svg"
style="width:1000px; display:block; margin-left:auto; margin-right:auto">

## Contributions

Our paper shows the following novel results:
- A standard abstract interpretation framework can be turned into a
  compiler: create a domain that is a **free-algebra** of the domain signature (i.e.
  a domain where each domain operation is a constructor creating an expression), then the analysis
  result **can be used to construct a new program**. Different abstract domain signatures correspond to different languages: the classical domain signature correspond to imperative programs expressed as a control-flow graph, and we provide a SSA domain signature corresponding to programs in SSA form.
- **Functors can implement compiler passes**.
  A functor is just a function that builds a new abstract domain on top of abstract
  domains received as arguments. Functors are modular, they can be proved independently
  and then combined in a full compilation chain. Functor soundness and completeness
  imply forward and backward simulation between source and compiled program respectively.

  Here is a small example of a sound and complete functor transformation:
  replacing a ternary operator with explicit control flow jumps.

  <img src="/assets/publications/pictures/2024-pldi-transformation-example.svg"
  style="width:600px; display:block; margin-left:auto; margin-right:auto">

- **Compiling to SSA recovers missing context and improves numerical analysis precision**.
  We describe a functor for compiling a small imperative language to SSA.
  This allows performing a numerical analysis on the SSA form, which leverages
  variable immutability to store information on expressions.
  This is always more precise than direct numerical analysis, while just adding a constant overhead.

  In particular, this domain can analyze compiled code with the same precision as source
  (when compilation corresponds, e.g., to transformation into three-address code).
  The usual precision loss resulting from compiling large expressions into
  instruction sequences with multiple intermediate variables is recovered thanks to our SSA-based analysis.

  Here are a few examples of assertion this domain can prove:
  - Propagate information across statements
    ```c
    c = (y >= 0);
    if (c) {
        assert(y >= 0);
    }
    ```
  - Learn from related variables:
    ```c
    y = x + 1;
    z = x * x;
    if (2 <= y && y <= 5) {
        assert(1 <= x && x <= 4);
        assert(1 <= z && z <= 16);
    }
    ```
  - Increase precision of the numeric abstraction: even though intervals
    can't represent `x != 0`, using our domain with interval abstraction can prove this
    ```c
    if (x != 0) {
        assert(x != 0);
    }
    ```

## Further information

- Read the [**paper**](/assets/publications/papers/2024-pldi.pdf)
- Presented at the [Programming Language Design and Implementation (PLDI) 2024 conference](https://pldi24.sigplan.org/). Watch the [**talk video**](https://www.youtube.com/watch?v=2Btkn9AvM8o) or look at the [**slides**](/assets/publications/slides/2024-pldi.pdf)
- Download the [**software artifact**](https://doi.org/10.5281/zenodo.10895582) from
  Zenodo to try out our example analyzer and explore the code.
