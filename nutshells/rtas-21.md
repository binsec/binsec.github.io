---
layout: post
title:  "RTAS'21: research paper [awarded RTAS'21 best paper]"
categories: new publication
paper-title: "No Crash, No Exploit: Automatic Verification of Embedded Kernels"
topic: "embedded system security; secure embedded kernels; formal methods; abstract interpretation; absence of privilege escalation"
pdf: "/assets/publications/papers/2021-rtas.pdf"
date: 2021-04-13
redirect_from: /new/publication/1970/01/01/nutshell-rtas-21.html
---


## Context

# Context
<img src="/assets/publications/pictures/rtas21.png" width="250" height="300"
     style="float: left; margin-right: 10px;" />
In most systems, the kernel is critical for security: it is the piece of
software that ensures isolation between programs, so you don't want it to fail
in that task. You don't want it to crash, either. We aim at verifying realistic
embedded kernels (e.g. used in the Internet of Things) and for that we want to devise an almost
fully automated approach that works on existing kernels.

We want to perform this verification at the executable level (“binary” code),
because all kernels use low-level, architecture-specific instructions. This is a
challenge due to the low-level nature of binary: no explicit control flow, no
types, legitimate integer overflows, etc.  Another challenge is unbounded loops.
Unbounded loops are loops whose number of iterations cannot be bounded
statically. A simple example of such a loop is iterating over a linked list.
Analyzing unbounded loops matters, because most real-time scheduling algorithms
contain such loops.

Another difficulty of analyzing embedded kernels is that they are usually
*parameterized*: values such as task priorities, or task memory region begin and
end addresses, may be constant throughout execution of the system, but are not
hard-coded in the kernel; instead, they are written in a special zone of memory
called the *interface* between kernel and tasks. The memory structure of this
zone is known but the precise values and addresses are not, which forces the
code analysis to step up in terms of complexity.

## Contributions

- A new method for verifying absence of privilege escalation and absence
  of crash in an embedded kernel, based on abstract interpretation (handles
  loops and verifies kernels independently of the application).
- A new abstraction of memory based on types, handling parameterization and
  improving scalability precision of the analysis.
- We applied the technique to two embedded kernels (including an industrial
  one), demonstrating that our method can verify kernels with less than 58 lines
  of manual annotations (in some cases, requiring 0 lines of annotations, i.e.
  the kernel is verified without human intervention).

## Further information

- Read the
  [**paper**](/assets/publications/papers/2021-rtas.pdf) and [extended technical report](/assets/publications/papers/2021-rtas-technical-report-analysis.pdf).
- To appear at the [Real-Time and Embedded Technology and Applications Symposium
  (RTAS'21)](http://2021.rtas.org/)
- Download the [tool and benchmark](https://github.com/binsec/rtas2021_artifact).
