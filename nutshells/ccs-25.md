---
layout: post
title:  "CCS'25: research paper"
categories: new publication
paper-title: "Augmenting Search-based Program Synthesis with Local Inference Rules to Improve Black-box Deobfuscation"
topic: "Reverse engineering, Deobfuscation, Artificial intelligence, Program synthesis"
pdf: "/assets/publications/papers/2025-ccs.pdf"
date: 2025-09-08
---

## Context

Obfuscation is a family of methods that take a program and translate it into a new program, harder to analyze and understand. 
While obfuscation can be used to protect intellectual property. It is also used by malware authors to impede detection and 
slow down remediation. 

Hence, multiple automated deobfuscation methods have been proposed to simplify obfuscated software. 
Especially, black-box deobfuscation relies on program synthesis to recover a simple version of a code.
It first sample the obfuscated code snippet, generating inputs randomly and monitoring its outputs. 
Then, with its input-output observations, black-box deobfuscation synthesizes a code that mimics the obfuscated code but which is simpler to understand.
As black-box deobfuscation relies only on input-output observations, it is totaly immune to standard deobfuscation.
Still, it is limited by the synthesis capabilities of current program synthesizers. 

<br>
<center>
<img src="/assets/img/black-box-deobf.png" alt="drawing" width="40%" title="Black-box deobfuscation process"/>
</center>

**Goal:** This paper aims to improve program synthesis capabilities to better handle deobfuscation tasks.
To to so, we propose *Search Modulo Inference Rules (SMIR)* that combines search and deduction the recover usually hard to synthesize expressions.

## Motivation

Consider the following code snippet, extracted from [Snapchat](https://hot3eed.github.io/snap_part1_obfuscations.html). As you may have guessed, this expression is obfuscated and so is very hard to understand. 
Unfortunately, the state-of-the-art of black-box deobfuscation cannot recover it, even in 1h.

```assembly
add     x0,sp,#0x1b8            ;struct timeval *tval
mov     x1,#0x0                 ;struct timezonze *tzone
adrp    x8,0x109499000
ldr     x8,[x8, #0x1d0]
blr     x8                      ;gettimeofday(tval,tzone)
ldr     x8,[sp, #0x1b8]         ;tval->tv_sec
mov     x9,#0x3e8
mul     x8,x8,x9
ldrsw   x9,[sp, #0x1c0]         ;tval->tv_usec
lsr     x9,x9,#0x3
mov     x10,#0xf7cf
movk    x10,#0xe353, LSL #16
movk    x10,#0x9ba5, LSL #32
movk    x10,#0x20c4, LSL #48
umulh   x9,x9,x10
mov     x10,#0xe6b3
movk    x10,#0x7dba, LSL #16
movk    x10,#0xecfa, LSL #32
movk    x10,#0xd0e1, LSL #48
add     x9,x10,x9, LSR #0x4
orr     x11,x9,x8
lsl     x11,x11,#0x1
eor     x8,x9,x8
sub     x8,x11,x8
eor     x9,x8,x10
mov     x10,#0xe6b3
movk    x10,#0x7dba, LSL #16
movk    x10,#0xecfa, LSL #32
movk    x10,#0x50e1, LSL #48
bic     x8,x10,x8
sub     x8,x9,x8, LSL #0x1
```

Indeed, the underlying expression computes `TVSEC = TVSEC * 1000`. This expression is hard to synthesize because 
current synthesizers cannot generate efficiently the `1000` constant value. 
They must re-create it using the `1` term, generating huge expressions like `(1 + 1) << 1 + ... + 1`.

**SMIR comes to the rescue:** Over this example, XSmir (our extension of [Xyntia](https://github.com/binsec/xyntia) with SMIR) can recover the expression in 2ms. Such good results 
generalizes to new other class of hard-to-sythesize expressions as shown in the following table:

<br>
<center>
<img src="/assets/img/smir-ccs25.png" alt="drawing" width="50%"/>
</center>


## Major Contributions

In summary, this paper makes the following major contributions:
* We propose *Search Modulo Inference Rules (Smir)*, a
new program synthesis scheme that extends search with inference 
rules to generate usually hard-to-synthesize expressions.
Inference rules serve two roles: they directly extend some candidate 
solutions into definitive solutions if possible, otherwise
they direct the search toward such candidate solutions;

* We implement Smir on top of the [Xyntia](https://github.com/binsec/xyntia) deobfuscator 
with the inference rules to handle usual operations over bitvectors (offsets,
masks, affine and polynomial expressions, etc.). 
It yields XSmir, the first search-based synthesizer handling these cases
efficiently;

* We evaluate XSmir against the state-of-the-art black-box deobfuscators 
Syntia and Xyntia, as well as the program synthesizers CVC4/5 and DryadSynth,
showing that it outperforms them on both real obfuscated and
non-obfuscated code. 
We also compared XSmir against
the state-of-the-art white-box deobfuscators ProMBA and
Gamba on MBA expressions. Results show that XSmir
always successfully simplifies more expressions than them.

## Further information

* Read the [paper](/assets/publications/papers/2025-ccs.pdf)
* To appear at the [ACM Conference on Computer and Communications Security (CCS) 2025](https://www.sigsac.org/ccs/CCS2025/) 
* Try [Xyntia](https://github.com/binsec/xyntia) and get our [artifacts](https://zenodo.org/records/17036259)
