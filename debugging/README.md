*This is not a lab, nor is it anything specific to writing compilers, but
rather a guide to help you debug programs.*

It is inevitable you will bump into bugs in your code (eg: random
segmentation faults). It helps to know about the existence of various
tools that will help you in debugging.

This document is, by no means, a thorough guideline on the available tools,
but rather, a introductory document. In other words, I am saying "these
tools exist and here's roughly what they do. There are decent resources
available in the internet to figure out what features to do they have. The
internet is your best friend." It is likely you will spend at least an hour
or two going through this whole document, but it is likely to save you many
more hours than that down the line (not just for this assessment, but for
pretty much all code you write in the future).

For simplicity, I will be assuming that you are in a
[Unix](https://en.wikipedia.org/wiki/Unix)-based environment.

In the case that the tools here will not fit your problems / needs,  it
_does_ help to simply lookup "How to fix XXXX" in a search
engine. You might be surprised by the results (This is how I found out about
valgrind, one of the tools presented below).

## Debuggers

If you have programmed in languages like Python or Java, you will be
familiar with backtraces when your program crashes.

```
fyquah@olaf$ python3 example-backtrace-1.py
# in python (example-1.py)
Traceback (most recent call last):
  File "example-1.py", line 9, in <module>
    main()
  File "example-1.py", line 5, in main
    x += a[i]
IndexError: list index out of range

# in Java (Example2.java)
fyquah@olaf$ java Example2
Exception in thread "main" java.lang.StackOverflowError
	at Example2.fibonaci(Example2.java:5)
	at Example2.fibonaci(Example2.java:5)
	at Example2.fibonaci(Example2.java:5)
        ....
	at Example2.fibonaci(Example2.java:5)
	at Example2.fibonaci(Example2.java:5)

# In C++, however ... :(  <-- this is a sad emoji
fyquah@olaf: debugging $ ./example-backtrace-3
Segmentation fault
```

Having a backtraces is incredibly helpful, as it aids you in pinpointing where
did the program crash. To get something similar in C/C++:

- Make sure you compile your object files with the `-g` flag. Read the g++'s
  manual pages (`man g++`) to see what this flag does. There are many ways
  to encode source information within binaries, one of the most popular
  ways is to use [DWARF](http://dwarfstd.org) (You are not required to
  understand how DWARF works, but nevertheless,
  [this introduction on DWARF](http://dwarfstd.org/doc/Debugging%20using%20DWARF-2012.pdf)
  is quite an entertaining read)
- Compilers allows you to compile your code with different level of
  optimisations, like `-O0` for fast compilation, `-O3` with agressive
  optimisation. DO NOT compile with `-O3`. Debuggers will still work,
  it is unlikely to yield very useful information.
- Execute your binary with a [debugger](https://en.wikipedia.org/wiki/Debugger)

Your choice of debuggers are as follows:

- On Linux, the GNU debugger (commonly referred to as gdb) should work out
  of the box. The EE lab machines are setup with gdb. Most distributions
  have gdb as part of their package managers.
- On MacOS, you can either use lldb, using this
  [command translation table](https://lldb.llvm.org/lldb-gdb.html)
  You can also try installing gdb and
  [code sign it](https://gist.github.com/gravitylow/fb595186ce6068537a6e9da6d8b5b96d).

*The following text will assume you are using gdb.*

There is a little program called [example-backtrace-3.cpp](./example-backtrace-3.cpp) which
takes a list of numbers as its input arguments sorts them, and outputs the
last element of the list. There is a bug in the program that will result in
a segmentaion fault (Exercise: Spot the bug).

To view a backtrace when running a program, 

```bash
fyquah@olaf: debugging $ gdb --args ./example-3
GNU gdb (GDB) 8.0
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-slackware-linux".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./example-3...done.
(gdb) run
Starting program: /home/fyquah/dev/eie/langproc-2017-cw/debugging/example-3
Running computationally heavy process!

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400d14 in process_arguments (argc=1, argv=0x7fffffffd8e8) at example-3.cpp:12
12        return *(numbers.end() - 1);
(gdb) bt
#0  0x0000000000400d14 in process_arguments (argc=1, argv=0x7fffffffd8e8) at example-3.cpp:12
#1  0x0000000000400d90 in main (argc=1, argv=0x7fffffffd8e8) at example-3.cpp:19
(gdb)
```

There is a few things going on here:

- Start `gdb` whilist specifying the target binary (and optionally
  command line arguments)
- Call `run` to execute the program
- Your program crashes (or suceededs, in which it you don't need to do
  anything)
- Call `bt` 
- Marvel in the abilities of debuggers -- "You can do that in C++??"
- (Optional) Print some variables to figure out the source of the problem
- To exit `gdb`, call `kill` then `quit`

In your compiler, you are most likely going to pass your compilation source
from stdin. To `run` with a file as stdin, call `run <compilation_source.c`.
To call gdb with a set of command line arguments, add them after the
binary in gdb args. eg: `gdb --args ./hello 1 2 3`.

There is a lot more you can do with  gdb, like printing variables, calling
functions, inserting breakpoints or even manually step through your code.
A little demo of some of its features:

```bash
(gdb) call (int) printf("%d\n", argc)
1
$11 = 2
(gdb) call (int) printf("%s\n", argv[0])
/home/fyquah/dev/eie/langproc-2017-cw/debugging/example-3
$12 = 58
(gdb) call (int) printf("%s\n", argv[1])
(null)
$13 = 7
(gdb) print numbers
$14 = {<std::_Vector_base<int, std::allocator<int> >> = {_M_impl = {<std::allocator<int>> = {<__gnu_cxx::new_allocator<int>> = {<No data fields>}, <No data fields>}, _M_start = 0x0, _M_finish = 0x0,
      _M_end_of_storage = 0x0}}, <No data fields>}
(gdb) call (int) process_arguments(3, ARRAY_OF_NUMBERS)
$1 = 99
```

I hope this convinces you that figuring out how to use GDB is better than
writing random print statements in your program. 

- It saves you a lot of time because you don't have to insert random
  print statements until you find the source of the bug. (Think `O(log N)`
  vs `O(1)`)
- It saves you a lot of time because you don't have to remove the print
  statements later
- It saves you a lot of time because you can thoroughly examine the problem
  at the crash site.
- It saves you a lot of time because you don't have to add print statements
  at the crash site to figure out the variables' values.
- It saves you a lot of time because you don't have to remove those
  print statements too.

GDB is an extremely powerful tool, that you can use to debug embedded
applications over USB or running applications over the network. One of
the most interesting "inception-like" use cases I have heard of is getting
the values of registers in a coprocessor, in a VM, nested within 3 ssh
sessions.

## Valgrind

[Valgrind](http://valgrind.org) is a dynamic analysis tool to detect various
issues, such as threading bugs, memory management issues, cache-profiling
and heap-profiling. We are primarily interested in memory checking (Memcheck),
the dynamic analysis that helps us detect memory errors. Valgrind is available
in most linux distributions.

Consider the example in [example-leak.c](./example-leak.c). There are two
two problems with the code: (1) `n5->left` and `n5->right` are not
assigned appropriately and (2) `n5` is not allocated enough memory (as `sizeof(int)` <
`sizeof(struct tree_t)`. Surprisingly enough, the code doesn't always crash.

```bash
fyquah@olaf: debugging $ make example-leak
cc -Wall -g example-leak.c -o example-leak
fyquah@olaf: debugging $ ./example-leak 
        6
    5
        8
10
        9
    7
```

There is various explaination you can give to why this program doesn't
crash, but this is not a behaviour you should not rely upon. (
[This stackoverflow post](https://stackoverflow.com/questions/8029584/why-does-malloc-initialize-the-values-to-0-in-gcc/8029624#8029624)
gives insight to why the values are zero-ed out as to why there wasn't a
segmentation fault, try to recall how memory and page tables are organised
in an operating system.) To rectify this memory problem, we can use valgrind
to diagnose the source of the problem:


```bash
fyquah@olaf: debugging $ valgrind ./example-leak
==7177== Memcheck, a memory error detector
==7177== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==7177== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==7177== Command: ./example-leak
==7177==
        6
    5
        8
10
==7177== Invalid read of size 8
==7177==    at 0x400830: inorder_print (example-leak.c:54)
==7177==    by 0x400843: inorder_print (example-leak.c:54)
==7177==    by 0x400894: inorder_print (example-leak.c:59)
==7177==    by 0x4008C7: main (example-leak.c:67)
==7177==  Address 0x5203228 is 4 bytes after a block of size 4 alloc'd
==7177==    at 0x4C2CB5F: malloc (vg_replace_malloc.c:299)
==7177==    by 0x400757: build_binary_tree (example-leak.c:20)
==7177==    by 0x4008B2: main (example-leak.c:66)
==7177==
        9
==7177== Invalid read of size 8
==7177==    at 0x400881: inorder_print (example-leak.c:59)
==7177==    by 0x400843: inorder_print (example-leak.c:54)
==7177==    by 0x400894: inorder_print (example-leak.c:59)
==7177==    by 0x4008C7: main (example-leak.c:67)
==7177==  Address 0x5203230 is 12 bytes after a block of size 4 alloc'd
==7177==    at 0x4C2CB5F: malloc (vg_replace_malloc.c:299)
==7177==    by 0x400757: build_binary_tree (example-leak.c:20)
==7177==    by 0x4008B2: main (example-leak.c:66)
==7177==
    7
==7177==
==7177== HEAP SUMMARY:
==7177==     in use at exit: 124 bytes in 6 blocks
==7177==   total heap usage: 7 allocs, 1 frees, 1,148 bytes allocated
==7177==
==7177== LEAK SUMMARY:
==7177==    definitely lost: 24 bytes in 1 blocks
==7177==    indirectly lost: 100 bytes in 5 blocks
==7177==      possibly lost: 0 bytes in 0 blocks
==7177==    still reachable: 0 bytes in 0 blocks
==7177==         suppressed: 0 bytes in 0 blocks
==7177== Rerun with --leak-check=full to see details of leaked memory
==7177==
==7177== For counts of detected and suppressed errors, rerun with: -v
==7177== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

There are two main things happening:

1. There are two error messages about address with an incorrect amount of 
   memory allocated. This is an extremely good demonstration of the power of
   valgrind - (a) It tells you when. (Thought exercise: Where did the numbers,
   "Address 0x5203228 is 4 bytes" and "Address 0x5203230 is 12 bytes"
   come from?)
2. Memory leak (as expected), as we are not de-allocating memory ourselves.

You might be curious _why_ this even works, it is possible that the memory
location allocated by followed by the heap-allocated memory location is

Let's fix the `malloc` call, then see what happens:

```
fyquah@olaf: debugging $ make example-leak
fyquah@olaf: debugging $ valgrind ./example-leak
==7892== Memcheck, a memory error detector
==7892== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==7892== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==7892== Command: ./example-leak
==7892==
        6
    5
        8
10
==7892== Conditional jump or move depends on uninitialised value(s)
==7892==    at 0x40082A: inorder_print (example-leak.c:50)
==7892==    by 0x400843: inorder_print (example-leak.c:54)
==7892==    by 0x400843: inorder_print (example-leak.c:54)
==7892==    by 0x400894: inorder_print (example-leak.c:59)
==7892==    by 0x4008C7: main (example-leak.c:67)
==7892==
        9
==7892== Conditional jump or move depends on uninitialised value(s)
==7892==    at 0x40082A: inorder_print (example-leak.c:50)
==7892==    by 0x400894: inorder_print (example-leak.c:59)
==7892==    by 0x400843: inorder_print (example-leak.c:54)
==7892==    by 0x400894: inorder_print (example-leak.c:59)
==7892==    by 0x4008C7: main (example-leak.c:67)
==7892==
    7
==7892==
==7892== HEAP SUMMARY:
==7892==     in use at exit: 144 bytes in 6 blocks
==7892==   total heap usage: 7 allocs, 1 frees, 1,168 bytes allocated
==7892==
==7892== LEAK SUMMARY:
==7892==    definitely lost: 24 bytes in 1 blocks
==7892==    indirectly lost: 120 bytes in 5 blocks
==7892==      possibly lost: 0 bytes in 0 blocks
==7892==    still reachable: 0 bytes in 0 blocks
==7892==         suppressed: 0 bytes in 0 blocks
==7892== Rerun with --leak-check=full to see details of leaked memory
==7892==
==7892== For counts of detected and suppressed errors, rerun with: -v
==7892== Use --track-origins=yes to see where uninitialised values come from
==7892== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

This is conditional statement in line 50 of `example-leak.c` is dependent
on an uninitialised value. This problem is admitedly harder to debug with
valgrind alone and it is more likely that gdb will be more useful
(when it crashes).

Anyway, if we fix the assignments (by assigning `n5->left` and `n5->right`
to NULL), we are left with:

```bash
fyquah@olaf: debugging $ make example-leak
fyquah@olaf: debugging $ valgrind ./example-leak
==8192== Memcheck, a memory error detector
==8192== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==8192== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==8192== Command: ./example-leak
==8192==
        6
    5
        8
10
        9
    7
==8192==
==8192== HEAP SUMMARY:
==8192==     in use at exit: 144 bytes in 6 blocks
==8192==   total heap usage: 7 allocs, 1 frees, 1,168 bytes allocated
==8192==
==8192== LEAK SUMMARY:
==8192==    definitely lost: 24 bytes in 1 blocks
==8192==    indirectly lost: 120 bytes in 5 blocks
==8192==      possibly lost: 0 bytes in 0 blocks
==8192==    still reachable: 0 bytes in 0 blocks
==8192==         suppressed: 0 bytes in 0 blocks
==8192== Rerun with --leak-check=full to see details of leaked memory
==8192==
==8192== For counts of detected and suppressed errors, rerun with: -v
==8192== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

The only problem left will be memory leaks. It is not reported as an error,
because there are often times when heap memory freed simply by program
termination is sufficient.

Unless you are writing your compiler in C (or C++ that looks like C), the
main avenue you will bump into the memory block size issue is the following
type of code:

```C++
class Statement : public Node {
 ...
}

static void
some_function()
{
  Node *node = node;
  Statement *stmt = (Staement*) node;  // This is an invariant: I know what I am doing
}
```

In most cases, you actually don't know what you are doing. For that reason,
you really should avoid such casts where you can. Unfortunately, compiler
authors, inevitably, write compilers with a lot of hand-maintained
invariants. When you see yourself writing a type-cast as such in your code,
it is worth asking yourself if it is possible to modify your code slightly
to avoid this cast altogether. If that's not trivially possible and you have
to perform type-casting, use `static_cast<>` or `dynamic_cast<>`
(See [this stackoverflow post](https://stackoverflow.com/a/1255015)
for an explaination on their differences).

## Static Analysis

Valgrind and GDB falls under the category of dynamic analysis, that is you
run the program and use runtime information to look for errors. Another
common form of program analysis is [static analysis](https://en.wikipedia.org/wiki/Static_program_analysis),
that is a form of check that tries to find problems in your code without
executing it. There are a lot of such tools available (for free!) to perform
such analysis, some are built into modern IDEs such as XCode while some
others require some form of setup. We will limit our discussion to one
specific tool, the [clang static analyser](https://clang-analyzer.llvm.org/).

The clang static analyser generates a really nice web UI for you to navigate
through the errors. Static analyser tools, in general, have very low
false positive rates. That is, it may not necessarily report all errors,
but all errors it reports are very likely to be genuine errors.

Setting up the clang static analyser, it is surprisingly simple:

0. Install the `clang` and `llvm` toolchains on your local setup (This is
   already done in the lab machines)
1. `scan-build make your_make_target`
2. Wait for compilation. You will notice that compilation is noticeably
   slower due to static analysis running simultaneously.
3. You will get a message that tells you how many errors it has found and
   how to view them.
4. You can either call `scan-view` as instructed, or simply navigate to
   `/tmp/scan-build-XXXXXXX/index.html` on your web browser.

```
fyquah@olaf: project $ scan-build make
scan-build: Using '/usr/bin/clang-4.0' for static analysis
flex -o src/c_lexer.yy.c src/c_lexer.lex
.... (snip) ...
scan-build: 9 bugs found.
scan-build: Run 'scan-view /tmp/scan-build-2018-02-14-024355-359-1' to examine bug reports.
fyquah@olaf: project $
```
Try compiling with `scan-build make example-leak`. The static analyser should
have managed to some errors in the memory leak example (You might need to
run make clean to force the static analyser to recompile the binaries
whilist running static analysis). Try compiling the backtrace example with
`scan-build make example-backtrace-3`. What do you observe? Why?

---

To fully understand the amount of analysis that static analysers can do
in code analysis, here are some really pretty screenshots.

![alt text](./images/scan-view-2.png "Scan View 2")

*The Clang static analysis tool generates a really nice table to display the
list of errors (You will notice some errors from the parsers).*

![alt text](./images/scan-view-1.png "Scan View 1")

*When you click on the links, it displays a thorough analysis on the code
path that can lead to errors. In this case, it found a null pointer deferencing.*

![alt text](./images/scan-view-3.png "Scan View 3")

*Here is another example of an invalid pointer derefence it has found.
In this case, the pointer is derefencing a freed pointer.*

And if you have not guessed, this _was_ from my compiler for this very
course a few years ago. I hope this is sufficient to convince you to
at least consider using static analysis.

Static analysis tools are usually not included in compilers:

- Compilers and static analysis are really a decoupled products. You can
  use the clang static analyser with gcc, clang, MSVC or your very own
  compiler!
- Running static analysis takes time, and it is sometimes undesirable to
  have compilation time increase just because of static analysis. (Settings
  where you _don't_ want to run thorough static analysis)
- Compiler authors usually have a [long list of issues and features to work on](https://gcc.gnu.org/bugzilla/buglist.cgi?component=c%2B%2B&product=gcc&resolution=---).

## More Resources

There are endless amount of tools that are available for checking your code
and making sure they work and a lot of resources online to help you do that.
Unfortunately, it is unlikely that you have time to go through all of it.
Feel free to dig around the resources given below, should you have time or
wish to learn more about the subject, but you do not feel the need to do so

- [GDB User Manual](https://www.gnu.org/software/gdb/documentation/)
- [A GDB Tutorial from UMD](https://www.cs.umd.edu/~srhuang/teaching/cmsc212/gdb-tutorial-handout.pdf)
- [A GDB Tutorial from UMich](http://web.eecs.umich.edu/~sugih/pointers/gdbQS.html)
- [A simple example in using breakpoints with GDB](https://www.thegeekstuff.com/2010/03/debug-c-program-using-gdb/)
- [Valgrind User Manual](http://valgrind.org/docs/manual/manual.html) - There
  is a lot in this manual, for our use cases, we are interested in chapters
  1 - 3 (for introductory material) and chapter 4 (Memcheck).
- [cppcheck](http://cppcheck.sourceforge.net/) - A tool for C++ static analysis.
- And the most useful tool of all, a search engine (Now that you know what
  you need to search).

## Acknowledgement

Thanks to bla bla bla for reading through initial drafts of this document.
