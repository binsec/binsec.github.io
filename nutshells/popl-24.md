---
layout: post
title:  "POPL24: research paper"
categories: new publication
paper-title: "Inference of Robust Reachability Constraints"
topic: "program analysis, abduction, precondition inference, symbolic execution"
pdf: "/assets/publications/papers/2024-popl.pdf"
date: 2024-01-19
---

## Motivation

Let us consider the synthetic memcopy function below.

```c
typedef struct { unsigned char bytes[32]; } uint256_t;

void kmemcpy (void *dst, const void *src, size_t n)
{
  if ((((intptr_t)dst | (intptr_t)src | n) & 0b11111)) // slow path
    for (size_t i = 0; i < n; i += 1) *((uint8_t*)dst + i) = *((uint8_t*)src + i);
  else // fast path
    for (size_t i = 0; i <= (n >> 5); i += 1) *((uint256_t*)dst + i) = *((uint256_t*)src + i);
}
```

This code copies `n` bytes from the `src` buffer to the `dst` buffer in two possible fashions.
Either both `src` and `dst` are pointers aligned on 32 bits, in which cases the function copies the data word by word through the *fast path*, or they're not and copy is made byte by byte through the *slow path*.
However, the fast path contains an error, as the loop termination comparison is performed with a `<=` instead of a `<`, which means that whenever the function follows its fast path, a buffer overflow will occur.
The goal of this work is to propose a method to detect such bugs precisely, that is, that they exist and under which conditions they can be triggered.

## The Problem

We can express the bug as a control flow reachability problem to a highjacking function (`g`) after a stack overflow:

```c
void f(const char *input)
{
  char buf[64] = { 0 };
  kmemcpy(buf, input, 64); buf[63] = 0;
  puts(buf);
  return;
}

void g() { puts("hijacked"); exit(-1); }

int main (int argc, char *argv[])
{
  char src[96] = "doing things";
  for (int i = 0; i < 32; i += sizeof(void (*)()))
    *(void (**)())(src + 64 + i) = &g;
  f(src);
  return 0;
}
```

This problem can be handled by usual techniques for reachability problems such as *bounded model checking* or *symbolic execution*.
However, what a reachability analysis will give us is that it is possible to execute a buffer overflow with a formal input to realize it.
Unfortunately, this realization will be a false positive in practice as replaying the input will not always trigger the bug.
This is due to the fact that part of the formal input returned by the reachability analysis contains a precise valuation for a part of the input space that the user has no ability to control; here, *e.g.* `&buf = 0xffffcc80` and `&input = 0xffffcd40`.

To mitigate this issue, [previous work](/assets/publications/papers/2021-cav.pdf) has introduced an upgrade version of reachability called *robust reachability*.
With robust reachability, we create a partition of the input space between what is controlled by the user and what is not, and assigns distinct quantifiers to each set in the path constraints that are built.
Typically, in this example, the addresses `buf` and `input` are not controlled while the content of `input` is.
Robust reachability will thus look for a value for the content `input` that permits to trigger the stack overflow independently of the value of the uncontrolled addresses.

Unfortunately, this is still not enough for this case as no such value exist.
Still, finding and characterizing this bug remains of importance as it is fairly easy to trigger.

## What we do

We propose a method to compute *robust reachability constraints*, that is, preconditions on the input space that suffice to achieve robust reachability.
Such robust reachability constraints allow in the end to derive both a controlled input and a constraint on the uncontrolled input space for which this input achieves robust reachability.  
For the aforementioned example for instance, the target constraint is `&buf % 32 = 0 && &input % 32 = 0`.
We perform this computation with an abduction algorithm that, given an *inference language*, succesively selects candidate solutions and check them against robust reachability until a suitable solution is found.
Further exploration and minimization is performed to obtain the most general solution present in the inference language.

## Contributions

This paper makes the following contributions:
* We propose the first program-level abduction algorithm for robust trace property satisfaction, an extension of robust reachability. The technique is agnostic *w.r.t.* oracles, and is correct, complete, and minimal *w.r.t.* its inference language;
* We design dedicated pruning techniques to ensure that the computation time remains affordable;
* We implement a specialization of our algorithm to robust reachability of program locations and evaluate it on standard benchmarks from software verification (SVComp) and security analysis (FISSC);
* We propose an application to a realistic security evaluation scenario where we study the characterization of vulnerabilities introduced by fault injection attacks on typical implementations of a security primitive

We show that our method permits a relevant bug characterization and security evaluation that outperforms previous methods, both in the quantity of characterized bugs and in the quality of the characterizations.
It is typically capable of generating the constraint for the example mentioned above.

## Further information

* Read the [paper](/assets/publications/papers/2024-popl.pdf)
* To appear at [the 51st ACM SIGPLAN Symposium on Principles of Programming Languages (POPL 2024)](https://popl24.sigplan.org/) 
* [Artifact](https://doi.org/10.5281/zenodo.8421879)

