---
title: "Conclusion: Software Portability and Optimization"
seoTitle: "Learnings of Software Portability and Optimization - SPO600"
seoDescription: "What I learnt in the course of Software Portability and Optimization at Seneca College"
datePublished: Tue Apr 23 2024 03:58:52 GMT+0000 (Coordinated Universal Time)
cuid: clvbuwj5y000j08jp6d709u9z
slug: conclusion-software-portability-and-optimization
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713844545252/e31a94b3-0b1c-41bf-bd1d-b0bbe31f1a36.png
tags: memes, gcc, gcc-compiler, spo600, afmv

---

This course was an intellectual rollercoaster for me, I learned so many things and got a dopamine kick whenever I learned something, especially the assembly languages like 6502, x86 and 64ARM. I personally love 6502 as it is one of the easiest assembly languages to learn. Chris(My Professor) even had a coffee mug which defined all the instructions for 6502.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713840110904/0497733e-894a-4f26-aa8c-39639d25d9ec.jpeg align="center")

This mug proves that the instruction set for 6502 is small and easy to learn. I even developed a Number guessing [game](https://stevenpillay.hashnode.dev/random-number-guessing-game-in-assembly) in 6502 Assembly. When I was just getting comfortable with 6502 Assembly, Chris informed us that learning 6502 was just a stepping stone towards learning the x86 and 64ARM.

Then we came across the 64-bit assemblers, I was surprised that, I actually understood the workings of the 64-bit assemblers considerably quickly all thanks to the 6502 Assembly. I was very proud of myself. What I learned in this course is very basic, there is actually a lot to learn about the 64-bit assembler, but this basic information was enough to understand simple Assembly operations.

Then I learned about compiler optimizations, and I was very surprised to know that, the code that we compile be it any language, it is not necessary that it has to run the way that we coded it to run. The compiler applies a lot of optimizations to the code for instance let's take a simple for loop that prints from 100 to 110.

```c
#include <stdio.h>

int main() {
  for (int i = 0; i < 11; i++) {
    const int number = 100;
    printf("%d\n", number + i);
  }
  return 0;
}
```

Now as you can see the constant value is declared inside the loop, and if we were to run this code as is, the constant would be declared 11 times as the loop runs 11 times. Now in order to increase the efficiency of the program the compiler will move the const outside the loop. So it becomes like this:

```c
#include <stdio.h>

int main() {
  const int number = 100;
  for (int i = 0; i < 11; i++) {
    printf("%d\n", number + i);
  }
  return 0;
}
```

This process is called **Hoisting** and is just an example of one of the optimizations the compiler performs. These different kinds of optimizations explained with illustrations can be found [here](https://toronto.tylers.info/dokuwiki-toronto/doku.php?id=spo600:compiler_optimizations). There are separate levels of optimizations with which we can compile our program, here is a [link](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html) to the official documentation if you want to learn more. Even if you choose the lowest optimization level for our program, the compiler will still perform some optimizations.

After all these optimizations, the compiler converts the code to Assembly language, for the OS to execute it.

We have a lot of ARM processors in the market, some support [SVE](https://developer.arm.com/Architectures/Scalable%20Vector%20Extensions) (Scalable Vector Extension), some support [SVE2](https://developer.arm.com/documentation/102340/0100/Introducing-SVE2) and some support none. Now the problem was if we wrote code specialized for SVE, it would not run on the machines that don't support SVE, Similarly, if we wrote code specialized for the general case (Neither SVE nor SVE2), we would not benefit from the cutting-edge technology that the user has.

To solve this problem came our exciting [AFMV](https://stevenpillay.hashnode.dev/afmv-project) Project. We aimed to optimize and clone the program's main(time-consuming) functions to produce the implementation for the SVE, SVE2 and the general case during compile time. There would also be a resolver function that would choose at runtime what function to run depending on the chip's capabilities. All of this would be done requiring zero effort from the developer himself. The developer needs to specify how much architecture to support while compiling the program the rest would be taken care of by the compiler.

I chose to work on the Diagnostic output for the Project, which will provide some info about the Function multi-versioning that took place during compile time, like how many functions were cloned and how many functions were pruned. I have created a test pass for now which can be found in our Project [Repository](https://github.com/Seneca-CDOT/gcc/tree/afmv-diagnostic-output). If you want to know how I created that pass, check out my [blog](https://stevenpillay.hashnode.dev/understanding-the-gcc-passes) about GCC passes.

The Project is not yet fully developed, we are still working on it but I am hoping we will finish this Project very soon and submit our Pull Request to the GCC repository. I also developed the Repository's Wiki so that first-timers can easily open an Issue and submit their pull request.

In the end, I would like to thank Chris and all my fellow students for accompanying me in my journey to learn about Software Portability and Optimization.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713844233527/9b62746a-6b0b-4ee7-9891-1b504946c82b.webp align="center")

I can hereby proudly declare that I can understand this meme🥳 as a result of rigorous training and practice.

## **Sources**

Tyler, C. (n.d.). *Software portability and optimization*. [**matrix.senecapolytechnic.ca**](http://matrix.senecapolytechnic.ca/)**.**[**matrix.senecapolytechnic.ca/~chris.tyler/wi**](http://matrix.senecapolytechnic.ca/~chris.tyler/wi)**..**

*GCC, the GNU Compiler Collection - GNU Project*. (n.d.). [**gcc.gnu.org**](https://gcc.gnu.org/)

[https://www.reddit.com/r/ProgrammerHumor/comments/9h8v2c/assembly\_ftw/](https://www.reddit.com/r/ProgrammerHumor/comments/9h8v2c/assembly_ftw/)

[https://stirringdragon.games/product/the-6502-reference-mug/](https://stirringdragon.games/product/the-6502-reference-mug/)