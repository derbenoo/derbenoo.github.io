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
peda.get_eflags()                          # Get flags value from EFLAGS register: dictionary of named flags
peda.set_eflags(flagname, value)           # Set/clear/toggle value of a flag register: Boolean (success)
```

## Breakpoints

```
peda.set_breakpoint(location)              # Set breakpoint at target function or address (String ot Int)
peda.get_breakpoint(num)                   # Get info of a specific breakpoint
peda.get_breakpoints()                     # Get list of current breakpoints: (Num(Int), Type(String), Disp(Bool), Nnb(Bool), Address(Int), commands(String))
```

## Debugger Control

```
peda.stepuntil(inst, mapname, depth)       # Step execution until next "inst" instruction within a specific memory range (mapname) with specific depth
peda.testjump(inst= None)                  # Test if jump instruction is taken or not: (status, address of target jumped instruction)
```

## Get instructions

```
peda.prev_inst(address, count= 1)          # Get previous 'count' instructions at an address as list of tuples: (address(Int), code(String))
peda.current_inst(address)                 # Parse instruction at an address as tuple of (address(Int), code(String))
peda.next_inst(address, count= 1)          # Get next 'count' instructions at an address as list of tuple (address(Int), code(String))
```

## Memory Operations

```
peda.dumpmem(start, end)                   # Dump process memory from start to end: raw bytes
peda.readmem(address, size)                # Read content of memory at an address: raw bytes
peda.read_int(address, intsize= None)      # Read an interger value from memory with forced intsize
peda.read_long(address)                    # Read a long long value from memory
peda.writemem(address, buf)                # Write buf to memory start at an address, return number of written bytes (Int)
peda.write_int(address, value, intsize)    # Write an interger value to memory (with size intsize)
peda.write_long(address, value)            # Write a long long value to memory
peda.cmpmem(start, end, buf)               # Compare contents of a memory region with a buffer: dictionary of array of diffed bytes in hex
peda.xormem(start, end, key)               # XOR a memory region with a key (String)
peda.searchmem(start, end, search)         # Search for all instances of a pattern (String or Python Regex) in memory from start to end: (address(Int), hexval(String))
peda.searchmem_by_range(mapname, search)   # Search for all instances of a pattern in virtual memory ranges (string): (address(Int), hexval(String))
peda.search_reference(search, mapname)     # Search for all references to a value in memory ranges (String): (address(Int), hexval(String))
peda.search_address(searchfor, belongto)   # Search for all valid addresses in memory ranges: (address(Int), value(Int))
peda.search_pointer(searchfor, belongto)   #
```

## Execution flow

```
peda.xref(search= "", filename= None)      #  Search for all call references or data access to a function/variable (String): list of (address(Int), asm instruction(String))
peda.get_function_args(argc= None)         # Get the guessed arguments passed to a function when stopped at a call instruction (argc= force num arguments): list of arguments
```

## Assemble/ Disassemble

```
peda.assemble(asmcode, bits= None)         # Assemble ASM instructions, multiple instructions are separated by ";" (String): return bin code (raw bytes)
peda.disassemble(*arg)                     # Wrapper for disassemble command (arg: args for disassemble command)
peda.disassemble_around(address, count= 8) # Disassemble instructions nearby current PC or an address: (String)
```

## Misc

```
peda.is_executable(address)                # Check if an address (Int) is executable: Boolean
peda.is_writable(address)                  # Check if an address (Int) is writable: Boolean
peda.is_address(value)                     # Check if a value (Int) is a valid address (belongs to a memory region): Boolean
peda.getfile()                             # Get full path to executable file
peda.get_status()                          # Get execution status of debugged program
peda.getpid()                              # Get PID of the debugged process
```
