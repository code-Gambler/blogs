---
title: "AFMV Project"
seoTitle: "Auto Function Multi Versioning"
seoDescription: "Project that automatically implements, Function Multi-Versioning for SVE and SVE2"
datePublished: Mon Apr 15 2024 21:40:49 GMT+0000 (Coordinated Universal Time)
cuid: clv1hbdrm000k08l486voaq89
slug: afmv-project
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713217136328/da2e2081-4d24-4159-b808-54ea721c3b90.jpeg
tags: gnu, gcc, gcc-compiler, spo600, projectstage1, afmv

---

The main aim of this Project is to upgrade the GCC compiler in such a way that the programs compiled by it can be run on machines with different capabilities like SVE or SVE2 while still benefitting from the cutting-edge technologies with zero effort from the developer. First of all, we need to start by building the GCC compiler.Auto

## Building the GCC Complier

I run WSL on my machine on top of Windows, so I concluded that It would a good Idea to build the compiler on my machine, but I am still going to try building it on my machines as well as on our class servers.

First of all, I started, by pulling the source code from the git repository, which can be found here: `git://`[`gcc.gnu.org/git/gcc.git`](http://gcc.gnu.org/git/gcc.git). I then ran the git clone command from the x86 Class machine, `git clone git://`[`gcc.gnu.org/git/gcc.git`](http://gcc.gnu.org/git/gcc.git), It took a while as the repository was big.

Once the pull is complete, we need to run the `configure` script found in the root of the repository, with some options. As instructed on our Course Website, I made a build directory just beside the source tree and used the `--prefix=/path/to/build/dir` option to specify the path of the build directory.

But the configuration failed as I didn't have all the dependencies installed, specifically `mpc`, `mpfr`, and `gmp`. So, I looked up how to install this software on the machines for hours and tried to install it, but failed. After hours of digging through the internet, I found a simple command `./contrib/download_prerequisites` which can be run from the root of the project to install all the dependencies while reading the official [documentation](https://gcc.gnu.org/install/download.html) for installing GCC. After running this command the configure command was successfully executed.

As a result of this, a Makefile along with some other files was created in the build directory. So I `cd`'d into the build directory and ran the `make` command, but I forgot to add the `-j` to specify how many jobs can run in parallel as I underestimated the time taken by the project to build, I waited for three hours but then after I decided to interrupt the make and start it again using `-j` option, I also used `lscpu` to get the number of cores for our Class server. This is what I got:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713208490847/f33de13d-bca1-431d-bd40-d20b46a38a0b.png align="center")

As we can see our machine has Seven cores, we can use any number between 8 and 15. I went ahead and used 10 cores to `make -j 10`. Still, it took about four to five hours to build but finally, it was built now it was the time to test.

I tested it by using two commands, `gcc --version` gives us the version of the GCC compiler that has been installed in the machine, and `/home/spillay6/gcc/build/bin/gcc --version` gives us the version of the GCC compiler that we just built.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713209497894/dede0f4c-e65f-422a-bf82-79f1f1cc898d.png align="center")

This Project is now divided into 12 tasks, which are as follows:

| **Name** | **Description** |
| --- | --- |
| Command-line Parsing | Parse the GCC command line to pick up AFMV options |
| Version List Processing | Process the version list received from the command-line to validate the architectural features |
| arch= Arguments | The current GCC AArch64 FMV capability accepts versions that are identified by feature flags (such as “sve2”) but does not accept “arch=” arguments such as “arch=armv9-a” (those type of arguments are accepted by the x86 FMV implementation). Add this functionality. |
| Apply FMV cloning to functions automatically | When the appropriate command-line options are provided, the compiler should automatically clone *all* functions, as if the target\_clone attribute was specified. |
| Produce an error message if AFMV and FMV are used together | Produce an error if the compiler is invoked with AFMV command-line options *and* there are FMV attributes specified in the code. |
| Prune Cloned Functions | Remove any AFMV-created clone functions that do not provide any significant benefit or differentiation. |
| Diagnostic Output | Provide diagnostic output (when activated by -fdump-…-… command-line options). |
| Git Wrangler | Mangage the repository, including frequent rebasing. |
| Update Documentation 1 | Update the existing GCC IFUNC and FMV documentation (all archs) |
| Update Documentation 2 | Update the existing ACLE documentation |
| Create AFMV Documentation | Create documentation for the AFMV feature. |
| Create Tests | Create a suite of tests for the AFMV capability. (This is in addition to individual tests that the various task owners will prepare). |

The two tasks that I am interested in are:

* Diagnostic Output
    
* Create AFMV Documentation
    

## Diagnostic Output

First of all, we need to create our file called the `diagnostic-dump.cc` inside the GCC folder which would have all the details of our Diagnostic pass. For stage 2 we will only be developing a test pass, that triggers when we use a specific command line argument and is also triggered when the user uses a `-f-dump-all` option. The second file that we need to update is the `passes.def` file which contains all the passes that are performed on the system. We need to insert our diagnostic pass after the `a-helloWorld.c.265t.optimized`. Our Diagnostic pass will mainly show two things:

* How many functions were cloned?
    
* How many functions were pruned?
    

I would develop it locally on my machines but not build it, as VS code is much easier to work with as compared to vim, and once I am done with the files, I will transfer the files to the `x86` class machine and run the `make` command.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713216432253/135096ff-f76e-4312-83df-017f9048066f.png align="center")

If everything goes well, we will see another pass, just below optimized called `a-helloWorld.c.265t.diagonstic-dump`. This work heavily depends on the following tasks:

* Apply FMV cloning to functions automatically
    
* Prune Cloned Functions
    

We will be providing the user with the values derived from these processes, but for Stage 2, I will create a test dump that does nothing but just says something like "If you are seeing this pass, the diagnostic dump is working."

As far as the time is concerned for this project, I am pretty confident that I can complete this task in about 20 hours. The time division is as follows:

* Create a test dump which is triggered only a `-afmv-dump` option is used - 10 hours
    
* Add the create dump file to `passes.def` - 5 hours
    
* Debug any errors or exceptions that come up - 5 hours
    

## Create AFMV Documentation

This task would require us to create the documentation for all the features and options that we developed during this project. I will start by learning how GCC maintains its current documentation by having a look at this repository `git://gcc.gnu.org/git/gcc-wwwdocs.git` which contains all the documentation for GCC. The documentation for contribution can be found [here](https://gcc.gnu.org/about.html#git). I will start by adding a section called AFMV, In which I will specify all the command line options and features available to the user. I will define each and every option in detail, and provide a brief overview of Auto Function-Multi-Versioning.

As this is the documentation part, its completion depends on the completion of all the remaining parts of the project. For Stage I can develop documentation on what is AFMV? and how it works. Once everyone turns in their part, I can work on the specific command line options and their features.

Documentation for Stage would require at least 30 hours, The time division is as follows:

* Research AFMV - 10 Hours
    
* Prepare the documentation - 10 Hours
    
* Implement corrections and changes pointed out in the peer review - 10 Hours
    

## Conclusion

I personally would like to work on the diagnostic dump. Mainly because the problem when working with a big project like GCC is not the actual coding but figuring out where and which file to change/update, as I have already figured out what files to change and what files to update, that's half work done right, Now I just need to implement a test-dump for Stage 2 and Integrate it with others in Stage 3, it seems like a straight forward task to me. Whereas If were to work on documentation I would have to write definitions and explanations, I would rather write code than documentation.

## Sources

Tyler, C. (n.d.). *Software portability and optimization*. [**matrix.senecapolytechnic.ca**](http://matrix.senecapolytechnic.ca/)**.**[**matrix.senecapolytechnic.ca/~chris.tyler/wi..**](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:start)

*GCC, the GNU Compiler Collection - GNU Project*. (n.d.). [https://gcc.gnu.org/](https://gcc.gnu.org/)

GCC Compiler. (2021). [https://www.linkedin.com/pulse/gcc-compiler-ran-kong/](https://www.linkedin.com/pulse/gcc-compiler-ran-kong/)