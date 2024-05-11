# Unions
```c
union {
  int myint;
  char mychar;
  char mystr[20];
} myun;
```

- similarity with structs
  - declaration and definition is the same
  - access is the same
- differences with structs
  - all the members have an offset of zero

## Unions And Base + Offset
- the compiler keeps track of offsets for each member of the struct in a symbol table

| member | offset |
| ------ | ------ |
| myint  | 0      |
| mychar | 0      |
| mystr  | 0      |

- assuming `mystruct` is located at memory address 1000, what will be the address of `myint`, `mychar`, and `mystr`?
  - they all have the address of 1000

## Why Unions?
- suppose we want to store information about athletes
  - for all we want:
    - name, jerseyNum, team, sport
  - for football players we want:
    - attempts, yards, touchdowns, interceptions
  - for baseball players we want:
    - wins, losses, innings, strikeouts
  - for basketball players we want:
    - shots, assists, rebounds, points

```c
struct player {
	char name[20];
	char jerseyNum[4];
	char team[20];
	int playerType;
	union sport {
		struct football { ... } footballstats;
		struct baseball { ... } baseballstats;
		struct basketball { ... } basketballstats;
	} thesport;
} theplayer;

theplayer.thesport.footballstats.touchdowns = 3;
```

- often used in implementing polymorphism found in object-oriented languages

## Unions May/May Not
- unions may:
  - be copied or assigned
  - have their address taken with &
  - have their members accessed
  - be passed as arguments to functions
  - be returned from functions
  - be initialized (but only the first member)
- unions may not be compared

---
# Function Pointers
- to store the address of a function, we use a pointer (similar to addresses of variables)

```c
// function that returns an int
int fi(void);

// function that returns a pointer to an int
int *fpi(void);

// pointer to a function that returns an int
int (*pfi)(void);
```

```c
// legal assignment
pfi = fi;

// illegal assignment
pfi = fi();
```

## Why Function Pointers?
- ex. you are writing a general purpose sorting functions and want it to be able to sort anything (numbers, strings, structs, unions, etc.)
  - comparing numbers, strings, and other stuff calls for at least two different techniques
  - function pointers allow you to write functions that do the comparison we need
    - a function to compare numbers
    - a function to compare strings
    - a function to compare other stuff
  - now when we call the function to do the sorting, we pass in a pointer to the appropriate function for the type of data we have
- Note: a pointer to a function generally stores the address of the first word of the function code

## `<stdlib.h>` Quicksort
```c
#include <stdlib.h>
void qsort(void *base, size_t nmemb, size_t size, int (*compar)(const void *, const void*));
```

- base: memory address where the array starts
- nmemb: length of the array
- size: size of a single element (in bytes)
- Note: to get documentation for an imported library's method, run `man <methodName>` in the terminal

## Comparison Functions With Quicksort
```c
#include <stdio.h>
#include <stdlib.h>
#define MAX 100

int compar_ints(const void *pa, const void *pb) {
	return *((int *)pa) - *((int *)pb);
}

int compar_strings(const void *ppa, const void *ppb) {
	return strcmp(*((char **)ppa), *((char **)ppb));
}

int main(int argc, char **argv) {
	char *strings[] = {"dec", "sun", "ibm", "apple", "hp", "ti", "univac"};
	int i, s;
	int a[MAX];
	if (argc == 2 && *(argv[1]) == 'a') {
		s = sizeof(strings) /sizeof(strings[0]);
		qsort(strings, s, sizeof(strings[0]), compar_strings);
		for (i = 0; i < s; i++) {
			printf(" %s", strings[i]);
		}
		printf("\n");
	} else {
		for (i = 0; i < MAX; i++) {
			a[i] = rand() % 100;
			printf(" %d", a[i]);
		}
		printf("\n\n");
		qsort(a, MAX, sizeof(int), compar_ints);
		for (i = 0; i < MAX; i++) {
			printf( "%d", a[i]);
		}
	}
	return 0;
}
```