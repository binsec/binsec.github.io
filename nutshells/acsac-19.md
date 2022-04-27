---
 layout: post
 title:  "ACSAC'19, research paper"
 categories: new publication
 paper-title: "How to Kill Symbolic Deobfuscation for Free (or Unleashing the Potential of Path-Oriented Protections)"
 topic: "Code Obfuscation; Protection against Symbolic Deobfuscators and Other Program Analysis-Based Attackers"
 pdf: "/assets/publications/papers/2019-acsac.pdf" 
 date: 2019-12-13
 redirect_from: /new/publication/2019/12/13/acsac19.html
---
 
 
## Motivation 
 
Code obfuscation is a major tool for protecting software intellectual property from attacks such as reverse engineering or code tampering. Yet, recently proposed (automated) attacks based on Dynamic Symbolic Execution (DSE) shows very promising results, hence threatening software integrity. Current defenses are not fully satisfactory, being either not efficient against symbolic reasoning, or affecting runtime performance too much, or being too easy to spot.
 
 
 
 
## Contributions
 
 
This paper presents and study a new class of anti-DSE protections coined as path-oriented protections targeting the weakest spot of DSE, namely path exploration. 

* We propose a predictive framework allowing to understand this class of protections (especially we identify the key property of "single-value path" protection); 

* We design two novel path-based protections achieving the single-value path criterion; 

* Extensive evaluation demonstrates that these approaches critically counter symbolic deobfuscation, impacting DSE more than three levels of virtualization at essentially no cost. 


From a methodological point of view, this work extends recent attempts at rigorous evaluation of obfuscation methods, with both an analytical evaluation of the protection and a refinement of prior  experimental setups. 
 
 
## Further information
 
 - Read the [paper][paper].
 - Presented at [The 35th Annual Computer Security Applications Conference][acsac19].
 
 
 [acsac19]: https://www.acsac.org/2019/
 [paper]: /assets/publications/papers/2019-acsac.pdf
 
