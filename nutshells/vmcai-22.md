---
layout: post
title:  "VMCAI'22: research paper"
categories: new publication
paper-title: "Lightweight Shape Analysis based on Physical Types"
topic: "type safety; formal methods; code analysis;"
pdf: "/assets/publications/papers/2022-vmcai.pdf"
date: 2021-11-24
redirect_from: /new/publication/1970/01/01/nutshell-vmcai-22.html
---

Memory errors are the source of the most pervasive and critical
security vulnerabilities.  Programs written in low-level systems
languages, like C/C++ and assembly, perform low-level pointer and
memory operations and are particularly subject to these kind of
errors.

In this paper, we are interested in memory abstractions expressive
enough to verify *type safety*, i.e. the *preservation of structural invariants
expressed by types*, in non-trivial linked data-structure manipulations in both
high- and low-level code (such as assembly or low-level C).
This type safety
entails *spatial memory safety*, namely that each memory
access is done on an address that was previously allocated (and thus
that null or out-of-bound pointer dereferences are impossible).
We also seek for a high level of automation (i.e., by avoiding the
requirement of complex handwritten program annotations) and of
efficiency.

Many verification techniques aimed at verifying the correctness of
memory manipulating programs have been developed.
On one end of the precision spectrum, *pointer analyses* infer basic
conservative relations between pointer values and can tackle basic memory
errors.
However, they are limited in that, unlike our analysis technique, they cannot establish safety when doing so  requires reasoning over the structure of data.

Consider for example the following function, manipulating a disjoint-set data
structure:

```c
typedef struct uf {
  struct uf* parent;
} uf;

uf *uf_find(uf *x) {
  while(x->parent != 0) {
    uf *parent = x->parent;
    if(parent->parent == 0)
      return parent;
    x->parent = parent->parent;
    x = parent->parent;
  }
  return x;
}
```

The correctness of this code requires some non-nullness analysis, as well as
hypotheses on the structure of the memory pointed by the function argument `x`.
Such reasoning goes beyond usual pointer analyses, while our analysis successfully validates this function.


On the other hand, *shape analyses* based on three-valued logics like TVLA or on
separation logics attempt to establish precise structural invariants, such as
the existence of some list or tree data-structures.
Such analyses can cope with the verification of memory safety in
presence of sophisticated structures, yet they are typically less
scalable than basic pointer analyses and also less resilient to a
local precision loss in the sense that losing precision over a
fragment of the memory often entails no information can be recovered
about that region. Another limitation is that such analyses are difficult to apply to low-level
code, like low-level C or binary code. On the contrary, our technique scales less due to being mostly a storeless abstraction, and was made to handle the low-level type-punning code patterns typical of low-level systems programs.


## Contributions

To achieve this, we propose:

- A novel memory abstraction that is inspired by the classical notion of types,
  but applies to the physical representation of data-structures. The analysis effectively
  performs a type inference of a dependent type system using abstract interpretation.
- Two independent extensions of the domain to track “retained” and “staged”
  points-to predicates to improve the precision of the analysis by representing
  memory updates in a flow-sensitive way.
- An experimental evaluation on a set of challenging benchmarks, showing that
  the combination naturally deals with both C and binary code manipulating
  dynamic data structures.


# Further information
- Read the
  [**paper**](/assets/publications/papers/2022-vmcai.pdf). 
- To appear at the [23rd International Conference on Verification, Model Checking, and Abstract Interpretation
  (VMCAI'22)](https://popl22.sigplan.org/home/VMCAI-2022#About)
- Download the [tool and benchmark](https://doi.org/10.5281/zenodo.5589489).
