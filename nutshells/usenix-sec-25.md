---
layout: post
title:  "Usenix Security'25: research paper"
categories: new publication
paper-title: "Attacker Control and Bug Prioritization"
topic: "bug prioritization, symbolic execution"
pdf: "/assets/publications/papers/2025-usenix-sec.pdf"
date: 2025-03-18
---

# Attacker Control and Bug Prioritization

## Motivation

Developing efficient bug-finding techniques has been a focus of research for decades.
As a result, bugs are currently being found at an unprecedented rate, far exceeding developers' abilities to fix them.
Due to the potential security impact of bugs, there thus is a need for efficient bug prioritization.

Unfortunately, existing prioritization techniques and practices are less than satisfactory.
Bugs are often either ranked based on their type of vulnerability or scored by opaque machine learning algorithms that rely solely on bug reports.
There is a distinct lack of formal approaches to this problem, and fine-grained prioritization has yet to be achieved.

Since exploitability is a hard to define concept, and formal methods need well-defined properties, we choose to isolate and focus on distinct aspects.
One such aspect is *attacker control*, the ability of an attacker to influence parameters of a vulnerability through program inputs.
While this concept is fairly common-place, we are the first to propose a detailed theoretical framework for it, with associated algorithms and concrete applications to bug prioritization.

## Example

```c
#define HEADER_SIZE 40

uint64_t check_header(char *input, 
        uint64_t input_size)
{
    //2) input[0->7] written on the stack
    uint64_t header = *((uint64_t *) input);
    return header <= 296;
}
    
void get_msg(char *buf, uint64_t buf_size, 
        char *input, uint64_t input_size)
{
    //3) not initialized => size = header
    uint64_t size;
    if(input_size <= buf_size + HEADER_SIZE)
        //4) input_size < 40 => integer underflow
        size = input_size - HEADER_SIZE;
    //5) buffer overflow!!!
    // a. input_size < 40 
    //    => 2^64 - 40 <= size < 2^64
    // b. input_size > (*@{296}@*) => size = header 
    memcpy(buf, input + HEADER_SIZE, size);
}

int main(...)
{
    //1) inputs: char *input, uint64_t input_size
    ...
    char buf[256];
    if(check_header(input, input_size))
        get_msg(buf, 256, input, input_size);
    ...
}
```

This program contains two variants of a buffer overflow, vulnerabilities a and b.
While vulnerability a only leads to very large out-of-bounds write sizes (> 2^63), and thus always causes crashes, vulnerability b allows an attacker to overwrite memory adjacent to the buffer, including a return address.

Common indicators traditionally associated with attacker control, such as taint or the presence of symbolic bytes, cannot distinguish vulnerability a from b.
Even model counting is not helpful, since both vulnerabilities have the same number of possible write sizes.
The key insight to be gleamed from this example is that different values can have different threat levels, and fine-grained prioritization cannot be achieved without taking this fact into account.

## Our Attacker Control Framework

In this paper, we base all of our notions of control on one key definition: the *domain of control (DoC)*, which is the set of values that can be obtained.
It then follows that |DoC| = 1 indicates a lack of control, |DoC| = 2 is the lowest possible level of control, which we call *weak control*, and |DoC| = max is the maximum level of control, which we call *strong control*.
Furthermore, following usual notions of quantitative information flow analysis would give us *quantitative control*, equal to |DoC|.

None of these notions are precise enough to achieve fine-grained prioritization on their own.
To properly score the dangerosity of a given domain of control, we need *weighted quantitative control*, the sum of the threat levels of each obtainable value.
To measure it, we first need to extract the domain of control from a path constraint using our *Shrink and Split* algorithm, which relies on the previously defined properties to iteratively refine the result as a set of intervals.
Then, weighted quantitative control can be easily computed by integrating a weight function over the domain.

## Implementation and Experiments

We implemented our algorithms in *Colorstreams*, a new dynamic binary-level analysis tool based on inline information-flow analysis.
It provides a variety of different analyses based on taint or symbolic execution, the latter relying on Binsec.
Colorstreams mainly focuses on automation and modularity in order to facilitate its practical usage and the integration of new analyses.

In our experiments, we found that our approach was the only one able to properly differentiate real-world out-of-bounds and control-flow hijack vulnerabilities.
The level of automation of our tool also allowed us to automatically rank out-of-bounds vulnerabilities from the MAGMA fuzzing benchmark with minimal human efforts.

## Contributions

In summary, we make the following contributions:

* We provide a generic theoretical framework for attacker control.
* We propose algorithms for the purpose of evaluating attacker control.
In particular, Shrink and Split allows to extract domains of control in an analysis- and human-friendly format.
* We implemented our algorithms in Colorstreams, a new dynamic analysis tool with symbolic execution capabilities.
* We evaluated our approach on real-world vulnerabilities and showed its effectiveness.
We also showed its practicality in a realistic scenario through our automatic end-to-end analysis of the MAGMA fuzzing benchmark.

## Further information

* Read the [paper](/assets/publications/papers/2025-usenix-sec.pdf)
* To appear at [34th USENIX Security Symposium](https://www.usenix.org/conference/usenixsecurity25) 
* [Artifact](https://zenodo.org/records/15075966)

