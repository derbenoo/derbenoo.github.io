---
category: ctf
layout: ctfpost
title: tablEZ
author: Benjamin Assadsolimani
event: CSAW CTF Qualification Round 2017
ctftime-task-id: 4656
tags: [Reversing]
points: 100
solves: 342
description: "Bobby was talking about tables a bunch, so I made some table stuff. I think this is what he was talking about..."
---


## Challenge Description

Bobby was talking about tables a bunch, so I made some table stuff. I think this is what he was talking about...

<!-- [tablEZ](https://ctf.csaw.io/files/d814c19920533b7dac9e20a2efb4fe4f/tablez) -->

## Overview

We check out the binary:

```
$ file tablez
tablez: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=72adea86090fb7deeb319e95681fd2c669dcc503, not stripped
```

When we execute the binary, we are prompted to input the flag. Since we don't know it (yet), the response is a simple "WRONG" message. We open the binary in IDA and see that it is quite small so we can quickly analyze the code and figure out what is going on:

![IDA binary overview]({{site.url}}/public/images/ctf/csaw_qual_17_ida_tablez.png)

So our input has to be 37 characters long and each character is replaced by a corresponding value that is stored in a table. Finally, the transformed input is compared to the constant value that was defined at the beginning and if they are equal, the "CORRECT <3" message is printed. There are many ways to reverse the table lookup for the constant value, but we decide to let the binary do the work for us.

## Let the binary do the work for us

The `get_tbl_entry` function looks like the following:

![IDA get_tbl_entry function dissassembly]({{site.url}}/public/images/ctf/csaw_qual_17_ida_get_tbl_entry.png)

The lookup table is constructed in the following way: `[a_0, b_0, a_1, b_1, ..., a_254, b_254]` where `a_1, a_2, ..., a_254` are the input bytes and `b_1, b_2, ..., b_254` are the corresponding bytes that they are mapped to (`a_i -> b_i`). The function first determines the index of the input byte by iterating through the array with a stepsize of 2 and checks if `a_i` is equals the input byte. If the right index was found, it is used to query the array with an offset of 1 to find `b_i`. If we want to reverse the mapping, all we would have to do is swap out the two references to the lookup table such that the function would first determine the index on the `b_i` value and then find the corresponding `a_i` value. This only requires two little patches to the binary: First, increment the reference to `trans_tbl` at `loc_85D` and decrement the next reference to `trans_tbl` by 1. We do so and save the patched binary.

Now we can give the binary the constant value and it will reverse it for us so that we obtain the flag! Since we don't know how to pass non-printable characters to a `read()` call, we execute the binary with gdb and set a breakpoint after the read call to manually change the bytes in memory:

```
$ gdb ./tablez

gdb-peda$ b *0x555555554938 // break after fgets
Breakpoint 1 at 0x555555554938

gdb-peda$ b *0x5555555549F7 // break on strcmp
Breakpoint 2 at 0x5555555549f7

gdb-peda$ r
Starting program: ./tablez
Please enter the flag: // input a 37 char long string
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, 0x0000555555554938 in main ()
gdb-peda$ n

gdb-peda$ p strcpy($rax, "\xCE\x65\x05\x99\x23\xF9\x27\x99\x23\xBE\x11\x11\x65\xB1\x99\x65\x23\x73\x99\x71\x1B\x30\xF4\xF9\xF9\xB3\x99\xBE\xB3\xB1\xE7\x11\xF5\x9D\x73\xB3\x27\x0a") // change string in memory to constant from binary
$1 = 0xffffe320
gdb-peda$ c
Continuing.

Breakpoint 2, 0x00005555555549f7 in main ()
gdb-peda$ x/s $rax // read flag from $rax after strcmp break
0x7fffffffe320: "}3m_r0f_rett3b_3ra_spuk00l_elb4t{galf"
```

Looks like the flag is reversed, let me fix that for you:
```
$ python -c 'print "}3m_r0f_rett3b_3ra_spuk00l_elb4t{galf"[::-1]'
flag{t4ble_l00kups_ar3_b3tter_f0r_m3}
```

Sweet!
