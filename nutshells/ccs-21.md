---
layout: post
title:  "CCS'21: research paper"
categories: new publication
paper-title: "Search-Based Local Black-Box Deobfuscation: Understand, Improve and Mitigate"
topic: "reverse engineering, deobfuscation, artificial intelligence, program synthesis"
pdf: "/assets/publications/papers/2021-ccs.pdf"
date: 2021-07-13
redirect_from: /new/publication/1970/01/01/nutshell-ccs-21.html
---

## Motivation

Lets consider that you are reverse engineering a program (e.g. a malware). If you are unlucky (and you will probably be) this code snippet will be obfuscated i.e. translated to an equivalent complex program to make analysis harder. So how can you understand it ? You could try to simplify the code by hand but this is error prone and very time consuming. 
Thus, you may leverage automated symbolic-based methods. However, some tricky *local* code snippets can be so complex that analysis will fail or return incomprehensible information. This is where **blackbox deobfuscation** (introduced by [Blazytko et al.](https://www.usenix.org/conference/usenixsecurity17/technical-sessions/presentation/blazytko)) comes into play: it simplifies local code fragments that have been highly obfuscated and returns understandable version of them.

**Example.** Consider the following code snippet (written in C instead of assembly for better readability):


```c
int a = 2*(x | 2*y) - (x ^ 2*y) - x - y;
int cond = 2*(x | y) - (~x & a) - (x & ~y);
if ( cond == 5 ) {
    ...
} else {
    ...
}
```

A natural question is: *when is the condition verified or not ?* Using usual symbolic methods, you will retrieve that the condition is verified if and only if 
`(2*(x | y) - (~x & (2 * (x | 2*y) - (x ^ (2 * y)) - x - y)) - (x & ~y) == 5)`. This is not really helpful. However, if we apply blackbox deobfuscation
we will easly found out that the condition is verified if and only if: `x + y == 5`. This is a lot more understandable.


## Blackbox deobfuscation

But how does it work ? Blackbox deobfuscation relies on observed inputs/outputs (I/O) relations to synthesize an expression which mimics observed behaviors. For example, to synthesize the previous condition, we compute `cond` for distinct value of `x` and `y`. Thus, we get the following I/O relations:

<center>

<style type="text/css">
.tg  {border-collapse:collapse;border-color:#93a1a1;border-spacing:0;}
.tg td{background-color:#fdf6e3;border-color:#93a1a1;border-style:solid;border-width:1px;color:#002b36;
  font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:6px 15px;word-break:normal;}
.tg th{background-color:#657b83;border-color:#93a1a1;border-style:solid;border-width:1px;color:#fdf6e3;
  font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:6px 15px;word-break:normal;}
.tg .tg-baqh{text-align:center;vertical-align:top}
.tg .tg-wa1i{font-weight:bold;text-align:center;vertical-align:middle}
.tg .tg-nrix{text-align:center;vertical-align:middle}
</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-wa1i">x</th>
    <th class="tg-wa1i">y</th>
    <th class="tg-wa1i">cond</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-nrix">0</td>
    <td class="tg-nrix">0</td>
    <td class="tg-nrix">0</td>
  </tr>
  <tr>
    <td class="tg-nrix">1</td>
    <td class="tg-nrix">4</td>
    <td class="tg-nrix">5</td>
  </tr>
  <tr>
    <td class="tg-nrix">10</td>
    <td class="tg-nrix">40</td>
    <td class="tg-nrix">50</td>
  </tr>
  <tr>
    <td class="tg-baqh">-5</td>
    <td class="tg-baqh">20</td>
    <td class="tg-baqh">15</td>
  </tr>
  <tr>
    <td class="tg-baqh">0</td>
    <td class="tg-baqh">10</td>
    <td class="tg-baqh">10</td>
  </tr>
  <tr>
    <td class="tg-baqh" colspan="3">...</td>
  </tr>
</tbody>
</table>

</center>

From these I/O samples, a blackbox deobfuscator synthesizes a coherent expression. In our example, it would infer that `cond = x + y` and we can conclude that the condition is `x + y == 5`.


**Goal.** Performing efficient synthesis is crucial for blackbox deobfuscation. It must return *correct* and *simple* results while being *fast* and able to *handle a wide variety of distinct expressions*. Thus, the goal of the paper is to study *how to perform efficient synthesis for blackbox deobfuscation*. This leads us to discuss limits of the approach and propose the first anti-blackbox deobfuscation protections.

## Major Contributions

In summary, this paper makes the following major contributions:
* We extend the evaluation of the state-of-the-art blackbox deobfuscator Syntia, relying on Monte Carlo Tree Search (MCTS) for the synthesis, and highlight new strenghts and weaknesses of Syntia;
* We promote the use of S-metaheuristics in place of MCTS for efficient blackbox deobfuscation. We implement our approach in a tool dubbed *Xyntia* and show that it highly outperforms Syntia, being faster and handling more complex expressions.
* Finaly, we propose the first protections against blackbox deobfuscation.


## Further information

* Read the [paper](/assets/publications/papers/2021-ccs.pdf)
* To appear at the [ACM Conference on Computer and Communications Security (CCS) 2021](https://www.sigsac.org/ccs/CCS2021/) 
* Try ([Xyntia](https://zenodo.org/record/5094898#.YO2UchMzbSw)) and get our ([benchmarks](https://github.com/binsec/xyntia))
