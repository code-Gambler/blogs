---
title: "How to find the time complexity of 6502 Assembly Code"
seoTitle: "6502 Assembler"
seoDescription: "Funding the time complexity of the 6502 assembler code"
datePublished: Sun Mar 17 2024 22:17:26 GMT+0000 (Coordinated Universal Time)
cuid: cltw2us5l00000ajrcizzavgd
slug: how-to-find-the-time-complexity-of-6502-assembly-code
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1710713113150/d1d74844-218d-4db4-aa30-3ed8e7d91705.jpeg
tags: spo600

---

First of all, in order to run and assemble our 6502 assembler code, we would need a 6502 chip or an Emulator, in this blog post we will be using a 6502 Emulator which can be found [here](https://matrix.senecapolytechnic.ca/~chris.tyler/6502/).

The Emulator looks something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709759125423/ba302ce4-d180-41cb-9ba4-c71710361572.png align="center")

The Emulator has the following Sections:

* Input Box - This is the box on the left and it is used for writing our assembly code
    
* A Bitmap Display - This is the black display in the middle which is used for Graphical Output
    
* Console Box - This is the box on the right which acts as the I/O for our Program
    

Let's start by finding the efficiency of the following source code written for the 6502 Assembler:

```plaintext
 	    LDA #$00	; set a pointer in memory location $40 to point to $0200
 	    STA $40		; ... low byte ($00) goes in address $40
 	    LDA #$02	
 	    STA $41		; ... high byte ($02) goes into address $41
 	    LDA #$07	; colour number
        LDY #$00	; set index to 0
 LOOP:	STA ($40),y	; set pixel colour at the address (pointer)+Y
 	    INY		; increment index
 	    BNE LOOP	; continue until done the page (256 pixels)
 	    INC $41		; increment the page
 	    LDX $41		; get the current page number
 	    CPX #$06	; compare with 6
 	    BNE LOOP	; continue until done all pages
```

What does this code do?

This code-snippet when run on the emulator colors the bitmap display bright yellow.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709759093857/cb3caade-6d77-4e7d-b45b-bd45f46ce047.png align="center")

How to find the **Run-Time** of the program?

First Step: Calculate the number of processor cycles the program takes to complete.

We can do that by referring to 6502's [Documentation](https://www.masswerk.at/6502/6502_instruction_set.html), where the processor cycles for each command is specified. So let's start by calculating the number of cycles for each line of our code. Commands like `BNE` (branch on not equal) have different cycle counts depending if the result is zero or not.

| Label | Command | Cycles | Cycle Count | Alt Cycles | Alt Count | Total |
| --- | --- | --- | --- | --- | --- | --- |
|  | LDA #$00 | 2 | 1 |  |  | 2 |
|  | STA $40 | 3 | 1 |  |  | 3 |
|  | LDA #$02 | 2 | 1 |  |  | 2 |
|  | STA $41 | 3 | 1 |  |  | 3 |
|  | LDA #$07 | 2 | 1 |  |  | 2 |
|  | LDY #$00 | 2 | 1 |  |  | 2 |
| loop: | STA ($40),y | 6 | 1024 |  |  | 6144 |
|  | INY | 2 | 1024 |  |  | 2048 |
|  | BNE LOOP | 3 | 1020 | 2 | 4 | 3068 |
|  | INC $41 | 5 | 4 |  |  | 20 |
|  | LDX $41 | 3 | 4 |  |  | 12 |
|  | CPX #$06 | 2 | 4 |  |  | 8 |
|  | BNE loop | 3 | 3 | 2 | 1 | 11 |

So if we add all the values in this table, we would get the total number of cycles of the program.

$$2+3+2+...+8+11=11325$$

Second Step: Now that we have the total number of cycles we can easily get the run-time of the program by multiplying it by the time it takes for our 6502 processor to complete one cycle. The clock speed for our 6502 processor is 1 microsecond (that is it takes one microsecond to complete one clock cycle).

$$Runtime=Cycles\times ClockSpeed$$

| Total Cycles | Runtime | Unit |
| --- | --- | --- |
| 11325 | 11325 | Micro Seconds |
|  | 11.325 | Milli Seconds |
|  | 0.011325 | Seconds |

Now let us find ways to improve the runtime:

If we observe this table, the runtime of the whole program majorly depends on these 3 lines of code:

| **Command** | **Cycles** | **Cycles Count** | Alt Cycles | Alt Count | **Total Cycles** |
| --- | --- | --- | --- | --- | --- |
| STA ($40),y | 6 | 1024 |  |  | 6144 |
| INY | 2 | 1024 |  |  | 2048 |
| BNE LOOP | 3 | 1020 | 2 | 4 | 3068 |

Since this is a loop, we can try loop unrolling(Unrolling means to repeating the inner portion of the loop multiple times making multiple copies of the code. This takes additional space, but reduces the number of times that the loop control condition is evaluated (Tyler, 2024).) the loop. I have unrolled the loop 32 times but "loop unrolling is often beneficial as long as the unrolled loop fits into cache; unrolling past that point will reduce performance"(Tyler, 2024). So our code should look something like this after unrolling.

```plaintext
    LDA #$00       ; Set a pointer in memory location $40 to point to $0200
    STA $40         ; Low byte ($00) goes in address $40
    LDA #$02
    STA $41         ; High byte ($02) goes into address $41
    LDA #$07        ; Color number
    LDY #$00        ; Set index to 0

; Unrolled loop: Process 32 pixels per iteration

LOOP:
    STA ($40),y ; Set pixel color for the next pixel
    INY         ; Increase the value of Y register
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    STA ($40),y
    INY
    BNE LOOP
    INC $41     ; Increment the page
    LDX $41     ; Get the current page number
    CPX #$06    ; Compare with 6
    BNE LOOP    ; Continue until done all pages
```

Now Let's change the initial code to display light a blue color on the bitmap display. The codes corresponding to the color can be found [here](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:6502_emulator). We just have to change the value stored in the accumulator from #7 to #e.

```plaintext
 	    LDA #$00	; set a pointer in memory location $40 to point to $0200
 	    STA $40		; ... low byte ($00) goes in address $40
 	    LDA #$02	
 	    STA $41		; ... high byte ($02) goes into address $41
 	    LDA #$e	    ; set the colour number to light blue
        LDY #$00	; set index to 0
 LOOP:	STA ($40),y	; set pixel colour at the address (pointer)+Y
 	    INY		; increment index
 	    BNE LOOP	; continue until done the page (256 pixels)
 	    INC $41		; increment the page
 	    LDX $41		; get the current page number
 	    CPX #$06	; compare with 6
 	    BNE LOOP	; continue until done all pages
```

Now let's make the 4 sections of the bit map display of different colors. One way to achieve that would be to store the value of color in a memory location($42), increment the value at that location and load it in the accumulator again.

```plaintext
	LDA #$00	; set a pointer in memory location $40 to point to $0200
 	STA $40		; ... low byte ($00) goes in address $40
 	LDA #$02	
 	STA $41		; ... high byte ($02) goes into address $41
 	LDA #$07	; color number
 	STA $42
	LDY #$00	; set index to 0
 LOOP:	
 	STA ($40),y	; set pixel color at the address (pointer)+Y
 	INY			; increment index
 	BNE LOOP	; continue until done the page (256 pixels)
	INC $42		; Increasing our color value by one
	LDA $42		; Loading the new value in the accumulator
 	INC $41		; increment the page
 	LDX $41		; get the current page number
 	CPX #$06	; compare with 6
 	BNE LOOP	; continue until done all pages
```

Sources:

Tyler, C. (n.d.). *Software portability and optimization*. [matrix.senecapolytechnic.ca](http://matrix.senecapolytechnic.ca). [https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:start](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:start)