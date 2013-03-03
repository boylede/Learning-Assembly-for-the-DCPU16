# Learning DCPU Assembly

## Requisite Knowledge 
It is not neccessary to understand binary completely to start out. All that you need to know is that binary is the only way computers think, and that they don't "know" that any data should be interpreted in a specific way other than how the programmer tells them to. Also, you must know that the smallest peice of data the DCPU deals with is 16 bits, which means that a peice of information can be any one of 65,536 values, and nothing else. This piece of data is called a ```word```.
 
## Instructions
Assembly is essentially a more easily human-readable representation of machine code (or machine code is a machine-readable representation of assembly). Each instruction in assembly tells the processor to do one specific operation. DCPU assembly differs from an actual assembly language in that the operations are all designed to be easy to use by humans, whereas actual modern processors have instruction sets that are designed to be written by compilers and are not very human-friendly.

In DCPU machine code, each instruction is stored in one to three ```words```, depending on complexity. The first part of the instruction is the opcode. The opcode tells the processor which operation it is going to do, and the rest of the instruction is concerned with what pieces of data the processer should be manipulating.

There are a number of options for telling the DPCU which data to use. There are 8 registers to store data in, and 65,536 ```words``` of memory. You can also use literal values (e.g. a number).

```
set a, 11
sub a, 4
```
These two instructions would each translate into a single ```word``` of machine code when compiled. The first one sets the value of the ```a``` register to 11. The operation is ```set``` and the two peices of data are the ```a``` register and the literal value 11.

The next operation is ```sub```, and as you might expect it instructs the processor to subtract 4 from ```a```. The result is stored in ```a```, because the DCPU always stores the result in the first value.

It is not important right now to know what exact numerical opcodes these instructions will compile to, but it is important to understand that each assembly instruction is just a human-readable way to represent what will be compiled to machine code.

## Registers
Registers are little peices of memory on the processor which are used to store values for immediate use. The dcpu has 6+2 general purpose registers and 4 special purpose registers.


###General Purpose Registers:

Because the DCPU is made to be fun and simplified for humans, every register is one word and can be manipulated directly, except ```IA``` which I'll talk about later. Actual CPU achitectures can have differently sized registers which are for specific things, and which are full of little rules and tricks.

```
A, B, C
X, Y, Z
I, J 		; these two are sort of unique, hence 6+2
```

Each of the general purpose registers can be used for any arbitrary usage that the programmer wants to, but remember that ```I``` and ```J``` are unique in that there are two instructions which affect them directly.
 
 ```
; sti means "set then increment". 
; std means "set then decrement".

; examples
sti A, 2 	; sets a to 2 and then increments I and J
std B, 1 	; sets b to 1 and then decrements I and J
```

### PC: Program Counter AKA Instruction Pointer
The special registers have specific uses. The most important to understand at first is ```PC```. This is the program counter, or instruction pointer, and it points to the location in memory where the current instruction is. After each instruction, ```pc``` is automatically incremented by 1 and then the instruction at that location is executed.

### SP: Stack Pointer
Like ```pc```, the stack pointer ```sp``` is used to keep track of a location in memory that the processor uses for other operations. In this case it is part of the stack, which is essentially an area of memory that values can be quickly stored in and retrieved from, but only in sequential order. We'll talk about the stack more later.

### IA: Interrupt Address
This is another pointer which we'll talk about later. Essentially a programmer can define a subroutine which gets run when hardware wants to talk to the DCPU or when another subroutine calls an Interrupt.

### EX: Excess Register
The exess register is used when a function overflows the 16 bit possible values. This is a very luxurious function of the DCPU because "real" architectures often only have 1 bit for overflow and it is shared between different uses. This provides 16 whole bits of extra information when an operation overflows.


## Pointers and Labels
In DCPU assembly, the ```[``` and ```]``` characters are used to enclose pointers. They mean that the value should be interpreted as a memory address, and that the value at that memory address is the one you are refering to. If that doesn't make sense, consider this  pretend map of some memory:

```
0000: 	0FF0 000F 0000 F000
0004: 	0012 0230 0205 07e0
0008:	0000 0001 0020 0000
000C: 	0300 4000 0050 0b00
``` 
I have represented memory in hexidecimal, which is often used in computing partly because it is more succinct than binary. Each four digit group represents one ```word``` of memory.

The values at the left are the offset, or the number of each ```word``` of memory. The first ```word``` is numbered 0, and up from there to the total of 65,536 possible on the DCPU. The offset isn't actually part of the memory, it is just used for visualization.

In this memory map, the pointer [0003] would point to the value 0xF000, which you can see  on the first line. You can see the the first value on the next line is at offset 0004, which makes sense because it is one after 0003. 


Consider the following snippet:

```
set [data], 5
sub pc, 1
data:
dat 0
```

Here, ```data``` is a label, it is defined with a colon ```:``` following a string. When the program is complied, labels are translated into the memory address of the instruction after a label is defined. In this example ```dat 0``` is the next instruction (dat tells the compiler to include some literal values in the program, in this case ```0000```). So the line ```set [data], 5``` means to set the value of memory at ```data``` to 5. We've used ```data``` as a pointer to "talk" to a specific peice of memory. 

Often in simple programs it is usefull to assign a specific piece of memory to store a given value which you can then refer to in code by its label. This is what we have done above, with the label ```data```. I sometimes call these labels "global variables" because they are at a predefined location in memory and can be refered to in any subroutine by name.


## Comments
Before we get to an example program, I want to mention comments.

In code, it is often usefull to annotate things with comments so that others, and even future you, can understand what the goal of a given subroutine is. This is done in DCPU-Assembly using semi-colons. Everything on that line after the semi-colon is ignored by the compiler.

```
; some random instructions:
set a, 4
sub i, 4
mul b, 2
; this is another comment
hwi 8
int 1 ; the comment doesnt have to be the first thing on the line.
rfi a
```

## An example program

````
	set a, 10
	set b, 0
loop:
	add b, a
	sub a, 1
	ifn a, 0
		set pc, loop
	sub pc, 1

````
In order to understand assembly you need to read through the program line by line. Hopefully, some of the instructions listed make sense even without learning the operations in advace, but lets talk about each line:

````
	set a, 10
	set b, 0
````
This is pretty self-explanatory: the ```a``` and ```b``` registers are set to some integer values, ```10``` and ```0```. These are represented in decimal, but they could be expressed in binary or hexidecimal also.

````
loop:
````
This line defines a label, named loop.

````
	add b, a
	sub a, 1
````
These two lines manipulate the registers we set up before. The first one adds the value of ```a``` into ```b```, the result is stored in ```b```. The next line reduces ```a``` by 1. 

````
	ifn a, 0
		set pc, loop
````
```ifn``` is part of a series of operations which control program flow, called conditional operators. These operators check the two values that are supplied, and will either skip the next operation or not.

```ifn``` checks to see if the first value is NOT equal to the second value. Since we set ```a``` to 10 and then subtracted 1, it will be 9 the first time it gets here. Since ```9 != 0```, the if statement will be true and the instruction after it **will** be executed. If it was false, it would skip the next instruction. The next instruction is indented in order to make it easier to read, but that is just for humans. The computer doesn't care how you indent things. 

```set pc, loop``` sets the instruction pointer to the label we set before, meaning that the next instruction that is evaluated will be ```add b, a```. This loop will be repeated until ```a == 0``` and then ```set pc, loop``` will be skipped, and ```sub pc, 1``` will be run.

```
	sub pc, 1
```
As you probably expect by now, this will subtract 1 from the instruction pointer. Each time the instruction pointer executes an instruction, it is automatically incremented so that it points to the next instruction to execute. Subtracting 1 undoes that addition, and ```pc``` will stay where it is forever. This is one way to stop execution. Currently, there is no real way to shut down the DCPU, so halting ```pc``` is the popular way to finish doing things.


## Hardware
Doing some basic math on the computer is all well and good, but lets explore talking with hardware.

On the DCPU, interfacing with hardware is done in a very consistant manner. First, you need to know what hardware is connected and what order it is connected in, so that your program can talk to each piece of hardware individually.

In order to do this, you first get the number of hardware devices connected,```hwn i```, and then iterate through them, talking to each one in turn to learn about it. ```hwq i``` will ask the hardware to return details about itself in the registers. ```hwi i``` will send a signal to the hardware called an Interrupt.

### Hardware Interrupts
Each piece of hardware is capable of talking directly with the DCPU's memory and getting the values of registers. The hardware is only allowed/supposed to do this when you ask it to by sending it an interrupt. 

Once you know which device you want to comminucate with you can use ```hwi``` to send it an interrupt.

```
; 	send an interrupt to the 21st piece of hardware connected
hwi 21
; 	send an interrupt to the i'th piece of hardware.
hwi i
```
Usually, the value of the ```a``` register, and possibly other registers, is set in order to communicate a certain intention to the piece of hardware.

Hardware specifications are posted to ```http:///www.dcpu.com```, and it is a good idea to peruse them in order to understand what kind of things you can do with hardware. All of the functionality is general enough to allow the programmer a fair amount of control over what is happening. This also means that not a whole lot is done for you, but that is OK as far as I am concerned.

## First Assignment
Design a command line shell that can read input commands, compare them to a list of built-in commands, and then redirect program flow to the desired command. 

This may seem rather daunting given the slim amount of information given above, but if you consider a multi-staged approach it is far from impossible.

### Initialize
Program an algorithm which analyzes the pieces of hardware that are connected and stores the number needed to communicate with each in a pre-set label. 
### Hello World
Add to the above the ability to write to the screen. 
### Text Edit
Add the ability for the user to type arbitrary strings to the screen, and erase without allowing the user to currupt memory! It is up to you whether the user can write more than one screen full of text, but make sure your subroutines prevent the user from accidentally overwriting memory outside the screen space.
### Command Interpet
Write an algorith that compares the input string to an internal list of strings until it finds a match, or fails. 
### Program Flow
Add the ability to redirect program flow to the desired command, and return to the shell when that "command" is finished.

## First Assignment Examples
Examples of each of the above milestones is included along with this document.


