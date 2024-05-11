# Subroutines
- all of the following are related to subroutines
	- methods
	- functions
	- procedures
- subroutines are about reusing code

## Issue 1: What If Our Subroutine Calls Another Subroutine?
- subroutines generally return R7
- however, if a subroutine calls another subroutine, we would lose the old return address
- as a result, we need to preserve the old return address R7 before calling the next subroutine

## Issue 2: We Only Have A Few Registers
- only 8 registers (R0-R7)
- our main program is using registers
	- but our subroutines also need to use registers
	- so we save old values from the registers we use, then put the original values back after the subroutine completes

# The Stack
## Saving Registers In A Subroutine - Naive Solution
- a naive solution
```assembly
;subroutine that needs to use r1, r2, and r3

subr		st	r1, r1save
			st	r2, r2save
			st	r3, r3save

;some work is done here

			ld	r1, r1save
			ld	r2, r2save
			ld	r3, r3save
			ret
			
r1save		.fill	0
r2save		.fill	0
r3save		.fill 	0
```
- however, the subroutines will use a lot of memory space if every subroutine declares space to save registers
	- ex. if a subroutine calls itself

## Saving Registers In A Subroutine - Stack Solution
- the stack
	- last in first out (LIFO)
	- supports push and pop operations
- the stack is located in some designated area of memory
- the address of the top of the stack is stored somewhere
- the stack can grow in either direction (up or down)

## Top Of The Stack
- we only need to keep track of
	- the top of the stack
	- the memory location where the last value was pushed onto the stack
	- which is also the next thing to pop off the stack
- let's use R6 to store the "stack pointer"

## Where Do We Put The Stack?
<img src="img/l10-stack-location.png" alt="stack-location" width="300">

- there is a convenient spot to put the stack in the LC-3
- since a heap grows up, let the stack grow down to lower memory addresses
- Note: memory addresses are listed low to high on the page (refer to the down direction as high to low)

## A Software Stack Convention
<img src="img/l10-software-stack-convention.png" alt="software-stack-convention" width="600">

## Push And Pop
- push
	- decrement stack point (our stack is growing down to lower memory addresses)
	- then write data in R0 to the new top of stack

```assembly
PUSH		ADD	R6, R7, #-1
			STR	R0, R6, #0
```

- pop
	- read the data at the current top of stack into R0
	- then increment stack pointer

```assembly
POP			LDR	R0, R6, #0
			ADD R6, R6, #1
```

## Running Out Of Memory
- what if a stack is already full or empty?
	- before pushing, we could test for overflow
	- before popping, we could test for underflow
- what happens when the stack pointer runs into the heap?
	- stack overflow

## How Does A Stack Help With Subroutines?
- we can push R7 onto the stack (to save the subroutine return address)
- we can push each of the registers onto the stack, to save copies of their old values
	- then we can borrow those registers to reuse inside our subroutine
- when our subroutine is done, just pop the return address and original registers back off the stack

## What Else Might A Subroutine Need To Know, Or Store?
- we're already going to save on the stack:
	- R7 (return address)
	- copies of registers
- what else do we need?
	- arguments
	- return value
	- local variables

# The LC-3 Calling Convention
## Stack Frame
- **stack frame**: all of the data on the stack, relevant to our subroutine
	- arguments
	- return value
	- local variables
	- saved return address (R7)
	- saved registers

## Frame Pointer
- keep a "known" location in a stack frame in another register (R5)
- R5 always points to the first local variable
	- we always reserve space for at least one local variable, even if we don't have any local variables
- the frame pointer (R5) is an anchor

<img src="img/l10-subroutine-ex.png" alt="subroutine-ex" width="450">

<img src="img/l10-subroutine-stack-frame-ex.png" alt="subroutine-stack-frame-ex" width="450">

## Special Purpose Registers
- R5 - frame pointer
- R6 - stack pointer
- R7 - return address
- Note: for subroutines, only use R0-R4 as general purpose registers

<img src="img/l10-general-stack-frame-ex.png" alt="general-stack-frame-ex" width="550">

## Definitions
- **stack frame/activation record**: the collection of all data on the stack associated with one subprogram call
	- generally includes the following components
		- return address
		- argument variables passed on the stack
		- local variables
		- saved copies of any registers modified by the subroutine that need to be restored
- **caller**: the code that will call a subroutine; responsible to push arguments onto stack frame
	- ex. ```m = mult(a,b)```
- **callee**: the subroutine definition; responsible for everything from return values to saved registers on stack frame
	- ex. ```int mult(a,b) { ... }```

## Caller
- what do we do to call a subroutine?
	- push arguments (in reverse order)
	- JSR
- what do we do after subroutine returns?
	- pop the return value
	- pop the arguments

<img src="img/l10-stack-changes-multiply-ex.png" alt="general-stack-frame-ex" width="600">

## What Is The Effect Of JSR?
- a subroutine effectively pushes its return value on the stack

## Assembly - Caller Of MULT(A,B)
```assembly
;let’s call mult
;M = mult(A,B);
;assume M, A, B, MULT are labels

;push arguments in reverse order
;push B
LD	R1, B
ADD	R6, R6, #-1
STR	R1, R6, #0

;push A
LD	R1, A
ADD	R6, R6, #-1
STR	R1, R6, #0

;call mult
JSR MULT

;after MULT returns
;pop the return value
LDR	R1, R6, #0
ADD	R6, R6, #1

;save the ret val in M
ST	R1, M

;pop the two args, A and B
ADD	R6, R6, #2
```

## Stack Frame - Callee
- when a subroutine is called (stack buildup)
	- save some registers (callee-saved)
	- make room for local variables
- do the work for the subroutine
- when procedure is about to return (stack teardown)
	- prepare return value
	- restore registers
	- RET

## Stack Frame Progression
<img src="img/l10-stack-frame-progression.png" alt="stack-frame-progression" width="600">

## LC-3 Calling Convention
```y = foo(a, b, c);```

|caller/callee|task|
|-|-|
|caller|push args onto stack right to left|
|caller|jump to subroutine|
|callee|decrement SP to leave four slots (ret value, ret addr, old FP, local var)|
|callee|save copy of R7 (ret addr), and copy of R5 (old FP)|
|callee|save callee - set R5 (frame pointer) to be R6 (stack pointer)|
|callee|allocate space (SP) for local variables & saved registers (R0-R4)|
|callee|save registers R0-R4 used by the function|
|callee|execute the code in the function|
|callee|save the ret val (at R5 + 3)|
|callee|restore saved registers (R0-R4)|
|callee|set SP to FP, to pop off local vars and saved registers|
|callee|restore the ret addr (to R7), and old FP (to R5)|
|callee|pop off 3 words (ret addr, old FP, first local var)|
|caller|grab the ret val|
|caller|deallocate space for ret val and args|

## 3 Oddities
- we always allocated space for no less than one local variable even when we don't need any
- the caller always pushes N words of arguments, but always pops N + 1 words (which includes the return value)
- the callee pushes M words of the stack frame, but pops M-1 to leave the return value on top of the stack

## Is The Stack Frame Same For Every Subroutine
- if you only have one local variable, your stack frame will always look like prior examples
- if you have two or more local variables, you will need to allocate more space and move the saved register locations up to make space for additional local variables

## Saving Registers
- you always save R7 (old return address) and R5 (old frame pointer)
- always save R0-R4 (five general purpose registers) on the stack frame

## Example: `mult(int a, int b)`
```assembly
;	int mult(int a, int b) {
;		int answer = 0;
;		while (a>0) {
;			answer += b;
;			a--;
;			return answer;
;		} 


;caller
;push arguments in reverse order
;mult(x,3)

		AND	R0, R0, 0		;push(3)
		ADD	R0, R0, 3
		ADD	R6, R6, -1
		STR	R0, R6, 0
		LD	R0, X			;push(x)
		ADD	R6, R6, -1
		STR	R0, R6, 0
		JSR	MULT			;mult(x,3)


;<========== START: SUBROUTINE MULT ==========>
;callee – the subroutine itself
;first, lay out our stack frame
;this is always the same for 2 args, 1 local variable
;and 5 saved registers
;for other counts, just adjust the amount of stack space as needed

MULT	ADD	R6, R6, -4		;push 4 wds
							;set rv later
		STR	R7, R6, 2		;save RA
		STR	R5, R6, 1		;save old FP
							;set local var later
		ADD	R5, R6, 0		;FP = SP
		ADD	R6, R6, -5		;push 5 words	
		STR	R0, R5, -1		;save SR1
		STR	R1, R5, -2		;save SR2
		STR	R2, R5, -3		;save SR3
		STR	R3, R5, -4		;save SR4
		STR	R4, R5, -5		;save SR5
		

;finally – we can do the work of mult()

AND		R0, R0, #0			;R0 = 0
		STR R0, R5, 0		;answer = R0
W1		LDR R0, R5, 4		;R0 = a
		BRnz END_W1	
		LDR R0, R5, 0		;R0 = answer
		LDR R1, R5, 5		;R1 = b
		ADD R0, R0, R1		;R0 = answer + b
		STR R0, R5, 0		;answer = R0
		LDR R0, R5, 4		;R0 = a
		ADD R0, R0, #-1		;R0 = a-1
		STR R0, R5, 4		;a = a-1
		BR W1				
END_W1	LDR R0, R5, 0		;R0 = answer
		STR R0, R5, 3		;set ret val to answer
		

; We need to tear down the stack frame
		LDR	R4, R5, -5		; restore R4
		LDR	R3, R5, -4		; restore R3
		LDR	R2, R5, -3		; restore R2
		LDR	R1, R5, -2		; restore R1
		LDR	R0, R5, -1		; restore R0
		ADD	R6, R5, 0		; pop saved regs,
							; and local vars
		LDR	R7, R5, 2		; R7 = ret addr
		LDR	R5, R5, 1		; FP = Old FP
		ADD	R6, R6, 3		; pop 3 words
		RET					; mult() is done!

;<========== END: SUBROUTINE MULT ==========>


;pop the return value off the stack
		LDR R0, R6, 0		;R0 = return value
		ADD R6, R6, 1
		;save return value at label m
		ST R0, m
		;pop the arguments off the stack
		ADD R6, R6, 2
```