---
layout: chapter
title: Chapter 2 Memory
subtitle: Some kind of subtitle.
---

# Memory
We can use registers to store information temporarily, but when we want something to be more durable we put it in memory so that we can refer to it later. We refer to it by remembering the address in memory that it is stored in. 

## Addresses
The DCPU has 2&#94;16 words of memory. Conveniently, a word is also 16 bits. This means that you could store enough values in 1 word of memory to refer to each word of memory distinctly. This value is called its address, and basically is the index of that specific word. The first word in memory is 0, the next one 1, etc, all the way up to 0xffff.

[ &lt;&lt;&lt; Previous Chapter](../1/) &nbsp; [Next Chapter &gt;&gt;&gt;](../3/)