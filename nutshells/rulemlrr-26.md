---
layout: post
title:  "RuleML+RR 2026: research paper"
categories: new publication
paper-title: "Precondition Constraint Acquisition Through Explanatory Queries"
topic: "machine learning, constraint acquisition, weakest precondition"
pdf: "/assets/publications/papers/2026-rulemlrr.pdf"
date: 2026-10-19
---

## Motivation

**Precondition Constraint Acquisition.** Function annotations in the form of function pre- and postconditions are crucial for many software engineering, program verification, and, more generally, reverse engineering applications. Indeed, preconditions specify the inputs on which a function executes without crashing, enabling modular software analysis. Unfortunately, such annotations are rarely available and must be retrofitted by hand. Recently, Constraint Acquisition (CA), a learning framework from Constraint Programming, has been extended to automatically infer program preconditions in a black-box manner, from input-output observations. This led to two main CA-based precondition inference frameworks [PreCA](/nutshells/ijcai-22.html) and [DCA](/nutshells/kr-23.html).

**Informativeness gap.** While relying on CA provides strong correctness guarantees, scalability remains a bottleneck. Indeed, both PreCA and DCA rely solely on membership queries reporting execution success or failure, requiring numerous executions to be generated and processed. Still, automated program analysis can overcome this limitation by extracting deeper context. Developing more informative, analysis-driven queries and extending CA to support them are the next crucial steps. In this paper, we propose a new type of queries, named explanatory queries, and show how to leverage them in constraint acquisition for program analysis.


## Major Contributions

This paper makes the following major contributions:

* We introduce a novel class of queries designed for oracles capable of providing structured feedback beyond mere classification. An explanatory query is a query that not only provides the classification of a given input (i.e., safe/unsafe/unknown) but also provides an explanation for its classification. To benefit from this extended answer, this explanation is expected to generalize the response and allow us to reduce the number of further queries we need to ask. We formalize the assumptions required for an explanatory query to be informative and for an oracle to support such queries;

* We propose the XAcq algorithm and its implementation, extending PreCA to utilize
explanatory queries actively. We show how XAcq can leverage the notion of path predicate in symbolic
execution, a classical static analysis approach, to create informative explanations. Under
reasonable hypotheses, we formalize and prove the termination and correctness properties
for XAcq, establishing that this novel approach is sound and capable of efficiently processing
structured feedback;

* Finally, we conducted a detailed experimental evaluation comparing XAcq with existing CA-based methods (PreCA and DCA) and PIE, a state-of-the-art baseline for precondition inference.
Across two shared precondition inference benchmarks, our empirical results demonstrate that
integrating explanatory queries substantially boosts the performance and scalability of CA-based
methods. Furthermore, XAcq yields higher-quality preconditions than PIE in terms of soundness
guarantees, while maintaining reasonable computational overhead.
## Further information

* Read the [paper](/assets/publications/papers/2026-rulemlrr.pdf)
* To appear at the [The International Joint Conference on Rules and Reasoning (RuleML+RR 2026)](https://2026.declarativeai.net/ruleml-rr) 
* Try [PreCA](https://github.com/binsec/preca)
