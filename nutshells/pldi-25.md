---
layout: post
title: "PLDI'25: research paper"
paper-title: "Relational Abstractions Based on Labeled Union-Find"
topic: "Relational abstract domain; Labeled union-find; Abstract Interpretation"
pdf: "/assets/publications/papers/2025-pldi.pdf"
redirect_from: /new/publication/1970/01/01/nutshell-pldi-25.html
categories: new publication
date: 2025-06-15
katex: true
---

## Context

In program analysis, various abstractions are used to store known facts about the
program being studied. Two popular choices are *non-relational abstractions*,
which store numeric information on variables (like intervals $$x \in [0:5]$$), or *relational
abstraction*, which store relations between variables (like $$3x + 2y \leqslant -5z$$).
The former is fast (complexity in $$\mathcal O(|\mathbb X|)$$ where $$|\mathbb X|$$ is the number of variables)
but imprecise, whereas the latter is very precise but cost-prohibitive
(polyhedra has $$\mathcal O(2^{|\mathbb X|})$$ complexity).

In the middle of this spectrum lie *weakly-relational abstractions*. They only
store relations between pairs of variables. For example, octagons
store relations $$\pm x \pm y \leqslant c$$ for some constant $$c$$. These abstractions are faster than
polyhedra, but still expensive, in large part due to having to compute
a transitive closure to find all known relations, which costs $$\mathcal O(|\mathbb X|^3)$$.

Our goal is to find **a new family of relational abstract domains that are cheaper than the weakly-relational domains**.
For this, a central question is **can we compute the expensive transitive closure much more cheaply?** The answer is yes, if we assume that the relation obtained on each path between
two variables is always the same. This allows eliminating the vast majority of relations, **we only need to
store a spanning tree** and can still recover any arbitrary relation in amortized almost-constant time, using a variation
of the efficient [union-find](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) data structure, that we call the **labeled union-find**.

## The labeled union-find data structure

<img src="/assets/publications/pictures/2025-pldi-labeled-uf-simple.svg"
style="width:400px; display:block; margin-left:auto; margin-right:auto">

Similarly to the classical union-find, the labeled union-find
is a rooted forest, i.e. a set of trees where each node points to its parent.
The
extension adds labels $$\mathbb L$$ to the parent edges,
representing relations. In the figure above, these relations are
affine constraints between two variables (of the form $$y = a*x +
b$$). From this labeled union-find, it is easy to infer the relation
between any two variables (if there is one, i.e. when they are in the same tree) by composing or reversing
relations. For instance, from $$z = y - 1$$ and $$y = 2 * r$$, we can
deduce that $$z = 2*r - 1$$. While computing this, we can shrink the
distance between the element $$z$$ and the root of the tree $$r$$, doing the
analog of path compression in the union-find structure.

Computing the relation between some variables may also require
inverting relations; for instance, the relation between $$y$$ and $$x$$ is
the composition of $$y = 2*r$$ and the inverse of $$x = 3*r+2$$,
i.e. $$\displaystyle r = \frac x 3 - \frac 2 3$$, yielding $$\displaystyle y = \frac 2 3 x - \frac 4 3$$. In general, the
labels thus have an associative composition operation, and an inverse
operation. Formally, labels must have a [**group
structure**](https://en.wikipedia.org/wiki/Group_(mathematics)).  This
requirement also derives fairly naturally from our previous assumption
(same relation on each path).


## The labeled union-find relational abstraction

When using labeled union-find to represent abstract relations between
variables, the soundness of operations places strong requirements on
the relations that can be used. We show that these relations **must
correspond to injective functions** between equivalence classes. This
allows to discover many suitable examples of relations, such as:

- Constant offset: relations of the form $$y = x + b$$ for some constant $$b$$
- Two Value per Equality (TVPE): $$ax + by + c = 0$$, with $$a,b,c \in \mathbb Z^3$$, $$\mathbb Q^3$$ or $$\mathbb R^3$$
- Modular TVPE: $$y = ax + b\mathop{\texttt{mod}} 2^{64}$$ between 64-bit vectors with $$a$$ odd.
- Xor and rotation: $$y = (x \mathop{\mathtt{xor}} c) \mathop{\mathtt{rot}} n$$ between fixed-size bitvector, which can encode many shifts
- Linear transformations $$Y = A * X + B$$ where $$Y$$ and $$X$$ are vectors, and $$A$$ is an invertible matrix.

However, labeled union-find cannot use relations like bounded
difference $$a \le y - x \le b$$, as they are not injective. Doing so
inevitably leads to precision loss.

## Application

Codex already performs [sophisticated constraint propagation](/papers/2024-pldi-compiling-with-abstract-interpretation.html)
using relations between the values computed by the program. However, the new domain can find new relations, e.g. from relating simultaneously incremented loop counters:
```c
int i = 0, j = 4;
while(i < 10) { i += 1; j += 3; }
```
Without labeled union-find, Codex learns that at the end of the loop, $$\mathtt{i} = 10$$,
$$\mathtt{j} \in [4:+\infty]$$ and $$\mathtt{j} \equiv 1 \mathop{\mathtt{mod}} 3$$. However,
with labeled union find and the TVPE relation, $$\mathtt{j} = 3\mathtt{i} + 4$$ is inferred.
Thus, at the end of the loop, Codex knows $$\mathtt{j} = 34$$.

The abstraction has also been implemented in the [Colibri2](https://colibri.frama-c.com/index.html) constraint solver to infer and propagate new relations efficiently.

## Combining with other abstractions

Labeled union-find groups variables into related classes, which each
point to the same representative. This can not only be used to provide new
information to other parts of the analysis, but also simplify other
abstractions. Instead of computing relations between individual
variables, we can just compute relations between groups of variables
related by labeled union-find.

For example, when combining the constant offset labeled union-find with the interval abstraction, we
only need to store intervals on representative elements, not on all variables, since these can be
recovered. If we know that $$y \xrightarrow{+2} x$$ and $$x \in [0:2]$$ then we do not need to store
$$y \in [2:4]$$. This reduces storage cost and propagation time, since all related variables are
updated at once any time new information is learned.

<img src="/assets/publications/pictures/2025-pldi-factorization.svg"
style="width:700px; display:block; margin-left:auto; margin-right:auto">
<center>
Fig. Using labeled union-find to factorize a non-relational abstraction. On the left, we associate an interval
to every node in the graph. On the right, we can group related nodes and only store an interval value
on the representative. The values of other nodes are recomputed as needed without precision loss.
</center>

Labeled union-find can also help relational abstraction similarly, shrinking their size and thus their
computation cost. Furthermore, it can be modified to detect any entailed equalities and notify other
abstractions of these facts.

## Going further

- Read the [**paper**](/assets/publications/papers/2025-pldi.pdf) or look at the [**slides**](/assets/publications/slides/2025-pldi.pdf).
- You can also read the [**WIP workshop paper**](https://hal.science/cea-04996700v1). It is only 4 pages long and less technical.
- Presented at the [Programming Language Design and Implementation (PLDI) 2025 conference](https://pldi25.sigplan.org/).
- Download the [**software artifact**](https://zenodo.org/records/15261356) from
  Zenodo to explore the code and see the performance results.
