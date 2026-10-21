---
layout: post
title:  "RuleML+RR 2026: research paper"
categories: new publication
paper-title: "Towards Anytime Constraint Acquisition"
topic: "anytime, machine learning, constraint acquisition, weakest precondition"
pdf: "/assets/publications/papers/2026-ruleml.pdf"
date: 2026-10-19
---

## Motivation

Constraint acquisition (CA) is a symbolic machine learning framework. 
It queries an oracle (often a human user) to classify examples as solutions or non-solutions to the problem it aims to learn. 
Interestingly, CA shows strong correctness guarantees. However, it often takes a lot of time to terminate. 

Still, in many contexts, such as precondition inference, an approximation of the solution may be sufficient. 
Indeed, precondition inference asks: for which inputs does compute terminate without
error? In program verification, waiting for the exact precondition is often unnecessary; it suffices to provide an under-approximation, i.e., a set of constraints such that all solutions lead to a correct execution (though it may miss some). 

**The anytime gap in CA.** Existing CA approaches share a fundamental weakness: neither produces a useful
intermediate approximation under a tight budget. We propose an extension of CA, titled CAnDy, for fast, anytime constraint acquisition.

## Major Contributions

This paper makes the following major contributions:
* We propose the first anytime approach to CA that focuses on rapidly generalizing the most specific network consistent with the information gathered so
far. Our method maintains this network and progressively refines the concept learned during the acquisition process;
* To accelerate this refinement, we introduce a new query generation strategy that prioritizes queries based on a distance criterion. This
strategy enables the learner to improve the current approximation more efficiently under limited time or query budgets;
* We evaluate the resulting approach, CAnDy, on program precondition inference benchmarks. The results show that it refines constraint networks significantly faster than the existing methods, achieving up to a 16 times speedup over state-of-the-art results. CAnDy also divides by 2 to 3 the waiting time between positive queries in the hardest cases compared to competitors.

## Further information

* Read the [paper](/assets/publications/papers/2026-ruleml.pdf)
* To appear at the [The International Joint Conference on Rules and Reasoning (RuleML+RR 2026)](https://2026.declarativeai.net/ruleml-rr) 
* Try [PreCA](https://github.com/binsec/preca)
