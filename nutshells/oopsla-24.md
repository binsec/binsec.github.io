---
layout: post
parent: Publications
title: "OOPSLA'24"
categories: new publication
paper-title: "A Dependent Nominal Physical Type System for Static Analysis of Memory in Low Level Code"
topic: "Static Analysis; Abstract interpretation; Memory Safety"
pdf: "/assets/publications/papers/2024-oopsla.pdf"
date: 2024-09-21
nav_order: 2024-09-21
---

## Context

When you program in C, it is easy to make programming mistakes that
directly compromise spatial memory safety, such as null pointer
dereferences, type confusion, or buffer overflows. These direct memory
corruption errors are a subclass of undefined behaviors in C, i.e.,
programming mistakes that you should not make, such as division by
zero, signed integer overflow, or breaking the C strict aliasing
rules. The compiler assumes that you do not make these mistakes, which
are not checked, meaning that if you do make them, your program
may crash, corrupt memory, or exhibit other unpredictable
behaviors. Moreover, these mistakes can be exploited by an attacker to
take control of your program,
and this represents both the most common and the most severe kind of
security vulnerabilities.


Thus, it is important to ensure that your program is free from
these undefined behaviors. Providing tools that do this practically
is one of the main purposes behind the development of
Codex, a sound static analyzer based on abstract
interpretation. The paper focuses on particular method that can
ensure spatial memory safety of C or binary programs almost
automatically, requiring only a small amount of type annotations.


## Example

This example comes from our
[tutorial](/docs/tutorial_oopsla2024.pdf), and extracted from an OS
code that we have analyzed.

Suppose that we are given a function in a library described using the following header file:

```c
// Linked list of messages, each containing a fixed-length buffer
struct message {
  struct message *next;
  char *buffer;
};

// Wrapper around the linked list, specifies the length of all buffers
struct message_box {
  int length;
  struct message *first;
};

void zeros_buffer(struct message_box* box);
```

An example of a memory layout that would fit this description is the following one:

<img src="/assets/publications/pictures/2024-oopsla-struct_layout.png"
style="width:700px; display:block; margin-left:auto; margin-right:auto">

In the image, we assumed that `message` is a singly-linked list, and that the `char *` pointer points to a single char.

Now, we want to verify that the implementation of `zeros_buffer`, a function that sets all the buffers in the `message_box` to zero, is memory-safe.

```c
void zeros_buffer(struct message_box *box) {

    struct message * first = box->first;
    struct message * current = first;

    int length = box->length;

    do {
        for (int i = 0; i < length; i++) {
            current->buffer[i] = 0 ;
        }
        current = current->next;
    } while(current != first) ;
}
```

Note that this function is memory-safe only if the `box` parameter follows some invariants, in particular:
1. Each `message` points to a `buffer` whose size corresponds to `box->length`.
2. The list of `message`s is circular (the code never tests the `next` field to see if it can be a null pointer)


So, if we try to analyze this code as is, Codex will correctly report that the code is not memory safe. Indeed, a main feature of Codex is that the analysis is sound: if there is a spatial memory safety issue, it should report it.

Luckily, it is easy to express the required invariants in our type system. It suffices to copy the header file, and edit it as follows:

```c
struct message(len) {
   struct message(len)+ next;
   char[len]+ buffer;
};

∃ mlen:integer with self > 0.
struct message_box {
  (integer with self = mlen) length;
  struct message(mlen)+ first;
};

void zeros_buffer(struct message_box+ box);
```

Here, the `+` for the pointer types (instead of the `*`) indicates
that the corresponding pointer is never null. The `struct message`
type now receives a `len` parameter corresponding to the length of the
`buffer` that it contains. Finally, the `struct message_box` type is
modified to include an invariant between the `length` field and the
`len` parameter of the message (i.e., the length of the buffers).

A possible layout for this description is the following one:

<img src="/assets/publications/pictures/2024-oopsla-parametrized_layout.png"
style="width:700px; display:block; margin-left:auto; margin-right:auto">

Now, this updated header file is not for the C compiler, but is used
by our Codex tool, that can now verify that `zeros_buffer` is
memory-safe (as it does not report any alarm) automatically. Note that
this proof relies on the hypothesis that the `box` argument of
`zeros_buffer` correspond to the memory layout described by the types.
This assumption is checked in any analyzed function that would call `zeros_buffer`.
Thus, if you verify all the functions in a program, we prove it memory-safe.

[//]: # {: .note }
[//]: # While codex **ensures spatial memory safety** (no invalid pointer read/write),
[//]: # it does **not ensure termination**.
[//]: # Even with our given types, the `zeros_buffer` function may loop infinitely.
[//]: # Indeed, we cannot express the invariant stating the list is circular. It is sort
[//]: # of implied by the constraints that the `next` pointer is never null, since memory
[//]: # is finite, the list will eventually reach a loop. However, we may have a lasso-shape,
[//]: # where the first few `message`s are not part of that loop.

Finally, this verification of `zeros_buffer` can be made not only on
the C source code, but also on the compiled machine code, i.e. Codex
can perform type-checking of both C and machine code automatically!

## Key contributions and take-away

One of the main challenges when analyzing C programs is the
representation of the memory. The paper proposes a **type system**,
inspired by that of C, as the basis for this abstraction. While
initial versions of this type system have been proposed in
[VMCAI'22](/papers/2022-vcmai-lightweight-shape-analysis.html) and
used in [RTAS'21](/papers/2021-rtas-no-crash-no-exploit.html), this
paper extends it significantly with new features like support for
union, parameterized, and existential types. The paper shows how to
combine all these features to encode many complex low-level idioms,
such as flexible array members or discriminated unions using a memory
tag or bit-stealing. This makes it possible to apply Codex to
challenging case studies, such as the unmodified Olden benchmark, or
parts of OS kernels or the Emacs Lisp runtime.

The reasons why a C or machine code program is memory-safe generally
depend on invariants on the values, such as "this pointer points to a
memory area whose length corresponds to the contents of this
integer". Thus, a type system that can be used to guarantee memory
safety must use dependent types. This makes type checking particularly
complex, which is why we use abstract interpretation to type-check the
program. Abstract interpretation also allows automatic inference
of other kinds of program invariants (beyond those expressed by the
type system), that helps the overall analysis to type-check the
program and verify its spatial memory safety.

Often, static analyzers are
[whole-program](/docs/concepts/whole_program_analysis.html), meaning
that they need to know the entire program to perform the
analysis. This is useful when using the analyzer for validation, but
prevents the use of the analyzer while developing. One of the nice
benefits of the type system is that it allows [modular
analysis](/docs/concepts/modular_analysis.html), using the types of
the function prototype rather than using the code in the function
definition.

Finally, we investigate properties of our type system. Contrary to
type systems like Rust's, our type system is structural: we don't
track all the references to an object, which eases analysis
automation. Such a type system is particularly convenient when there
are complicated sharing patterns and no issues of temporal memory
safety, as found in statically-allocated OS kernels or
garbage-collected programs. We investigate the properties of our type
system and have discovered interesting features like the *mild
update*, allowing switching between the cases in a union provided
that some monotonicity conditions are met.

## Further information

- If you want to know more, the [**slides**](/assets/publications/slides/2024-oopsla.pdf) should be quite accessible.

- Interested in using it? Be an early adopter, check the
[**tutorial**](/assets/docs/tutorial_oopsla2024.pdf) and try proving that your
code is (spatially) memory-safe now!

- Interested in the theory? Read the
[**paper**](/assets/publications/papers/2024-oopsla.pdf).

- You can check the [artifact](https://zenodo.org/records/13383433),
which contains the version of Codex used by the artifact reviewers, as
well as the benchmarks used.

- Appeared at
  [OOPSLA'2024](https://2024.splashcon.org/track/splash-2024-oopsla).
