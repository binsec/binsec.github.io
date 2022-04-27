---
layout: post
title:  "FPS'21: short research paper"
categories: new publication
paper-title: "A Tight Integration of Symbolic Execution and Fuzzing"
topic: "vulnerability detection; testing; fuzzing; symbolic execution;"
pdf: "/assets/publications/papers/2021-fps.pdf"
date: 2021-11-30
redirect_from: /new/publication/1970/01/01/nutshell-fps-21.html
---

# Context

*Automated test generation* is a key element of software engineering and security. It notably aims at generating test suites achieving a high coverage of the program's execution paths.

Two techniques are typically leveraged to achieve this:
- **Symbolic Execution** [1] (SE) methodically explores the execution paths of the program.
For each path, the symbolic analysis reasons about the input and creates a *path predicate*: a constraint on the input, with the guarantee that solutions will follow the targeted path.
By using an off-the-shelf constraint solver, SE generates a solution for each path predicate, theoretically exploring every single execution path of the program.
In practice, SE often fails to explore every path in a reasonable time frame, since any conditional statement or loop increases the number of paths, making SE easily a victim to a path explosion, while complex path predicates may blow up constraint solvers.
- **Fuzzing** [2] is a test technique which relies on brute-force test case generation.
The key idea is that by running the Program Under Test (PUT) on many different test cases, most of the execution paths will be explored, making it efficient to find vulnerabilities.
The first generation of fuzzers were blackbox fuzzers, generating test cases without any knowledge about the PUT, in a way akin to random testing.
Greybox fuzzers improved on the technique, keeping the core idea of brute-force test case generation, but combining it with lightweight analyses to guide the fuzzing.
For example, AFL [3], the state of the art of greybox fuzzers, uses branch coverage information.
It maintains a queue of "interesting" test cases, from which it selects test case to mutate in order to create new ones.
The PUT is executed on each new test case, and the coverage information is retrieved and compared to the coverage achieved until this point.
If the test case explored a new part of the program, it is considered to be interesting, and added to the queue.
Though this helps AFL improve coverage compared to blackbox fuzzers, it still struggles with nested conditions, as we show below.

# Motivation

The following program explifies some of the issues (nested conditions, loops) faced by symbolic execution and fuzzing.

```c
int main(int argc, char** argv) {

  char buf[64];
  int x, y;

  read(0, buf, 64);

  int cpt;
  for (cpt = 10; cpt < 30; cpt++) {
    if (buf[cpt] == cpt % 20)
      y += 1;
  }

  printf("%i\n", y);

  if (buf[0] == 'a')
    if (buf[4] == 'F')
      if (buf[7] == '6')
        x = 1;
      else
        x = 2;
    else
      x = 3;
  else
    x = 4;

  printf("%i\n", x);

  return 0;
}

```

## AFL

Two things prevent AFL from efficiently exploring the nested conditions in the program.
First, it needs to find a solution to the first condition, `buf[0] == 'a'`.
Without information about the semantics of the program, it has to randomly mutate the original test case until a solution is created.
Let us imagine such a solution, "aest" is created.
Using the coverage information, AFL detects that this test case followed a new execution path, and flags it as interesting.
At this point, we still need to create solutions to the second and third conditions, `buf[4] == 'F'` and `buf[7] == '6'`, while still satisfying the first one.
However, AFL does not understant what makes this test case pass through the first condition and what would make it pass through the two other ones.
As a result, it may mutate the first byte as often as the rest of the input, creating numerous test cases that do not satisfy the first condition anymore, let alone the following ones.
As for the loop, it is quickly explored by AFL, since it focuses on code coverage rather than path coverage.

## KLEE

KLEE [4] is a State of the Art symbolic execution tool.
As a result of the technique described above, solving the specific conditions is not an issue.
The symbolic analysis merely creates the path predicates for each path, `buf[0] = 'a'`, `buf[0] = 'a' and buf[4] = 'F'`, and so on, and uses a constraint solver to create solutions.
On the other hand, KLEE will actively try to explore every possible path of the for loop, leading to path explosion.
We see this when we modify the number of loop iterations, KLEE going from 0.3s to explore the whole program when there is no iteration, to 133s when there are 20 iterations.

# Our Proposal

In this paper, we describe our proposed solution: a combination of symbolic execution and fuzzing, where both techniques have been modified to allow for a tight integration.
The resulting tool uses symbolic reasoning to guide the exploration past difficult conditions, while keeping fuzzing's efficient test generation to speed up code exploration.

In particular, we created **Lightweight Symbolic Execution** (LSE), a variant of Symbolic Execution which targets conditions in the path and generates easily-enumerable path predicates.
The target language for said predicates is a subset of the constraint language usually used in symbolic execution.
As a result, our path predicates are harder to generate and usually over-approximated -- though we prove them to be correct -- but solvable directly by a fuzzer, without relying on an heavyweight constraint solver.

We combined LSE to a constrained fuzzer, which we use to generate the solution test cases.
Built on top of AFL, and using its coverage technique, our constrained fuzzer not only mutates test cases, but keeps track of the path predicate associated with each test case, systematically creating solutions to the predicates.

In particular, if we have the "aest" solution for the program above, LSE will identify what makes the test case interesting (here, that `buf[0] = 'a'`), and constrained fuzzing will create multiple solutions (here, exploring the parts of the cond beyond that condition).
Moreover, we can also leverage LSE to find solutions to new conditions by negating the condition from the trace.
For example here, explicitely leading the exploration past the `buf[4] = 'F'` condition by crafting a solution.

## Contribution

- We introduce Lightweight Symbolic Execution (LSE), a variant of symbolic execution tailored for tight integration with fuzzing.
LSE relies on the novel notion of easily-enumerable path predicates, and avoids the need for any external constraint solver;
- We show how Lightweight Symbolic Execution can be smoothly integrated with fuzzing, through the novel idea of Constrained Fuzzing, communicating through easily-enumerable path predicates, yielding fast (solver-less) test case enumeration together with targeted symbolic reasoning;
- Finally, we have implemented these ideas in an early prototype named ConFuzz, built on top of Binsec and AFL, and provide promising preliminary experiments against standard tools.

# Further information

- Read the
  [**paper**](/assets/publications/papers/2021-fps.pdf). 
- To appear at the [International Symposium on Foundations & Practice of Security (FPS)](http://www.fps-2021.com)

# References
- [1] [C. Cadar and K. Sen - *Symbolic execution for software testing: three decades later*, ACM Communications '13](https://dl.acm.org/doi/10.1145/2408776.2408795)
- [2] [V. J. M. Manès, H. Han, C. Han, S. K. Cha, M. Egele, E. J. Schwartz, and M. Woo - *The art, science, and engineering of fuzzing: A survey*, IEEE TSE '19.](https://www.computer.org/csdl/journal/ts/5555/01/08863940/1e0YnO3GyJO)
- [3] [AFL's website](https://lcamtuf.coredump.cx/afl/)
- [4] [C. Cadar, D. Dunbar, D. R. Engler, et al - *Klee: unassisted and automatic generation of high-coverage tests for complex systems programs*, OSDI '08.](https://www.usenix.org/conference/osdi-08/klee-unassisted-and-automatic-generation-high-coverage-tests-complex-systems)

