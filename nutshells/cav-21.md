---
layout: post
title:  "CAV'21: research paper"
categories: new publication
paper-title: "Not All Bugs Are Created Equal, But Robust Reachability Can Tell The Difference"
topic: "program reliability and security; reachability; symbolic execution;"
pdf: "/assets/publications/papers/2021-cav.pdf"
date: 2021-06-21
redirect_from: /new/publication/1970/01/01/nutshell-cav-21.html
---

# Context

Many problems in software verification are encoded as reachability queries of
some undesired condition—a bug, the exploitation of a vulnerability, etc. When
a verification engine establishes that a certain buggy location in the program
is reachable, an input triggering the bug is reported to the developer so that
it can be fixed. In the case of techniques based on an under-approximation of
program behaviors, like symbolic execution or bounded model checking, we even
have in principle the guarantee that the reported issue is real: there are no
false positives.

However, things are more subtle in practice, as some bugs can be triggered
reliably whereas others only happen in very specific and highly improbable
initial conditions. While standard reachability cannot tell the difference,
this distinction is crucial in many real-life scenarios, notably in
security-oriented contexts (bug triage, bug prioritization, criticality
assessment). For example, with symbolic execution one can prove that some
executable affected by [CVE-2019-15900](https://nvd.nist.gov/vuln/detail/CVE-2019-15900) is vulnerable when (among others) the
initial memory address `0xffefffff` contains 42 and the stack starts at
`0xfff0001f`. On modern systems, the address of the stack is intentionally
randomized and changes on each execution, and initial memory can contain
anything. For this reason, these specific initial conditions are highly
improbable, and the attacker cannot make them happen. As a result, this report
does not tell us much about whether this can happen in practice and could be
weaponized by an attacker: while symbolic execution rightfully proves that this
bug is *reachable*, it might possibly not be a security issue, but rather a
form of *false positive*.

In fact, we would like to prove a *stronger property*: can this bug be
triggered *reliably*, e.g. independently of the address of the stack and of the
content of initial memory in the example above? The goal of this paper is to
introduce a formalisation of this intuition under the form of a new property
called *robust reachability*.  Our approach consists in partitioning inputs of
the program into controlled inputs and uncontrolled inputs.  The initial
content of the memory would typically be labeled as uncontrolled (think of
“uncontrolled by the attacker”) in typical threat models.  A bug is robustly
reachable if there exist controlled inputs, such that for all uncontrolled
inputs, this bug is reached.  In other words, the attacker can take advantage
of the bug *reliably* without relying on good luck to get the right
uncontrolled inputs.

Equipped with this new property, we can show that in our executable
CVE-2019-15900 is robustly reachable: there are cases where the attacker wins
every time, whatever the initial content of memory! This time, the result of
the analysis is a clear signal of the threat posed by this bug.

# Contributions

- We formally introduce the concept of robust reachability and motivate its use, giving practical examples where standard reachability
leads to false positives in practice (whatever the underlying verification technology). We also characterize robust reachability in terms of temporal logic
and hyperproperties, and compare it with non-interference
- We revisit Symbolic Execution and Bounded Model Checking and show how they can be lifted to the robust case. While they
both have the same deductive power in the standard case, they do not anymore in the robust setting—yet, path merging allows Robust SE to pace up
with Robust BMC. Finally, we show how to adapt standard optimizations
for Symbolic Execution and Bounded Model Checking
- We implement and evaluate the first symbolic execution engine
dedicated to robust reachability, namely Binsec/RSE. We show how to
use it for criticality assessment of 4 existing vulnerabilities (CVEs), and
compare it with standard symbolic execution. RSE appears to be tractable
with reasonable overhead, yielding false-positive-free symbolic reasoning.

# Further information
- [Read the paper](/assets/publications/papers/2021-cav.pdf)
- To appear at [CAV 2021](http://i-cav.org/2021/)
- Download the source of our implementation [Binsec/RSE](https://github.com/binsec/cav2021-artifacts) or pre-installed [artifacts](https://zenodo.org/record/4721753)
