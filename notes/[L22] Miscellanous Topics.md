# Control Flow
- there is a goto in C but it is not recommended for several reasons
    - optimization
    - performance
    - clarity
- even worse... is using `setjmp.h`

```c
#include <setjmp.h>

// holds location x
jmp_buf x;
  
// ... some code

// marks this location as x
// t = 0 for normal control flow
// t = 3 if longjmp(x, 3) used to get to this line of code
t = setjmp(x);

// ... some more code

// transfers back to location x and returns 3
longjmp(x, 3);
```

# Variadic Functions
- **variadic function**: a function that takes in a variable number of arguments
- there must be at least 1 argument in the call and the function prototype
- it is the responsibility of the called function to figure out how many arguments were passed to it

```c
#include <stdarg.h>
void printargs(int arg1, ...);

int main(void) {
	printargs(5, 2, 14, 84, 97, 15, -1, 48, -1);
	printargs(84, 51, -1);
	printargs(-1);
	printargs(1, -1);
}

void printargs(int arg1, ...) {
  va_list ap;
  int i;
  
  va_start(ap, arg1);
  for (int i = arg1; i >= 0; i = va_arg(ap, int)) {
  	printf("%d", i);
  }
  
  va_end(ap);
  putchar('\n');
}

// OUTPUT:
// 5 2 14 84 97 15
// 84 51
//
// 1
```

- `printf()` prototype
    - `void printf(char *format, ...);`
    - Note: `printf()` and similar functions use the format string to determine how many arguments are present

# `main()` Return Value
-   `return 0`  = okay
-   `return <anythingButZero>` = problem
-   `main()` can return an 8-bit value
-   any function can exit and set the return value
    -   ex. `exit(99)` exits the function and returns 99
-   you can see the returned value on the command line with `echo $?`

# Idioms For Opening Files
-   in the C environment, there are three files opened for you on pre-defined "streams", typically connected to your keyboard/display
    -   stdin - standard input
    -   stdout - standard output (`printf(...)` defaults to stdout)
    -   stderr - standard error output (use `fprintf(stderr, ...)`)
-   any other files must be opened and closed by calling a standard IO routine

```c
// both the functions below do the same thing

FILE *infile;
if ((infile = fopen("f.txt", "r")) == NULL) {
	// handle error
}

// ---------------

FILE *infile;
if (!(infile = fopen("f.txt", "r"))) {
	// handle error
}
```

# Unformatted (Binary) I/O
```c
#include <stdio.h>

// read size * nmemb bytes from the stream
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);

// write size * nmemb bytes to the stream
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
```

# Buffered Output
-   the standard IO library provides buffering improvements to minimize the number of system call traps that need to be made to `read()` and `write()`
-   for the stdout, the output is line-buffered if it is written to a terminal and block-buffered if it is written to a disk file
    -   if your program crashes, the buffer contents may be lost, especially if the file has been redirected to a disk file instead of a terminal
    -   that means you can lose your debugging output from printf if you don't end it with a newline and don't allow it to go directly to your terminal
-   if you execute these printf statements just before a segfault, you may not see any output...

```c
printf("checkpoint 1...");

// ... some code

printf("checkpoint 2...");
```

-   you can force the output buffer to be flushed with `fflush()` but it is also buffered
    -   so if the program crashes, the buffered contents may be lost

```c
printf("checkpoint 1...");
fflush(stdout);

// ... some code

printf("checkpoint 2...");
fflush(stdout);
```

# `stderr` Output
-   Note: for stderr, the output is set to not buffered
    -   so output to stderr will always cause immediate I/O

```c
fprintf(stderrl, "checkpoint 1");

// ... some code

fprintf(stderr2, "checkpoint 2");
```

# Inline Functions
```c
inline int myfunc(int a, int b) {
	return a + b;
}
```

-   inline functions don't build a stack frame and don't jump to subroutine
-   inline functions simulate the actions of the function in the source code

# Optimization
-   you use the `-Olevel` optimization to control optimization
    -   0 means don't move code around (default)
    -   1 means quick optimizations that don't take much compilation time
    -   2 means all optimizations that don't involve speed/space tradeoff
    -   3 means even more optimization
-   the higher the optimization level, the more likely your code will not work properly if you don't follow the C rules precisely
-   for use with a debugger use `-O0` or `-O1`

# Writing Tricky/Efficient Code
-   **profiling**: instrumenting and measuring a running program to discover where the CPUs spend their time in order to find the most effective places to optimize the code
-   don't write tricky code designed to be very efficient
    -   you may just defeat the optimizer by hiding what you are trying to accomplish
    -   until you measure it, programmers rarely know where the CPU is spending its time in their code (premature optimization)
    -   if you are really trying to be efficient
        -   measure (aka profile) your program to see where it spends its time
        -   look at the optimized assembly language output (-S) for your hot spots to see how the code is being generated
        -   modify your C code to improve the optimized output
        -   measure it again

