---
layout: chapter
title: Chapter 2 Memory
subtitle: Some kind of subtitle.
---

# Memory
We can use registers to store information temporarily, but when we want something to be more durable we put it in memory so that we can refer to it later. We refer to it by remembering the address in memory that it is stored in. 

## Addresses
The DCPU has 2&#94;16 words of memory. Conveniently, a word is also 16 bits. This means that you could store enough values in 1 word of memory to refer to each word of memory distinctly. This value is called its address, and basically is the index of that specific word. The first word in memory is 0, the next one 1, etc, all the way up to 0xffff.

## The Stack
One of the special registers that we saw earlier was the stack pointer. The stack pointer stores an address in memory which is cached by the DCPU for faster access. When you add a value onto the stack, the stack pointer is decreased by 1 and the value is written to that address in memory. 

Basically, the stack grows from the end of memory. The first value you write will be at the end, the next one just before it.

```
set push, 8
set push, b
set a, pop
set b, pop
```

You put values on the stack by setting push, and you get them back using pop. In this example, a is set to whatever b was, and b is set to 8. Values are retrieved in the opposite order that they were put in.

You can access the value at the stack pointer without modyfying the stack pointer using `peek`, and you can access values near the stack pointer by using `pick` and a number.


```
set push, 8
set push, 2
set a, peek   	; set a to 2
set a, pick 1 	; set a to 8
```

## Labels
While you are writing instructions, you can define a label to refer to a specific place in the code. A label is translated by the compiler into the address of next line of code after the label definition. 

```
	set a, 1
:label
	add a, 1
	set pc, label
```

Labels are defined by prefixing the label with a semicolon. Some compilers support putting the semicolon after the label, which is how real assembly works, but not all DCPU compilers will recognize those labels at the moment.

In the example above, label would translate to the number 1. The first instruction, `set a, 1` is encoded into the 0th word of memory, and `add a, 1` is encoded into the 1st word. When `set pc, label` is compiled, it will be translated into `set pc, 1`.

By accessing `pc` directly, you change the program flow. The program will run in a continuous loop because the instruction pointer keeps getting set to the same value.

Label definitions themselves are not translated into machine code, so the above example would compile into three words.

## Globals
If you want to keep track of a value but you don't want to hog a register for it, and you need to access it from a lot of places so the stack is out of the question, you can put it in a specific address in memory and then always refer to it by its address.

You can use a label to act as a stand-in for the address, and that way you don't need to remember the specific address.

```
set a, [aValue]
add a, 16
set [aValue], a

; . . .

:aValue
dat 0
```

When you use square-brackets `[ ]` around a number, the DCPU will interpret the number as an address in memory and access the value at that address. Since labels are translated into a number at compile time, you don't need to keep track of the specific address.

I call these globals because any part of the program could access them.

[ &lt;&lt;&lt; Previous Chapter](../1/) &nbsp; [Next Chapter &gt;&gt;&gt;](../3/)