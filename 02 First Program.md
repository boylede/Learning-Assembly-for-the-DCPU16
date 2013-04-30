# Using Hardware
Now that the basic terms are defined, lets write an actual program. In order to make a "hello world" program, you need to write things to the screen. Lets start by initializing the screen.

## Initialize Program
Lets write a program that can check to see what hardware is connected to the DCPU and remember it for later.
### The plan
we want to figure out how many peices of hardware are connected, and then ask each peice of hardware what it is, and optionally do specific things with a piece of hardware when we've found it.

1. enumerate hardware
2. get info about each piece of hardware.
3. store the device id of the screen and keyboard.

The first part is easy, there is an opcode to do just that.

```
hwn i
```
This stores the number of connected devices in the register `i`.

### Looping
When the DCPU boots up, each piece of hardware is given an ID. If there are 7 peices of hardware, there will be 7 IDs and `hwn` will store 7 in `i`.

e.g.

```
0. A screen
1. A keyboard
2. a clock
3. disk drive
4. phasers
5. warp drive
6. security cameras
```
Hardware ID's start at 0, so the last peice of hardware has an ID of 6. We can easily talk to each peice of hardware by just counting down from the number that `hwn` produced.

```
:init_loop
sub i, 1
hwq i
```
`sub i, 1` subtracts `1` from `i` each time the loop is run. `hwq i` queries the piece of hardware which has ID `i` and sets the `a`,`b`,`c`,`x`, and`y` registers with information about it.

Later we will set pc back to `init_loop`, but first we need a way to decide when to get out of the loop.

### Conditionals 
The DCPU provides a few different opcodes which are usefull for controlling program flow. 

These operations will either skip or execute the operation which follows them.

```
ife 	; if equal
ifn		; if not equal

ifg		; if greater than
ifl		; if less than
	
; among others...
```

You use them like this:


```
ifn i, 0
	set pc, init_loop
```
Which means "if i is not equal to 0, set pc to init_loop".
It is good practice to indent the operation that will be skipped, for better readability.


### Globals

Now we have a loop, and it gets some values pertaining to each peice of hardware, but it doesn't do anything with them.

What we really want to do is be able to refer to the screen or the keybaord directly, without knowing its id while we are coding. 

```
hwi "the screen"
```

In order to do that, we can use a label to reference a specific chunk of data and store the device ID there. 

Sometimes I call these Globals because they are kind of like global variables, any subroutine can access them without having to keep them in a specific register all the time.

```
; if this device is the screen:
set [screen], i
; if this device is the keyboard:
set [keyboard], i

; . . .

:screen
dat 0 		; "dat" is a keyword to insert the given value at compile time
:keyboard
dat 0 		; it isn't a valid operation, so dont get them mixed up with your code
```

Just as in the addressing section, when you use square-brackets `[ ]` around a number, the DCPU will interpret the number as an address in memory and access the value at that address. Since labels are translated into a number at compile time, you don't need to keep track of the specific address.

Now we can use conditionals to check if this peice of hardware is one we want to keep track of. We know from the spec that each peice of hardware will report a given manufacturer and product id, so we can check that against what we are given from `hwq`.

```
; I only check "a", but you could check each of abcxy.
ife a, 0x7406
	set [keyboard], i
ife a, 0xf615
	set [screen], i
```
Here's what this routine looks like as a whole:

```
	hwn i
:init_loop
	sub i, 1
	hwq i
	ife a, 0x7406
		set [keyboard], i
	ife a, 0xf615
		set [screen], i
	ifn i, 0
		set pc, init_loop
	set A, 0
	set B, [video]
	hwi [screen]
	
; ... somewhere else that the instruction pointer never reaches:
:screen
	dat 0
:keyboard
	dat 0
```

Now that we have the hardware IDs of the screen and keyboard tucked away where we can get them whenever we want, we can start communicating with the hardware.

###Talking to hardware

You can tell hardware that you want it to do something by sending it an interrupt, using "hardware-interrupt" `hwi`. The hardware can read the state of the registers to decide what you want it to do.

In order to use the screen we need to tell it to watch a part of memory. We will then write to that part of memory and the screen hardware will copy from it to draw things on the screen. If you set `a` to `0`, then run `hwi [screen]`, the screen will use the value of `b` for its memory.

```
set A, 0
set B, [video] ; I've put the value I want it to use in a global, too.
hwi [screen]
```

The screen accepts a handful of other values in `a`, which we'll explore more later.

```
a =		| 	effect:
	0	|	set screen memory area to b.
	1	|	set font memory area to b.
	2	|	set palette memory area to b.
	3	|	set the color of the screen border.
	4	|	write the internal font buffer to memory
	5	|	write the internal palette buffer to memory.
```
### Stoping execution
When we are making sample programs we want them to stop at a certain point. If you were to write a program that takes user input, you might not; In our sample program we want to stop now though. 

```
sub pc, 1
```
This will subract 1 from `pc`, but `pc` gets incremented every instruction so it will end up back here, looping forever. This is currently the way to stop execution. 

In total, our program looks like this:



```
	hwn i
:init_loop
	sub i, 1
	hwq i
	
	ife a, 0x7406
		set [keyboard], i
	ife a, 0xf615
		set [screen], i
	ifn i, 0
		set pc, init_loop

	set A, 0
	set B, [video]
	hwi [screen]

	sub pc, 1

:screen
	dat 0xffff
:keyboard
	dat 0xffff
:video
	dat 0x8000

```

Next chapter we will print to the screen.

