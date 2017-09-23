---
category: cheat-sheet
layout: cheat-sheet
title: GDB Python/ PEDA
author: Benjamin Assadsolimani
---


## Setup

```
git clone https://github.com/longld/peda.git ~/Repositories/peda    # clone PEDA repository
echo "source ~/Repositories/peda/peda.py" >> ~/.gdbinit             # make GDB load PEDA
```

Run scripts like this: `gdb -x script.py`

## Execute GDB command

```
gdb.execute(gdb_command, to_string= False) # Execute GDB command and return its output (if to_string= True)
peda.execute(gdb_command)                  # Execute GDB command (with exception wrapper)
peda.execute_redirect(gdb_command)         # Execute GDB command and return its output
```

## Get register values

```
peda.getreg(register)                      # Get value (Int) of a specific register name (String)
peda.getregs(reglist)                      # Get values of specified registers as dict: {regname(String) : value(Int)}
```

## Breakpoints

```
peda.set_breakpoint(location)              # Set breakpoint at target function or address (String ot Int)
peda.get_breakpoint(num)                   # Get info of a specific breakpoint
peda.get_breakpoints()                     # Get list of current breakpoints: (Num(Int), Type(String), Disp(Bool), Nnb(Bool), Address(Int), commands(String))
```

## Get instructions

```
peda.prev_inst(address, count= 1)          # Get previous 'count' instructions at an address as list of tuples: (address(Int), code(String))
peda.current_inst(address)                 # Parse instruction at an address as tuple of (address(Int), code(String))
peda.next_inst(address, count= 1)          # Get next 'count' instructions at an address as list of tuple (address(Int), code(String))
```

## Misc

```
peda.assemble(asmcode, bits= None)         # Assemble ASM instructions, multiple instructions are separated by ";" (String): return bin code (raw bytes)
peda.disassemble(*arg)                     # Wrapper for disassemble command (arg: args for disassemble command)
peda.getfile()                             # Get full path to executable file
peda.get_status()                          # Get execution status of debugged program
peda.getpid()                              # Get PID of the debugged process
```
