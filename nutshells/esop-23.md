---
layout: post
title:  "ESOP'23: research paper"
categories: new publication
paper-title: "Adversarial Reachability for Program-level Security Analysis"
topic: "fault injection, symbolic execution"
pdf: "/assets/publications/papers/2023-esop.pdf"
date: 2023-04-21
---


## Context

Major works have delved into program analysis over the last decades to hunt for software vulnerabilities and bugs in programs, leading to  industrial adoption in some leading companies.
Yet, stepping back from these successes, it appears that all these methods consider a rather weak threat model, where the attacker can only craft smart ``inputs of death'' through legitimate input sources of the program, exploiting corner cases in the code itself.  
Tools only looking for bugs and software vulnerabilities may deem a program secure while the bar remains quite low for an *advanced attacker*, able for example to  take advantage of attack vectors such as (physical) hardware fault injections, micro-architectural attacks, software-based hardware attacks like Rowhammer, or any combination of vectors.  
While previously limited to high-security devices and systems such as smart cards and cryptography modules, these fault-based attacks can now target a wider spectrum of systems, such as  bootloaders, firmware update modules, security enclaves, etc. 

## Motivation

Consider the following program, inspired by VerifyPIN.

```c
bool g_authenticated;
int u1, u2, u3, u4, ref1, ref2, ref3, ref4;

void verifyPIN() {
    int res = 1;
    res = res * (u1 == ref1);
    res = res * (u2 == ref2);
    res = res * (u3 == ref3);
    res = res * (u4 == ref4);
    g_authenticated = res;
}

void main(int argc, char const *argv[]) {
    /* State initialization */
    assert(u1!=ref1 || u2!=ref2 || u3!=ref3 || u4!=ref4);
    /* Call to targeted code */
    verifyPIN();
    /* Security oracle */
    assert(g_authenticated == true); 
}

```
  
The user PIN digits *u1* to *u4* are checked against the reference digits *ref1* to *ref4*, using the accumulator *res*. The attacker seeks to be authenticated (validate the assert l.16) without knowing the right digits (l.15).  

As expected, standard symbolic execution tools do not report any violation here, as they consider the simplest possible attacker. The attacker indeed cannot succeed by only crafting legitimate inputs. However, an advanced attacker can leverage more powerful attack vectors to inject faults into the program in order to succeed. For instance, corrupting *g_authenticated* to *true* at l.10 achieves the attacker goal. It could be obtained for example through a physical or Rowhammer attack.

Tool simulating fault injection at software-level are called software-implemented fault injection tools (SWiFI). The rare prior works in the field rely mostly on two techniques, each yielding scalability issues. 
Mutant generation consists in creating copies of the program with a slight modification representing the effect of a fault. Each mutant is then analyzed, leading to a mutant explosion.
A forking analysis explores program paths, forking at each possible fault location, creating a path where the fault occurred and also pursuing the path where there is no fault at this location. This results in a path explosion.

We can use SWiFI techniques to evaluate the security of our motivating program. They will find possible attack paths, yet they do not scale with multiple faults. For instance, a forking analysis (keep in mind that mutant generation scales worse) would explore 166 paths in 0.6 seconds for 1 fault,  2994 paths in 11 seconds for 2 faults, and it gets exponentially worse (x10) for each extra fault (we halted at 4 faults with a 12h timeout).

On the contrary, our novel *forkless* algorithm, simulates fault injection without creating new paths.  In this example, it shows a constant runtime as the number of faults increases from 1 to 10. We explore 9 paths in 0.2 seconds in all cases.


## Adversarial Reachability & BINSEC/ASE

The first challenge is to provide a formal framework to study what an advanced attacker can do to attack a  program. Interestingly, while such frameworks are routinely used in cryptographic protocol verification, none has been studied for program-level analysis.
We propose an extension to the transition system representing a program, where we add a new type of transition, the *faulted* transitions, representing the effect of a fault onto the program. The attacker model in translated to constrains on the *faulted* transitions. For instance, where they can occur, how many can be injected, what part of the state they can affect, etc. 
We define Adversarial Reachability, extending the concept of standard reachability in this extended transition system. It enables to reason about the reachability under attack of a location of interest to an attacker.

The second challenge is to design an efficient algorithm to assess the vulnerability of a program to a given attacker model, while adding capabilities to the attacker naturally gives rise to a significant path explosion -- especially in the case of multiple fault analysis. 
We propose Adversarial Symbolic Execution (ASE), an algorithm which includes fault injection into standard symbolic execution with a new forkless encoding. This encoding wraps arithmetically an assignment with the effect of a fault, and can support a variety of fault models. The forkless encoding prevents path explosion experienced by the state-of-the-art and enables scaling in number of faults.
We implemented ASE inside the BINSEC[1] symbolic execution engine.


## Major Contributions

In summary, this paper makes the following major contributions:
- We formalize the adversarial reachability problem extending standard reachability to take into account an advanced attacker, together with the associated correctness and completeness definitions;
- We describe a new symbolic exploration method, adversarial symbolic execution, to answer adversarial reachability, featuring a novel forkless
fault encoding to prevent path explosion and two optimization strategies to reduce fault injection. We establish their correctness and completeness;
- We propose an implementation of our techniques for binary-level analysis, on top of the BINSEC framework [1]. We systematically evaluate its performances against prior work, using a standard SWiFI benchmark from physical fault attacks and smart cards. Experiments show a very significant performance gain against prior approaches, for example up to x10 and x215 times on average for 1 and 2 faults respectively with a similar reduction in the number of adversarial paths. Moreover, our approach scales up to 10 faults whereas the state-of-the-art starts to timeout for 3 faults ;
- We finally perform a security analysis of the WooKey bootloader, a very well tested real-life security-focused program. We were able to find known attacks and evaluate the adequacy of some of the countermeasures.Especially, we found an attack not taken into account in a recently proposed patch [2], and proposed a new patch to the developers.

## Further Information

- Read the paper.
- To appear at the [2023 ETAPS conference](https://etaps.org/2023/conferences/).
- Download the [tool and benchmarks](https://github.com/binsec/esop2023_artefact).

## References

[1] DAVID, Robin, BARDIN, Sébastien, TA, Thanh Dinh, et al. BINSEC/SE: A dynamic symbolic execution toolkit for binary-level analysis. SANER 2016.

[2] LACOMBE, Guilhem, FELIOT, David, BOESPFLUG, Etienne, et al. Combining Static Analysis and Dynamic Symbolic Execution in a Toolchain to detect Fault Injection Vulnerabilities. PROOFS WORKSHOP 2021.
