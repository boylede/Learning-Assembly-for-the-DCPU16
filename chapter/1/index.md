---
layout: chapter
title: Chapter 1 - Basic Concepts
subtitle: A guided tour of the DCPU-16 specification.
---

# Basic Concepts

## Numbers 
### Binary
It is not neccessary to understand binary completely to start out. All that you need to know is that binary is the only way computers think, and that they don't "know" that any data should be interpreted in a specific way other than how the programmer tells them to. 

Binary is merely a way to represent numerical values, so essentially everything that the computer knows boils down to one value or another.

Digits in binary are called bits, and the smallest peice of data the DCPU deals with is 16 bits, which means that a peice of information can be any one of 65,536 values, and nothing else. This piece of data is called a **word**.

Writing sixteen 1's and 0's every time you want to talk about a value would get tedious fast, so programmers often use hexidecimal to represent binary values.

### Hexidecimal
Hexidecimal, or simply hex, is a way to write numbers just like binary and decimal. It is used because it is compatible with binary - each hex digit is 4 bits of information. Decimal does not have such a direct comparison.

Hexidecimal digits can be any one of 16 values, represented by the numbers 0-9 and the letters a-f. The hex digit "a" is equivalent to the decimal number 10, b=11, etc, up to f=15. In order to denote that a given number is written in hex, the prefix ```0x``` is used. 

```0x12f4``` is equal to ```4852``` in decimal or ```0001 0010 1111 0100``` in binary.

## Assembly Code

Assembly consists of "simple" computations which are executed one at a time, in order. These are called **instructions**. Each instruction contains an operation and one or two values. 

```dasm

	set a, 12
	add a, 4
	set b, 2
	sub a, b
	
```

In the example above there are several different instructions shown, using the ```set```, ```add```, and ```sub``` operations. As you can probably guess, ```add``` and ```sub``` perform arithmatic on integers. Set allows you to copy a value to another place, in this example we store two values in two registers, called ```a``` and ```b```. Registers are like little pockets of memory on the processor which can be accessed quickly.
