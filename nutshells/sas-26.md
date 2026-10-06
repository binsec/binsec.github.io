---
layout: post
title: "SAS'26: research paper"
paper-title: "A Lattice of Union-Finds"
topic: "Union-find; Lattice; Persistence; Abstract Interpretation"
pdf: "/assets/publications/papers/2026-sas.pdf"
redirect_from: /new/publication/1970/01/01/nutshell-sas-26.html
categories: new publication
date: 2026-10-06
katex: true
---

## Context

Using [union-find data-structures](https://en.wikipedia.org/wiki/Disjoint-set_data_structure)
is a very efficient way to represent changing sets of equalities or equivalence relations.
They render a normally complex computation, the transitive closure, extremely fast.
However, to achieve this they suffer from strong limitations. The classical presentation
of union-find only allows adding equalities, a process which destroys previous version
of the set with fewer equalities.
An extension (known as persistent union-find, see [Conchon and Filliâtre, *A Persistent Union-Find Data Structure*, 2007](https://dl.acm.org/doi/10.1145/1292535.1292541)) allows backtracking, i.e. forgetting
the most recently added equalities to revert to a previous version.

However, when representing relations in program analysis, such a tree of
versions is not sufficient. We want to be able to compare information coming
from different control-flow branches, and combine them to obtain the set of
equalities that hold after joining such branches. In other words, we need **a
directed acyclic graph (DAG) of versions, not just a tree**. In practice, we
propose a lattice, which is more expressive than a DAG, as it has two
distinct merges (conjunctions and disjunctions), and allows comparing versions.

## Lattice Operations on Union-Find

We formally define the operations required to turn persistent union-find into a lattice:
- `join` combines two union-finds to only keep the set of equalities that are true in both
  (i.e. the conjunction of the represented equivalence relations);
- `meet` combines two union-finds to keep the set of equalities that are true in either
  (i.e. the disjunction of the represented equivalence relations);
- an `inclusion` operator allows checking whether a union-find is more precise than another
  (does it contain all the other's equalities).

<img src='/assets/publications/pictures/2026-sas-join-meet.svg'
  alt="Union-find join and meet example"
  style="width:700px; display:block; margin-left:auto; margin-right:auto"/>
<center>
Fig. Example of a <code>join</code> and <code>meet</code> of union-find structures.
Each dot represents a node, each potato a class and each arrow a parent pointer
</center>

## Join Algorithm

We provide fast algorithms to compute all three operations. Our algorithms are
remarkably simple, even if they rely on non-trivial correction arguments.
Of the three operations, `join` is the most complex, as it must compute the intersection
of all equivalence classes, which creates more classes than there are in either argument.
Furthermore, the notion of a class itself is not directly visible in union-find:
there are no easily accessible sets of related node. Instead, each node points
to a parent, which may be (or transitively point to) the unique representative.

What we realized is that we can identify the classes of the intersection by
pairs of representatives from `u` and `v`. We can then construct a map `new_classes: repr_u * repr_v -> repr_join`.
Crucially, this map construction is linear, since finding the representatives of
a node is an almost-constant time operation:
```ocaml
let linear_join : UF.t -> UF.t -> UF.t = fun u v ->
    let new_classes = Hashtbl.create () in
    let result = ref (UF.make ()) in
    for x in UF.nodes do
        let ru = UF.find u x in
        let rv = UF.find v x in
        match Hashtbl.lookup new_classes (ru,rv) with
        | Some r -> result := UF.union result x r
        | None -> Hashtbl.add new_classes (ru,rv) x
    done;
    !result
```

This algorithm has complexity $$O(n \times \alpha(n))$$ where $$n$$ is the total number of
nodes and $$\alpha$$ the inverse Ackermann function. It is almost-optimal in the general case,
meaning we can find examples where creating the `join` union-find requires $$O(n)$$ writes.

However, in program analysis, we often join structures that only have small differences.
In that case, we want to avoid scanning the entire set of nodes, and only focus on
the much smaller set of nodes with differing parents. We can use
either Patricia trees (see our [OCaml patricia-tree library](https://codex.top/api/patricia-tree/)
with faster than linear union and intersection)
or persistent array data structures for fast access to that difference set.
We can then adapt the above algorithm to take advantage of this `diff` operator:
```ocaml
let lookup tbl (x,y) =
    if x == y
    then Some x
    else Hashtbl.lookup tbl (x,y)

let diff_join : UF.t -> UF.t -> UF.t = fun u v ->
    let new_classes = Hashtbl.create () in
    (* Create a new pointer to u *)
    let result = ref { parent = u.parent } in
    (* Reset parents of the diff *)
    for x in diff u v do !result.parent[x] <- x done
    (* Proceed as for the linear algorithm *)
    for x in diff u v do
        let ru = UF.find u x in
        let rv = UF.find v x in
        match lookup new_classes (ru,rv) with
        | Some r -> result := UF.union result x r
        | None -> Hashtbl.add new_classes (ru,rv) x
    done;
    result
```
This new version runs in $$O(\delta \times \alpha(n))$$, which can be considerably
faster when $$\delta$$ is small relative to $$n$$. Our experimental benchmarks confirm
this good performance, for even very large $$n$$.

## Meet and Inclusion Algorithms

Meet and inclusion are considerably simpler, as they can be implemented using
the `union` and `find` operations directly, and do not require looking deeply at
the internals of a union-find structure. Just like `join`, they also benefit
from a difference operator to avoid scanning the entire set of nodes:
```ocaml
let diff_meet : UF.t -> UF.t -> UF.t = fun u v ->
  diff u v
  |> fold (fun result x -> UF.union result x v.parent[x]) u

let diff_incl : UF.t -> UF.t -> bool = fun u v ->
  diff u v
  |> forall (fun x -> UF.find u x = UF.find u v.parent[x])
```
Both of these run in $$O(\delta \times \alpha(n))$$, and inclusion can early return
if an equality that holds in `v` is not found in `u`.

## Going further

- Read the [**paper**](/assets/publications/papers/2026-sas.pdf).
- Presented at the [Static Analysis Symposium (SAS) 2026 conference](https://conf.researchr.org/home/splash-issta-2026/sas-2026), look at the [slides](/assets/publications/slides/2026-sas.pdf).
- Download the [**software artifact**](https://zenodo.org/records/21217874) from
  Zenodo to explore the code and see the performance results, or take a look at
  the [union-find-lattice](https://codex.top/api/union-find-lattice/) library that we've released on [opam](https://opam.ocaml.org/).
