# Arithmetic Operations
- add
- and
- not
- subtract
- or
- clear a register
- copy from one register to another
- increment a register by n, <img src="https://render.githubusercontent.com/render/math?math=-16\leq n\leq 15">

## Subtracting
```assembly
;SUB R1, R2, R3	;instruction does not exist
;how to subtract

NOT R3, R3		;flip the bits of R3
ADD R3, R3, #1	;add 1 to R3 ;now R3 is -R3
ADD R1, R2, R3
```

## OR
```assembly
;OR R1, R2, R3	;instruction does non exist: R1 = R2|R3
;how to do OR	;use DeMorgan's Law

NOT R2, R2		;R2 = ~R2
NOT R3, R3		;R3 = ~R3
AND R1, R2, R3	;R1 = ~R2 & ~R3
NOT R1, R1		;R1 = ~(~R2 & ~R3)
```

## Clearing Registers
```assembly
;CLR R1			;ex. set R1 = 0
AND R1, R1, #0	;R1 = R1 & 0
```

## Moving From Register To Another
```assembly
;MOV R1, R2		;ex. R1 = R2
ADD R1, R2, #0	;R1 = R2 + 0
```
- this is how you copy from one register to another
- Note: do not use LD instruction to move, it will not work

# Control Instructions
## `BR`
- if, then, else
- for
- while
- do while
- conditional branch
- unconditional branch (BRnzp or BR)
- never branch (NOP)

## `JMP`
- go to some location
- branches long distances
- Note: we rarely use JMP in this course, BR is more common

## Differences Between BR And JMP
- BR
	- can branch on N, Z, and P conditions
	- can always branch (BR or BRnzp)
	- can never branch (NOP)
	- destination address is always PCoffset9
	- can't branch more than -256 to 255 words
- JMP
	- always branches
	- destination address is always in a register
	- can branch to any memory address

## How Do We Do `IF` Using A `BR`?
- first do an operation (ADD, AND, NOT, LD, LDI, LDR, ST, STI, STR) to set the condition codes
- then BR with the appropriate combination of NZP conditions in the instruction

## Every Condition: Comparison With Zero
|comparison|condition code|
|-|-|
|<|N|
|<=|NZ|
|==|Z|
|!=|NP|
|>=|ZP|
|>|P|

- always => NZP (we abbreviate BRnzp as just BR, branch always)
- never => no condition codes (we use NOP = no operation)

# Stylized Assembly Coding
- assembly programming gives the programmer a lot of freedom
	- freedom to optimize performance whether needed or not
	- freedom to write impossibly complex code
	- freedom to make a zillion different errors
	- freedom to debug everything in binary
- is this always a good idea?
	- if you are just learning
	- if you don't want to make a career of it
	- if you recognize that a compiler/optimizer can do a better job

# Act Like A Compiler
- write your algorithm in a high-level language
- write down where you are going to store your variables (register, static memory, stack offset, etc.)
	- use comments for registers and stack offsets
	- use assembler directives to reserve memory
- copy your algorithm with ";" at the beginning of each line to create assembly language comments
- translate each statement into a stanza of the appropriate machine language instructions
	- for complex statements like IF, WHILE, CALL, use templates
	- make sure that each stanza you translate is independent of your other stanzas
		- at the start, don't depend on the contents of temporary registers
		- at the start, don't depend on the condition codes
		- at the end, store your modified values where they belong
- this will not result in the most efficient code but it will be correct code
- Note: Complx doesn't know what is data and what is an instruction code, you need to differentiate while writing code

# Code Templates
## if (R1>0) then ... else ...
```assembly
		ADD		R1, R1, #0		;if (R1>0) then
		BRnz	ELSE1

		...[THEN part]...

		BR		ENDIF1
ELSE1	NOP						;else

		... [ELSE part]

ENDIF1	NOP						;endif
```

## for (init; R1>0; reinit)
```assembly
		...[initialize loop]...	;for (init;

FOR1	ADD 	R1, R1, #0
		BRnz 	ENDF1			;R1>0;
	
		...[FOR body]...
		
		...[reinitialize]...	;reinit)
	
		BR		FOR1
ENDF1	NOP
```

## while (R1>0)
```assembly
WHILE1	ADD 	R1, R1, #0		;while(R1>0)
		BRnz 	ENDW1
		
		...[WHILE body]...
		
		BR		WHILE1
ENDW1	NOP						;endwhile
```

## do ... while(R1>0)
```assembly
DO1		NOP						;do

		...[DO WHILE body]...
		
		ADD 	R1, R1, #0
		BRp		DO1				;while(x)
```

## if (A=0 && B=0)
```assembly
		; go to ELSE part if either condition is false
		
		LD		R0, A
		BRnp	ELSE
		LD		R0, B
		BRnp	ELSE2
		
		;THEN code goes here
		
		BR		ENDIF2
ELSE2	NOP

		;ELSE code goes here
		
ENDIF2	NOP
```

## Array Addressing
```assembly
;i is R3, q is R4
;short q

;short a[5]
a		.blkw	5

;short i=3
		AND 	R3, R3, #0
		ADD		R3, R3, #3

;q=a[2]
		LEA		R0, a
		LDR		R4, R0, #2
		
;q=a[i];array is close
		LEA		R0, a
		ADD		R0, R0, R3
		LDR		R4, R0, #0
		
;q=a[i];array is far
		LD		R0, aaddr
		ADD		R0, R0, R3
		LDR		R4, R0, #0
		
		...
		
		aaddr	.fill a
		
;a[i]=q
		LEA		R0, a
		ADD		R0, R0, R3
		STR		R4, R0, #0

```

# Templates For Indexing Into An Array
## Fetching ARRAY[I] (nearby)
```assembly
LEA	R1, ARRAY	;R1 is address of ARRAY[0]
LD	R2, I		;R2 is index number
ADD	R1, R1, R2	;R1 is address of ARRAY[I]
LDR R1, R1, #0	;R1 is value of ARRAY[I]
```

## Fetching ARRAY[I] (far away)
```assembly
LD	R1, ARRAY	;R1 is address of ARRAY[0]
LD	R2, I		;R2 is index number
ADD	R1, R1, R2	;R1 is address of ARRAY[I]
LDR	R1, R1, #0	;R1 is value of ARRAY[I]
BR	SK			;Don’t execute the address
AD	.fill ARRAY
SK	NOP
```

## Storing ARRAY[I] (nearby)
```assembly
LEA R1, ARRAY	;R1 is address of ARRAY[0]
LD R2, I		;R2 is index number
ADD R1, R1, R2	;R1 is address of ARRAY[I]
STR R3, R1, #0	;value of ARRAY[I] is R3
```

## Storing ARRAY[I] (far away)
```assembly
LD R1, ARRAY	;R1 is address of ARRAY[0]
LD R2, I		;R2 is index number
ADD R1, R1, R2	;R1 is address of ARRAY[I]
STR R3, R1, #0	;value of ARRAY[I] is R3
BR	SK			;Don’t execute the address
AD	.fill ARRAY
SK	NOP
```