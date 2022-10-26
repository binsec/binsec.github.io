---
layout: post
title:  "Postdocs, PhDs and research internships in software security and program analysis"
categories: jobs open
date: 2022-10-25
---
We have several open positions to **hire postdocs, PhD students and research interns** in software security and program analysis. Successful candidates will work in our BINSEC team, on topics like vulnerablity detection and analysis, software reverse engineering and deobfuscation, binary-level formal verification and code protection. 
**All positions are fully-funded**.



## CONTENT 
* [About us](#about-us)
* [Topics](#topics)
* [Practical details about the hiring procedure and the positions](#practical-details-about-the-hiring-procedure-and-the-positions)
* [Working and living in Paris](#working-and-living-in-paris)


## ABOUT US

**OUR TEAM** - The BINary-level SECurity research group (BINSEC) is a dynamic team of [4 senior and 9 junior researchers][team], which offers a **stimulating and open-minded scientific environment in English** to all its members. The group has [frequent publications][publications] in top-tier security, formal methods and software engineering conferences. We work in close collaboration with other French and international research teams, industrial partners and national agencies. The team is part of Université Paris-Saclay, the world’s 16th and European Union’s 1st university, according to the Shanghai ARWU Ranking in 2022.  

**OUR WORK** - The team has high-level expertise in several code analysis approaches, namely symbolic execution, abstract interpretation and fuzzing. We apply these techniques to [improve software security][walloffame], covering notably vulnerability detection and analysis, code (de)obfuscation and formal verification. See [our website][website] for additional information. 

## TOPICS

Available topics notably **involve but are not strictly limited** to the following (in alphabetical order).
**Supervision** will be provided by one permanent researcher of our team (*main supervisor*), typically **in collaboration with** other researchers from the team and outside of it, to provide an optimal combination of expertise, availability and seniority.

**ARTFUL VULNERABILITY DETECTION WITH FUZZING** (*main advisor*: [Michaël Marcozzi][marcozzi]) - [Fuzzing][fuzzing] refers to a process of repeatedly running a program with automatically generated inputs to discover bugs, which can then be fixed. A major challenge in the field is moving from indistinct program exploration towards artful triggering of dangerous vulnerabilities. Taking advantage of our team’s expertise and previous works, the selected candidate will propose, implement and evaluate ways to overcome this challenge, such as [finer-grained guidance mechanisms][ndssfuzz] or [combination with symbolic execution][fps]. 

**BINARY-LEVEL PROGRAM ANALYSIS AT SCALE** (*main advisor*: [Sébastien Bardin][bardin]) - Program analysis is considered to be harder to conduct at the binary level than at the source level, due to a loss of abstraction and an increase in size. The goal here will be to challenge this folklore knowledge, and to design more efficient binary-level program analysis techniques, especially symbolic execution. The selected candidate will explore different strategies, such as combining different methods together (e.g., static and symbolic analysis), or revisiting source-level optimizations for the binary-level case. The candidate will benefit from the team's expertise on symbolic execution.   

**COMBINATION OF SEMANTIC AND SYMBOLIC ANALYSIS USING SSA-BASED ABSTRACT INTERPRETATION** (*main advisor*: [Matthieu Lemerre][lemerre]) - SSA is a popular intermediate representation (underpinning for instance LLVM) that simplifies and makes more efficient some program analyses. Until recently, SSA translation was performed using ad-hoc algorithms. In a forthcoming POPL paper, we recently discovered that SSA translation could be viewed as a standard dataflow analysis by abstract interpretation, enabling the translation to SSA and the usage of SSA to be done simultaneously, rather than in sequence. We are looking for candidates interested in investigating the consequences of this discovery.

**DOMESTICATING COMPLEX SOFTWARE VULNERABILITIES** (*main advisor*: [Michaël Marcozzi][marcozzi]) - Mitigation and exploitation of software vulnerabilities is a game of cat and mouse. The resulting arms race can lead attackers to rely on more and more complex vulnerabilities, like backdoors, exploit chains, microarchitectural exploits or side-channel attacks. Such complex vulnerabilities are still barely understood by cybersecurity researchers. Following previous scientific expeditions in these wild lands (see e.g. [Thomas et al.][backdoors], [Daniel et al.][SP2020] or [Daniel et al.][NDSS2021]), the selected candidate will be responsible for exploring and documenting (parts of) the landscape of complex security vulnerabilities. The main goal will be to propose a highly-needed taxonomy of these threats, leading to systematic ways to mitigate them.  

**OS VERIFICATION AND MEMORY ANALYSIS USING  ABSTRACT INTERPRETATION AND DEPENDENT TYPES** (*main advisor*: [Matthieu Lemerre][lemerre]) - Memory corruption is the most common and severe type of software vulnerability. In general, reasoning about memory is essential, especially in systems software like [OS kernels][RTAS2021] or critical libraries, that perform complex low-level memory manipulation. We want to address this problem in an original way using [abstract interpretation to verify memory safety of a dependent type system][VMCAI2022], trying to attain a sweet spot between the analysis precision and efficiency, and the need to supply user annotations, and we welcome candidates interested in any combination  of these topics.

**REVERSE ENGINEERING AND DEOBFUSCATION** (*main advisor*: [Sébastien Bardin][bardin]) - Program analysis methods have been used for some years now in code understanding and code deobfuscation ("symbolic deobfuscation"), yet protections emerge against such techniques. More recently, black-box methods based on AI techniques have been shown effective against obfuscated code, being able in some cases to recover the initial semantic of protected code fragments. Still, again, dedicated protections emerge. The selected candidate will explore how these different methods can be combined in order to design efficient deobfuscation methods. 

**SECURITY-ORIENTED PROGRAM ANALYSIS** (*main advisor*: [Sébastien Bardin][bardin]) - Standard program analysis techniques have been primarily designed for safety concerns rather than for security issues. While effective at finding bugs, they have more difficulties at distinguishing true vulnerabilities from mere benign bugs. The selected candidate will explore how program analysis could be enhanced toward searching for bugs "that matter". The notion of "robust reachability", [recently proposed by the team][CAV2021], is a natural starting point. Extensions can include both alternative definitions or better algorithmic techniques. 

**SYMBOLIC REASONING FOR MICRO-ARCHITECTURAL ATTACKS** (*main advisor*: [Sébastien Bardin][bardin]) - Micro-architectural attacks such as Spectre and Meltdown bring new threats to highly-sensitive programs such as cryptographic libraries, kernels or enclave. The selected candidate  will explore how program analysis techniques such as symbolic execution can be efficiently lifted to address these vulnerabilities at scale. The challenge will be to model these attacks in a way amenable to efficient automated analysis. This work will naturally build upon our previous proposal for efficient [relational symbolic execution][SP2020] and [specualtive symbolic execution][NDSS2021].  

## PRACTICAL DETAILS ABOUT THE HIRING PROCEDURE AND THE POSITIONS

Candidates should **send a CV** to <binsec-jobs@saxifrage.saclay.cea.fr> **as soon as possible**. Applications will be reviewed as they arrive (first come, first served), depending on our availability, and additional information may be requested from you.

Interesting candidates will be contacted for an **online interview**, to evaluate notably their *academic and programming skills*, together with their *English proficiency*. Each position is expected to start as soon as possible (upon completion of all administrative requirements). 

Supervised by our senior researchers, the **recruited candidates will be involved in the scientific activities of the team**. These activities mainly include solving research problems, formalising research ideas, implementing and evaluating software prototypes, publishing and presenting at top conferences, as well as participating to the scientific life of the team (paper reading groups, hackatons, etc). Postdocs and PhD students will be able to dedicate a *small* fraction of their time to **teaching**, if they want so. Our former team members have been able to secure stimulating positions in academia or industry and we will support you in advancing your career.

**SPECIFIC INFO FOR POSTDOC CANDIDATES** - Postdoc candidates should have (or be close to have) a **Ph.D. in Computer Science**. We are primarily looking for candidates with a good research experience in one or more of the following domains: software security, formal methods, software engineering, programming languages, architecture or systems. Postdocs will be expected to tackle more challenging problems with less supervision than PhD students. They may be involved in supervising interns. Each postdoc position will have a **duration of 2 years**.

**SPECIFIC INFO FOR PHD CANDIDATES** - PhD candidates should have (or be close to have) a **Master (or equivalent) in Computer Science**, with good results in topics related to the doctoral work that they would conduct in our team. They should be persistent, rigorous, motivated and curious individuals. Authorship of a scientific publication or real-world coding experience are a plus. Each PhD position will have a **duration of 3 years**.

**SPECIFIC INFO FOR RESEARCH INTERNSHIP CANDIDATES** - Research internship candidates should be **preparing a Master (or equivalent) in Computer Science**, with good results in topics related to the work that they would conduct in our team. Under the close supervision of our researchers, interns will be expected to support the activties of the team, either by reviewing scientific litterature, or by testing and formalising research ideas, or by coding research prototypes, or by conducting research experiments. Each internship will have a **duration of 2.5 to 6 months**. Successful internships may open the door to doing a PhD in the team.

## WORKING AND LIVING IN PARIS

Our offices are located in [Nano-Innov][nano], at the heart of Plateau de Saclay, south of Paris, Europe’s biggest research and industry cluster. Agencies like [Science Accueil][scienceaccueil] or [Cité Internationale Universitaire de Paris][ciup] are available to help foreign candidates find their home and settle here. Most of us live either in the wooden and quiet southern suburbs of Paris or closer to the bustling historical center of the city. Paris is the capital of France, a metropolis of 12.5 million people and one of the most visited travel destinations in the world, in the heart of western Europe.


[backdoors]: https://dx.doi.org/10.1007/978-3-030-00470-5_5
[NDSS2021]: https://binsec.github.io/nutshells/ndss-21.html
[SP2020]: https://binsec.github.io/nutshells/sp-20.html
[CAV2021]: https://binsec.github.io/nutshells/cav-21.html
[VMCAI2022]: https://binsec.github.io/nutshells/vmcai-22.html
[RTAS2021]: https://binsec.github.io/nutshells/rtas-21.html
[fps]: https://binsec.github.io/nutshells/fps-21.html
[ndssfuzz]: https://binsec.github.io/nutshells/fuzzing-22.html
[fuzzing]: https://www.fuzzingbook.org/
[bardin]: http://sebastien.bardin.free.fr/
[lemerre]: https://binsec.github.io/people/lemerre.html
[marcozzi]: http://www.marcozzi.net
[team]: https://binsec.github.io/#people
[nano]: https://goo.gl/maps/Swn77dLqrKQki7zt9
[publications]: https://binsec.github.io/publications
[walloffame]: https://binsec.github.io/achievements
[website]: https://binsec.github.io
[scienceaccueil]: https://www.science-accueil.org/en/
[ciup]: https://www.ciup.fr/en/
