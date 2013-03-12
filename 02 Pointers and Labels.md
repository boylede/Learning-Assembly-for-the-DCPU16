



## Addressing
````
set a, 8 		; set the A register to the literal value 8
set a, [8]		; set the A register to the value in memory at address 0x0008
set a, [8+b]	; set the A register to the value in memory at address (0x0008 plus B)
set [a], 8		; set the value at the memory address that is in the A register to 8 
````
Addressing means how you refer to a specific peice of data. On the DCPU, data can either be stored in a register or stored in memory. Using pointers to address a peice of memory is simple enough, just enclose the memory address in square brackets ```[address]``` and the DCPU will read from or write to that address. 

In the third example above, the literal value 8 is added to the value stored in the ```a``` register to determine which memory address to read. (The value of the ```a``` register is not altered.) For example, this can be used in a loop to refer to one value after another in memory.

```
set a, 0
:loop
set [label+a], 0xe900
add a, 1
ifn a, 0xf
	set pc, loop
	sub pc, 1
```
This will set 16 **words** of memory to 0xe900, because 0xf is 15 in hexidecimal (and we started at 0).

This might be useful if we want to copy a string out to the screen. Let's say we have configured the screen to read from memory at 0x8000, and we want to print the string stored at the label "string" onto the screen in yellow text on a blue background.

```
set i, 0x8000
set j, string
:loop
bor [j], [color]
sti [i], [j]
ifn [j], 0
    set pc, loop
sub pc, 1

:color
dat 0xe900 
:string
dat "Hello World!", 0
```
The number 0xe900 represents yellow text on a blue background, we store it at a label so we can change it easily when we want, and the string too. Strings are often terminated with a 0 so that you know when the string is over without knowing how long it is in advance.


## Pointers and Labels
In DCPU assembly, the ```[``` and ```]``` characters are used to enclose pointers. They mean that the value should be interpreted as a memory address, and that the value at that memory address is the one you are refering to. Consider this  pretend map of some memory:

```
0000: 	0FF0 000F 0000 F000
0004: 	0012 0230 0205 07e0
0008:	0000 0001 0020 0000
000C: 	0300 4000 0050 0b00
``` 
I have represented memory in hexidecimal, which is often used in computing partly because it is more succinct than binary. Each four digit group represents one **word** of memory.

The values at the left are the offset, or the address, of each **word** of memory. The first **word** is numbered 0, and up from there to the total of 65,536 possible on the DCPU. The offset isn't actually part of the memory, it is just used for visualization.

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

