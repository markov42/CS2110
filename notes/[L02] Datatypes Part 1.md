# Datatypes And Data Representations
- **data representation**: the set of values from which a variable, constant, function, or other expression may take its value and includes the meaning of those values; tells the compiler or interpreter how the programmer intends to use it
	- ex. the process and result of adding two variables differs greatly according to whether they are integers, floating point numbers, or strings
- **data type**: a particular representation if there are operations in the computer that can operate on information that is encoded in that representation

# Bits
- **bit (binary digit)**: takes exactly two values 0 and 1
- binary is the most effective way to store and calculate using today's electronics
- in general, n bits can represent <img src="https://render.githubusercontent.com/render/math?math=2^n"> different states

# Binary Tricks
```
128    64    32    16    8    4    2    1
0      1     1     1     1    1    1    1

0111 1111 = 127
```

- `01111111` has a string of seven 1's (starting from the first power of 2 less than 127), we know that `01111111` is just `128 - 1 = 127`

# Signed Bits
- first bit (left most bit) denotes sign
    - `+` is denoted by 0
    - `-` is denoted by 1

# 1’s Complement
- flip all bits (1 becomes 0, 0 becomes 1)
- bad because we waste a spot for -0

# 2’s Complement
- to negate (make positive to negative or negative to positive)
	- flip the bits, add one
- you know a 2’s complement number is negative if there is a “1” on the left-most bit
- given a negative number in 2’s complement and convert to decimal
	- flip the bits, add one, convert to decimal, take the negative of the result

# Representations Of Binary Number In Other Forms
- for n=4, the following are the representations of the binary number in other forms

| binary | unsigned | signed | 1's complement | 2's complement |
| ------ | -------- | ------ | -------------- | -------------- |
| 0000   | 0        | 0      | 0              | 0              |
| 0001   | 1        | 1      | 1              | 1              |
| 0010   | 2        | 2      | 2              | 2              |
| 0011   | 3        | 3      | 3              | 3              |
| 0100   | 4        | 4      | 4              | 4              |
| 0101   | 5        | 5      | 5              | 5              |
| 0110   | 6        | 6      | 6              | 6              |
| 0111   | 7        | 7      | 7              | 7              |
| 1000   | 8        | -0     | -7             | -8             |
| 1001   | 9        | -1     | -6             | -7             |
| 1010   | 10       | -2     | -5             | -6             |
| 1011   | 11       | -3     | -4             | -5             |
| 1100   | 12       | -4     | -3             | -4             |
| 1101   | 13       | -5     | -2             | -3             |
| 1110   | 14       | -6     | -1             | -2             |
| 1111   | 15       | -7     | -0             | -1             |

- unsigned 4-bit binary number ranges from `0` to `15`
- signed 4-bit binary number ranges from `-8` to `7`
- unsigned n-bit binary number ranges from `0` to <img src="https://render.githubusercontent.com/render/math?math=2^n - 1"> 
- signed n-bit w/ 2's complement ranges from <img src="https://render.githubusercontent.com/render/math?math=-2^{n-1}"> to <img src="https://render.githubusercontent.com/render/math?math=2^{n-1} - 1">

# 2's Complement Addition (No Overflow)
- Note: anytime you see subtraction `a-b` convert it to `a + (-b)`
- Note: if you ever need to carry past the specified number of bits, DISCARD IT
```
5 - 2
convert to 5 + (-2)

 2 = 0010
-2 = 1110	(flip the bits and add 1)

       1100
 5      0101
-2      1110
---     -----
 3      0011
```

```
5 - (-1)
convert to 5 + 1

     0001
5     0101
1     0001
--    -----
6     0110
```

```
What is the sum of 101111 and 001010 (in base 2)?

001110
 101111
 001010
 ------
 111001
```

# 2's Complement Addition (Overflow)
- check carry in and out of the leading digit (left most digit)
- overflow occurs when 
	- there is a carry into the sign bit but no carry out
	- there is a carry out of the sign bit but no carry in
- adding two positive numbers overflows if you carry into the sign
- adding two negative numbers overflows if you don't carry in and out of the sign
- adding two opposite signed numbers NEVER OVERFLOWS

```
Adding two positive numbers (no overflow since carry-in and carry-out are both 0).
3 + 3

     0011
3     0011			
3     0011
--    -----
6    00110
```

```
Adding two positive numbers (overflow due to carry-in = 1).
4 + 4

     0100
4     0100			
4     0100
--    -----
8    01000
```

```
Adding two negative numbers (no overflow since carry-in and carry-out are both 1).
-5 + (-3)

       1111
-5      1011			
-3      1101
--      -----
-8     11000

Note: throw out the extra 1 (so solution is 1000)
```

```
Adding two negative numbers (overflow due to carry-in = 0 but carry-out = 1)
-4 + (-5)

       1000
-4      1100
-5      1011
---     -----
-9     10111

Note: throw out the extra 1 (so solution is 0111)
```

# Sign Extension
- to add bits to the left of a 2's complement number
	- fill in new bits on the left with the value of the sign bit
- ex. `1101` => `1111 1101`
- ex. `0011` => `0000 0011`

```
Adding numbers with different number of bits (SIGN EXTEND)
-3 + 64

 -3    11111101
+64    01000000
---    --------
61     00111101
```

# Adding A Number To Itself
- <img src="https://render.githubusercontent.com/render/math?math=A"> + <img src="https://render.githubusercontent.com/render/math?math=A\iff 2A\iff A\ll 1">
  - <img src="https://render.githubusercontent.com/render/math?math=\ll"> is a left-shift operator
- <img src="https://render.githubusercontent.com/render/math?math=A\ll n \iff A^n"> 

# Fractional Binary Numbers
```
8   4   2   1   .5   .25   .125   .625
1   0   1   0    1    1     0      0

1010.1100 (base 2) = 10.75 (base 10)
```
