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
of the efficient union-find data structure.

<img src="/assets/publications/pictures/2025-pldi-spanning-tree.svg"
style="width:900px; display:block; margin-left:auto; margin-right:auto">

<center>
Fig. Graph of relations between variables. Each arrow represents a relation, labels have been omitted.
Left is the initial configuration, middle is the computed closure, and right is the minimal spanning tree that our domains only have to maintain.
</center>

## Labeled union-find

<img src="/assets/publications/pictures/2025-pldi-labeled-uf-example.svg"
style="width:900px; display:block; margin-left:auto; margin-right:auto">

<center>
Fig. Example of a labeled union-find data structure. Left is the actual data structure
(each arrow is a pointer). Right is the implied transitive closure
(each dashed arrow can be computed in constant time).
At the bottom, we have an example of path compression:
shortening the path from z to the representative r
without changing the transitive closure (the dotted arrow is replaced by the dashed one).
</center>

We can use an extension of the classical
[union-find](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) data
structure to represent this spanning tree. The extension proceeds by adding
labels $$\mathbb L$$ (representing relations) to the parent edges, and is thus
called *labeled union-find*. In order to properly adapt the union-find
algorithms, these labels need an associative composition operation $$\mathbb
;$$, an inverse $$\cdot^{-1}$$ and a neutral element. That is to say, they must
have a [**group structure**](https://en.wikipedia.org/wiki/Group_(mathematics)).
This requirement also derives fairly naturally from our previous assumption
(same relation on each path).

<img src="/assets/publications/pictures/2025-pldi-labeled-uf.svg"
style="width:700px; display:block; margin-left:auto; margin-right:auto">
<center>
Fig. Main operation of labeled union-find. Compared to classical union-find,
the "union" operation has been renamed "add_relation", and the get_relation operation
is new.
</center>

## The labeled union-find relational abstraction

When using labeled union-find to represent abstract relations between variables,
the soundness of operations places strong requirements on the relations that
can be used. We show that these relations **must correspond to injective functions**
between equivalence classes. The following examples relations are suitable:

- Constant offset: relations of the form $$y = x + b$$ for some constant $$b$$
- Two Value per Equality (TVPE): $$ax + by + c = 0$$, with $$a,b,c \in \mathbb Z^3$$, $$\mathbb Q^3$$ or $$\mathbb R^3$$
- Modular TVPE: $$y = ax + b\mathop{\texttt{mod}} 2^{64}$$ between 64-bit vectors with $$a$$ odd.
- Xor and rotation: $$y = (x \mathop{\mathtt{xor}} c) \mathop{\mathtt{rot}} n$$ between fixed-size bitvector, which can encode many shifts

However, we cannot use relations like bounded difference $$y - x \in [a:b]$$, as
they are not injective. Doing so inevitably leads to precision loss.

## Combining with other abstractions

Labeled union-find groups variables into related class, which each point to the
same representative. This can often be used to simplify other abstractions,
especially whenever information on any element can be deduced from information
on the representative.

For example, when combining the constant offset labeled union-find with the interval abstraction, we
only need to store intervals on representative elements, not on all variables, since these can be
recovered. If we know that $$y \xrightarrow{+2} x$$ and $$x \in [0:2]$$ then we do not need to store
$$y \in [2:4]$$. This reduces storage cost and propagation time, since all related variables are
updated at once any time new information is learned.

<img src="/assets/publications/pictures/2025-pldi-factorization.svg"
style="width:1200px; display:block; margin-left:auto; margin-right:auto">
<center>
Fig. Using labeled union-find to factorize a non-relational abstraction.
Left: without LUF, we store an interval value on all nodes of this term graph.
Right: using LUF, we can group related nodes together and only store an interval value
on the representative. We can recompute the values of other nodes as needed without precision loss.
</center>

Labeled union-find can also help relational abstraction similarly, shrinking their size and thus their
computation cost. Furthermore, it can be modified to detect any entailed equalities and notify other
abstractions of these facts.

## Examples

We have implemented labeled union-find domains both in [Codex](https://codex.top),
a sound static analyzer based on abstract interpretation, and in [Colibri2](https://colibri.frama-c.com/index.html), a constraint solver.

Codex already performs constraint propagation using relations between the values computed by the program. However, the new domain can find new relations; in particular, an important source of improvement comes from relating
simultaneously incremented loop counters. For instance, consider the following C snippet:
```c
int i = 0, j = 4;
while(i < 10) { i += 1; j += 3; }
```
Without labeled union-find, Codex learns that at the end of the loop, $$\mathtt{i} = 10$$,
$$\mathtt{j} \in [4:+\infty]$$ and $$\mathtt{j} \equiv 1 \mathop{\mathtt{mod}} 3$$. However,
with labeled union find and the TVPE relation, $$\mathtt{j} = 3\mathtt{i} + 4$$ is inferred.
Thus, at the end of the loop, Codex knows $$\mathtt{j} = 34$$.

In Colibri2, we were able to increase propagations when using constant difference
alongside the interval domain. For example, instance, if $$f(x) = 2a+x+3b$$
for some $$a,b$$, and if we know that $$f(4) < 10$$, then we can learn that
$$f(9)^2 \leqslant 225$$ is unsatifsiable. This sort of reasoning seems easy, but
in practice, a decision procedure for non linear-arithmetic is difficult to implement and costly.
Using labeled union-find (to relate $$f(4)$$ and $$f(9)$$) enables solving some
easy cases such as these.

## Going further

- Read the [**paper**](/assets/publications/pdfs/2025-pldi-relational-abstractions-labeled-uf-with-appendices.pdf).
- You can also read the [**WIP workshop paper**](https://hal.science/cea-04996700v1). It is only 4 pages long and less technical.
- To be presented at the [Programming Language Design and Implementation (PLDI) 2025 conference](https://pldi25.sigplan.org/).
- Download the [**software artifact**](https://zenodo.org/records/15261356) from
  Zenodo to explore the code and see the performance results.
