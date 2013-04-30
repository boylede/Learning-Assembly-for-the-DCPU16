
#The LEM 1802 Monitor
Now that we are able to figure out what hardware is connected, lets take a look at the specification for the screen. The official spec is found at [dcpu.com/monitor](http://dcpu.com/monitor/).

We already learned that `hwi` is used to talk with devices, and that it reads the value of `a` to decide what to do, lets look all the options.

```
A value:		Name:
0				MEM_MAP_SCREEN
1				MEM_MAP_FONT
2				MEM_MAP_PALETTE
3				SET_BORDER_COLOR
4				MEM_DUMP_FONT
5				MEM_DUMP_PALETTE

```

Each of the mem_map functions tells the monitor to use a region of memory, defined by the `b` register, for a certain data structure.

The mem_dump options allow you to write out the default data structure of the font and the palette.

Finally, the set border color option is pretty straight forward. The screen has a border around it that can be any of the 16 colors in the palette.

## The Screen memory
Once you map the screen memory, the monitor will start to draw the screen based on the values in that region of memory. The screen's display is a 32 wide by 12 tall grid of characters. Each character setting takes up one **word** of memory, for a total of 384 **words**. Each **word** of the screen memory works like this:

```
binary:
ffff bbbb Bccc cccc
```
The highest 4 bits - `f` - choose the foreground color from the 16 color palette. The next 4 - `b` - choose the background color. The `B` bit sets whether this character should "blink" on and off, and the lowest 7 bits choose which of the 128 possible characters to display. 

To write to the screen, we need to at minimum set the lowest 7 bits to the character we want, and then set the foreground color. We could also set a background color and/or set `blink` to have the character blink on and off.

As we saw when storing globals, you can use `dat` to insert raw data into the compiled code.

```
:hellostring
dat "Hello World", 0
```
This puts the data values for the characters in the string into the compiled output, followed by a blank (zero) **word**.

Right now the upper bits of the characters that define the colors are not set - if we copied them to the screen like this they would show up as black on black. To make them visible, all we need to do is change the foreground color.

Since we know the bits are unset, we can use `bor` to set them. If we didn't know we would have to use `and` to clear them first.

```
; set forground to white
bor [hellostring], 0xf000

; to clear color & blink bit
and [hellostring] 0x7f
```

As we learned before, a label only points to one **word**, the **word** following the label's definition. So, right now we have only changed the first character's color. We need to make a loop to do this for all the characters.
We'll know it is time to stop when we reach the `0`. While we're at it, we'll copy the letters from the program's part of memory into the screen's mapped part of memory.

```
:start
	set j, hellostring 	; this is where we will copy from
	set i, [video] 		; this is where we will copy to
	
:printLoop

; this adds a color to each character, so its not black on black.
	bor [j], 0xf000
	
; set the value at i to the value at j, then increment i and j.
	sti [i], [j]
	
; now check if we are at the end of the string.
; earlier, put a 0 value at the end of the string for this purpose.
	ifn [j], 0
		set pc, printLoop
		
; this will stop the program.
	sub pc, 1
```

In this example, I've set the color on the string before I move it to screen memory. This is because I wanted to use `sti`, but the downside is that if you did it again with a different color the result would be unexpected. Lets make a general function that writes strings out to the screen for us. 

##Subroutines
We've already been using labels to define certain peices of data that we want to use later, but you can also use them to point to a group of instructions that you want to run more than once, in different contexts. These are called subroutines.

Since subroutines are accessed from different places in the program, you need to keep track of where you were before you used the subroutine so that you can get back there. The stack is great for that, because it allows you to store a sequential list of values so even if you call another subroutine from inside the first one you can get back to where you are supposed to.

The instruction `jsr` makes this easier by pushing the `pc` to the stack, and then setting `pc` to the value given.

```
set a, 6
jsr aSubroutine

set a, 8
jsr aSubroutine

set a, 2
jsr aSubroutine

; stop here
sub pc, 1

; anywhere else in the program...

:aSubroutine
add a, 6
div a, 2
add [accumulate], a
set pc, pop

:accumulate
dat 0
```

Remember, labels just translate to numbers, so when you write `jsr aSubroutine`, it translates to something like `jsr 0x5402`.

When you jump to a subroutine, it might be refferred to as "calling" that subroutine. In order to talk about subroutines in a general sense, you might refer to the place in the program where the subroutine is called from the "caller" and you might refer to the subroutine itself the "callee".

Since we are going to use the subroutine in different contexts, we want to make sure we don't overwrite registers that the caller might have been using. We can use the stack for that too.

```
:aSubroutine
set push, a  	; put a on the stack
set push, b		; put b on the stack

sub b, 2
add a, b
div a, 2
add [accumulate], a

set b, pop		; get b back from the stack
set a, pop 		; get a back from the stack
set pc, pop

:accumulate
dat 0
```

Since the stack is sequential, if we had forgotten to get `a`, `b`, and `pc` back in the right order, we would set those registers to the wrong values.

##Print String
So lets write a subroutine that prints things out onto the screen. 

```
; a function to write a string to the screen
; excpects a pointer to a zero-terminated string in register A.
; and the color values in b.
:print
	set push, i
	set push, j

	set i, [cursor]
	add i, [video]
	set j, a	
	
:printLoop
	and [j], 0x7f
	bor [j], b
	sti [i], [j]
	ifn [j], 0
		set pc, printLoop

	sub i, [video]
	set [cursor], i
	
	set i, pop
	set j, pop
	set pc, pop
	
:cursor
	dat 0
:video
	dat 0

```

This is just an example print subroutine, and it isn't perfect. For instance, it sets the color on the actual string given as input, and doesn't ever remove it. It also doesn't check to see if the string runs off the screen, or if `b` is even a valid color.

In the next chapter, we'll take a look at keyboard input, and maybe handle some of those problems.

# Appendix

## The Font
This monitor has an internal font with 128 characters. Each character is defined by two **words** - each bit represents a pixel of the 8x4 pixels available for each character. You can choose to have the screen use your own font by mapping part of the memory for use as the font.

## The Palette

The LEM-1802 has an internal palette of 16 colors. These can be any rgb color with 4 bits per channel - meaning that there are a total of 4096 possible colors. If you want, you can write your own palette and tell the monitor to use it instead. 

The palette is 16 **words** - one per color. Each **word** has 4 bits - that is one hex digit - per r,g,b channel. The highest 4 bits are always 0.
