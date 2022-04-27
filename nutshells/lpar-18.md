---
layout: post
title:  "LPAR '18: extended abstract" 
date:   2019-04-04
categories: conference paper abstract
paper-title: "Arrays Made Simpler: An Efficient, Scalable and Thorough Preprocessing"
topic: "Constraint preprocessing for efficient Symbolic Execution"
pdf: http://sebastien.bardin.free.fr/2018-lpar.pdf
from-binsec-version: 0.2
redirect_from: /conference/paper/abstract/2019/04/04/lpar18.html
---


## Motivation 

Automatic decision procedures for Satisfiability Modulo Theory are at the heart
of almost all recent formal verification methods, including Symbolic Execution
(SE). Especially, the theory of arrays is key as it allows to model memory or
essential data structures such as maps, vectors and hash tables.

However, this theory is known to be hard to solve in both theory and practice.
Even more so in the case of very long formulas coming from binary-level
SE. Standard simplification techniques *à la* Read-over-Write (check the paper
for more details) have 2 main drawbacks:

1. They do not scale on very long sequences of stores because of a quadratic
   worst-case complexity.  This is a major issue in practice: for example,
   Symbolic Execution over malware or obfuscated programs may need to consider
   execution traces of several millions of instructions.
   
2. They miss many simplification opportunities because of crude approximate
    equality checks (typically, syntactic term equality). With such checks,
    index equality may be sometimes proven, but disequality can never be —
    except in the very restricted case of constant-value indexes. 
    
The theory of arrays can then quickly become a bottleneck of constraint
solving and Symbolic Execution. 


## Contributions

This papers presents a novel efficient and thorough approach to array
simplification based upon 3 key components:

- dedicated data structure;
- base normalization;
- domain reasoning.

Experimental results in BINSEC demonstrate that this approach consistently
improves the resolution times of the leading SMT solvers (Boolector, Yices,
Z3). Moreover, it also scales over very large formulas (several hundreds of
thousands of array accesses), with drastic gains in terms of runtime — passing
from hours to seconds on the reverse engineering of a code protected by AsPack.


