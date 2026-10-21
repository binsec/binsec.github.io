---
layout: post
title:  "CCS'26: research paper"
categories: new publication
paper-title: "Redefining Grey-Box Deobfuscation through Context-Aware Program Synthesis, and Application to MBA"
topic: "Reverse engineering, Deobfuscation, Artificial intelligence, Program synthesis"
pdf: "/assets/publications/papers/2026-ccs.pdf"
date: 2026-10-19
---

## Context

Obfuscation is a family of methods that takes a program and translates it into a new program, harder to analyze and understand. 
While obfuscation can be used to protect intellectual property, malware authors also use it to impede detection and 
slow down remediation. 

Hence, multiple automated deobfuscation methods have been proposed to simplify obfuscated software. 
While black-box deobfuscation, via program synthesis, elegantly bypasses syntactic complexity by reasoning purely over input/output semantics, it suffers from fundamental scalability limitations as the underlying semantic complexity grows. On the other hand, white-box approaches struggle to cope with the increase of syntactic complexity. The few existing grey-box approaches attempt to bridge this gap, but suffer from fundamental limitations that hinder their effectiveness on large and complex obfuscated programs. We argue that these limitations stem from a missing abstraction layer between syntax and semantics. 

**Goal:** In this paper, we precisely revisit grey-box deobfuscation as the missing abstraction layer between syntax and semantics, enabling scalable deobfuscation by combining structural guidance with observational semantics.


## Motivation

As shown in our paper, grey-box deobfuscation faces multiple core challenges. Consider the expression: *(5x ^ 6y) + 2 * (5x & 6y)*. It is an obfuscated version of *5x + 6y*. To simplify it, grey-box approaches must specify which sub-parts of the expression synthesis must be called on. A standard solution is to use a horizon, from which all sub-expressions are anonymized. For instance, at horizon 3, we would get the expression:

<br>
<center>
<img src="/assets/img/grey-box.png" alt="drawing" width="50%"/>
</center>

Unfortunately, applying horizons incrementally is not an effective solution. Indeed, as shown in the table below, the resulting expressions are either too simple to yield a simplification (horizons 2 and 3) or the synthesis fails (horizons 4 and 5). In the end, grey-box approaches fail to simplify this expression. 
A new framework for grey-box deobfuscation is thus needed. 

<br>
<center>
<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:0px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:0px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-ewnm{border-color:inherit;border-width:1px;text-align:center;vertical-align:top}
.tg .tg-6a0s{border-color:inherit;border-width:1px;text-align:left;vertical-align:top}
.tg .tg-an7b{border-width:1px;text-align:center;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-6a0s"><span style="font-weight:bold">Horizon</span></th>
    <th class="tg-6a0s"><span style="font-weight:bold">Expression to synthesize</span></th>
    <th class="tg-6a0s"><span style="font-weight:bold">Synthesis success</span></th>
    <th class="tg-6a0s"><span style="font-weight:bold">Simplification</span></th>
    <th class="tg-an7b"><span style="font-weight:bold">Failure reason</span></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-6a0s">h=2</td>
    <td class="tg-ewnm">a1 + a2</td>
    <td class="tg-ewnm">yes</td>
    <td class="tg-ewnm">no</td>
    <td class="tg-an7b">No simplification possible</td>
  </tr>
  <tr>
    <td class="tg-6a0s">h=3</td>
    <td class="tg-ewnm">(a1 ^ a2) + 2 * a3</td>
    <td class="tg-ewnm">yes</td>
    <td class="tg-ewnm">no</td>
    <td class="tg-an7b">No simplification possible</td>
  </tr>
  <tr>
    <td class="tg-6a0s">h=4</td>
    <td class="tg-ewnm">(5x ^ 6y) + 2 * (a1 &amp; a2)</td>
    <td class="tg-ewnm">no</td>
    <td class="tg-ewnm">no</td>
    <td class="tg-an7b">Mix of 5x, 6y, a1, and a2, without relations between them</td>
  </tr>
  <tr>
    <td class="tg-6a0s">h=5</td>
    <td class="tg-ewnm">(5x ^ 6y) + 2 * (5x &amp; 6y)</td>
    <td class="tg-ewnm">no</td>
    <td class="tg-ewnm">no</td>
    <td class="tg-an7b">To complex for synthesis</td>
  </tr>
</tbody></table>
</center>


## Major Contributions

In summary, this paper makes the following major contributions:

1. **Conceptual Framework:** We redefine the shift from isolated black-box synthesis to a contextual grey-box
strategy that uses AST structural properties to guide local synthesis. Our new definition extends the tool-centric view
proposed in prior work to provide a general framework for grey-box deobfuscation. Moreover, we highlight 4 main challenges 
faced when building grey-box deobfuscation. We argue that solving these challenges is key to get efficient and robust grey-box methods;

2. **GreySynth Algorithm:** We present our dual-engine architecture, composed of OES and CHS, to recover heavily obfuscated expressions efficiently. This leads to the
GreySynth framework: the first principled method that tackles the core challenges of grey-box deobfuscation;

3. **Real-world dataset with ground truth expressions:** Current evaluations of deobfuscation often rely on datasets containing 
only synthetic expressions. This is a major limitation, especially when evaluating grey-box approaches, which are
affected by both the syntax and the semantics of the code. Hence, we build a new dataset of real-world expressions,
extracted from a large corpus of well-known C libraries, and obfuscated with the obfuscators Tigress, OMVLL, and MBA Obfuscator.

4. **Experimental Evaluation:** We xtensively evaluate GreySynth against the state-of-the-art black-, white- and grey-box deobfuscators. Experiments show that it highly 
outperforms all competitors. In particular, it achieves near-perfect simplification of general obfuscated expressions, while competitors produce expressions 4 to 135 times larger. Moreover, we show that GreySynth is the only solution that scales to extremely complex obfuscations. In contrast, competitors' performance breaks down (GreySynth retrieves expressions 2 times larger than the ground truth in the worst cases, whereas competitors' results are 494 times larger at best).

## Further information

* Read the [paper](/assets/publications/papers/2026-ccs.pdf)
* To appear at the [ACM Conference on Computer and Communications Security (CCS) 2026](https://www.sigsac.org/ccs/CCS2026/) 
* Try [Xyntia](https://github.com/binsec/xyntia) and get our [artifacts](https://doi.org/10.5281/zenodo.22649859)
