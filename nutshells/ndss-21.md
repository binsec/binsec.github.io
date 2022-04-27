---
layout: post
title:  "NDSS'21: research paper"
categories: new publication
paper-title: "Hunting the Haunter — Efficient Relational Symbolic Execution for Spectre with Haunted RelSE"
topic: "Relational symbolic-execution; Binary analysis; Cryptographic constant-time; Spectre attacks"
pdf: "/assets/publications/papers/2021-ndss.pdf"
date: 2021-03-08
redirect_from: /new/publication/1970/01/01/nutshell-ndss-21.html
---

# Motivation
**Spectre attacks** [1] are microarchitectural attacks, made public in
2018, that exploit processor optimizations in order to leak secret
data manipulated by a program. In particular, these attacks exploit
the *speculation mechanisms* used in processors to execute instruction
ahead of time and improve the performance.

An attacker can mistrain the predictor in order to trigger a wrong
prediction on the victim side. Following the wrong prediction, the
victim executes *transient* instructions (i.e. incorrect instructions
resulting from a misprediction). When the processor realizes that its
guess is incorrect, it reverts the architectural state to the state
before the prediction. However, transient execution leave
microarchitectural side effects that can be exploited by an attacker
to recover secrets data.

Take for instance the following program where the `idx` argument is
attacker controlled and a secret is stored in `secretarray`. Note that
the function `leakThis(toLeak)` leak the value of `toLeak` by encoding
it in the cache where it can later be retrieved by an attacker via
cache attacks.

In the *regular semantics* of the program, the attacker does not have
access to the value of `secretarray` (even if we consider that
they can observe the state of the cache and recover the value of
`toLeak`).

However, with Spectre attacks, an attacker can mistrain the branch
predictor to force the victim's program to speculatively execute the
*true* branch of the if-statement and execute
`leakThis(publicarray[131088])` which leaks the value of
`secretarray[0]`.

``` c
uint32_t publicarray_size = 16;
uint8_t publicarray[16] = { 1 .. 16 };
uint8_t publicarray2[512 * 256];
uint8_t secretarray[16]; // Secret data

// This function encodes toLeak in the cache
void leakThis(uint8_t toLeak) {
  tmp &= publicarray2[toLeak * 512];
}

void case_1_masked(uint32_t idx) { // idx=131088
  if(idx < publicarray_size) {   // Mispeculated
    // Out-of-bound read, reads secretarray[0]
    uint8_t toLeak = publicarray[idx];
    leakThis(toLeak); //Leaks secretarray[0]
  }
}
```

This type of Spectre attack which exploits the conditional branch
predictor, is called Spectre-PHT (or Spectre-v1). There are four
variants of Spectre based on the speculation mechanism they exploit,
and we refer the interested reader to an excellent survey by Canella,
Claudio, et al. [2] if they want to learn more about these variants.
In our work we focus on two variants, Spectre-PHT (a.k.a Spectre v1)
and Spectre-STL (a.k.a Spectre v4) respectively exploiting conditional
branch predictions and memory disambiguation mechanism in
store-to-load forwarding (more details in the paper).

In the end, Spectre attacks exploit *speculation mechanisms* and
*microarchitectural side-channels*.

## Speculative Constant-time
To protect against microarchitectural side-channels, cryptographic
libraries such as OpenSSL, Libsodium, etc., use **constant-time
programming**.  It basically means that the program is designed in
such a way that the **timing behavior of the program and the
microarchitectural state are independent from the secrets** (see [our
article on
Binsec/Rel](/new/publication/2020/05/18/s&p20.html)
for more information on timing attacks and constant-time).

The problem with constant-time is that **it is not sufficient to
protect against Spectre attacks** (for instance our illustrative
example is constant-time but still vulnerable to Spectre attacks).

**Speculative constant-time** [3] is an adaptation of constant-time
that includes the speculative semantics of the program and allows to
reason about Spectre attacks. Still, few verification tools are able
to detect Spectre attacks or verify speculative constant-time and most
of them do not scale.

**Challenge:** The speculative semantics of the program introduces new
*transient* behaviors that must be modeled by verification tools. If
not handled carefully, these new behaviors can quickly leads to state
explosion. The challenge is to optimize the exploration in order to
make the analysis applicable to real code.
<br/><br/>

# Our proposal
Symbolic analyzers for Spectre model the speculative behavior
*explicitly*, by forking the execution to explore transient paths.
The adaptation of symbolic execution to constant-time-like properties,
known as relational symbolic execution (RelSE), has proven very
successful in terms of scalability and precision for binary level.
Our key technical insight is adapt RelSE to represent transient
executions *at the same time* as regular executions (i.e. executions
related to correct speculations). We name this technique *Haunted
RelSE*.

We implement Haunted RelSE in a tool called
[*Binsec/Haunted*](https://github.com/binsec/haunted), built on top of
[Binsec/Rel](https://binsec.github.io/new/publication/2020/05/18/s&p20.html)
[4], our prior tool for constant-time analysis at binary level. We
Evaluate it on small test cases as well as real-world cryptographic
code from donna, Libsodium and OpenSSL libraries. Interestingly, our
experiments revealed that (1) index-masking, a well-known defense used
against Spectre-PHT, may introduce new Spectre-STL
vulnerabilities---for which we propose and verify correct
implementations; (2) and that a popular option of gcc to generate
position-independent code (PIC) may introduce Spectre-STL
vulnerabilities.

## Contributions
- We design a dedicated technique on top of RelSE, named Haunted
  RelSE, to efficiently analyze speculative executions and detect
  Spectre-PHT and -STL violations, and formally prove that this
  technique is correct;
- We propose a verification tool, called Binsec/Haunted, implementing
  Haunted RelSE and evaluate it on small test cases as well as
  real-world cryptographic code. Our experimental evaluation shows
  that Binsec/Haunted can find violations of speculative constant-time
  in real-world cryptographic code, such as donna, Libsodium and
  OpenSSL libraries and is more efficient than prior tools;
- To the best of our knowledge, we are the first to report that
  index-masking--a well-known defense against Spectre-PHT--and PIC
  options from the gcc compiler may introduce Spectre-STL
  vulnerabilities.

# Further information
- Read the
  [**paper**](/assets/publications/papers/2021-ndss.pdf)
- To appear at the [Network and Distributed System Security Symposium
  (NDSS'21)](https://www.ndss-symposium.org/ndss2021/)
- Download the [tool](https://github.com/binsec/haunted) and
[benchmarks](https://github.com/binsec/haunted_bench).
- See our related [LASER workshop talk](/assets/publications/slides/2021-laser.pdf) about our experimental evaluation.
<br/><br/>

# References
- \[1\] [Kocher, P., Horn, J., Fogh, A., Genkin, D., Gruss, D., Haas,
  W., Hamburg, M., Lipp, M., Mangard, S., Prescher, T. and Schwarz,
  M. -- *Spectre attacks: Exploiting speculative execution.*,
  SP'19](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=8835233)
- \[2\] [Canella, C., Van Bulck, J., Schwarz, M., Lipp, M., Von Berg,
  B., Ortner, P., Piessens, F., Evtyushkin, D. and Gruss, D. -- *A
  systematic evaluation of transient execution attacks and defenses.*,
  USENIX Security
  '19](https://www.usenix.org/system/files/sec19-canella.pdf)
- \[3\] [Cauligi, S., Disselkoen, C., von Gleissenthall, K., Tullsen,
  D.M., Stefan, D., Rezk, T. and Barthe, G., *Constant-time
  foundations for the new spectre era.*,
  PLDI'20](http://cseweb.ucsd.edu/~dstefan/pubs/cauligi:2020:ct-foundations.pdf)
- \[4\] [Daniel, L., Bardin, S., Rezk, T., *Binsec/Rel: Efficient
  Relational Symbolic Execution for Constant-Time at Binary-Level*,
  SP'20](/assets/publications/papers/2020-sp.pdf)
