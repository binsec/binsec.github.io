---
layout: post
title:  "DIMVA'25: research paper"
categories: new publication
paper-title: "Exploring the Potential of LLMs for Code Deobfuscation"
topic: "Large language models, deobfuscation, program analysis, machine learning"
pdf: "/assets/publications/papers/2025-dimva.pdf"
date: 2025-05-06
---

## Motivation

**Obfuscation and deobfuscation.** Software can include many secret in their code, that could be retrieved by a reverse engineer. 
Hence, to protect against it, software editors can use obfuscation to make the code very hard to understand and analyze. 
Unfortunatelly, these obfuscation transformations are used by malware authors to hide their malicious behaviors.
Hence, different deobfuscation methods aim to simplify obfuscated code. 

**Large Language Models (LLM).** Recently LLMs have drawn the attention from the research community. Interrestingly, it was shown
efficient beyond the usual Natural Language Processing (NLP) case. LLMs can notably generate code or infer function names. 

In this work, we study to which extent the good results of LLMs on code generation can transfer to deobfuscation. 
This is an interresting application scenario to assess LLMs code comprehension. Indeed, it requires manipulating the
complex constructs included by obfuscation and simplifying them while conserving code semantics. 

## Can LLMs be used for deobfuscation ?

To assess the capacity of LLMs to simplify obfuscated code, we obfuscated a dataset of C functions with multiple transformation from the Tigress obfuscator.
We fine-tuned different models to generate a simplified code from the obfuscated one. 
Experiments then tackled the three following research questions:
* Can LLMs deobfuscate state-of-the-art code obfuscation transformations?
* Can LLMs deobfuscate code in a real-world scenario where multiple transformations are chained?
* How much is memorization affecting the performance?

**Takeaways.**
* Fine tuning small coding models for deobfuscation show promising results (even when obfuscations are chained);
* Code correctness is the main challenge faced by LLMs for deobfuscation. Still, using bigger models would likely reduce this problem;
* Experiments show that memorization was not significant, indicating a genuine code understanding capabilities of LLMs.

## Major Contributions

In summary, this paper makes the following major contributions:
* We systematically analyze the potential of LLMs for code deobfuscation, comparing general-purpose models, specialized code
models, and instruction-tuned models; 
* We highlight the strengths of LLMs in tackling code obfuscation, but it also shows their current limitations, such as generating semantically incorrect code; 
* We build the first scalable dataset for training and evaluating the performance of LLMs for the deobfuscation task that can be used with arbitrary C programs.

## Further information

* Read the [paper](/assets/publications/papers/2025-dimva.pdf)
* To appear at the [22nd Conference on Detection of Intrusions and Malware & Vulnerability Assessment (DIMVA '25)](https://www.dimva.org/dimva2025/) 
* Artifacts and datasets are available [here](https://github.com/DavidBeste/llm-code-deobfuscation)

