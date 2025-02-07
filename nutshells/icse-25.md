---
layout: post
title:  "ICSE'25: research paper"
categories: new publication
paper-title: "ROSA: Finding Backdoors with Fuzzing"
topic: "fuzzing, dynamic analysis, metamorphic testing, backdoors, vulnerability detection"
# TODO[Dimitri]: add preprint (`pdf`) here.
date: 2025-01-29
---

## Motivation
A **code-level backdoor** is a hidden access, concealed within the code of a program. It
allows whoever is aware of which program inputs activate it to either _trigger a privilege
escalation within the program_ or to _access underlying resources_ that should otherwise not be
available.

An **illustrative example** can be given considering the well-known [Sudo](https://www.sudo.ws/)
Unix utility. Sudo's authentication-handling code looks something like this, in the case where
authentication is performed via a password:
```c
int verify_user(const struct sudoers_context* ctx, const char* password) {
    int ret = AUTH_FAILURE;
    // ...
    ret = ctx->verify(password);
    // ...
    return ret;
}
```

We assume that `ctx->verify(password)` performs a cryptographically secure verification of the
password against a stored password hash.

Now, imagine that an attacker has managed to poison the upstream version of Sudo (used by
essentially every Linux distribution), injecting the following code:
```c
int verify_user(const struct sudoers_context* ctx, const char* password) {
    int ret = AUTH_FAILURE;
    // ...
    ret = ctx->verify(password);
    // ---  Attacker-injected code begin ---
    if (strcmp(password, "let_me_in") == 0) {
        ret = AUTH_SUCCESS; 
    }
    // ---  Attacker-injected code end   ---
    // ...
    return ret;
}
```

With this version of Sudo, anyone aware of the special `"let_me_in"` password can bypass
authentication and have full root access to all systems where it has been installed!

In practice, **confirmed software supply-chain attacks** have led to the injection of such
backdoors into popular open-source projects, like PHP, ProFTPD, vsFTPd and xz.
**Backdoors have also been discovered in the binary firmware of popular network routers**. This
last type of attack is even more difficult to protect against, since often the end user will only
see the _binary_ version of the program, as the infected firmware is often closed-source.

Historically, the only way to vet for such backdoors was to **manually reverse-engineer the
(binary) program** and look through the code. This task requires human experts with
years of expertise working for a long time, and even domain-specific knowledge depending on the
program. When we consider vetting scenarios such as for large-scale deployment of new routers in a
company's infrastructure, it is difficult to scale this manual and costly process to hundreds of
binary programs in the firmware. And this comes with cascading, multi-user impact, should this
manual vetting fail to detect a real threat. **A few academic approaches have tried to partly
automate this process**, but they cover a limited scope of backdoors and programs, and a
significant amount of manual reverse-engineering remains necessary. 

**Fuzzing** is a form of automated program testing. It generates test inputs automatically, runs
the program with them and detects program misbehaviors (like crashes) automatically at runtime.
Among advanced capabilities, modern fuzzers, like the community-maintained AFL++, are equipped for
testing binary-only programs and for efficiently exploring complex branching conditions in the
tested code. These tools are currently attracting a lot of popularity because of their reported
ability to discover misbehaviors caused by software vulnerabilities, such as a crash caused by a
buffer overflow. In this work, we aim at taking advantage of fuzzers to detect misbehaviors
corresponding to the triggering of a backdoor. Yet, **current knowledge does not offer any means to
automatically characterize the triggering of a backdoor at runtime**. In addition, **benchmarking
backdoor detection capabilities over multiple programs and backdoors is difficult**, as backdoor
reports are scarce and often point to lost samples or undocumented binary firmware, running on
obsolete and difficult-to-obtain appliances.

## Proposal
In this work, we propose **a new automatic approach for the detection of backdoors, called ROSA**.
It couples a state-of-the-art fuzzer (AFL++) with a novel mechanism (based on the principle of
metamorphic test oracles) capable of detecting backdoor triggers at runtime. The key intuition
behind this mechanism is that, for example, fuzzing the aforementioned backdoored version of Sudo
with incorrect credentials should always cause _similar observable reactions_; however, among the
generated wrong credentials, the ones that trigger the backdoor will cause a _different reaction_,
enabling ROSA to detect them.

Let us illustrate the different steps through which ROSA can detect the aforementioned backdoor in
Sudo (assuming all other security protocols are adhered to and a strong root password is chosen):
- **Phase 1**: the fuzzer is run for a short period (around one minute) to collect the
  representative inputs of the program. More precisely, it tries various incorrect passwords
  (e.g., `"test"`, `"testtttttttt"`, `"aaaaaaBBBBBBBBBBBB"`, ...), which are stored in a
  _representative inputs database_ if they cover a fresh set of edges in the Control-Flow Graph
  (CFG) of Sudo. The external program behavior triggered by each of these representative inputs is
  also saved, by recording the system calls issued by Sudo when run with the password.
- **Phase 2**: the fuzzer is run for a longer period (a few hours) and discovers many new (still
  incorrect) passwords. For each such password, the representative inputs database is searched for
  an input that covers a similar set of edges in the CFG and triggers the same system calls. If
  no such input is found, then the discovered password is considered as a suspicious outlier and
  reported as a possible backdoor. For example:
  - Password `"testtt\nttttt"`: most similar representative input is `"test"`, and while there is a
    slight difference in CFG edges, there is no difference in emitted system calls, so this input
    is considered **safe**.
  - Password `"let_me_in"`: most similar representative input is `"testtttttttt"`, and there is
    **considerable difference** in emitted system calls, as the new input _succeeds_ in passing the
    authentication function, _forks_ the running program and _executes_ the given command. This
    input is therefore marked as **suspicious**.
- **Post-processing**: a human expert goes through the suspicious inputs and analyzes them under a
  process tracing program (e.g., [strace](https://strace.io/)). The expert quickly realizes that
  the `"let_me_in"` input successfully runs the command passed to Sudo, which is not supposed to
  happen, since the _real_ root password is cryptographically protected and was never given to the
  fuzzer. The expert therefore correctly concludes that **there must be a backdoor in Sudo** which
  allows users to authenticate with the special `"let_me_in"` password.

To facilitate the experimental evaluation of ROSA and its comparison with existing tools, we have
created and made available **the novel ROSARUM backdoor detection benchmark**. ROSARUM includes 17
_authentic_ (found in the wild) and _synthetic_ (injected by us) backdoors of different varieties,
found in different types of programs.

## Experiments and results
We have implemented the ROSA approach into an [open-source tool](https://github.com/binsec/rosa)
and evaluated it over the
[ROSARUM backdoor detection benchmark](https://github.com/binsec/rosarum).

In a set of 10 ROSA runs of 8 hours each _for each program in ROSARUM_, we measure **whether or not
ROSA successfully finds the backdoor**, the **time it took to find the first backdoor-triggering
input** as well as **the number of reported suspicious inputs** to vet semi-automatically. We also
compare against a different state-of-the-art backdoor detection approach, which still fundamentally
relies on reverse-engineering the binary program.

Our measurements show that ROSA **detects all 17 backdoors** in ROSARUM in **1h30** on average,
with an average of **7 suspicious inputs** to vet with our proposed semi-automatic method. Compared
to the (faster) state-of-the-art backdoor detection approach, ROSA **detects all the backdoors**
(as opposed to only 4 out of 17), it produces **44 times fewer suspicious inputs** to vet, and
since it provides the full inputs which trigger the suspicious behavior as well as their suspicious
system calls, there is **no need** for manual reverse-engineering.

## Further information
{% comment %}
    TODO[Dimitri]: link to paper (for the "paper" word in the following sentence).
{% endcomment %}
- Read the paper, download the [artifact](https://zenodo.org/records/14724251), try out [ROSA](
  https://hub.docker.com/r/plumtrie/rosa) and [ROSARUM](https://hub.docker.com/r/plumtrie/rosarum).
{% comment %}
    TODO[Dimitri]: link to paper (for the "Published" word in the following sentence).
{% endcomment %}
- Published in the [47th International Conference on Software
  Engineering](https://conf.researchr.org/home/icse-2025).
