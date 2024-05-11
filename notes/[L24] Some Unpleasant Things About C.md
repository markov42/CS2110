# (Im)Proper Bounds Checking
```c
char *read_data(int sockfd) {
	char *buf;

  // reads 32-bit data length from network stream
  int length = network_get_int(sockfd);
  
  if ((buf = (char *)malloc(MAXCHARS)) != NULL) {
    // malloc null check
  }

  if (length < 0 || length + 1 >= MAXCHARS) {
    free(buf);
	}
  
  if (read(sockfd, buf, length) <= 0) {
    free(buf);
  }
}
```

- Note: if length = <img src="https://render.githubusercontent.com/render/math?math=2^{32 (bits) - 1}-1"> (the max value of a 2s complement integer value), the bounds checking fails
    - the bounds checking line is essentially bypassed
    - the next if statement performing `read()` results in a buffer overflow which can be used by a hacker to take over the system

# Overflowing A Buffer (`strcat` Example)
```c
char *starcat (char *dest, const char *src) {
	char *d = dest
	
	while (*d) {
		++d;
	}
	
	while ((*d++ = *src++) != '\0');
	
	return dest;
}
```

- if `*dest` does not have enough space (aka buffer) to copy all the characters of `src` to `dest`, you get a buffer overflow and start writing into other areas of memory in the code
- once again, a hacker can take advantage of a buffer overflow to take over your system

# Dinosaurs That Should Be Extinct
- avoid `strcpy()`
    - use `strncpy(dst, src, size)`
- avoid `strcat()`
    - use `strncat(dst, src, size)`
- avoid `sprintf()`
    - use `snprintf(dst, size, format, ...)`
- avoid any string function that copies to a destination that doesn't have a size bound
- because C is open source, if there is no bounds checking, a hacker could theoretically write into memory (outside of the alloted buffer space for a string) and take over the entire system

# Smash Game
- if a function utilizes a do-while loop in a function, the hacker can take over  by passing enough values to get to and edit the return address of the stack frame

# How To Prevent Vulnerabilities
- bound check
- non-executable stack
    - hardware prevents you from jumping to another part of the stack because the hardware tells you where to execute code
- canaries (help from the compiler, stack teardown checks a canary to see if anything was corrupted)
    - terminator - use `\0`, or `-1` to make it hard to use `strcpy()`
    - random - memory allocator chooses a random number at startup
    - random XOR - XOR the metadata from a block into the random number