---
layout: post
title:  "ICSE'21: research paper"
categories: new publication
paper-title: "Interface Compliance of Inline Assembly: Automatically Check, Patch and Refine"
topic: "low-level programming, inline assembly, compilation issues, program analysis"
pdf: ""
date: 1970-01-01
---

## Motivation

“GCC-style inline assembly is notoriously hard to write correctly”
[[D54891](https://reviews.llvm.org/D54891)].

Indeed, messing around with assembly code is quite easy.
But inlining the assembly straight into the *C* code is even more hazardous.

Let see how through a small example.
The following macro is defined in
[**libtomcrypt**](https://github.com/libtom/libtomcrypt)
to store a 32 bits value in a big endian cipher (reverse the byte order).

```c
#define STORE32H(x, y)           \
asm __volatile__ (               \
   "bswapl %0     \n\t"          \
   "movl   %0,(%1)\n\t"          \
   "bswapl %0     \n\t"          \
      ::"r"(x), "r"(y));
```

The inline assembly chunk is split in two parts:
- assembly instructions written directly in *C* formatted strings;
- an interface making the link between assembly and *C* operands.

The tokens ```%0``` and ```%1``` will be replaced (*à la* ```printf```)
by the registers (```"r"``` constraints) chosen for the value ```x``` and
the pointer ```y``` by the compiler.

Let's try to call this macro within a *C* program:

```c
#include <stdio.h>
int main ()
{
  char y[4] = "fail";
  STORE32H(0x646f6e65 /* "done" */, y);
  printf("%c%c%c%c\n", y[0], y[1], y[2], y[3]);
  return 0;
}
```

The main function simply tries to write the ASCII value of the **done** string
to overwrite the value **fail** stored in ```y``` and then print the value of ```y```.

Using **gcc**, it works well:
```bash
$ gcc -m32 main.c
$ ./a.out

> done
```

Wait! Let's try to increase the optimisation level:
```bash
$ gcc -m32 -O1 main.c
$ ./a.out

> fail
```

We are experiencing what we call a *latent* bug caused by a
**compliance issue**.
It's all about the compiler missing
a critical information about the chunk behavior: the interface never states that
the content pointed by ```y``` is written.

So basically, the compiler propagates the content of
the initial buffer to the ```printf``` call -- actually bypassing our writes.


Inline assembly is a common practice in software engineering, especially
in optimized library (**GMP**, **libtomcrypt**, etc.) or multimedia software
(**ALSA**, **FFMPEG**, etc.). As compiler optimisations are becoming more
complex and more aggressive, every compliance issue, lurking
without causing any visbible issue, is actually an opportunity to trigger a hard to find
bug.

That is why we propose **RUSTInA**, an automatic tool to **check** the
*compliance* and, generally, **fix** the problem by strengthening the interface.
Of course, in order to check this so-called *compliance*, we need to
define it first. Let's have a look at the paper!

## Contributions

In summary, this paper makes the following contributions:
- A novel semantic and comprehensive formalization of the problem of
interface compliance;
- A new method to automatically verify the compliance of an inline assembly
chunk and to generate a corrective patch for the majority of compliance
issues;
- Thorough experiments of a prototype implementation on a large set
of *x86* real-world example.


Thus, we have found that **half** of the **2k** assembly chunks coming from
a typical Debian distribution are actually *non-compliant*, with more than
**2k** individual issues reported by **RUSTInA**. This work helped us in
identify 6 patterns of **bad coding practices** responsible of more than
**80%** of the issues. Finally, we've started to submit patches to
the project owners and **7** of them have already accepted and applied
the changes.


## Further information

- Preprint is coming *very soon*!
- Artifact is coming *soon*!
- Dont miss the presentation @ [the 43rd **International Conference on Software Engineering**](https://conf.researchr.org/venue/icse-2021/icse-2021-VirutalVenue)