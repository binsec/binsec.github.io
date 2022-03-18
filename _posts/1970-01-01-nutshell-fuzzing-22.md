---
layout: post
title:  "FUZZING'22: registered report"
categories: new publication
paper-title: "Fine-Grained Coverage-Based Fuzzing"
topic: "automated software testing; fuzzing;"
pdf: "/assets/publications/papers/2022-fuzzing.pdf"
date: 1970-01-01
---

## Motivation

Fuzzing is an effective software testing method that discovers bugs by feeding target applications with (usually a massive amount of) automatically generated inputs. Many state of-art fuzzers use branch coverage as a feedback metric to guide the fuzzing process. The fuzzer retains inputs for further mutation only if branch coverage is increased. However, branch coverage only provides a shallow sampling of program behaviours and hence may discard inputs that might be interesting to mutate. This work aims at taking advantage of the large body of research over defining finer-grained code coverage metrics (such as mutation coverage) and make these metrics easily available as better proxies to select interesting inputs for mutation.

## Our proposal

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


## Contributions
- We propose to make state-of-the-art coverage-based fuzzers support most standard fine-grained coverage metrics out of the box
- We develop a Clang pass to automatically
instrument C code with the Multiple Condition Coverage and
Mutation Coverage metrics objectives. 
- We use this pass to transform the four programs from Lava-M, a
standard and basic fuzzer benchmark.
- We run the AFL++ and Qsym fuzzers five times for 24 hours over the original Lava-M programs and their transformed versions, showing promising improvements on fuzzing performance.

## Further information
- Read the [paper](/assets/publications/papers/2022-fuzzing.pdf)
- To appear at the [The 1st International Fuzzing Workshop (FUZZING) 2022](https://fuzzingworkshop.github.io/#guides)
- The FUZZING workshop comes with a preregistration-based publication process in a top journal (see [details](http://fuzzbench.com/blog/2021/04/22/special-issue/))

