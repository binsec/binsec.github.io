---
layout: post
title:  "RAID'20: research paper"
categories: new publication
paper-title: "Binary-level Directed Fuzzing for Use-After-Free Vulnerabilities"
topic: "Directed Fuzzing; Use-After-Free"
pdf: "/assets/publications/papers/2020-raid.pdf"
date: 2020-10-14
redirect_from: /new/publication/2020/10/14/raid20.html
---
## Motivation
**Directed Greybox Fuzzing** like AFLGo [1] aims to perform stress testing on pre-selected potentially vulnerable target locations, with applications to different security contexts: (1) bug reproduction, (2) patch testing or (3) static analysis report verification. Despite tremendous progress in the past few years, current (directed or not) greybox fuzzers still have a hard time finding complex vulnerabilities such as Use-After-Free (UAF), non-interference or flaky bugs, which require bug-triggering paths satisfying very specific properties. 

**We focus on UAF bugs** which appear when a heap element is used after having been freed. They are currently identified as one of the most _critical exploitable vulnerabilities_ due to the lack of mitigation techniques compared to other types of bugs, and they may have serious consequences such as data corruption, information leaks and denial-of-service attacks. Why is detecting UAF hard for existing fuzzers?
- **Complexity** – Exercising UAF bugs requires to generate inputs triggering a _sequence_ of 3 events – _alloc, free_ and _use_ – _on the same memory location_, spanning multiple functions of the Program Under Test (PUT), where exercising bugs like buffer overflows only require a single out-of-bound memory access. This combination of both _temporal_ and _spatial_ constraints is extremely difficult to meet in practice;
- **Silence** – UAF bugs often have _no observable effect_, such as segmentation faults. In this case, fuzzers simply observing crashing behaviors do not detect that a test case triggered such a memory bug. Sadly, popular profiling tools such as ASan or Valgrind cannot be used in our fuzzing context due to their high runtime overhead.

For example, let's consider the following simple UAF bug, where a possible Proof of Concept (PoC) demonstrating the bug is running the program with `AFU` as an input:

``` c
int *p, *p_alias;
char buf[10];

void bad_func() {
    free(p); // exit() is missing
}

int main (int argc, char *argv[]) {
    int f = open(argv[1], O_RDONLY);
    read(f, buf, 10);
    p = malloc(sizeof(int));
    
    if (buf[0] == 'A') {
        p_alias = malloc(sizeof(int));   
        p = p_alias;    
    }
    if (buf[1] == 'F')
        bad_func();
    if (buf[2] == 'U')
       *p = 1;
    return 0;
}
```
 Both AFL-QEMU and even directed fuzzer AFLGo with targets at source-level cannot detect this bug _within 6 hours_, while UAFuzz can detect it _within minutes_ with the help of targets extracted from a Valgrind's UAF report.

## Contributions
_We propose UAFuzz, the first (binary-level) directed greybox fuzzer tailored to UAF bugs_. Our contribution is the following:
- We systematically revisit the three main ingredients of directed fuzzing (selection heuristic, power schedule, input metrics) and specialize them to UAF;
- We develop an **UAFuzztoolchain** on top of AFL and BINSEC;
- We release a **fuzzing benchmark dedicated to UAF**, comprising 30 real bugs from 17 widely-used projects;
- In a **bug reproduction** setting, we demonstrate that UAFuzz is **highly effective** and significantly outperforms state-of-the-art competitors: 2x faster in average to trigger bugs (up to 43x), +34% more successful runs in average (up to +300%) and 17x faster in triaging bugs (up to 130x, with 99% spare checks);
- In **patch testing** setting, UAFuzz found **30** unknown bugs (11 UAFs, 7 CVEs) in projects like Perl, GPAC, MuPDF and GNU Patch (including 4 buggy patches).

## Further information

- Read the [paper](/assets/publications/papers/2020-raid.pdf)
- Download the [tool](https://github.com/binsec/uafuzz) and [benchmarks](https://github.com/binsec/uafbench)
- Presented at [The 23rd International Symposium on Research in Attacks, Intrusions and Defenses (RAID 2020)](https://raid2020.org/)
- Presented at [Black Hat USA 2020](https://www.blackhat.com/us-20/briefings/schedule/#about-directed-fuzzing-and-use-after-free-how-to-find-complex--silent-bugs-20835)

## References
[1] Marcel Böhme, Van-Thuan Pham, Manh-Dung Nguyen, and Abhik Roychoudhury. Directed greybox fuzzing. CCS 2017
