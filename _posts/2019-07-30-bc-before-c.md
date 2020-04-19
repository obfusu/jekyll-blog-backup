---
layout: post
title: BC - Before C
date: '2019-07-30T00:22:00.000+05:30'
author: 77ganesh
tags: 
modified_time: '2019-07-31T14:44:14.730+05:30'
---

Before going to C, lets see why programming languages were invented.

Computers are the most dumbest thing in this world. They understand only 2 terms - 0 or 1, and this language is called as binary.

The very first programs were exactly written in this language. For example to add 2 numbers, the program would look like this.

```
10110000 00000101
01000000 00000010
11110100
```

Since only 2 terms are there, a simple program would be lengthy spanning several pages. Since reading a lot of 0s & 1s is mentally tedious and confusing, they came up with hex notation.

Hex is just a shortcut for binary. Hex is base 16. So you can represent 16 different unique combinations using a single digit (0-9 and then A-F). Binary is base 2. So you can only represent 2 different unique combination - 0 & 1.

To represent 16 different unique combinations in binary, how many digits do you need?
1 digit in binary can represent 2 unique values - 0 or 1
2 digits in binary can represent - 0 or 1 followed by 0 or 1
i.e. 2 x 2 = 4 unique combinations
3 digits can similarly represent - 2 x 2 x 2 = 8
And 4 digits can represent - 2 x 2 x 2 x 2 = 16

Here is the conversion mapping:

Hex - Binary
```
0 - 0000
1 - 0001
2 - 0010
3 - 0011
4 - 0100
5 - 0101
6 - 0110
7 - 0111
8 - 1000
9 - 1001
A - 1010
B - 1011
C - 1100
D - 1101
E - 1110
F - 1111
```

So the same program in hex would look like this
```
B0 05
80 02
F4
```

This still looks weird, but at least it is readable, unlike an overflowing confusing stream of 0s and 1s.

As the next step, programmers invented the assembly language. Assembly language consists of mnemonics. Mnemonics is another fancy word for shorthand. Just like how hex is used to represent binary, we will now use mnemonics to represent hex.

```nasm
MOV AL, 5
ADD AL, 2
HLT
```

As you can see, it slightly starts to make sense.
Lets go line by line.

MOV AL, 5

This states that move the number 5 into register AL. We'll see what registers are in the next post. For now, assume registers means variables.

ADD AL, 2

Means AL = AL + 2

HLT

Halt and end the program.


The above program just adds the number 5 & 2. If you want to provide input and print the output, we'll have to write a handful of much more complicated mnemonics.

This way of writing programs takes a lot of time, effort and cost to develop and maintain. Hence programming languages like C were invented to make life easier. 