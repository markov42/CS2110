# The ISA
## Instruction Set Architecture (ISA)
- **ISA**: specifies all the information about the computer that the software needs to be aware of
- ISA is a programmer’s view of the CPU (ex. LC-3) from the outside (how do I write machine code/assembly?)
- the internal implementation of the LC-3 is the datapath and microcode

## Who Uses An ISA?
- machine language programmers
- assembly language programmers
- compiler writers
- high-level language programmers interested in performance, debugging, etc.

## ISA: What Is Specified
- memory organization
- registers
- instruction set
	- opcodes
	- data types
	- addressing modes

## LC-3 Memory Organization
- address space
	- 16 bit addresses
	- 0-65535
	- x0-xFFFF
- addressability
	- 16 bits
- word addressable/byte addressable?
	- strictly word addressable with 16 bit words

## LC-3 Registers
- general purpose registers
	- 8 general purpose registers
	- R0-R7
- Is the PC considered to be a GPR?
	- no
- conventions
	- certain registers will have designated purposes (that comes later, for now you can use all 8 of them)

## LC-3 Instruction Set
- Instruction
	- OP Codeq
	- Operand
- Instruction Set
	- Op Codes
		- Operate (ALU)
		- Data movement (load/store memory) 
		- Control (conditionals, loops, etc.)
	- Data Type
	- Addressing Modes

## How Many Instructions? (ISA Design Philosophies)
- lots
	- CISC (complex instruction set computer)
	- ex. Intel X86, DEC VAX, IBM Z-series
	- do as much as you can in a single instruction
	- relatively easy to write efficient machine code by hand
- few
	- RISC (reduced instruction set computer)
	- ARM, PowerPC, MIPS, LC-3
	- expose as much as you can to the compiler so that it can optimize it
	- hard to write efficient machine code by hand

## LC-3 Machine Code And Assembly Language
- for now, we are focused on the LC-3 datapath
	- the circuit implementation and the state machine
	- the control signals that make each instruction happen
- you will need to understand what each LC-3 machine code instruction does (atomically, in isolation)
	- what registers or memory locations does it change?
	- which ALU operation is happening?
	- how does the data flow through each MUX and the bus to make the operation execute with the desired behavior?
- you do NOT need to understand why or how you would use these instructions in a program yet
	- will be covered later, with assembly programming
	- for now, just understand each instruction’s behavior, so you can know how to implement it in the datapath and microcode, using control signals

# LC-3 Instruction Set
- 16-bit instructions
- 4 bits for opcode (bits 15-12)
- how many instructions?
	- 15 (code 1101 is reserved for future use)

## LC-3 Assembly Language
- a symbolic (text, human readable) representation of the binary machine language
- instructions are one-to-one between assembly language and machine language
- ex. ADD R1, R2, R3

## Machine Instruction Cycle
- FETCH, FETCH, FETCH, DECODE, EXECUTE
	- FETCH and DECODE are the same for every instruction

## Categories Of LC-3 Instructions
- operate (ALU) - only takes 1 cycle for the EXECUTE phase to run
- data movement (memory)
- control
	- BR
	- JMP
	- JSR
	- JSRR
	- RET
	- RTI
	- TRAP

## Basic Format For Instructions
<img src="img/l07-machine-instruction-format.png" alt="machine-instruction-format" width="400">

### Operate (ALU) Instructions
- ADD
- AND
- NOT

<img src="img/l07-add-and-instructions.png" alt="add-and-instructions" width="350">

<img src="img/l07-not-instruction.png" alt="not-instruction" width="350">

- `imm5` refers to an immediate value or literal
	- 1 and -1 are the most commonly used literals in high level language
	- using `imm5` allows the machine to use only 1 register (rather than using 2 registers)
- the addressing mode depends on bit 5

#### Addressing Modes (For ALU Operations)
- register
	- all operands come from register file
	- ex. `ADD R1, R2, R3`
- immediate or literal
	- some operands come from bits in the instruction itself (get them from the instruction register (IR)
	- immediate values are 5-bit two’s complement (for ADD and AND)
		- from bits `[0:4]` in the instruction itself
	- sign-extend the immediate value to 16 bits two’s complement
		- ex. `ADD R1, R2, #3`
		- ex. `AND R1, R2, #-3`
	
#### Example: Addressing Modes (For ALU Operations)
```
ADD R3, R3, R2		; 0001 011 011 0 00 010 (register addressing mode)

ADD R3, R3, #2		; 0001 011 011 1 00010 (literal or immediate addressing mode)
```

### Data Movement Instructions
|load|store|
|-|-|
|LD|ST|
|LDR|STR|
|LDI|STI|
|LEA|----|

- purpose
	- **load** data into a register (usually from memory)
	- **store** a register’s value out to memory

<img src="img/l07-load-data-mvmt-instructions.png" alt="load-data-mvmt-instructions">

<img src="img/l07-store-data-mvmt-instructions.png" alt="store-data-mvmt-instructions">

- LD/ST (PC-relative)
- LDR/STR (base + offset)
- LDI/STI (indirect, takes 5 states to complete)
- LEA (immediate, no memory access, 1 state to complete)

#### Addressing Modes (For Data Movement)
- where can operands (data) be found?
	1.  in the instruction (literal or Immediate)
	2. in the registers
	3. in the memory
- **effective address**: used to describe the memory location that the instruction uses for its operands
- 3 addressing modes involve memory locations, at the effective address
	- PC-relative (pc + offset)
	- base + offset
	- indirect
- other 2 addressing modes
	- immediate or literal
	- register
- each instruction allows a subset of these addressing modes (a common approach in a RISC architecture)

#### Calculating PC-Relative Effective Address
- PC-relative uses a 9-bit offset value so to find effective address:
	- sign extend the 9-bit offset value and add it to the 16-bit memory address

<img src="img/l07-pc-relative-addressing-mode-ex.png" alt="pc-relative-addressing-mode-ex" width="550">

#### Effective Address
- how big is a machine instruction?
	- 16 bits
- how big is a memory address?
	- 16 bits
- can we fit a full memory address in a machine instruction?
	- no, because we need some bits for the op code and other parts of the instruction

#### Example: Addressing Modes (For Data Movement)
- imagine memory in a large array
	- `int mem[65536]`
- calculate the effective address (EA), based on the addressing mode
	- PC relative: `mem[PC + offset]`
	- base + offset: `mem[BaseReg + offset]`
	- indirect: `mem[mem[PC + offset]]`

#### Loading From Memory
- takes 3+ cycles to complete

1. send the address of where the data is located to MAR (memory address register)
2. memory is read from that address and "driven" to the MDR (memory data register)
3. send data from MDR specified destination register

- recall FETCH (get a machine code instruction from memory)
- load (LD) is similar, but its execution state gets data from memory into a register

#### Storing To Memory
- takes 3+ cycles
1. data from a register is placed into the MDR
2. address of where to store data is placed into the MAR
3. data is stored in memory at the address specified by the MAR

- NOTE: indirect addressing mode takes 5 cycles
	- STI (store indirect)
	- LDI (load indirect)

#### What Is The Operand?
|Addressing Mode|Operand|
|-|-|
|immediate|SEXT(IR\[4:0\])|
|register|use the contents of the specified register|
|PC-relative|computer PC + SEXT(IR\[8:0\]) or PC + SEXT(IR\[10:0\]) and deference (note that PC is already incremented by this time|
|base + offset|computer the specified register + SEXT(IR\[5:0\]) and deference|
|indirect|compute PC-relative address but deference twice|

- **deference**: interpret what you have as a memory address and fetch

#### Comparing Data Movement Methods
- LEA: DR <- (PC + PCOffset9)
	- puts effective address (PC + offset) itself into a register (no memory access)
- LD: DR <- Mem\[PC + PCOffset9\]
	- put the contents of some memory at the effective address (PC + offset) into a register
- LDR: DR <- Mem\[BaseR + Offset6\]
	- puts the contents of the effective address (base register + offset)
- LDI: DR <- Mem\[Mem\[PC + PCOffset9\]\]
	-	goes to the effective address (PC + offset) and gets a value
	-	treat the value as another memory address and get data from there into a register (accesses memory twice)
- ST:
	- puts the contents of a register into memory at the effective address (PC + offset)
- STR:
	- puts the contents of a register into memory at the effective address (base register + offset)
- STI:
	- puts the contents of a register into a memory location whose address is stored in memory at the effective address (PC + offset)
	- accesses memory twice

#### Example: LD
<img src="img/l07-ld-instructions-ex.png" alt="ld-instructions-ex" width="550">

#### Example: LDI
<img src="img/l07-ldi-instructions-ex.png" alt="ldi-instructions-ex" width="550">

#### Example: STR
<img src="img/l07-str-instructions-ex.png" alt="str-instructions-ex" width="550">

#### Example: LEA
<img src="img/l07-lea-instructions-ex.png" alt="lea-instructions-ex" width="550">

### Control Instructions
<img src="img/l07-control-instructions-1.png" alt="control-instructions-1" width="550">

<img src="img/l07-control-instructions-2.png" alt="control-instructions-2" width="550">

#### Changing The Sequence Of Instructions
- in the FETCH phase, we increment PC by 1
	- but what if we don't want to execute the instruction that follows the previous (ex. loop, if-then, function call)
- control instructions
	- jumps are unconditional - always changes the PC
	- branches are conditional - changes the PC only if some condition is true (ex. the result of an ADD is 0)

#### LC-3 JMP Instruction Example
- set the PC to the value contained in a register (which becomes the address of the next instruction to fetch)

<img src="img/l07-jmp-instruction-ex.png" alt="jmp-instruction-ex" width="550">

#### Control Instructions Examples
<img src="img/l07-control-instructions-ex.png" alt="control-instructions-ex" width="550">

#### Condition Codes
- N - negative
- Z - zero
- P - positive

- the condition code register is set:
	- during EXECUTE when LD.CC is high
	- only set by ADD, AND, NOT, LD, LDR, LDI
	- otherwise the control codes remain unchanged
- condition codes are 3 bits in the BR instruction that are ANDed with the NZP condition code bits
	- if any bits are on, the branch is taken
	- otherwise, the instruction in the following word is fetched next

#### Condition Code Logic
<img src="img/l07-condition-code-logic.png" alt="condition-code-logic" width="500">

#### Which Instructions Set Condition Codes?
- all the ALU instructions
	- ADD, AND, NOT
- the LOAD (data)
	- LD, LDR, LDI
- in other words, every instruction that changes the value of a general purpose register, R0-R7
	- except for LEA
- all other instructions do not change the condition code

---
# Microsequencer And Microcode
<img src="img/l07-lc3-fsm-controller.png" alt="lc3-fsm-controller" width="450">

- **microsequencer**: describes the combinational circuit that chooses the next state from the 10 input lines and the current state
- **control unit**: controls the flow of data in the LC-3 data path
- microcode: a built-in program that the control unit executes
	- each cycle through the microcode program execute one LC-3 machine instruction
	- runs on a smaller sequential logic circuit to carefully choose a sequence from 64 possible states
- components of the control unit
	- control store
	- microsequencer