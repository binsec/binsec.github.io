---
layout: post
title:  "TOSEM (2023): research paper"
categories: new publication
paper-title: "Fine-Grained Coverage-Based Fuzzing"
topic: "automated software testing; software vulnerability detection; fuzzing; code coverage criteria; mutation testing"
pdf: "/assets/publications/papers/2023-tosem.pdf"
date: 2023-04-25
redirect_from: /new/publication/1970/01/01/nutshell-tosem-23.html
---

## Motivation

<strong>Fuzzing</strong> refers to a process of repeatedly running a program with automatically generated inputs to trigger faults. The usual motive is to detect bugs as early as possible, before they cause failures or get exploited as vulnerabilities in production. Fuzzers are reported to have discovered many new [CVE](https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures) vulnerabilities within a wide range of applications in the recent years.

 Many state of-art fuzzers use <strong>branch coverage</strong> as a feedback metric to guide the generation of inputs. More precisely, the fuzzer retains the generated inputs as seeds from which to generate newer inputs, but only in the case where they improve branch coverage within the control-flow graph of the fuzzed program. At the same time, automatic code analyses (such as taint tracking or symbolic execution) scrutinize the fuzzed program, in order to help the fuzzer discover inputs covering the branches that have a low triggering probability.
 
 However, branch coverage provides a shallow sampling of program behaviours and hence may discard inputs that might be interesting to discover and keep as seeds. The software testing community has for long defined standard <strong>coverage metrics that are finer-grained than simply counting branches</strong> (like multiple conditions or mutation coverage). Yet, the fuzzing community has just started to investigate ways to integrate one or another of these finer-grained metrics within a specific fuzzer. As a consquence, the general ability of such metrics to improve fuzzing guidance remains unknown.

<strong>In this work</strong>, we intend to challenge the position of branch coverage as the de facto guidance metric for fuzzing. Concretely, <strong>we aim at enabling the use of finer-grained metrics in fuzzers</strong>, as well as at <strong>evaluating whether and how much it improves fuzzing guidance</strong>. In particular, we want to avoid requiring to dig into the internals of every fuzzer implementation to extend it with ad hoc support for every additional metric. Instead, we aim at <strong>providing a generic runtime guidance mechanism that would work with existing fuzzers out of the box</strong>, and which could be used to support most fine-grained metrics and beyond.

## Proposal

We provide out-of-the-box support for fine-grained coverage metrics in fuzzers by <strong>transforming the code of the fuzzed program</strong>. Indeed, the coverage objectives from most fine-grained metrics (like conditions to activate or mutants to kill) can be made explicit in the code of the fuzzed program. As a consequence, we can transform this code to <strong>add new no-op branches corresponding to these fine-grained objectives</strong>. Covering one of the added branches is then equivalent to covering the corresponding objective from the metric. The instrumentation is performed carefully, in order to avoid modifying the program semantics. 

<strong>For example</strong>, let us consider the following code snippet as the fuzzed program. It is basically a C function checking if an appliance is running outside its allowed temperature limit and taking corrective actions if so. We suppose that the implementation is buggy, but that the bug only triggers a program crash (enabling a fuzzer to detect it) when the temperature is `50` and some rare values are provided in the `data` argument (requiring the fuzzer to generate many inputs with current_temp equal to `50` to actually trigger it).

```c
void check(int current_temp,char *data[] ){
if(current_temp>=50) // Bug: should be current_temp>50
    {
    // Deal with appliance running outside
    // the allowed temperature limit
    ...  
        // The bug triggers a detectable crash 
        // only when current_temp==50
        // and when rare specific values
        // are present in data
    }
}
```

Our approach makes the objectives from fine-grained coverage metrics explicit in such code by adding no-op branches (empty conditional statements) corresponding respectively to each of them. For example, the following empty conditional statement would be inserted to encode one of the objective of the Weak Mutation metric (ROR operator): 
```c
void check(int current_temp,char *data[]){
...
if (current_temp>=50 != current_temp>50) {} // Killing mutant where >= is replaced by > 
...
if(current_temp>=50) // Bug should be current_temp>50
    {
    // Deal with appliance running outside
    // the allowed temperature limit
    ...  
        // The bug triggers a detectable crash 
        // only when current_temp==50
        // and when rare specific values
        // are present in data
    }
}
```

<strong>Fuzzing such a transformed program</strong> with a fuzzer guided by branch coverage is equivalent to fuzzing the original program, but with the fuzzer also <strong>saving as seeds inputs covering the fine-grained objectives</strong>. In addition, all the analyses helping the fuzzer to cover low-probability branches will also serve to <strong>help cover low-probability fine-grained objectives</strong>. 

In our example, the then branch of the new `if (current_temp>=50 != current_temp>50)` statement explicitly forces the fuzzer to discover and maintain as seed an input where `current_temp` equals `50`. This seed should increase the chance for the fuzzer to trigger a crash revealing the bug, which would make bug detection faster in average.

TODO : parler des autres cas où l'instru peut être utile

## Experiments and results

TODO

## Further information
- Read the [paper](/assets/publications/papers/2023-tosem.pdf) and download the [artifact](https://git.frama-c.com/bnongpoh/cannotate) and [benchmark](https://zenodo.org/record/7275184).
- [Published](https://dl.acm.org/doi/10.1145/3587158) in the "ACM Transactions on Software Engineering and Methodology" journal (TOSEM).


