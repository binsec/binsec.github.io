---
 layout: post
 title:  "PLDI'24, research paper"
 categories: new publication
 paper-title: "Quantitative Robustness for Vulnerability Assessment"
 topic: "program reliability and security; reachability; symbolic execution;"
 pdf: "/assets/publications/papers/2024-pldi-qrse.pdf"
 date: 2024-05-31
---


# Quantitative Robustness for Vulnerability Assessment

## Motivation

Most software analysis techniques focus on bug reachability, i.e., proving the existence of inputs triggering a bug.
However, this approach is not ideal for security evaluation as it does not take into account how reliably said bugs can be triggered.
Indeed, some inputs may not be easily controlled by an attacker in practice, such as sources of randomness for instance.

**Robust reachability.** 
To tackle this issue, we introduced the notion of *robust reachability* in a previous paper (*put link here*).
Given a partition of inputs between *controlled* and *uncontrolled* ones, a program location is *robustly reachable* if and only if it can always be reached regardless of the value of uncontrolled inputs.

While robust reachability is a strong indicator of reliability, it suffers from its lack of nuance.
Indeed, some bugs may not be robustly reachable, yet "easy enough" to trigger to be exploitable.

## Example

```
/* main privilege levels */
# define DEFAULT_LEVEL 1
# define OPERATOR_LEVEL 100
# define ADMIN_LEVEL 9000
/* commands */
# define DROP_PRIVILEGE 0
# define DROP_PRIVILEGE_LEGACY 1
# define GET_VERSION 2
# define SUDO 3

uint32_t uninit ; /* random data */
uint32_t privilege_level = DEFAULT_LEVEL ;

void set_privilege_level ( uint32_t new) {
	privilege_level = new;
}

uint32_t get_privilege_level () {
	// bug: return uninitialized memory
	return uninit ;
}

void prog1 ( uint32_t command , uint32_t argument ) {
	if ( command == GET_VERSION ) {
		/* harmless */
	} else {
		/* command is sudo */
		if ( get_privilege_level () == OPERATOR_LEVEL ) {
			set_privilege_level ( ADMIN_LEVEL );
		}
	}
}

void prog2 ( uint32_t command , uint32_t argument ) {
	switch ( command ) {
		case GET_VERSION : /* harmless */ break ;
		case DROP_PRIVILEGE :
		case DROP_PRIVILEGE_LEGACY :
			if ( argument < get_privilege_level ()) {
				set_privilege_level ( argument );
			}
	}
}
```

In prog1, when attackers play perfectly by choosing command = 1, they need to be lucky to : only one value of uninit out of $2^{32}$ grants them admin privileges. 
However in prog2, for command = 1 and argument = 9000 over 99% of the values that uninit can take will let attackers achieve their goal.

## Quantitative Robustness

In this paper, we define a new, *quantitative* notion of robust reachability, *quantitative robustness*, as the maximum proportion of uncontrolled inputs triggering the bug for a chosen controlled input.
Using this definition, we can now distinguish the two bugs from the previous example.

**Quantitative Robust Symbolic Execution (QRSE).**
To determine the quantitative robustness of a bug, we first use symbolic execution to obtain constraints for the execution paths reaching it.
We can then compute quantitative robustness either for each path separately, or on a single, merged constraint.
Since it is often enough to shown that overall quantitative robustness exceeds a threshold in practice, we can stop exploring new paths once this condition is met.

**Measuring quantitative robustness.**
Quantitative robustness can be reduced to the *f-E-MAJSAT* problem, which previously had applications in some AI-related fields.
Existing algorithms can thus be used and we also propose a new approximate one called *Relax*.

**Experiments**
We applied QRSE to the benchmark from our previous robust reachability paper, which includes several real-world vulnerabilities, as well as to a fault analysis benchmark.
The results showcase the usefulness of the increase in precision from robust reachability to quantitative robustness.
In addition, we compared the performance of existing f-E-MAJSAT solvers as well as Relax on 117 formulas from our previous experiments.
Overall, most existing algorithms perform poorly in the context of quantitative robustness, except for the most naive one, surprisingly.
Relax also proves useful in approximately solving additional instances.

## Contributions

In summary, we make the following contributions:
- We define a quantitative pendant of robust reachability called quantitative robustness, which generalize both reachability and robust reachability.
- We propose Quantitative Robust Symbolic Execution (QRSE), a variant of symbolic execution for computing quantitative robustness. We show that QRSE does not strictly require path merging, contrary to robust reachability analysis.
- We propose a method reducing path-wise quantitative robustness with finite variable domains (e.g., bitvectors and arrays) to f-E-MAJSAT, a counting problem studied in some sub-fields of AI. 
As off-the-shelf methods turn out to be too inefficient or imprecise for our purposes, we introduce a new approximate algorithm called Relax.
- We have implemented these ideas in two tools: BINSEC/QRSE and the Popcon solver. 
First experiments on security-oriented case studies demonstrate that QRSE enables fine-grained bug triage compared to symbolic execution and robust symbolic execution.
In addition, Relax solves more problems arising from QRSE than prior techniques while minimizing approximation.
