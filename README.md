# smdsh

A shell that doesn't rely on stack or heap allocation.  Keep commands short; there isn't a lot of error checking here.

## Rules
1. no heap memory allocation (malloc, brk, mmap, new processes, etc.)
2. no stack allocation (meaning the adjustment of rbp or rsp)
3. no calling libraries that violate rules 1 or 2
4. using memory that is already allocated at the start of the program is ok, but don't abuse it
5. storing read-only data in .text is ok
6. no self-modifying code (sssslllloooowwww!)

## Memory used
 - 128-byte red zone after rsp
 - regular x86 and x64 registers
 - xmm0 - xmm7

### Memory map
|      | 0x0 - 0x8 | 0x9 - 0xF |
|------|-----------|-----------|
| 0x00 | **argv    | **argv    |
| 0x10 | **argv    | **argv    |
| 0x20 | **argv    | **argv    |
| 0x30 | **argv    | **argv    |
| 0x40 | argv[0]   | argv[1]   |
| 0x50 | argv[2]   | argv[3]   |
| 0x60 | NULL      |           |
| 0x70 |           |           |

## Features
 - uses only 128 bytes of RAM (the Linux x86-64 red zone)
 - all environment variables defined at launch are passed to children
 - basic builtin support
 - extensive use of SIMD instructions to speed up data processing

`old_variants` contains older versions of smdsh that didn't work for various reasons.
 - `smdsh_xmm_str.s` used SSE string processing instructions for everything, which was a huge hassle and hard to read and adjust.  Better method is to use regular cmp and mask instructions.
 - `smdsh_tight.s` packed all commands and args as tight as possible in XMM registers which made it really hard to do string processing or shifting without losing data.

