# Recursion
- using a stack gives us recursion for free
- the following sections of code for subroutines stay the same for recursion:
	- initial code (push arguments and JSR, make room for local variables, save registers)
	- ending code (prepare return value, restore registers, clean up stack, RET)
- the body code just contains a recursive call to the subroutine
- if you use a well-designed calling sequence, a recursive function call looks just like any other function call; a recursive function looks just like any other function