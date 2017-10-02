---
category: ctf
layout: ctfpost
event: CMU Binary Bomb Challenge
title: Level 6
author: Benjamin Assadsolimani
description: "Level 6 of the CMU Binary Bomb challenge."
---


I recently stumbled upon the [CMU Binary Bomb challenge](http://www.cs.cmu.edu/afs/cs/academic/class/15213-s02/www/applications/labs/lab2/bomblab.html). I grabbed the binary of [here](https://github.com/petroav/CMU-assembly-challenge/blob/master/binary?raw=true) and started to defuse the thing. You quickly figure out that the binary is divided into several levels where each level can be solved on its own. Level 1-5 are pretty straight forward and can be solved with ease using your favorite debugger and/or disassembler (it's probably IDA isn't it). Now, level 6 is a little bit trickier. It starts off fine by parsing 6 numbers from your provided input. However, those numbers are then processed with a bunch of assembly code that does some weird stuff which we are totally too lazy to analyze. Instead, we notice that the code does not contain libary calls, system calls or any other crap that would be hard for `angr` to deal with. Therefore, we make the lazy decision to write a simple angr script that solves the level for us. After all, we already solved level 1-5 in the traditionally way by reading and understanding the respective assembly code...

## Keep calm - use angr

So the action plan is pretty simple:
* Use angr
* Call the function `phase_6`
* Tell angr where the 6 numbers are stored in memory and that those are symbolic variables
* Let angr find its way through the assembly jungle
* ??? (angr symbolic execution magic)
* Profit (output the solution)

And here it is:

``` python
import angr

find             = 0x08048E87
avoid            = 0x080494FC
phase_6          = 0x08048D98
read_six_numbers = 0x08048FD8
numbers_array    = 0x7ffeffcc

proj = angr.Project("./binary")

# hook read_six_numbers function with a stub as we will write our numbers directly to memory
stub_func = angr.SIM_PROCEDURES['stubs']['ReturnUnconstrained']
proj.hook(read_six_numbers, stub_func())

state = proj.factory.call_state(phase_6)
numbers= [state.solver.BVS("num" + str(i), 32) for i in range(6)]

for i in range(6):
    state.mem[numbers_array + 4 * i].uint32_t= numbers[i]

simgr= proj.factory.simgr(state)
simgr.explore(find=[find], avoid=[avoid])

print ' '.join([str(simgr.found[0].solver.eval(num)) for num in numbers])
```

After some time (about four minutes), angr tells us the solution: `4 2 6 3 1 5`.

Great!