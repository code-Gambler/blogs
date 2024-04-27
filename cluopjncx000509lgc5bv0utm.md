---
title: "Introduction to 64 bit Assembly"
seoTitle: "Introduction to 64 bit Assembly"
seoDescription: "Simple loop program coded for x86 and 64 Arm/Aarch 64 Assembler"
datePublished: Sat Apr 06 2024 23:10:11 GMT+0000 (Coordinated Universal Time)
cuid: cluopjncx000509lgc5bv0utm
slug: introduction-to-64-bit-assembly
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1712444982158/5a119b59-bc2f-4ddb-b4a4-c824679cde9d.jpeg
tags: assembly-language, assembly, 64-bit-processor, assembly-x8664-assembly-programming, spo600, lab3

---

In this blog, I am going to talk about x86 and Aarch64 assemblers, and their basic commands and we are going to code a simple loop that prints the index. Seems like a simple task right? Guess again, as we are going to do it in assembly language for two different architectures(x86 and Aarch64). It is going to be an awful lot of code to print a simple index, so bear with me until the end.

## Arch 64 Assembler

```plaintext
 .text
 .globl _start
 min = 0                          /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 10                         /* loop exits when the index hits this number (loop condition is i<max) */
 _start:
     mov     x19, min
 loop:
     /* ... body of the loop ... do something useful here ... */
     add     x19, x19, 1
     cmp     x19, max
     b.ne    loop
     mov     x0, 0           /* status -> 0 */
     mov     x8, 93          /* exit is syscall #93 */
     svc     0               /* invoke syscall */
```

Aarch 64 is a Reduced Instruction Set Computer (RISC) processor. In simple words, the instructions for this assembler are simple and can only do one task at a time. Here is a fairly simple piece of Aarch 64 code, which loops 10 times but doesn't do anything inside the loop. The program starts at the label start, and in the last 3 lines of code, we use the syscall to exit the program. Notice our program starts at `_start` but if we use the c-complier instead of the assembler, our program is going start at label `main`. Lots of talking now let's get this code to print loop 10 times. We will be using this quick start [guide](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:aarch64_register_and_instruction_quick_start) to code.

In order to do that we are going to define our msg which is going to be "Loop" and we are going to do that in the `.data` section. After that, we are going to load the `x0` register with 1, which is stdout, following that register is going to be our `x1` register which is going to contain the address of our message and lastly `x2` is going to contain the length of the message, and after that we are going invoke the system call which is going to print the message to the screen.

```plaintext
 .text
 .globl _start
 min = 0                          /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 10                         /* loop exits when the index hits this number (loop condition is i<max) */
 _start:
     mov     x19, min
 loop:
     /* Print the Message */

     mov     x0, 1           /* file descriptor: 1 is stdout */
     adr     x1, msg         /* message location (memory address) */
     mov     x2, len         /* message length (bytes) */

     mov     x8, 64          /* write is syscall #64 */
     svc     0               /* invoke syscall */

     /* Continue the Loop */
     add     x19, x19, 1
     cmp     x19, max
     b.ne    loop

     /* Exit the Program */
     mov     x0, 0           /* status -> 0 */
     mov     x8, 93          /* exit is syscall #93 */
     svc     0               /* invoke syscall */

.data
msg:    .ascii      "Loop\n"
len=    . - msg
```

Here is the output of the above code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712435712891/eb4ecb80-0a0a-4f0b-a064-8b27e97793ba.png align="center")

Now let's print the index number along with "Loop", it should look something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712435759402/aeb2ed51-6a07-48ce-8076-5041db0cbff7.png align="center")

In order to do that we first add some filler character to our message (We are using '#'), and inside our loop, we are going to replace this filler with our index, but first we would have to convert our index to a character, we can do it simply by adding 48(ASCII code for '0') to our value. Here is the assembly code that prints the loop index.

```plaintext
 .text
 .globl _start
 min = 0                          /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 31                         /* loop exits when the index hits this number (loop condition is i<max) */
div = 10
 _start:
     mov     x19, min
 loop:
     /* Convert Loop counter to char */
     add     x15, x19, '0'
     adr     x14, msg+6
     strb    w15, [x14]

     /* Print the Message */

     mov     x0, 1           /* file descriptor: 1 is stdout */
     adr     x1, msg         /* message location (memory address) */
     mov     x2, len         /* message length (bytes) */

     mov     x8, 64          /* write is syscall #64 */
     svc     0               /* invoke syscall */

     /* Continue the Loop */
     add     x19, x19, 1
     cmp     x19, max
     b.ne    loop

     /* Exit the Program */
     mov     x0, 0           /* status -> 0 */
     mov     x8, 93          /* exit is syscall #93 */
     svc     0               /* invoke syscall */

.data
msg:    .ascii      "Loop: #\n"
len=    . - msg
```

There is a limitation for the above code it can only print a single-digit index, let's modify our code so that it can print numbers up to 30. We can do that by dividing our index by 10, the quotient is the first digit and the remainder is the second digit. The only problem is the `udiv` instruction in Arch 64 doesn't calculate the reminder, instead, we will be using the `msub` instruction to calculate the reminder.

```plaintext
 .text
 .globl _start
 min = 0                          /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 31                         /* loop exits when the index hits this number (loop condition is i<max) */
div = 10
 _start:
     mov     x19, min
 loop:
     /* Initializing the registers */
     mov     x10, x19
     mov     x12, div
     
     /* Calculating the Quotient (First Digit) */
     udiv    x11, x10, x12
     
     /* Calculating the Reminder (Second Digit) */
     msub    x13, x12, x11, x10

     /* Convert Loop counter to char and replacing the First Digit in the message */
     add     x15, x11, '0'
     adr     x14, msg+6
     strb    w15, [x14]

     /* Convert Loop counter to char and replacing the Second Digit in the message */
     add     x16, x13, '0'
     adr     x17, msg+7
     strb    w16, [x17]
     /* Print the Message */

     mov     x0, 1           /* file descriptor: 1 is stdout */
     adr     x1, msg         /* message location (memory address) */
     mov     x2, len         /* message length (bytes) */

     mov     x8, 64          /* write is syscall #64 */
     svc     0               /* invoke syscall */

     /* Continue the Loop */
     add     x19, x19, 1
     cmp     x19, max
     b.ne    loop

     /* Exit the Program */
     mov     x0, 0           /* status -> 0 */
     mov     x8, 93          /* exit is syscall #93 */
     svc     0               /* invoke syscall */

.data
msg:    .ascii      "Loop:  #\n"
len=    . - msg 
```

Here is the output for the following code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712436637027/d443023d-df03-431e-9ff8-304803f54e79.png align="center")

Let's make this output a little prettier by suppressing the leading zero. In order to do that we will be using the `cpm` and `b.eq` instructions to create a conditional jump to printing the second digit if the first digit is a zero.

```plaintext
 .text
 .globl _start
 min = 0                          /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 31                         /* loop exits when the index hits this number (loop condition is i<max) */
div = 10
 _start:
     mov     x19, min
 loop:
     /* Convert Loop counter to char */
     mov     x10, x19
     mov     x12, div
     udiv    x11, x10, x12
     msub    x13, x12, x11, x10

     cmp     x11, 0
     b.eq    second
first:
     add     x15, x11, '0'
     adr     x14, msg+6
     strb    w15, [x14]
second:
     add     x16, x13, '0'
     adr     x17, msg+7
     strb    w16, [x17]
     /* Print the Message */

     mov     x0, 1           /* file descriptor: 1 is stdout */
     adr     x1, msg         /* message location (memory address) */
     mov     x2, len         /* message length (bytes) */

     mov     x8, 64          /* write is syscall #64 */
     svc     0               /* invoke syscall */

     /* Continue the Loop */
     add     x19, x19, 1
     cmp     x19, max
     b.ne    loop

     /* Exit the Program */
     mov     x0, 0           /* status -> 0 */
     mov     x8, 93          /* exit is syscall #93 */
     svc     0               /* invoke syscall */

.data
msg:    .ascii      "Loop:  #\n"
len=    . - msg
```

Here is the output of this code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712436903121/31934a60-e156-450f-9079-ea85abc1b561.png align="center")

Now Let's jump to the x86 Assembler

## x86 Assembler

We are going to code the same loop but this time with the x86 assembler. Here is the starter code, which is just a simple loop but it prints nothing. We will be using this quick start [guide](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:x86_64_register_and_instruction_quick_start) for our reference.

```plaintext
 .text
 .globl    _start
 min = 0                         /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 10                        /* loop exits when the index hits this number (loop condition is i<max) */
 _start:
     mov     $min,%r15           /* loop index */
 loop:
     /* ... body of the loop ... do something useful here ... */
     inc     %r15                /* increment index */
     cmp     $max,%r15           /* see if we're done */
     jne     loop                /* loop if we're not */
     mov     $0,%rdi             /* exit status */
     mov     $60,%rax            /* syscall sys_exit */
     syscall
```

x86 is a Complex Instruction Set Computer (CISC) processor. In simple words, the instructions for this assembler are complex and can do multiple tasks. Now let's print "Loop" using this code. In order to do that we will have to add our msg in the `.section .data`. After that it is pretty much the same as 64Arm we set up the registers containing the address to the message and the length of the message and finally we perform a syscall.

```plaintext
 .text
 .globl    _start
 min = 0                         /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 10                        /* loop exits when the index hits this number (loop condition is i<max) */
 _start:
     mov     $min,%r15           /* loop index */
 loop:
     /* ... body of the loop ... do something useful here ... */
     movq    $len,%rdx                       /* message length */
     movq    $msg,%rsi                       /* message location */
     movq    $1,%rdi                         /* file descriptor stdout */
     movq    $1,%rax                         /* syscall sys_write */
     syscall


     inc     %r15                /* increment index */
     cmp     $max,%r15           /* see if we're done */
     jne     loop                /* loop if we're not */
     mov     $0,%rdi             /* exit status */
     mov     $60,%rax            /* syscall sys_exit */
     syscall

.section .data

msg:    .ascii      "Loop #\n"
        len = . - msg
```

Here is the output for the following code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712438212663/a667ef7d-8bf3-4d05-8192-38fc6c3d2753.png align="center")

Now, Let's add an index number in the message by replacing the '#' as before.

```plaintext
 .text
 .globl    _start
 min = 0                         /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 10                        /* loop exits when the index hits this number (loop condition is i<max) */
 _start:
     mov     $min,%r15           /* loop index */
 loop:
     /* Convert Loop Counter to char  */
     mov     %r15, %r14
     add     $'0', %r14
     mov     %r14b, msg+6

     /* Print the Message */
     movq    $len,%rdx                       /* message length */
     movq    $msg,%rsi                       /* message location */
     movq    $1,%rdi                         /* file descriptor stdout */
     movq    $1,%rax                         /* syscall sys_write */
     syscall

     /* Continue the Loop */
     inc     %r15                /* increment index */
     cmp     $max,%r15           /* see if we're done */
     jne     loop                /* loop if we're not */

     /* Exit the program */
     mov     $0,%rdi             /* exit status */
     mov     $60,%rax            /* syscall sys_exit */
     syscall

.section .data

msg:    .ascii      "Loop: #\n"
        len = . - msg
```

Here is the output of the following code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712438286622/a406ccf7-c04f-4e10-9a89-810e561eaeb4.png align="center")

  
Now let's extend this loop to print numbers until 30, luckily as x86 is a CISC processor, the `div` instruction does calculate the remainder, unlike Aarch64. So we can calculate the first digit which is the quotient (It will be stored in the rax register) and the second digit which is the reminder (It will be stored in the rdx register).

```plaintext
 .text
 .globl    _start
 min = 0                         /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 31                        /* loop exits when the index hits this number (loop condition is i<max) */
div = 10
 _start:
     mov     $min,%r15           /* loop index */
 loop:
     /* Initilize the Registers */
     mov     $0, %rdx
     mov     %r15, %rax
     mov     $div, %r13
     
     /* Perform the Divison */
     div     %r13

     /* Move the Quotient and Reminder to different registers */
     mov     %rax, %r12
     mov     %rdx, %r11
    
     /* Replace the First Digit in the message */
     add     $'0', %r12
     mov     %r12b, msg+6

     /* Replace the Second Digit in the message */
     add     $'0', %r11
     mov     %r11b, msg+7

     /* Print the Message */
     movq    $len,%rdx                       /* message length */
     movq    $msg,%rsi                       /* message location */
     movq    $1,%rdi                         /* file descriptor stdout */
     movq    $1,%rax                         /* syscall sys_write */
     syscall

     /* Continue the Loop */
     inc     %r15                /* increment index */
     cmp     $max,%r15           /* see if we're done */
     jne     loop                /* loop if we're not */

     /* Exit the program */
     mov     $0,%rdi             /* exit status */
     mov     $60,%rax            /* syscall sys_exit */
     syscall

.section .data

msg:    .ascii      "Loop: ##\n"
        len = . - msg
```

Here is the Output of the following Program:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712438857317/a173f9e6-d7d9-4227-8bdf-a3ac10ce470c.png align="center")

Now let's remove the leading zeros, we do that by using the `cmp`  
and `je` instruction.

```plaintext
 .text
 .globl    _start
 min = 0                         /* starting value for the loop index; **note that this is a symbol (constant)**, not a variable */
 max = 31                        /* loop exits when the index hits this number (loop condition is i<max) */
div = 10
 _start:
     mov     $min,%r15           /* loop index */
 loop:
     /* Convert Loop Counter to char  */
     mov     $0, %rdx
     mov     %r15, %rax
     mov     $div, %r13
     div     %r13
     mov     %rax, %r12
     mov     %rdx, %r11

     cmp     $0, %r12
     je      second

first:
     add     $'0', %r12
     mov     %r12b, msg+6

second:
     add     $'0', %r11
     mov     %r11b, msg+7

     /* Print the Message */
     movq    $len,%rdx                       /* message length */
     movq    $msg,%rsi                       /* message location */
     movq    $1,%rdi                         /* file descriptor stdout */
     movq    $1,%rax                         /* syscall sys_write */
     syscall

     /* Continue the Loop */
     inc     %r15                /* increment index */
     cmp     $max,%r15           /* see if we're done */
     jne     loop                /* loop if we're not */

     /* Exit the program */
     mov     $0,%rdi             /* exit status */
     mov     $60,%rax            /* syscall sys_exit */
     syscall

.section .data

msg:    .ascii      "Loop:   \n"
        len = . - msg
```

## My Experience

When coding assembler, surprisingly I was not stuck on a single error for a long time, which happens when I code in other languages maybe because I was accomplishing a simple task. My existing knowledge about the 6502 processor really helped me understand both assembly languages. It made me comfortable to work with registers instead of variables and labels instead of functions. One thing that I was happy about when coding in 64-bit assembly was that I no longer had to store values in memory and load them back in the register for mathematical operations as I had 4 times the amount of register in 64-bit Assembly compared to the 6502 processor.

If I had to choose between x86 and 64 Arm, I would go with 64 Arm as the instructions are very simple as well as the syntax. Debugging in Aarch 64 is also easier than in 64 Arm, as there are very few compile errors, at least I did not face any compile errors during this lab, but there were some semantic errors in my code, for instance, I was adding 30 to the index value to convert to a number but I didn't realize that I have to 30 in hexadecimal not in decimal, these were some of the minor errors that I faced during this lab. As always all the above code can be found [here](https://github.com/code-Gambler/64-bit-Assembly). Please feel free to propose changes by opening a Pull request.

Sources:

Tyler, C. (n.d.). *Software portability and optimization*. [**matrix.senecapolytechnic.ca**](http://matrix.senecapolytechnic.ca/)**.** [**matrix.senecapolytechnic.ca/~chris.tyler/wi..**](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:start)