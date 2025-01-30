---
title: HackDay 2025 - Just dig
date: 2025-01-26
categories:
  - CTFs
  - HackDay 2025
  - Reverse
tags:
  - reverse
description: Reverse Engineering challenge with an ELF binary, where debugging with GDB allowed decoding hexadecimal values in Base64 to obtain the flag. 🕵️‍♂️
---

## Description 

Category: Reverse Engineering

>Researchers have recently uncovered a mysterious ancient artifact beneath the Cogswell Halls. Its purpose remains unknown, but its potential seems invaluable. Who knows? The secrets it holds could revolutionize modern knowledge. They need an expert eye—yours! Let's dig in !

File : `Just_dig.out`

[Link of the event](https://ctftime.org/event/2615)

## Solve the challenge

### Examining the Binary

We begin by analyzing the provided binary file using the `file` command:
```bash
file Just_dig.out

Just_dig.out: ELF 32-bit LSB pie executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=6d1be85b880b780e60c14a840ba27bcc0ea90214, for GNU/Linux 3.2.0, not stripped
```

This tells us that the binary is a 32-bit ELF file, compiled for Linux systems. The binary is PIE (Position Independent Executable), meaning the memory addresses may vary at runtime, but since the binary is **not stripped**, we can still use tools like GDB to debug and analyze it effectively.

### Debugging with GDB

Using GDB, we identify a function called `HackDay`, which contains a series of comparisons. These comparisons iterate from `1` to `0xC` (12), each time **setting a specific value into a register before performing a comparison.**

```
0x565562a1 <+0>:	mov    ebx,0x56556195   # Load string address into EBX
0x565562a6 <+5>:	inc    DWORD PTR ds:0x56559008      # 1
0x565562ac <+11>:	cmp    DWORD PTR ds:0x56559008,0x1  # CMP 1,1
0x565562b3 <+18>:	je     0x56556231 <qRHxrv9HwO>
```

Each step involves examining a memory address and extracting its content. For example, at `0x56556195`, we find the following:

```
x/4x 0x56556195 
=> 0x53	0x45 0x46 0x44 
x/s 0x56556195
=> "SEFDdkVf\023\371\333\310\001bI\306blRvXzRzr1\224\abX0=RGlHWXtp(\003H\f"
```
* The extracted value here is `SEFD`.

### Extracting and Decoding

By analyzing each step of the `HackDay` function, we repeat this process for all 12 iterations. We extract hexadecimal values, convert them to ASCII characters, and then concatenate them.

```
1: 53h, 45h, 46h 44h => SEFD => HAC
2: 53h, 30h, 52h 42h => S0RB => DA
3: 57h, 58h, 74h 70h => WXtp => Y{i
4: 58h, 30h, 78h 76h => X0xv => _Lo
5: 64h, 6Bh, 56h 66h => dkVf => vE_
6: 52h, 47h, 6Ch 48h => RGlH => DiG
7: 5Ah, 79h, 46h     => ZyF  => g!
8: 4Fh, 56h, 38h 78h => OV8x => 9_1
9: 62h, 6Ch, 52h 76h => blRv => nTo
10: 58h, 7Ah, 52h 7Ah => XzRz => _4s
11: 62h, 58h, 30h 3Dh => bX0= => m}
12: 0h, 0h, 0h, 0h 
```

At this point, we notice that the resulting string resembles **Base64-encoded text**, ending with the `=` character, which is common for Base64 padding. We decode each extracted part from Base64 to reveal the final flag.

Flag : `HACKDAY{i_LovE_DiGg!9_1nTo_4sm}`