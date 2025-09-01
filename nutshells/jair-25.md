---
layout: post
title:  "JAIR'25: journal paper"
categories: new publication
paper-title: "A Query-Based Constraint Acquisition Approach for Enhanced Precision in Program Precondition Inference"
topic: "machine learning, constraint acquisition, weakest precondition, software engineering"
pdf: "/assets/publications/papers/2025-jair.pdf"
date: 2025-02-16
---

## Motivation

Consider that you want to use a function from a proprietary library. Of course, its source code is not available or is [obfuscated](https://en.wikipedia.org/wiki/Obfuscation_(software)) [1]. Moreover, you do not know on which inputs you can call it. To use this function safely, what you need is its [weakest precondition](https://en.wikipedia.org/wiki/Predicate_transformer_semantics) [2]. To infer it, you can try to reverse engineer the binary or the obfuscated code by hand. However, this is error prone, very hard and very time consuming. Thus, you may leverage automated methods. You can try whitebox precondition inference but functions are often too complex to fulfill the analysis automatically. 

So what about *blackbox precondition learners* ? These are indeed less impacted by code complexity. However, the state-o-the-art offers no correctness guarantees. This is a major drawback as a wrong precondition could lead to unsafe behavior in your code. This is where *PreCA* comes into play. It enables to perform *blackbox weakest precondition inference and shows good correctness properties*.

**Example.** Consider the function `find_first_of`:


```c

int find(int* t, int n, int val) {
    for (int i = 0; i < n; i++) {
        if (t[i] == val)
            return i;
    }
    return n;
}

int find_first_of(int* a, int m, int* b, int n) {
    for (int i = 0; i < m; i++) {
        if (find(b, n, a[i]) < n)
            return i;
    }
    return m;
}

```

Without any access to the source code, *PreCA* will infer its implicit weakest precondition: [ m > 0 => valid(a) ] and [ m > 0 and n > 0 => valid(b) ] in 172s with 45 automatically generated test-cases.


## Our proposal: PreCA

We propose *PreCA*, the first constraint acquisition [3] based method to infer *weakest preconditions*. 
*PreCA* performs active learning (no need for the user to give test-cases) and enjoys good theoretical and practical results.

**But how does it work ?** *PreCA* relies on active constraint acquisition, especially [Conacq](https://www.lirmm.fr/constraintacquisition/conacq.html), to infer the target weakest precondition. It takes as input a finite set of constraint B and supposes that the target weakest precondition can be expressed as a conjunction of constraints -- called a constraint network -- from B. *PreCA* then automatically generates test-cases and asks an oracle to classify them (positive or negative). It can then prune its search space depending on the oracle answers. Note that *PreCA* generates as many test-cases as needed to enforce convergence i.e., to find a unique constraint network coherent with all test-cases. To classify a test-case, the oracle runs the function under analysis over it. It is classified negatively if it leads to a runtime error or a postcondition violation and positively otherwise.

**What about the theoretical properties ?** *PreCA* shows better theoretical properties than the state-of-the-art blackbox precondition inference methods.
Especially, if the target weakest precondition WP can be expressed as a conjunction of constraints from B, then *PreCA* is assured to converge to a constraint network L equivalent to WP. 


## Major Contributions

In summary, this paper makes the following major contributions:
* We propose *PreCA*, the first ever (Conacq-like) framework based on active constraint acquisition and dedicated to infer preconditions;
* We show in that *PreCA* enjoys much better theoretical correctness properties than prior black-box approaches;
* We describe a specialization of *PreCA* to the important case of memory-related preconditions;
* We experimentally evaluate the benefit of our technique on several benchmark functions. The results show that *PreCA* significantly outperforms prior precondition learners, be it black-boxes or white-boxes.

----------------------

*This paper is an extension of our [IJCAI'22 paper](/nutshells/ijcai-22.html). It extends it in five main ways:*
* *We provide a revised version of the PreCA algorithm;*
* *We give a detailed presentation of each component of PreCA (description of the memory representation and how it impacts the oracle);*
* *We extend the constraint language by introducing new constraints related to strings;*
* *We explore new research questions in our experimental evaluation;*
* *We present two use-cases from the libc and the mbedtls libraries to highlight its practical utility.*

----------------------

## Further information

* Read the [paper](/assets/publications/papers/2025-jair.pdf)
* To appear at the [The Journal of Artificial Intelligence Research (JAIR)](https://jair.org/index.php/jair/index) 

## References

[1] [Christian Collberg, Clark Thomborson, and Douglas Low. 1997. A taxonomy of obfuscating transformations](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.68.2651&rep=rep1&type=pdf)

[2] [Edsger W Dijkstra. A constructive approach to the problem of program correctness. BIT Numerical Mathematics, 1968](https://link.springer.com/article/10.1007/BF01933419)

[3] [Christian Bessiere, Frédéric Koriche, Nadjib Lazaar, and Barry O’Sullivan. Constraint acquisi- tion. Artificial Intelligence, 2017](https://www.sciencedirect.com/science/article/pii/S0004370215001162)

