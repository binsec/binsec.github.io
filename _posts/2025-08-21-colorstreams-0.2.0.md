---
layout: post
title:  "Open source release of Colorstreams is out"
categories: releases colorstreams
date: 2025-08-21
---


BINSEC/Colorstreams 0.2 is now released. Check out its features and how you can analyze execution traces on-the-fly through analysis policies.


## What is Colorstreams?

Colorstreams is a dynamic data-flow analysis tool for x86-64 programs, with a focus on precision and flexibility.
It implements various analysis policies performing taint analysis, symbolic execution, as well as other tasks useful for automation.
These analysis policies can be combined in a single run, enhancing their capabilities while new ones can be easily integrated.

## A quick example

The following program contains two similar vulnerabilities with different effects: both are buffer overflow writes, but [Vuln. A] always leads to a crash because only large size is permitted, while [Vuln. B](#vuln-b) allows overwriting the return address.

```c
#include "stdio.h"
#include "stdint.h"
#include "string.h"

#define HEADER_SIZE 40

//checks that the message header is valid
uint64_t check_header(char *input, uint64_t input_size)
{
    //2) header sprayed on the stack
    uint64_t id = *((uint64_t *) input) & 0xfff;
    if(id <= 296)
    {
        uint64_t a = ((uint64_t *) input)[1];
        uint64_t b = ((uint64_t *) input)[2];
        //...
        return 1;
    }
    return 0;
}
    
//copies the message into a buffer
void get_msg(char *buf, uint64_t buf_size, 
        char *input, uint64_t input_size)
{
    //3) not initialized => size = sprayed header
    uint64_t size;
    if(input_size <= buf_size + HEADER_SIZE)
        //4) input_size < 40 => integer underflow
        size = input_size - HEADER_SIZE;
    //5) buffer overflow!!!
    // A. input_size < 40: 2^64 - 40 <= size < 2^64
    // B. input_size > 264: size = header 
    printf("copying %llu bytes into a %llu bytes buffer...\n", size, buf_size);
    memcpy(buf, input + HEADER_SIZE, size);
    printf("done\n");
}

//parses command line inputs
uint64_t get_input(int argc, char **argv, char **msg)
{
    if(argc != 3)
    {
        printf("Usage: target message message_size\n");
        exit(0);
    }
    *msg = argv[1];
    return atoi(argv[2]);
}

int main(int argc, char **argv)
{
    //1) inputs: char *input, uint64_t input_size
    char *input = NULL;
    uint64_t input_size = get_input(argc, argv, &input);
    char buf[256];
    if(check_header(input, input_size))
        get_msg(buf, 256, input, input_size);
    else
        printf("invalid message header\n");
    return 0;
}
```

Using Colorstreams, we can measure and display those differing capabilities by computing the constraints on the write size with symbolic execution. We can then extract the set of feasible values via the *Shrink and Split* strategy (see our [USENIX 2025 paper](https://www.usenix.org/conference/usenixsecurity25/presentation/lacombe)).
With the help of Colorstreams automation utilities, we can also avoid the need for manual instrumentation of the program.
To do so, we can mark the user inputs as sources while function arguments that can lead to out-of-bounds write are marked as sinks.

### Vuln. A

Here is the full command line for [Vuln. A](#vuln-a):
```c
colorstreams -main main -p compose -compose-producers "autosource;oobautosink" -autosource-source-stubs "get_input:bytes_to_buf(ret,ret<gdb<*((char**)@2)>>,ret<strlen<gdb<*((char**)@2)>>>,message,controlled);get_input:ret_reg(controlled)" -oobautosink-data-lim 8 -oobcheck-dont-check "_IO_printf;_IO_puts" -compose-consumers symbolic -symbolic-cw "_IO_printf;_IO_puts" -symbolic-p "S&S" -args "\"(a\" 0" ./target.run
```

Which yields the output:
```c
[COLORSTREAMS] Chosen policies: compose
[Compose <4> → Auto-Source <0> → SOURCE] get_input_ret <0> <REG<rax>[8]>: controlled (auto ret_reg)
[Compose <4> → Auto-Source <0> → SOURCE] message <1> <MEM<0x7ffd870e059e>[2]>: controlled (auto bytes_to_buf)
[Compose <4> → Symbolic Policy <3> → SOURCE] get_input_ret <0> <REG<rax>[8]>: controlled (auto ret_reg)
[Compose <4> → Symbolic Policy <3> → SOURCE] message <1> <MEM<0x7ffd870e059e>[2]>: controlled (auto bytes_to_buf)
[Compose <4> → OOB Auto-Sink <1> → OOB Checker <2> → INFO] stopping on memory violation
[Compose <4> → OOB Auto-Sink <1> → OOB Checker <2> → RESULT]
Detection 0:
├─target: MEM<0x7ffd870e05c6>[4611686018427387903]
├─r/w: read
├─valid: False
├─reason:
│ ├─type: memory mapping violation
│ └─description: partially unmapped location
└─location:
  ├─??? : memcpy@plt : ???
  ├─/home/guilhem/dev/colorstreams/doc/tutorial/target2.c : get_msg : 37
  └─/home/guilhem/dev/colorstreams/doc/tutorial/target2.c : main : 60
[Compose <4> → OOB Auto-Sink <1> → OOB Checker <2> → INFO] stopping on memory violation
[Compose <4> → OOB Auto-Sink <1> → OOB Checker <2> → RESULT]
Detection 1:
├─target: MEM<0x7ffd870df450>[4611686018427387903]
├─r/w: write
├─valid: False
├─reason:
│ ├─type: out-of-bounds
│ └─object:
│   ├─kind: stack variable <buf>
│   ├─object: MEM<0x7ffd870df450>[256]
│   └─id: 0x2
└─location:
  ├─??? : memcpy@plt : ???
  ├─/home/guilhem/dev/colorstreams/doc/tutorial/target2.c : get_msg : 37
  └─/home/guilhem/dev/colorstreams/doc/tutorial/target2.c : main : 60
[Compose <4> → OOB Auto-Sink <1> → SINK] Detection_1_OOB_write_base <0> <REG<rdi>[8]> (auto OOB base)
[Compose <4> → OOB Auto-Sink <1> → SINK] Detection_1_OOB_write_size <1> <REG<rdx>[8]> (auto OOB size)
[Compose <4> → OOB Auto-Sink <1> → SINK] Detection_1_OOB_write_wdata <2> <(0->7 MEM<0x7ffd870e05c6>[4611686018427387903])> (auto OOB written data)
[Compose <4> → OOB Auto-Sink <1> → SINK] Detection_0_OOB_read_base <3> <REG<rsi>[8]> (auto OOB base)
[Compose <4> → OOB Auto-Sink <1> → SINK] Detection_0_OOB_read_size <4> <REG<rdx>[8]> (auto OOB size)
[Compose <4> → Symbolic Policy <3> → SINK] Detection_1_OOB_write_base <0> <REG<rdi>[8]> (auto OOB base)
[Compose <4> → Symbolic Policy <3> → SINK] Detection_1_OOB_write_size <1> <REG<rdx>[8]> (auto OOB size)
[Compose <4> → Symbolic Policy <3> → RESULT]
Detection_1_OOB_write_size <1>:
├─Sink: REG<rdx>[8]
├─Symbolic: <True, True, True, True, True, True, True, True>
│ ├─Symbolic execution runtime: 0.103s
│ └─Tracer runtime (target execution): 0.384s
└─S&S:
  ├─projection: Detection_1_OOB_write_size_1_proj_0
  └─result:
    └─S&S:
      └─no split limit:
        ├─Regularity constraint: None
        ├─Bitsize: 0x40
        ├─I0:
        │ ├─interval: [0x0; 0x100] (0x101)
        │ └─status: strong
        ├─I1:
        │ ├─interval: [0xffffffffffffffd8; 0xffffffffffffffff] (0x28)
        │ └─status: strong
        ├─Count:
        │ ├─min: 0x129
        │ ├─maybe min: 0x129
        │ ├─max: 0x129
        │ └─pmc fallback: False
        ├─Splits: 1
        ├─Maximum Splits: -1
        ├─Runtime: 0.136s
        └─Timeout: false
[Compose <4> → Symbolic Policy <3> → SINK] Detection_1_OOB_write_wdata <2> <(0->7 MEM<0x7ffd870e05c6>[4611686018427387903])> (auto OOB written data)
[Compose <4> → Symbolic Policy <3> → SINK] Detection_0_OOB_read_base <3> <REG<rsi>[8]> (auto OOB base)
[Compose <4> → Symbolic Policy <3> → SINK] Detection_0_OOB_read_size <4> <REG<rdx>[8]> (auto OOB size)
[Compose <4> → Symbolic Policy <3> → RESULT]
Detection_0_OOB_read_size <4>:
├─Sink: REG<rdx>[8]
├─Symbolic: <True, True, True, True, True, True, True, True>
│ ├─Symbolic execution runtime: 0.103s
│ └─Tracer runtime (target execution): 0.384s
└─S&S:
  ├─projection: Detection_0_OOB_read_size_4_proj_0
  └─result:
    └─S&S:
      └─no split limit:
        ├─Regularity constraint: None
        ├─Bitsize: 0x40
        ├─I0:
        │ ├─interval: [0x0; 0x100] (0x101)
        │ └─status: strong
        ├─I1:
        │ ├─interval: [0xffffffffffffffd8; 0xffffffffffffffff] (0x28)
        │ └─status: strong
        ├─Count:
        │ ├─min: 0x129
        │ ├─maybe min: 0x129
        │ ├─max: 0x129
        │ └─pmc fallback: False
        ├─Splits: 1
        ├─Maximum Splits: -1
        ├─Runtime: 0.130s
        └─Timeout: false
[Compose <4> → Symbolic Policy <3> → ERROR <0>] Uncaught exception: memcpy size too large!
[ERROR <1>] fatal error
[Compose <4> → STATS]
Compose <4> stats:
├─Processed function calls: 71
├─Processed instructions: 3541
├─Unique processed instructions: 1585
├─Sources: 0
├─Sinks: 0
├─Positive sinks: 0
└─Analysis runtime: 4.090s
[STATS]
General stats:
├─Warnings: 1
└─Total runtime: 8.730s
[DONE]
```

In particular, we see that write sizes are either within the buffer or very large here:
```c
└─S&S:
  ├─projection: Detection_1_OOB_write_size_1_proj_0
  └─result:
    └─S&S:
      └─no split limit:
        ├─Regularity constraint: None
        ├─Bitsize: 0x40
        ├─I0:
        │ ├─interval: [0x0; 0x100] (0x101)
        │ └─status: strong
        ├─I1:
        │ ├─interval: [0xffffffffffffffd8; 0xffffffffffffffff] (0x28)
        │ └─status: strong
        ├─Count:
        │ ├─min: 0x129
        │ ├─maybe min: 0x129
        │ ├─max: 0x129
        │ └─pmc fallback: False
        ├─Splits: 1
        ├─Maximum Splits: -1
        ├─Runtime: 0.136s
        └─Timeout: false
```

### Vuln. B

For [Vuln. B](#vuln-b), the command line is identical, aside from the program inputs:
```c
colorstreams -main main -p compose -compose-producers "autosource;oobautosink" -autosource-source-stubs "get_input:bytes_to_buf(ret,ret<gdb<*((char**)@2)>>,ret<strlen<gdb<*((char**)@2)>>>,message,controlled);get_input:ret_reg(controlled)" -oobautosink-data-lim 8 -oobcheck-dont-check "_IO_printf;_IO_puts" -compose-consumers symbolic -symbolic-cw "_IO_printf;_IO_puts" -symbolic-p "S&S" -args "\"(a\" 5000" ./target.run
```

This time however, we see that write sizes are much more useful, allowing to overwrite memory next to the buffer:
```c
└─S&S:
  ├─projection: Detection_0_OOB_write_size_1_proj_0
  └─result:
    └─S&S:
      └─no split limit:
        ├─Regularity constraint: None
        ├─Bitsize: 0x40
        ├─I0:
        │ ├─interval: [0x0; 0x128] (0x129)
        │ └─status: strong
        ├─Count:
        │ ├─min: 0x129
        │ ├─maybe min: 0x129
        │ ├─max: 0x129
        │ └─pmc fallback: False
        ├─Splits: 0
        ├─Maximum Splits: -1
        ├─Runtime: 0.110s
        └─Timeout: false
```

**What is new?**

Colorstreams 0.2 is now open-source and available on [GitHub](https://github.com/binsec/colorstreams). It brings some improvement from the USENIX 2025 artifact:
* greatly improved tracing performance
* updated BINSEC and solver portfolio
* (oobcapa) added the ability to load and merge OOB capabilities from different runs for scoring
* and many new features helping with automation!

You can learn how to use Colorstreams with the following [tutorials](https://github.com/binsec/colorstreams/tree/main/doc/tutorial) or read our paper [Attacker Control and Bug Prioritization (USENIX 2025)](https://www.usenix.org/conference/usenixsecurity25/presentation/lacombe).
