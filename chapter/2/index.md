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
; test
```

[ &lt;&lt;&lt; Previous Chapter](../1/) &nbsp; [Next Chapter &gt;&gt;&gt;](../3/)