# Complementary Transistors
- n-Type MOS
	- switch is normally open
	- applying current closes the switch
- p-Type MOS
	- switch is normally closed
	- applying current opens the switch

<img src="img/l04-transistors.png" alt="transistors" width="400">

- a wire with some designated voltage (ex. +9.0V) can represent a logical 1
- a wire with some designated voltage (ex. 0V or ground) can represent a logical 0
- a wire that is not connected to 9.0V or ground is in a floating or high impedance state (value randomly varies from 0 to 1)

<img src="img/l04-transistors-voltage.png" alt="transistors-voltage" width="400">

<img src="img/l04-logical-operators.png" alt="logical-operators" width="400">

# Logic Gates
## De Morgan's Law
- (A' & B')' = A | B
- (A' | B')' = A & B

##  Conte Bubble Theorem
<img src="img/l04-conte-bubble-theorem.png" alt="conte-bubble-theorem" width="400">

- flip gates (OR -> AND, AND -> OR)
- flip inputs and outputs by adding or removing a bubble (NOT)

## Larger Gates
<img src="img/l04-larger-gates.png" alt="larger-gates" width="400">

- (AB)(CD) is faster than (A(B(CD)))

# Combinational Logic
- a combination of AND, OR, NOT (plus NAND and NOR)
- Note: the same inputs produce the same output

## Decoder
- will turn on at most 1 output, based on which input bits are set
- `n` input bits will produce <img src="https://render.githubusercontent.com/render/math?math=2^n">  outputs for a decoder
- fewer inputs than outputs

<img src="img/l04-decoder.png" alt="decoder" width="400">

## Multiplexor (MUX)
- a multiplexor has:
	- <img src="https://render.githubusercontent.com/render/math?math=2^n"> inputs
	- `n` selector bits (aka control inputs)
	- 1 output
- selects between inputs using a selector
- more inputs than outputs

<img src="img/l04-multiplexor.png" alt="multiplexor" width="400">

- bubble = 0
- wire = 1
- each gate of the decoder is "represented" by a binary number using bubbles and wires

## Demultiplexor (DEMUX)
- sends the input across exactly one of the output lines
- other outputs remain 0
- a demultiplexor has:
	- <img src="https://render.githubusercontent.com/render/math?math=2^n"> ouptuts
	- `n` selector bits
	- 1 input
- more outputs than inputs

<img src="img/l04-demultiplexor.png" alt="demultiplexor" width="400">

## Simple Adder
- sum -> use XOR gate

## Half-Adder
- sum -> use A XOR B
- carry out -> use A AND B

## Full-Adder
- add carry-in bits

<img src="img/l04-full-adder-truth-table.png" alt="full-adder-truth-table" width="450">
<img src="img/l04-full-adder-circuit.png" alt="full-adder-circuit" width="450">

# Simplification
## Boolean Simplification
<img src="img/l04-boolean-simplification.png" alt="boolean-simplification" width="700">

## Classic Sequence vs. Gray Code Sequence
- gray code sequence never has two switches changing at the same time

|Classic Sequence|Gray Code Sequence|
|-|-|
|000|000|
|001|001|
|010|011|
|011|010|
|100|110|
|101|111|
|110|101|
|111|100|

- gray code also allows you to loop around from the end to beginning (100 -> 000)

## Karnaugh Map
- a method of simplifying boolean expressions by grouping together related terms
	- two adjacent 1's in the map means there is a `x+x'` in the formula
- results in the simplest sum-of-products expression possible
- allows for "don't care" outputs

|A|B|C|func(A,B,C)|
|-|-|-|-|
|0|0|0|X|
|0|0|1|0|
|0|1|0|0|
|0|1|1|1|
|1|0|0|1|
|1|0|1|1|
|1|1|0|0|
|1|1|1|1|

1. Create the K-Map
	- using a truth table distribute variables across rows and columns using gray code order
	- fill in corresponding entries

|.|AB|AB'|A'B'|A'B|
|-|-|-|-|-|
|**C**|1|1|0|1|
|**C'**|0|1|X|0|

2. Make groupings
	- grouping rules
		- groups must be rectangular (may wrap around edges!)
		- groups may only contain 1s or Xs
		- all 1s must be contained within at least one group
		- groups must be as large as possible
		- the size of a group must be a power of 2
		- overlaps are allowed

|.|AB|AB'|A'B'|A'B|
|-|-|-|-|-|
|**C**|1*|1**|0|1*|
|**C'**|0|1**|X|0|

`*` denotes one group
`**` denotes another group

3.  Write the simplified equation
	- pull out the simplified expression based on K-Map groupings
		- (ABC + A'BC) + (AB'C + AB'C')
		- BC + AB'

## Programmable Logic Array/(Field) Programmable Gate Array (PLA/PGA/FPGA)
- given a truth table, we can implement its outputs using a series of NOT gates, AND gates, and then OR gates
- we need <img src="https://render.githubusercontent.com/render/math?math=2^n"> AND gates where `n` is the number of inputs
- we need 1 OR gate for each output
- PLA/PGA/FPGA devices exist for this reason