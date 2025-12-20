---
layout: single
title: Program Memory
date: 2025-12-06
classes: wide
tags:
  - Stack
  - Heap
---

> Operating systems use virtual memory to allow processes to work as if they have the full addressable space, even if physical RAM is smaller.
- [Memory Addressing: Physical and Virtual Memory](/memory-addressing/#physical-and-virtual-memory)

The following diagram illustrates a typical program memory layout on a 32-bit system, with virtual addresses spanning from `0x00000000` to `0xFFFFFFFF`.

```
High Address (0xFFFFFFFF)  ->  |-----------------------------|
                               |                             |
                               |   Command-line arguments    |
                               |  and environment variables  |
                               |                             |
                               |-----------------------------|
                               |                             |
                               |            Stack            | 
                               |                             |
                               |.............................|
                               |              |              |
                               |              V              |
                               |                             |
                               |                             |
                               |                             |
                               |              ^              |
                               |              |              |
                               |.............................|
                               |                             |
                               |            Heap             | 
                               |                             |
                               |-----------------------------|
                               |                             |
                               |     Data Segment (BSS)      |
                               |       (Uninitialised)       | 
                               |                             |
                               |-----------------------------|
                               |                             |
                               |         Data Segment        |
                               |         (Initialised)       | 
                               |                             |
                               |-----------------------------|
                               |                             |
                               |         Text Segment        |
                               |        (Program Code)       |
                               |                             |
Low Address  (0x00000000)  ->  |-----------------------------|
```

At the bottom of the address space lies the Text Segment, also called the Code Segment, which stores the program’s instructions.

Directly above the Text segment is the Data segment, which is divided into two areas:
	1.	Initialized data - where variables with predefined values are stored.
	2.	Uninitialized data (BSS) - where variables without initial values reside.

> Note: Global variables that are not explicitly initialized by the program are automatically set to 0 by the process model. This behavior does not apply to local variables.

All contents of the Text and Data segments are determined at compile time, so the compiler knows their locations and encodes this information in the executable.

At the very top of the memory space are the command-line arguments and environment variables, which are set when the process starts. Immediately below them is the Stack, which holds local variables as well as metadata used for function calls and returns.

Between the Data segment and the Stack lies the Heap, the area managed by dynamic memory functions.

Unlike the Text and Data segments, the contents of the Stack and Heap are determined at runtime, and their organization and growth depend on the program’s execution.

As the program requires more memory for the Heap, it expands upward toward higher memory addresses, whereas the Stack grows downward toward lower addresses when more space is needed.

During execution, the program keeps track of the Stack Pointer, which marks the current top of the Stack.

```
0xFFFFFFFF  |-----------------------------|
            |                             |
            |             ...             |
            |                             |
            |-----------------------------|
            |                             |
            |            Stack            | 
            |                             |
            |.............................| <- Stack Pointer
            |                             |
            |                             |
            |                             |
            |.............................|
            |                             |
            |            Heap             | 
            |                             |
            |-----------------------------|
            |                             |
            |             ...             |
            |                             |
            |-----------------------------|
            |                             |
            |        section .text        |
            |        global _start        |
            |                             |
            |        _start:              |
            |            push 0x539;      |
            |                             |
0x00000000  |-----------------------------|
```

When a push instruction is executed, the value is placed onto the Stack, and the Stack Pointer is updated accordingly.

```
0xFFFFFFFF  |-----------------------------|                      0xFFFFFFFF  |-----------------------------|
            |                             |                                  |                             |
            |             ...             |                                  |             ...             |
            |                             |                                  |                             |
            |-----------------------------|                                  |-----------------------------|
            |                             |                                  |                             |
            |            Stack            |                                  |            Stack            |
            |                             |                                  |                             |
            |.............................|                                  |.............................|
            |            1337             |                                  |            1337             |                    
            |.............................| <- Stack Pointer                 |.............................|
            |                             |                                  |            7331             | 
            |                             |                                  |.............................| <- Stack Pointer
            |                             |                                  |                             |
            |                             |                                  |                             |
            |                             |                                  |                             |
            |.............................|                                  |.............................|
            |                             |                                  |                             |
            |            Heap             |                                  |            Heap             |
            |                             |                                  |                             |
            |-----------------------------|                                  |-----------------------------|
            |                             |                                  |                             |
            |             ...             |                                  |             ...             |
            |                             |                                  |                             |
            |-----------------------------|                                  |-----------------------------|
            |                             |                                  |                             |
            |        section .text        |                                  |        section .text        |
            |        global _start        |                                  |        global _start        |
            |                             |                                  |                             |
            |        _start:              |                                  |        _start:              |
            |            push 0x539;      |                                  |            push 0x539;      |
            |                             |                                  |            push 0x1CB3;     |
            |                             |                                  |                             |
0x00000000  |-----------------------------|                                  |-----------------------------|
```

In addition to storing local variables, the Stack is used to manage function calls and returns, holding return addresses and other bookkeeping information.
