# Struct Delcaration/Definition
```c
struct [ <optional tag> ] [ {
	<type declaration>;
	<type declaration>;
	...
} ] [ <optional variable list> ];
```

- struct declarations that have a member list in curly braces define a new type (specifically a struct type)
- if the `<optional tag>` is included, it creates an unamed struct type that is different from every other struct type
- if the `<optional variable list>` is included, you are DECLARING and DEFINING a struct
- struct tags are in a separately-scoped name space from variables
  - ex. a struct variable can have the same name as a struct tag without causing confusion
- struct member names are in yet another name space local to the struct type
  - ex. every struct type could have a member named `next`

# Initializing A Struct
```c
struct mystruct_tag {
	int myint;
	char mychar;
	char mystr[20];
};
```
- you can initialize a struct with `struct mystruct_tag ms = {42, 'f', "goofy"};`
  - Note: make sure to specify the data is in the correct order
  - Note: the definition of a struct occurs at compile time and is the only time where `"goofy"` is "being assigned to" an array type (normally you cannot assign anything to an array type!)

```c
struct mystruct_tag {
	int myint;
	char mychar;
	char mystr[20];
}, ms;

// Q: Is the following legal?
// A: The following is illegal because mystr is an array name (aka a constant pointer to the array). Assigning anything to an array name is illegal (ex. arrayName = <anything>

ms.mystr = "foo";
```
- Note: however `ms.mystr[0] = 'f'` is allowed
	- because you can change the contents of the array, but never the pointer to the array

# Example: Copying Arrays
```c
struct s {
  int i;
  char c;
} s1, s2;

s1.i = 42;
s1.c = 'a';

// makes a clone of s1 and stores it in s2
s2 = s1; 
s1.c = 'b';

// s2.i contains 42
// s2.c contains a
```

```c
struct s {
  int i;
  char c[8];
}, s1, s2;

s1.i = 42;
strcpy(s1.c, "foobar");
s2 = s1;
// s2.i contains 42
// s2.c contains 'f', 'o', 'o', 'b', 'a', 'r'
```

- Note: `s2.c = s1.c` is illegal. You either need to copy structs to copy the string over or just use `strcpy();`

# Where Do Struct Members Get Stored In Memory?
```c
struct a {
	char mychar;
	int myint;
	char mystr[19];
} mystruct;
```

|          |          |           |          |
| -------- | -------- | --------- | -------- |
| mychar   | \<filler\> | \<filler\>  | \<filler\> |
| myint    | ...      | ...       | ...      |
| mystr[0] | mystr[1] | mystr[2]  | mystr[3] |
| mystr[4] | ...      | ...       | ...      |
| ...      | ...      | ...       |          |
| ...      | ...      | ...       | ...      |
| ...      | ...      | mystr[18] | \<filler\> |

- structs are usually filled out to meet the most stringent alignment of its members
- in the example, `sizeof(mystruct)=28`
- the compiler keeps track of the offsets of each member in a table for each struct

| member | offset                                                     |
| ------ | ---------------------------------------------------------- |
| mychar | 0                                                          |
| myint  | 4                                                          |
| mystr  | 8 (sum of sizes of all previous elements including filler) |

## Example: Calculating Location In Struct
```c
struct a {
	char mychar;
	int myint;
	char mystr[19];
} mystruct;
```

- `mystruct` is located at location 1000
  - address of `mystruct.mychar` is 1000
  - address of `mystruct.myint` is 1004
  - address of `mystruct.mystr` is 1008

|                                                              |                |
| ------------------------------------------------------------ | -------------- |
| base address of struct                                       | &mystruct      |
| offset of mystr within structoffset of element 4 within mystruct.mystr | 8              |
| offset of element 4 within mystruct.mystr                    | 4              |
|                                                              | &mystruct + 12 |

- what is the address of `mystruct.mystr[4]`?
  - if `mystruct` is located at location 1000, then the address of `mystruct.mystr[4]` is 1012

## Example: Calculating Location In Array Of Structs
```
struct a {
	char mychar;
	int myint;
	char mystr[19];
} mystruct;

struct a astruct[25];
astruct[6].mystr[3] = 'y';
```

|                                           |                 |
| ----------------------------------------- | --------------- |
| base address of struct                    | &mystruct       |
| offset of element 6 of mystruct           | 6 * sizeof(a)   |
| offset of mystr within struct             | 8               |
| offset of element 3 within mystruct.mystr | 3               |
|                                           | &mystruct + 179 |

what is the address of `astruct[6].mystr[3]`?

- Note:`sizeof(a)=28`

- if `astruct` is located at location 1000, then the address of `astruct[6].mystr[3]` is 179

## Example: Advanced Calculation Of Location
```c
static struct {
	int n;
	char m[3];
	double p;
} s[12];

// member offset
// n: 0
// m: 4
// p: 8

// struct size = 16
```

- if `s` is stored at memory address `0x0e3c`, where is `s[5].m[2]` stored?
  - `s[5]` is stored at `0x0e3c + 0x005c = 0x038c`
  - `s[5].m` is stored at `0x038c + 0x0004 = 0x0e90`
  - `s[5].m[2]` is stored at `0x0e90 + 0x0002 = 0x0e92`

## Structures May/May Not...
- structures may:
  - be copied or assigned
  - have their address taken with &
  - have their members accessed
  - be passed as arguments to functions
  - be returned from functions
- structures may not be compared