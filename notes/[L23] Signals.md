# IPC Mechanism
-   **IPC mechanism signal**: tells a process that some event occurs
    -   occurs in:
        -   the kill command
            -   `kill -l` (lists all 64 kill signals)
            -   `kill -s INT ####(pid)`
        -   when `ctrl-C` is typed (SIGINT)
        -   when `ctrl-\` is typed (SIGQUIT)
        -   when a child exits (SIGCHILD to parent)
        -   when a timer expires
        -   when a CPU execution error occurs
        -   etc.
    -   a form of interprocess communication

# Available Actions And Signals
-   when a process receives a signal, it performs one of the following three options
    -   ignore the signal
    -   perform the default operation
    -   catch the signal (perform a user defined operation)
-   commonly used signals (all defined in `signal.h`)
    -   SIGABRT
    -   SIGALRM
    -   SIGCHILD
    -   SIGHUP
    -   SIGINT
    -   SIGUSR1
    -   SIGUSR2
    -   SIGTERM
    -   SIGKILL
    -   SIGSTOP
    -   SIGSEGV
    -   SIGILL

# Processing Signals
-   similar to an interrupt (software interrupt)
-   when a process receives a signal:
    -   pause the execution
    -   call the signal handler routine
    -   continue execution
-   signal can be received at any point in the program
-   most default signal handlers will terminate the program
-   you can change the way your program responses to signals
    -   ex. make `ctrl-C` have no effect

# Simplified Signal Interface
-   ANSI C signal function to change the signal handler
-   syntax (both code segments serve the same purpose):

```c
#include <signal.h>
void (*signal(int sig, void *(disp)(int)))(int);
```

```c
#include <signal.h>
typedef void (*signalhandler)(int);
sighandler signal(int sig, sighandler disp);
```

-   semantics:
    -   `sig` - signal (defined in `signal.h`)
    -   `disp` - SIG_IGN, SIG_DFL or the address of the signal handler
    -   handler may be reset to SIG_DFL after one invocation
        -   AT&T UNINX does the reset, but BSD UNINX does not do the reset
    -   using the signal with a handler function isn't portable (use `sigaction(2)` instead)
-   a call to `signal()` in C establishes the behavior for the process when it receives a particular signal

# A Non-Portable Example Using `signal(2)`
```c
#include <signal.h>

void sigcatcher(int);
void sigexiter(int);

int main(intn argc, char *argv[]) {
	signal(SIGINT, sigcatcher);	# ctrl-C
	signal(SIGQUIT, sigcatcher);	# ctrl-\
	signal(SIGTERM, sigexiter);	# "kill process-id"
	
	printf("my process id is %d\n", getpid());
	while (1) {
		printf("waiting for 30 seconds on a signal\n");
		sleep(30);
	}
}

void sigcatcher(int s) {
	signal(s, sigcatcher);
	printf("caught signal %d\n", s);
}

void sigexiter(int s) {
	printf("exiting on signal %d\n", s);
	exit(1);
}
```

- `printf()` is not signal safe and can cause a deadlock in the signal handler
    - however, `write()` is okay
- `signal()` resets the signal handler if the OS is one that resets it to default (C is one that resets to default)
    - as a result, there is a race condition if another signal comes in just before the `signal()` call, causing the default action (due to the reset) rather than calling the handler
- `sigcatcher` itself without the `()` is a pointer to a function

# Blocking Temporarily Suspends Signal Actions
- block/unblock signals

```c
// manipulate signal sets

#include <signal.h>
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signo);
int sigdelset(sigset_t *set, int signon);
int sigismember(const sigset_t *set, int signo);
```

```c
// manipulate signal mask of a process
// how: SIG_BLOCK, SIG_UNBLOCK, SIG_SETMASK

int sigprocmask(int how, const sigset_t *set, isgset_t *oset);
```

# Deferring A Signal Example
- for a critical region where you don't want a certain signal to be deferred, the program will look like this

```c
#include <signal.h>
sigset_t newmask, oldmask;
sigemptyset(newmask);
sigaddset(newmask, SIGINT);

sigprocmask(SIG_BLOCK, &newmask, &oldmask);
/* critical region */
/* code that should run before it gets interrupted */
/* ex. bank debit and credit for transactions should complete before interruption*/
sigprocmask(SIG_SETMASK, &oldmask, NULL);
```

- `newmask` includes signals that we want to block
- `oldmask` includes signals that were used before
- we need to block oldmask during the critical region and then restore it after the critical region

# Send A Signal
- kill
    - send a signal to a process
        - `pid > 0` = normal
        - `pid == 0` = all processes whose group ID is the current process' group id
        - `pid < 0` = all processes whose group ID = |pid|

```c
#include <signal.h>
#include <sys/types.h>
int kill(pid_t pid, intn signo);
```

# C Signal Characteristics Summary
- signals can be deferred until the program is ready to process them
- a process can send a signal to another process or to itself
- the receipt of a signal by the program can be ignored, can be processed according to a default rule, or can cause a function to be called

- Note: signals are processed whenever they are called
    - they are not just processed at entry and exits of C functions

# Example: Counting Child Processes
- keeping track of number of child processes in a shell
    - when a process exits, it sends a SIGCHILD to its parent

```c
#include <stdio.h>
#include <stddef.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <string.h>

int numofchild = 0;

// If we catch a SIGCHLD, decrement the number of active children
void sigchildhandler() {
  numofchild --;
  write(1, "Child exited\n", sizeof("Child exited\n"));
}

int main() {
  char cmd[1000], buf[1000], *argv[2];
  struct sigaction abc;
  int pid;
  
  // Set a sigchildhandler() to catch SIGCHLD signals
  abc.sa_handler = sigchildhandler;    sigemptyset(&abc.sa_mask);
  abc.sa_flags = 0;
  sigaction(SIGCHLD, &abc, NULL);
  
  while(1) {
    // Read in a command name to execute
    printf("<%d>", numofchild);
    fflush(stdout);
    while(fgets(buf, 100, stdin) == NULL);
    sscanf(buf, "%s", cmd);
    
    // Command is "quit"; exit if all children are complete     
    if (strcmp(cmd, "quit") == 0) {
      if (numofchild == 0) {
        exit(0);
      } else {
      	printf("There are still %d children.\n", numofchild);  
      }    
      // Execute a child process running the specified command
    } else if ((pid = fork()) == 0) {
      argv[0] = cmd;
      argv[1] = NULL;
      execv(argv[0], argv);
      exit(0);
      // If fork() doesn't fail, increment the number of children
    } else if (pid != -1) {
      numofchild ++;
    }
  }
}
```

- Note: if you are the parent, `fork()` returns the pid of your child
- Note: if you are the child, `fork()` returns 0
