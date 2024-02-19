---
layout: post
title:  "ESEC/FSE (2023): research paper"
categories: new publication
paper-title: "Scalable Program Clone Search through Spectral Analysis"
topic: "binary similarity, clone seach, binary analysis"
pdf: "/assets/publications/papers/2023-fse.pdf"
date: 2023-12-04
redirect_from: /new/publication/1970/01/01/nutshell-fse-23.html
---

## Motivation
Searching for similarities between x86 or ARM binaries over a large program repository is necessary when the original program written in source code is unavailable, which happens with commercial off-the-shelf (COTS), legacy programs, firmware or malware. For example, detecting malware clones is a major issue, as most malware are actually variants of a few major families active for more than five years. Another application is the identification of libraries, which is both a software engineering issue and a cybersecurity issue due to vulnerabilities inside dynamically linked libraries. These cases require a more global view of the code than just the function-level approach. 

## Problem

<p align="center">
<img src="/assets/img/CloneSearch.png" alt="Clone search workflow" style="height: 200px;" title="Architecture of a program clone search procedure"/>
</p>


Given a query composed of a target program and a repository, the program clone search ranks repository programs by their similarity to the target program. The search is successful if the most similar program is a clone of the target program. These clones could have been (i) compiled from the same source code with a different compiler chain, or (ii) produced using a slightly different version of the source code. 

## Example
Let us consider a repository containing 1420 libraries obtained from the compilation of `20` libraries with 4 optimization levels, 5 versions of GCC, 4 versions of clang, and to the 32 and 64 bits x86 platforms. Next, let us imagine we have the 20 libraries as targets (compiled for x86 32 bits with gcc 6.4 and the `-O2` optimization level).

We consider the following function-level methods and lift them to program-level - Asm2Vec, Gemini, SAFE, AlphaDiff. In order to do so, one has to match functions of the target and functions of a candidate which takes n² step where n is the number of functions. 
We also consider LibDB which does not require a lifting and has a way to filter candidate.

We report the proportion of successes that we call the precision score, as well as clone searches total runtime. PSS is precise and successful in all clone searches, however, it only takes 26s in total, while other methods take from 17h up to 160h. Even with pre-filtering, LibDB is close to 2h. Moreover, PSS runtime is due to its preprocessing; the total similarity checks runtime is negligible. 

## PSS

<p align="center">
<img src="/assets/img/CallGraph.png" alt="Call Graph" style="height: 200px;" title="Example of a Call Graph"/>
</p>

We devise a quick and stable similarity metric called Program Spectral Similarity (PSS), which is based on the combination of two criteria - the spectral distance between call graphs and a coarse spectral analysis of function control flow graphs. Call graphs are useful on a number of tasks. For example, GraphEvo has been able to understand software evolution through call graphs. Moreover, compiler optimizations usually have a small-scale effect on the call graph.

## Contributions
- We invent a new metric called Program Spectral Similarity (PSS) to measure the similarity of programs in the context of program clone searches over large repositories.
- We set up an evaluation framework including 97,760 Linux programs, 19,959 IoT malware programs, 84,992 Windows programs and a smaller Linux dataset of 950 programs, along with 15 methods from the state-of-the-art and 3 baselines.
- We show that PSS, and specifically its optimized version PSSO, achieves an ideal balance of speed, precision and robustness. PSS is robust even against architecture changes and certain obfuscations.
- We highlight the separation between methods needing relevant literal identifiers (such as constants) and those that do not, such as PSS. For the former, we propose two very simple methods (FunctionSet and StringSet) of which the latter surpasses the state-of-the-art in terms of precision.
- We draw conclusions about the different classes of possible methods. In particular, we conclude that many previous works on function clone search cannot cope with program clone search due to a lack of speed and robustness.

## Further information

* Read the [paper](/assets/publications/papers/2023-fse.pdf)
* To appear at the [31st ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering](https://conf.researchr.org/home/fse-2023) 
* Try it ([artifacts](https://doi.org/10.5281/zenodo.8289599))

