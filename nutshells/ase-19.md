---
layout: post
title:  "ASE '19: extended abstract"
date:   2019-12-20
categories: conference paper abstract
paper-title: "Get rid of inline assembly through verification-oriented lifting"
topic: "Automatic analysis of C & inline assembly;
        Precise decompilation of inline assembly"
pdf: https://arxiv.org/pdf/1903.06407.pdf
redirect_from: /conference/paper/abstract/2019/12/20/ase.html
---


## Motivation

Have you ever seen this kind of code?

```c
# 76 "ffmpeg/libavcodec/x86/mathops.h"
static inline int mid_pred(int a, int b, int c)
{
    int i=b;
    __asm__ (
        "cmp    %2, %1 \n\t"
        "cmovg  %1, %0 \n\t"
        "cmovg  %2, %1 \n\t"
        "cmp    %3, %1 \n\t"
        "cmovl  %3, %1 \n\t"
        "cmp    %1, %0 \n\t"
        "cmovg  %1, %0 \n\t"
        :"+&r"(i), "+&r"(a)
        :"r"(b), "r"(c)
    );
    return i;
}
```

The fragment above is C code using the **GNU inline assembly** extension, which
allows to embed assembly instructions into your C programs.  A 2018
[survey](http://ssw.jku.at/General/Staff/ManuelRigger/VEE18.pdf) reported that
more than **1/4** of trending **GitHub** projects actually contain it. We
actually did find it in more than **10%** of package sources written in C/C++ of
the **Debian** distribution. This is usually used either to implement more
efficient routines or access low-level hardware instructions.

State-of-the-art C program analyzers do not behave very well in presence of such code.

For instance, the [**KLEE**](http://klee.github.io/) symbolic execution engine
simply stops when it hits an assembly chunk. [**Frama-C**](http://frama-c.com/)
plugins, on the other hand, will handle it, severely over-approximating its
behavior in the process.  This leads to either incomplete or imprecise results
-- when they are not simply wrong.

That is why we proposed **TInA** *-- for Taming Inline Assembly --* an automated,
generic, verification-friendly and trustworthy lifting technique such that we can reuse, as is, our favorite C analyzers.

## Contributions

In summary, this paper makes the following contributions:

- A new principled method lifting inline assembly to high-level C amenable to
  further formal analysis;
- The  automated  validation  of  said  method  to  make  the lifter
  trustworthy;
- Thorough  experiments of  a  prototype  implementation  on  real-world
  examples.

Thus, we have shown TInA to be widely applicable and trustworthy: in less than
**1/2 hour**, it was able to lift **2k** assembly chunks coming from a typical Debian distribution (including [**FFMPEG**](https://www.ffmpeg.org/),
[**ALSA**](https://alsa-project.org/wiki/Main_Page),
[**GMP**](https://gmplib.org/), ...)
and validate **all** of the lifted code.
Moreover, these results stand whatever the compiler
was (**GCC** or **Clang**) while the technique also handles different
instruction sets (**x86** or **ARM**).
Last but not least, experiments have shown the produced C code actually enhances
for automatic analyses (for both [**KLEE**](http://klee.github.io/) and
[**Frama-C**](http://frama-c.com/)).

## Further information

- See [the **paper**](https://arxiv.org/pdf/1903.06407.pdf)
- Presented
@ [the 34th IEEE/ACM International Conference on **Automated Software Engineering**](https://2019.ase-conferences.org/)
