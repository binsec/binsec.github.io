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

Fuzzing refers to a process of repeatedly running a program with automatically generated inputs to trigger faults. The usual motive is to detect bugs as early as possible, before they cause failures or get exploited as vulnerabilities in production. Fuzzers are reported to have discovered many new [CVE](https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures) software vulnerabilities within a wide range of applications in the recent years.

 Many state of-art fuzzers use branch coverage as a feedback metric to guide the generation of inputs. The fuzzer retains inputs for further mutation only if they increase the coverage count of branches in the control-flow graph of the target program. In the same time, code analyses help finding inputs covering the branches that have a low triggering probability.
 
 However, branch coverage provides a shallow sampling of program behaviours and hence may discard inputs that might be interesting to mutate. To solve this issue, the software testing community has defined standard coverage metrics that are finer-grained than simply counting branches. Yet, the fuzzing community has just started to investigate ways to support some specific finer-grained metrics within specific fuzzers, and the general ability of these fine-grained metrics to improve fuzzing guidance remains unknown.

In this work, we intend to challenge the position of branch coverage as the de facto guidance metric for fuzzing, and evaluate how much using a finer-grained metric for guidance would improve fuzzers. In particular, we want to avoid digging into the internals of every fuzzer implementation to extend them with ad hoc support for every additional metric, but provide instead an out-of-the-box and generic runtime guidance mechanism for existing fuzzers, which could be used to support most fine-grained metrics and beyond.

## Proposal

TOREWRITE

Let us consider the following code snippet on how our approach can make a state-of-the-art fuzzer (based on branch coverage) support a fine-grained coverage metric out-of-the-box (without changing the fuzzer), by transforming the code of the program under test.
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
It is basically a C function checking if an appliance is running outside its allowed temperature limit and taking corrective actions if so. We suppose that the implementation is buggy, but that the bug only triggers a program crash (enabling a fuzzer to detect it) when the temperature is `50` and other rare conditions are met (requiring the fuzzer to generate many inputs with current_temp equal to `50` to actually trigger it).

Our approach makes the test objectives from standard coverage metrics explicit in the code of the Program Under Test (PUT), by adding no-op branches (empty conditional statements) corresponding respectively to each of them. For example, considering the weak mutation testing coverage metric, the following no-op branch would notably be inserted in our example: 
```c
void check(int current_temp,char *data[]){
void check(int current_temp,char *data[] ){
...
if (current_temp>=50 != current_temp>50) { // Killing mutant where >= is replaced by >
  // No-op branch added for encoding WM ROR criterion
  // Inputs entering here should have current_temp==50
  // At least one input entering here will be saved as seed
  // Other things being equal, forcing the fuzzer to save 
  // a seed with current_temp==50 will increase the 
  // probablity to trigger the crash faster
}
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

The new `if (current_temp>=50 != current_temp>50)` branch explicitly forces the fuzzer to maintain and mutate an input where `current_temp` equals `50` as soon as it generates one (these mutations will mostly affect the values in `data`). This will increase the chance for the fuzzer to trigger a crash revealing the bug, making bug detection faster in average.

## Experiments and results

TODO

## Contributions

TODO

## Further information
- Read the [paper](/assets/publications/papers/2023-tosem.pdf) and download the [artifact](https://git.frama-c.com/bnongpoh/cannotate) and [benchmark](https://zenodo.org/record/7275184).
- [Published](https://dl.acm.org/doi/10.1145/3587158) in the "ACM Transactions on Software Engineering and Methodology" journal (TOSEM).


