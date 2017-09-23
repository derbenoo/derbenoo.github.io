---
category: ctf
layout: ctfpost
title: WarmUp
author: Benjamin Assadsolimani
event: EKOPARTY CTF 2017
ctftime-task-id: 4648
tags: [Reversing]
points: 397
solves: 63
description: "It's time to warm up your reversing skills!"
---


## Challenge Description

It's time to warm up your reversing skills!

## Overview

We run the binary and get the following prompt:
```
$ ./warmup
Enter your values: ABC

```

Looks like we have a statically linked and stripped binary on our hands, let's open it in IDA. Unfortunately, IDA does not seem to be able to identify the used libraries so we end up with a huge pile of unnamed assembly code. Anyways, let's start at the `start`. We know that the `start` function usually calls `__libc_start_main` and passes the address of the `main` function to it as a parameter:

![IDA start function disassembly]({{site.url}}/public/images/ctf/ekoparty_ctf_2017_warmup_start.png)

we follow the `sub_400E5F` function which is most likely our `main` function and see that it makes 2 function calls (`sub_4009AE` and `sub_400E4F`). The first function seems to be a simple read call as we find the string that was also output by the binary before we were prompted to input our value:

![IDA read function disassembly]({{site.url}}/public/images/ctf/ekoparty_ctf_2017_warmup_read.png)

We assume that the second function then verifies our input. We follow a very long trail of function calls (luckily all functions have the same size and structure so that we can simply spam double click to get through) until we arrive here:

![Input check disassembly graph overview]({{site.url}}/public/images/ctf/ekoparty_ctf_2017_warmup_input_check.png)

Most of the blocks look like this:

![Input check disassembly single compare block]({{site.url}}/public/images/ctf/ekoparty_ctf_2017_warmup_byte_check.png)

So in each block, a byte of our input is compared with a byte of the "correct" input. If the compare checks out, the program goes to the next block. If not, it simply exits. When all compares where successful, we arrive at a block that prints the string "valid!". The problem is that the compares are not sequential but jump around in our input string. Additionally, the locations of the correct input bytes seem to be random as well and some blocks do more complicated operations to obtain the correct input byte before the compare. While we could manually write down at what position of the input the compare expects which byte, it is easier and way cooler to write a GDB Python script that does this for us.

## GDB/ PEDA Script

The script that automatically extracts the flag looks like the following:

``` python
# run like this: gdb -x warmup.py

offset      = 50
len_input   = 50
filename    = "in"

with open(filename, "w") as f: # write bytes starting from offset to input file
    f.write(''.join([chr(i) for i in range(offset, offset+len_input)]))

flag= ["\x00" for _ in range(len_input)] # initialize flag array with null bytes

peda.execute('file ./warmup')
peda.set_breakpoint(0x4009D8) # break before first input reference
peda.execute("run < %s" % filename) # run with input file

rip= peda.getreg("rip")
while rip < 0x400c6c: # run until hitting the "valid!" part or failure
    rip= peda.getreg("rip")
    instr= peda.current_inst(rip)[1][:3]

    if instr == "cmp":
        al= peda.getreg("rax") # flag char
        dl= peda.getreg("rdx") # input char
        flag[dl-offset]= chr(al)

    if instr == "jne":
        peda.set_eflags("zero", True)

    gdb.execute("n", to_string=True)

print(''.join(flag[:flag.index("\x00")])) # print flag until first null byte
peda.execute('quit')
```

And here's how it works. We basically want the following things:
* Stop at each `cmp` instruction and write down at which position of our input which byte was checked for
* Prevent each `jne` instruction from jumping so we end up in the next block, regardless of the result of the `cmp` instruction

Ok, preventing `jne` from jumping is easy as we simply have to set the zero flag to 1 before executing it. For the `cmp` instruction, we thought of a nifty trick: The registers `dl` and `al` are compared where `dl` contains the byte of our input and `al` the byte of the "correct" input. Now, if we provide the binary with an input of the structure `\x00\x01\x02\x03 ... \x50`, each input byte is equal to its position inside the string. This way, we simply have to note down the byte in the `al` register at the position `dl` and get the flag! In the above script, we had to start from an offset (50) because the byte `\x0a` is interpreted as a newline character by the read call, which then terminates. We write the input bytes to a file so we can easily provide them as an input to the binary in GDB. The flag variable is initialized as a list of null bytes so that we can easily replace a character at a position (a string would be cumbersome as strings are immutable in Python).

We let the script run and receive the flag:

```
gdb -x warmup.py
[...]
EKO{1s_th1s_ju5t_4_w4rm_up?}
```
