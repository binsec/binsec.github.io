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

 Many state of-art fuzzers use <strong>branch coverage</strong> as a feedback metric to guide the generation of inputs. More precisely, the fuzzer retains some of the already generated inputs as seeds from which to generate newer inputs, but only in the case where they have improved the coverage of branches within the control-flow graph of the fuzzed program. At the same time, automatic code analyses (such as taint tracking or symbolic execution) scrutinize the fuzzed program, in order to help the fuzzer discover inputs covering the branches that have a low triggering probability.
 
 However, branch coverage provides a shallow sampling of program behaviours and hence may ignore inputs that might be interesting to discover and keep as seeds. The software testing community has for long proposed standard <strong>coverage metrics that are finer-grained than simply counting branches</strong>, like control-low, data-flow or mutation coverage. For example, the Multiple Condition Coverage (MCC) metric takes into account subtle variations within the program’s control-low logic, which are not captured by branch coverage, while the Weak Mutation (WM) coverage metric takes into account the code behaviours where common programming mistakes would corrupt the program state. Yet, the fuzzing community has just started to investigate ways to integrate one or another of these finer-grained metrics within a specific fuzzer. As a consequence, the general ability of such metrics to improve fuzzing guidance remains unknown.

<strong>In this work</strong>, we intend to challenge the position of branch coverage as the de facto guidance metric for fuzzing. Concretely, <strong>we aim at enabling the use of finer-grained metrics in fuzzers</strong>, as well as at <strong>evaluating whether and how much it improves fuzzing guidance</strong>. In particular, we want to avoid requiring to dig into the internals of every fuzzer implementation to extend it with ad hoc support for every additional metric. Instead, we aim at <strong>providing a generic runtime guidance mechanism that would work with existing fuzzers out of the box</strong>, and which could be used to support most fine-grained metrics and beyond.

## Proposal

We provide out-of-the-box support for fine-grained coverage metrics in fuzzers by <strong>transforming the code of the fuzzed program</strong>. Indeed, the coverage objectives from most fine-grained metrics (like conditions to activate or program state corruptions to detect) can be made explicit as assertions in the code of the fuzzed program. As a consequence, we can transform this code to <strong>add new no-op branches</strong> (guarded by the assertion predicate) <strong>corresponding to these fine-grained objectives</strong>. Covering one of the added branches is then equivalent to covering the corresponding objective from the metric. The instrumentation is performed carefully, in order to avoid modifying the program semantics or TODO tampered compiler/fuzzer harness. 

<strong>For example</strong>, let us consider the following code snippet as the fuzzed program. It is basically a C function checking if an appliance is running outside its allowed temperature limit and taking corrective actions if so. We suppose that the implementation is buggy, but that the bug only triggers a program crash (enabling a fuzzer to detect it) when the temperature is `50` and some rare values are provided in the `data` argument (requiring the fuzzer to generate many inputs with `current_temp` equal to `50` to actually trigger it).

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

Our approach makes the objectives from fine-grained coverage metrics explicit in such code by adding no-op branches (empty conditional statements) corresponding respectively to each of them. For example, the following empty conditional statement would be inserted to encode one of the objectives from the Weak Mutation metric (ROR operator): 
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

In our example, the then branch of the new `if (current_temp>=50 != current_temp>50)` statement explicitly forces the fuzzer to discover and maintain as seed an input that enters it, i.e. where `current_temp` equals `50`. This seed will increase the proportion of generated inputs with `current_temp` equal to `50`, which should increase the chance for the fuzzer to trigger a crash revealing the bug, making bug detection faster in average.

The idea of providing <strong>runtime guidance to fuzzers</strong> out of the box, by inserting dedicated no-op branches into the fuzzed program, enables encoding objectives from fine-grained coverage metrics, but could also be used in other contexts. For example, the no-op branches could be guarded by predicates written by <strong>human developers</strong> (willing to tell the fuzzer which program behaviours are interesting to explore) or computed by <strong>static analysers</strong> (having computed fault preconditions, so that inputs satifying such preconditions should be favoured by the fuzzer).

## Experiments and results

We <strong>implement our proposal</strong> and support two representative fine-grained coverage metrics (multiple condition coverage and weak mutation) in two state-of-the-art fuzzers (AFL++ and QSYM). We measure the impact of this support over performance while fuzzing more than seven hundreds of thousands of lines of code, from the standard LAVA-M and MAGMA benchmarks, for more than two years and a half of CPU time. 

This <strong>measurement reveals that</strong>...
1. The <strong>impact of using the fine-grained metrics over the fuzzer performance is hard to predict before fuzzing and most of the time either neutral or negative</strong>. This result suggests that fine-grained metrics should not be used as a general means to guide fuzzers. This is confirmed by simultaneous studies conducted by other teams, which do not rely on a generic and black-box support of fine-grained metrics in fuzzers, but where support for a single fine-grained metric is integrated into a specific fuzzer, by modifying its internal implementation. 
2. <strong>In favorable circumstances, fine-grained metrics can significantly increase fuzzing performance</strong>. More precisely, fine-grained metrics seem to make fuzzers able to focus on a much denser sampling of the subtle local diferences of behaviour in the code, probably at the expense of a broader coverage of the complete codebase. In addition, the need to monitor and react to the coverage of the many additional coverage objectives from fine-grained metrics slows down the fuzzer. Situtations where the density of fine-grained objectives is too high should be avoided, as the resulting reduction of fuzzing throughput can impede the fuzzer's ability to explore a broader part of the codebase and to find bugs. 

<strong>Our work calls for further research</strong> to better investigate situtations where favorable circumstances would make fine-grained metrics profitable for fuzzing, like when only fragile or sensitive parts of the codebase would be instrumented, or as a complement to classical fuzzing campaigns. In addition, as our approach, which provides runtime guidance to fuzzers out of the box, appears effective, efforts are needed to make it even better integrated with fuzzer instrumentation and to deploy it to encode directives from human users or static analysers.

## Further information
- Read the [paper](/assets/publications/papers/2023-tosem.pdf) and download the [artifact](https://git.frama-c.com/bnongpoh/cannotate) and [benchmark](https://zenodo.org/record/7275184).
- [Published](https://dl.acm.org/doi/10.1145/3587158) in the "ACM Transactions on Software Engineering and Methodology" journal (TOSEM).


