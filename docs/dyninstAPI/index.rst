==========
DyninstAPI
==========

The normal cycle of developing a program is to edit the source code,
compile it, and then execute the resulting binary. However, sometimes
this cycle can be too restrictive. We may wish to change the program
while it is executing or after it has been linked, thus avoiding the
process of re-compiling, re-linking, or even re-executing the program to
change the binary. At first, this may seem like a bizarre goal, however,
there are several practical reasons why we may wish to have such a
system. For example, if we are measuring the performance of a program
and discover a performance problem, it might be necessary to insert
additional instrumentation into the program to understand the problem.
Another application is performance steering; for large simulations,
computational scientists often find it advantageous to be able to make
modifications to the code and data while the simulation is executing.

This document describes an Application Program Interface (API) to permit
the insertion of code into a computer application that is either running
or on disk. The API for inserting code into a running application,
called dynamic instrumentation, shares much of the same structure as the
API for inserting code into an executable file or library, known as
static instrumentation. The API also permits changing or removing
subroutine calls from the application program. Binary code changes are
useful to support a variety of applications including debugging,
performance monitoring, and to support composing applications out of
existing packages. The goal of this API is to provide a machine
independent interface to permit the creation of tools and applications
that use runtime and static code patching. The API and a simple test
application are described in [1]. This API is based on the idea of
dynamic instrumentation described in [3].

The key features of this interface are the abilities to:

-  Insert and change instrumentation in a running program.

-  Insert instrumentation into a binary on disk and write a new copy of
   that binary back to disk.

-  Perform static and dynamic analysis on binaries and processes.

The goal of this API is to keep the interface small and easy to
understand. At the same time, it needs to be sufficiently expressive to
be useful for a variety of applications. We accomplished this goal by
providing a simple set of abstractions and a way to specify which code
to insert into the application [1]_.

Abstractions
============

The DyninstAPI library provides an interface for instrumenting and
working with binaries and processes. The user writes a *mutator*, which
uses the DyninstAPI library to operate on the application. The process
that contains the *mutator* and DyninstAPI library is known as the
*mutator process*. The *mutator process* operates on other processes or
on-disk binaries, which are known as *mutatees.*

The API is based on abstractions of a program. For dynamic
instrumentation, it can be based on the state while in execution. The
two primary abstractions in the API are *points* and *snippets*. A
*point* is a location in a program where instrumentation can be
inserted. A *snippet* is a representation of some executable code to be
inserted into a program at a point. For example, if we wished to record
the number of times a procedure was invoked, the *point* would be entry
point of the procedure, and the *snippets* would be a statement to
increment a counter. *Snippets* can include conditionals and function
calls.

*Mutatees* are represented using an *address space* abstraction. For
dynamic instrumentation, the *address space* represents a process and
includes any dynamic libraries loaded with the process. For static
instrumentation, the *address space* includes a disk executable and
includes any dynamic library files on which the executable depends. The
*address space* abstraction is extended by *process* and *binary*
abstractions for dynamic and static instrumentation. The *process*
abstraction represents information about a running process such as
threads or stack state. The *binary* abstraction represents information
about a binary found on disk.

The code and data represented by an *address space* is broken up into
*function* and *variable* abstractions. *Function*\ s contain *points*,
which specify locations to insert instrumentation. *Functions* also
contain a *control flow graph* abstraction, which contains information
about *basic blocks*, *edges*, *loops*, and *instructions*. If the
*mutatee* contains debug information, DyninstAPI will also provide
abstractions about variable and function *types*, *local variables,*
*function parameters*, and *source code line information*. The
collection of *functions* and *variables* in a mutatee is represented as
an *image*.

The API includes a simple type system based on structural equivalence.
If mutatee programs have been compiled with debugging symbols and the
symbols are in a format that Dyninst understands, type checking is
performed on code to be inserted into the mutatee. See Section 4.28 for
a complete description of the type system.

Due to language constructs or compiler optimizations, it may be possible
for multiple functions to *overlap* (that is, share part of the same
function body) or for a single function to have multiple *entry points*.
In practice, it is impossible to determine the difference between
multiple overlapping functions and a single function with multiple entry
points. The DyninstAPI uses a model where each function (BPatch_function
object) has a single entry point, and multiple functions may overlap
(share code). We guarantee that instrumentation inserted in a particular
function is only executed in the context of that function, even if
instrumentation is inserted into a location that exists in multiple
functions.

Examples
========

To illustrate the ideas of the API, we present several short examples
that demonstrate how the API can be used. The full details of the
interface are presented in the next section. To prevent confusion, we
refer to the application process or binary that is being modified as the
mutatee, and the program that uses the API to modify the application as
the mutator. The mutator is a separate process from the application
process.

The examples in this section are simple code snippets, not complete
programs. Appendix A - Complete Examples provides several examples of
complete Dyninst programs.

Instrumenting a function
------------------------

A mutator program must create a single instance of the class BPatch.
This object is used to access functions and information that are global
to the library. It must not be destroyed until the mutator has
completely finished using the library. For this example, we assume that
the mutator program has declared a global variable called bpatch of
class BPatch.

All instrumentation is done with a BPatch_addressSpace object, which
allows us to write codes that work for both dynamic and static
instrumentation. During initialization we use either BPatch_process to
attach to or create a process, or BPatch_binaryEdit to open a file on
disk. When instrumentation is completed, we will either run the
BPatch_process, or write the BPatch_binaryEdit back onto the disk.

The mutator first needs to identify the application to be modified. If
the process is already in execution, this can be done by specifying the
executable file name and process id of the application as arguments in
order to create an instance of a process object:

BPatch_process \*appProc = bpatch.processAttach(name, processId);

This creates a new instance of the BPatch_process class that refers to
the existing process. It had no effect on the state of the process
(i.e., running or stopped). If the process has not been started, the
mutator specifies the pathname and argument list of a program it seeks
to execute:

BPatch_process \*appProc = bpatch.processCreate(pathname, argv);

If the mutator is opening a file for static binary rewriting, it
executes:

BPatch_binaryEdit \*appBin = bpatch.openBinary(pathname);

The above statements create either a BPatch_process object or
BPatch_binaryEdit object, depending on whether Dyninst is doing dynamic
or static instrumentation. The instrumentation and analysis code can be
made agnostic towards static or dynamic modes by using a
BPatch_addressSpace object. Both BPatch_process and BPatch_binaryEdit
inherit from BPatch_addressSpace, so we can use cast operations to move
between the two:

BPatch_process \*appProc = static_cast<BPatch_process \*>(appAddrSpace)

-or-

BPatch_binaryEdit \*appBin = static_cast<BPatch_binaryEdit
\*>(appAddrSpace)

Similarly, all instrumentation commands can be performed on a
BPatch_addressSpace object, allowing similar codes to be used between
dynamic instrumentation and binary rewriting:

BPatch_addressSpace \*app = appProc;

-or-

BPatch_addressSpace \*app = appBin;

Once the address space has been created, the mutator defines the snippet
of code to be inserted and identifies where the points should be
inserted.

If the mutator wants to instrument the entry point of
InterestingProcedure, it should get a BPatch_function from the
application’s BPatch_image, and get the entry BPatch_point from that
function:

std::vector<BPatch_function \*> functions;

std::vector<BPatch_point \*> \*points;

BPatch_image \*appImage = app->getImage();

appImage->findFunction(“InterestingProcedure”, functions);

points = functions[0]->findPoint(BPatch_locEntry);

The mutator also needs to construct the instrumentation that it will
insert at the BPatch_point. It can do this by allocating an integer in
the application to store instrumentation results, and then creating a
BPatch_snippet to increment that integer:

BPatch_variableExpr \*intCounter =

   app->malloc(*(appImage->findType("int")));

BPatch_arithExpr addOne(BPatch_assign, \*intCounter,

BPatch_arithExpr(BPatch_plus, \*intCounter, BPatch_constExpr(1)));

The mutator can set the BPatch_snippet to be run at the BPatch_point by
executing an insert­Snippet call:

app->insertSnippet(addOne, \*points);

Finally, the mutator should either continue the mutate process and wait
for it to finish, or write the resulting binary onto the disk, depending
on whether it is doing dynamic or static instrumentation:

appProc->continueExecution();

while (!appProc->isTerminated()) {

bpatch.waitForStatusChange();

}

-or-

appBin->writeFile(newPath);

A complete example can be found in Appendix A - Complete Examples.

Binary Analysis
---------------

This example will illustrate how to use Dyninst to iterate over a
function’s control flow graph and inspect instructions. These are steps
that would usually be part of a larger data flow or control flow
analysis. Specifically, this example will collect every basic block in a
function, iterate over them, and count the number of instructions that
access memory.

Unlike the previous instrumentation example, this example will analyze a
binary file on disk. Bear in mind, these techniques can also be applied
when working with processes. This example makes use of InstructionAPI,
details of which can be found in the InstructionAPI Reference Manual.

Similar to the above example, the mutator will start by creating a
BPatch object and opening a file to operate on:

BPatch bpatch;

BPatch_binaryEdit \*binedit = bpatch.openFile(pathname);

The mutator needs to get a handle to a function to do analysis on. This
example will look up a function by name; alternatively, it could have
iterated over every function in BPatch_image or BPatch_module:

BPatch_image \*appImage = binedit->getImage();

std::vector<BPatch_function \*> funcs;

image->findFunction(“InterestingProcedure”, funcs);

A function’s control flow graph is represented by the BPatch_flowGraph
class. The BPatch_flowGraph contains, among other things, a set of
BPatch_basicBlock objects connected by BPatch_edge objects. This example
will simply collect a list of the basic blocks in BPatch_flowGraph and
iterate over each one:

BPatch_flowGraph \*fg = funcs[0]->getCFG();

std::set<BPatch_basicBlock \*> blocks;

fg->getAllBasicBlocks(blocks);

| Each basic block has a list of instructions. Each instruction is
  represented by a
| Dyninst::InstructionAPI::Instruction::Ptr object.

std::set<BPatch_basicBlock \*>::iterator block_iter;

for (block_iter = blocks.begin(); block_iter != blocks.end();
++block_iter) {

BPatch_basicBlock \*block = \*block_iter;

std::vector<Dyninst::InstructionAPI::Instruction::Ptr> insns;

block->getInstructions(insns);

}

Given an Instruction object, which is described in the InstructionAPI
Reference Manual, we can query for properties of this instruction.
InstructionAPI has numerous methods for inspecting the memory accesses,
registers, and other properties of an instruction. This example simply
checks whether this instruction accesses memory:

std::vector<Dyninst::InstructionAPI::Instruction::Ptr>::iterator

   insn_iter;

for (insn_iter = insns.begin(); insn_iter != insns.end(); ++insn_iter)

{

   Dyninst::InstructionAPI::Instruction::Ptr insn = \*insn_iter;

   if (insn->readsMemory() \|\| insn->writesMemory()) {

   insns_access_memory++;

   }

}

Instrumenting Memory Accesses
-----------------------------

There are two snippets useful for memory access instrumentation:
BPatch_effectiveAddressExpr and BPatch_bytesAccessedExpr. Both have
nullary constructors; the result of the snippet depends on the
instrumentation point where the snippet is inserted.
BPatch_effectiveAddressExpr has type void*, while
BPatch_bytesAccessedExpr has type int.

These snippets may be used to instrument a given instrumentation point
if and only if the point has memory access information attached to it.
In this release the only way to create instrumentation points that have
memory access information attached is via
BPatch_function.findPoint(const std::set<BPatch_opCode>&). For example,
to instrument all the loads and stores in a function named
InterestingProcedure with a call to printf, one may write:

BPatch_addressSpace \*app = ...;

BPatch_image \*appImage = proc->getImage();

// We’re interested in loads and stores

std::set<BPatch_opCode> axs;

axs.insert(BPatch_opLoad);

axs.insert(BPatch_opStore);

// Scan the function InterestingProcedure and create instrumentation
points

std::vector<BPatch_function*> funcs;

appImage->findFunction(“InterestingProcedure”, funcs);

std::vector<BPatch_point*>\* points = funcs[0]->findPoint(axs);

// Create the printf function call snippet

std::vector<BPatch_snippet*> printfArgs;

BPatch_snippet \*fmt = new BPatch_constExpr("Access at: %p.\n");

printfArgs.push_back(fmt);

BPatch_snippet \*eae = new BPatch_effectiveAddressExpr();

printfArgs.push_back(eae);

// Find the printf function

std::vector<BPatch_function \*> printfFuncs;

appImage->findFunction("printf", printfFuncs);

// Construct the function call snippet

BPatch_funcCallExpr printfCall(*(printfFuncs[0]), printfArgs);

// Insert the snippet at the instrumentation points

app->insertSnippet(printfCall, \*points);



Using DyninstAPI with the component libraries
=============================================

In this section, we describe how to access the underlying component
library abstractions from corresponding Dyninst abstractions. The
component libraries (SymtabAPI, InstructionAPI, ParseAPI, and PatchAPI)
often provide greater functionality and cleaner interfaces than Dyninst,
and thus users may wish to use a mix of abstractions. In general, users
may access component library abstractions via a convert function, which
is overloaded and namespaced to give consistent behavior. The
definitions of all component library abstractions are located in the
appropriate documentation.

PatchAPI::PatchMgrPtr PatchAPI::convert(BPatch_addressSpace \*);

PatchAPI::PatchObject \*PatchAPI::convert(BPatch_object \*);

ParseAPI::CodeObject \*ParseAPI::convert(BPatch_object \*);

SymtabAPI::Symtab \*SymtabAPI::convert(BPatch_object \*);

SymtabAPI::Module \*SymtabAPI::convert(BPatch_module \*);

PatchAPI::PatchFunction \*PatchAPI::convert(BPatch_function \*);

ParseAPI::Function \*ParseAPI::convert(BPatch_function \*);

PatchAPI::PatchBlock \*PatchAPI::convert(BPatch_basicBlock \*);

ParseAPI::Block \*ParseAPI::convert(BPatch_basicBlock \*);

PatchAPI::PatchEdge \*PatchAPI::convert(BPatch_edge \*);

ParseAPI::Edge \*ParseAPI::convert(BPatch_edge \*);

PatchAPI::Point \*PatchAPI::convert(BPatch_point \*, BPatch_callWhen);

PatchAPI::SnippetPtr PatchAPI::convert(BPatch_snippet \*);

SymtabAPI::Type \*SymtabAPI::convert(BPatch_type \*);

Using the API
=============

In this section, we describe the steps needed to compile your mutator
and mutatee programs and to run them. First we give you an overview of
the major steps and then we explain each one in detail.

Overview of Major Steps
-----------------------

To use Dyninst, you have to:

(1) *Build and install DyninstAP:* DyninstAPI can be installed a package
    system such as Spack or can be compiled from source. Our github
    webpage contains detailed instructions for installing Dyninst:
    https://github.com/dyninst/dyninst.

(2) *Create a mutator program (Section 6.2):* You need to create a
    program that will modify some other program. For an example, see the
    mutator shown in Appendix A.

(3) *Set up the mutatee (Section 6.3):* On some platforms, you need to
    link your application with Dyninst’s run time instrumentation
    library. [**NOTE**: This step is only needed in the current release
    of the API. Future releases will eliminate this restriction.]

(4) *Run the mutator (Section 6.4):* The mutator will either create a
    new process or attach to an existing one (depending on the whether
    createProcess or attachProcess is used).

Sections 6.2 through 6.4 explain these steps in more detail.

Creating a Mutator Program
--------------------------

The first step in using Dyninst is to create a mutator program. The
mutator program specifies the mutatee (either by naming an executable to
start or by supplying a process ID for an existing process). In
addition, your mutator will include the calls to the API library to
modify the mutatee. For the rest of this section, we assume that the
mutator is the sample program given in Appendix A - Complete Examples.

The following fragment of a Makefile shows how to link your mutator
program with the Dyninst library on most platforms:

   # DYNINST_INCLUDE and DYNINST_LIB should be set to locations

   # where Dyninst header and library files were installed, respectively

retee.o: retee.c

$(CC) -c $(CFLAGS) -I$(DYNINST_INCLUDE) retee.c –std=c++11x

| retee: retee.o
| $(CC) retee.o -L$(DYNINST_LIB) -ldyninstAPI -o retee –std=c++11x

On Linux, the options -lelf and -ldw may be required at the link step.
You will also need to make sure that the LD_LIBRARY_PATH environment
variable includes the directory that contains the Dyninst shared
library.

Since Dyninst uses the C++11x standard, you will also need to enable
this option for your compiler. For GCC versions 4.3 and later, this is
done by specifying -std=c++0x. For GCC versions 4.7 and later, this is
done by specifying -std=c++11. Some of these libraries, such as libdwarf
and libelf, may not be standard on various platforms. Check the README
file in dyninst/dyninstAPI for more information on where to find these
libraries.

Under Windows NT, the mutator also needs to be linked with the dbghelp
library, which is included in the Microsoft Platform SDK. Below is a
fragment from a Makefile for Windows NT:

   # DYNINST_INCLUDE and DYNINST_LIB should be set to locations

   # where Dyninst header and library files were installed, respectively

CC = cl

retee.obj: retee.c

   $(CC) -c $(CFLAGS) -I$(DYNINST_INCLUDE)/h

| retee.exe: retee.obj
| link -out:retee.exe retee.obj $(DYNINST_LIB)\libdyninstAPI.lib \\

   dbghelp.lib

Setting Up the Application Program (mutatee)
--------------------------------------------

On most platforms, any additional code that your mutator might need to
call in the mutatee (for example files containing instrumentation
functions that were too complex to write directly using the API) can be
put into a dynamically loaded shared library, which your mutator program
can load into the mutatee at runtime using the loadLibrary member
function of BPatch_process.

To locate the runtime library that Dyninst needs to load into your
program, an additional environment variable must be set. The variable
DYNINSTAPI_RT_LIB should be set to the full pathname of the run time
instrumentation library, which should be:

   NOTE: DYNINST_LIB should be set to the location where Dyninst library
   files were installed

$(DYNINST_LIB)/libdyninstAPI_RT.so (UNIX)

%DYNINST_LIB/libdyninstAPI_RT.dll (Windows)

Running the Mutator
-------------------

At this point, you should be ready to run your application program with
your mutator. For example, to start the sample program shown in Appendix
A - Complete Examples:

% retee foo <pid>

Optimizing Dyninst Performance
------------------------------

This section describes how to tune Dyninst for optimum performance.
During the course of a run, Dyninst will perform several types of
analysis on the binary, make safety assumptions about instrumentation
that is inserted, and rewrite the binary (perhaps several times). Given
some guidance from the user, Dyninst can make assumptions about what
work it needs to do and can deliver significant performance
improvements.

There are two areas of Dyninst performance users typically care about.
First, the time it takes Dyninst to parse and instrument a program. This
is typically the time it takes Dyninst to start and analyze a program,
and the time it takes to modify the program when putting in
instrumentation. Second, many users care about the time instrumentation
takes in the modified mutatee. This time is highly dependent on both the
amount and type of instrumentation put it, but it is still possible to
eliminate some of the Dyninst overhead around the instrumentation.

The following subsections describe techniques for improving the
performance of these two areas.

Optimizing Mutator Performance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CPU time in the Dyninst mutator is usually consumed by either parsing or
instrumenting binaries. When a new binary is loaded, Dyninst will
analyze the code looking for instrumentation points, global variables,
and attempting to identify functions in areas of code that may not have
symbols. Upon user request, Dyninst will also parse debug information
from the binary, which includes local variable, line, and type
information.

Since Dyninst 10.0.0, Dyninst supports parsing binaries in parallel,
which significantly improve the analysis speed. We typically have about
4X speedup when analyzing binaries with 8 threads. By default, Dyninst
will use all the available cores on your system. Please set environment
variable OMP_NUM_THREADS to the number of desired threads.

Debugging information is lazily parsed separately from the rest of the
binary parsing. Accessing line, type, or local variable information will
cause Dyninst to parse the debug information for all three of these.

Another common source of mutator time is spent re-writing the mutatee to
add instrumentation. When instrumentation is inserted into a function,
Dyninst may need to rewrite some or all of the function to fit the
instrumentation in. If multiple pieces of instrumentation are being
inserted into a function, Dyninst may need to rewrite that function
multiple times.

If the user knows that they will be inserting multiple pieces of
instrumentation into one function, they can batch the instrumentation
into one bundle, so that the function will only be re-written once,
using the BPatch_process::beginInsertionSet and
BPatch_­process::end­Inser­tion­Set functions (see section 4.4). Using
these functions can result in a significant performance win when
inserting instrumentation in many locations.

To use the insertion set functions, add a call to beginInsertionSet
before inserting instrumentation. Dyninst will start buffering up all
instrumentation insertions. After the last piece of instrumentation is
inserted, call finalizeInsertionSet, and all instrumentation will be
atomically inserted into the mutatee, with each function being rewritten
at most once.

Optimizing Mutatee Performance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As instrumentation is inserted into a mutatee, it will start to run
slower. The slowdown is heavily influenced by three factors: the number
of points being instrumented, the instrumentation itself, and the
Dyninst overhead around each piece of instrumentation. The Dyninst
overhead comes from pieces of protection code (described in more detail
below) that do things such as saving/restoring registers around
instrumentation, checking for instrumentation recursion, and performing
thread safety checks.

The factor by which Dyninst overhead influences mutatee run-time depends
on the type of instrumentation being inserted. When inserting
instrumentation that runs a memory cache simulator, the Dyninst overhead
may be negligible. On the other-hand, when inserting instrumentation
that increments a counter, the Dyninst overhead will dominate the time
spent in instrumentation. Remember, optimizing the instrumentation being
inserted may sometimes be more important than optimizing the Dyninst
overhead. Many users have had success writing tools that make use of
Dyninst’s ability to dynamically remove instrumentation as a performance
improvement.

The instrumentation overhead results from safety and correctness checks
inserted by Dyninst around instrumentation. Dyninst will automatically
attempt to remove as much of this overhead as possible, however it
sometimes must make a conservative decision to leave the overhead in.
Given additional, user-provided information Dyninst can make better
choices about what safety checks to leave in. An unoptimized
post-Dyninst 5.0 instrumentation snippet looks like the following:

+----------------------------------+----------------------------------+
| **Save General Purpose           | In order to ensure that          |
| Registers**                      | instrumentation doesn’t corrupt  |
|                                  | the program, Dyninst saves all   |
|                                  | live general purpose registers.  |
+----------------------------------+----------------------------------+
| **Save Floating Point            | Dyninst may decide to separately |
| Registers**                      | save any floating point          |
|                                  | registers that may be corrupted  |
|                                  | by instrumentation.              |
+----------------------------------+----------------------------------+
| **Generate A Stack Frame**       | Dyninst builds a stack frame for |
|                                  | instrumentation to run under.    |
|                                  | This provides the illusion to    |
|                                  | instrumentation that it is       |
|                                  | running as its own function.     |
+----------------------------------+----------------------------------+
| **Calculate Thread Index**       | Calculate an index value that    |
|                                  | identifies the current thread.   |
|                                  | This is primarily used as input  |
|                                  | to the Trampoline Guard.         |
+----------------------------------+----------------------------------+
| **Test and Set Trampoline        | Test to see if we are already    |
| Guard**                          | recursively executing under      |
|                                  | instrumentation, and skip the    |
|                                  | user instrumentation if we are.  |
+----------------------------------+----------------------------------+
| **Execute User Instrumentation** | Execute any BPatch_snippet code. |
+----------------------------------+----------------------------------+
| **Unset Trampoline Guard**       | Marks the this thread as no      |
|                                  | longer being in instrumentation  |
+----------------------------------+----------------------------------+
| **Clean Stack Frame**            | Clean the stack frame that was   |
|                                  | generated for instrumentation.   |
+----------------------------------+----------------------------------+
| **Restore Floating Point         | Restore the floating point       |
| Registers**                      | registers to their original      |
|                                  | state.                           |
+----------------------------------+----------------------------------+
| **Restore General Purpose        | Restore the general purpose      |
| Registers**                      | registers to their original      |
|                                  | state.                           |
+----------------------------------+----------------------------------+

Dyninst will attempt to eliminate as much of its overhead as is
possible. The Dyninst user can assist Dyninst by doing the following:

-  **Write BPatch_snippet code that avoids making function calls.**
   Dyninst will attempt to perform analysis on the user written
   instrumentation to determine which general purpose and floating point
   registers can be saved. It is difficult to analyze function calls
   that may be nested arbitrarily deep. Dyninst will not analyze any
   deeper than two levels of function calls before assuming that the
   instrumentation clobbers all registers and it needs to save
   everything.

..

   In addition, not making function calls from instrumentation allows
   Dyninst to eliminate its tramp guard and thread index calculation.
   Instrumentation that does not make a function call cannot recursively
   execute more instrumentation.

-  **Call BPatch::setTrampRecursive(true) if instrumentation cannot
   execute recursively.** If instrumentation must make a function call,
   but will not execute recursively, then enable trampoline recursion.
   This will cause Dyninst to stop generating a trampoline guard and
   thread index calculation on all future pieces of instrumentation. An
   example of instrumentation recursion would be instrumenting a call to
   write with instrumentation that calls printf—write will start calling
   printf printf will re-call write.

-  **Call BPatch::setSaveFPR(false) if instrumentation will not clobber
   floating point registers**. This will cause Dyninst to stop saving
   floating point registers, which can be a significant win on some
   platforms.

-  **Use simple BPatch_snippet objects when possible**. Dyninst will
   attempt to recognize, peep-hole optimize, and simplify frequently
   used code snippets when it finds them. For example, on x86 based
   platforms Dyninst will recognize snippets that do operations like
   ‘var = constant’ or ‘var++’ and turn these into optimized assembly
   instructions that take advantage of CISC machine instructions.

-  **Call BPatch::setInstrStackFrames(false) before inserting
   instrumentation that does not need to set up stack frames. Dyninst
   allows you to force stack frames to be generated for all
   instrumentation. This is useful for some applications (e.g.,
   debugging your instrumentation code) but allowing Dyninst to omit
   stack frames wherever possible will improve performance. This flag is
   false by default; it should be enabled for as little instrumentation
   as possible in order to maximize the benefit from optimizing away
   stack frames.**

-  **Avoid conditional instrumentation wherever possible.** Conditional
   logic in your instrumentation makes it more difficult to avoid saving
   the state of the flags.

-  **Avoid unnecessary instrumentation.** Dyninst provides you with all
   kinds of information that you can use to select only the points of
   actual interest for instrumentation. Use this information to
   instrument as selectively as possible. The best way to optimize your
   instrumentation, ultimately, is to know *a priori* that it was
   unnecessary and not insert it.

Appendix A - Complete Examples
==============================

In this section we show two complete examples: the programs from Section
3 and a complete Dyninst program, retee.

.. _instrumenting-a-function-1:

Instrumenting a function
------------------------

#include <stdio.h>

#include "BPatch.h"

#include "BPatch_addressSpace.h"

#include "BPatch_process.h"

#include "BPatch_binaryEdit.h"

#include "BPatch_point.h"

#include "BPatch_function.h"

using namespace std;

using namespace Dyninst;

// Create an instance of class BPatch

BPatch bpatch;

// Different ways to perform instrumentation

typedef enum {

create,

attach,

open

} accessType_t;

// Attach, create, or open a file for rewriting

BPatch_addressSpace\* startInstrumenting(accessType_t accessType,

const char\* name,

int pid,

const char\* argv[]) {

BPatch_addressSpace\* handle = NULL;

switch(accessType) {

case create:

handle = bpatch.processCreate(name, argv);

if (!handle) { fprintf(stderr, "processCreate failed\n"); }

break;

case attach:

handle = bpatch.processAttach(name, pid);

if (!handle) { fprintf(stderr, "processAttach failed\n"); }

break;

case open:

// Open the binary file and all dependencies

handle = bpatch.openBinary(name, true);

if (!handle) { fprintf(stderr, "openBinary failed\n"); }

break;

}

return handle;

}

// Find a point at which to insert instrumentation

std::vector<BPatch_point*>\* findPoint(BPatch_addressSpace\* app,

const char\* name,

BPatch_procedureLocation loc) {

std::vector<BPatch_function*> functions;

std::vector<BPatch_point*>\* points;

// Scan for functions named "name"

BPatch_image\* appImage = app->getImage();

appImage->findFunction(name, functions);

if (functions.size() == 0) {

fprintf(stderr, "No function %s\n", name);

return points;

} else if (functions.size() > 1) {

fprintf(stderr, "More than one %s; using the first one\n", name);

}

// Locate the relevant points

points = functions[0]->findPoint(loc);

return points;

}

// Create and insert an increment snippet

bool createAndInsertSnippet(BPatch_addressSpace\* app,

std::vector<BPatch_point*>\* points) {

BPatch_image\* appImage = app->getImage();

// Create an increment snippet

BPatch_variableExpr\* intCounter =

app->malloc(*(appImage->findType("int")), "myCounter");

BPatch_arithExpr addOne(BPatch_assign,

\*intCounter,

BPatch_arithExpr(BPatch_plus,

\*intCounter,

BPatch_constExpr(1)));

// Insert the snippet

if (!app->insertSnippet(addOne, \*points)) {

fprintf(stderr, "insertSnippet failed\n");

return false;

}

return true;

}

// Create and insert a printf snippet

bool createAndInsertSnippet2(BPatch_addressSpace\* app,

std::vector<BPatch_point*>\* points) {

BPatch_image\* appImage = app->getImage();

// Create the printf function call snippet

std::vector<BPatch_snippet*> printfArgs;

BPatch_snippet\* fmt =

new BPatch_constExpr("InterestingProcedure called %d times\n");

printfArgs.push_back(fmt);

BPatch_variableExpr\* var = appImage->findVariable("myCounter");

if (!var) {

fprintf(stderr, "Could not find 'myCounter' variable\n");

return false;

} else {

printfArgs.push_back(var);

}

// Find the printf function

std::vector<BPatch_function*> printfFuncs;

appImage->findFunction("printf", printfFuncs);

if (printfFuncs.size() == 0) {

fprintf(stderr, "Could not find printf\n");

return false;

}

// Construct a function call snippet

BPatch_funcCallExpr printfCall(*(printfFuncs[0]), printfArgs);

// Insert the snippet

if (!app->insertSnippet(printfCall, \*points)) {

fprintf(stderr, "insertSnippet failed\n");

return false;

}

return true;

}

void finishInstrumenting(BPatch_addressSpace\* app, const char\*
newName)

{

BPatch_process\* appProc = dynamic_cast<BPatch_process*>(app);

BPatch_binaryEdit\* appBin = dynamic_cast<BPatch_binaryEdit*>(app);

if (appProc) {

if (!appProc->continueExecution()) {

fprintf(stderr, "continueExecution failed\n");

}

while (!appProc->isTerminated()) {

bpatch.waitForStatusChange();

}

} else if (appBin) {

if (!appBin->writeFile(newName)) {

fprintf(stderr, "writeFile failed\n");

}

}

}

int main() {

// Set up information about the program to be instrumented

const char\* progName = "InterestingProgram";

int progPID = 42;

const char\* progArgv[] = {"InterestingProgram", "-h", NULL};

accessType_t mode = create;

// Create/attach/open a binary

BPatch_addressSpace\* app =

startInstrumenting(mode, progName, progPID, progArgv);

if (!app) {

fprintf(stderr, "startInstrumenting failed\n");

exit(1);

}

// Find the entry point for function InterestingProcedure

const char\* interestingFuncName = "InterestingProcedure";

std::vector<BPatch_point*>\* entryPoint =

findPoint(app, interestingFuncName, BPatch_entry);

if (!entryPoint \|\| entryPoint->size() == 0) {

fprintf(stderr, "No entry points for %s\n", interestingFuncName);

exit(1);

}

// Create and insert instrumentation snippet

if (!createAndInsertSnippet(app, entryPoint)) {

fprintf(stderr, "createAndInsertSnippet failed\n");

exit(1);

}

// Find the exit point of main

std::vector<BPatch_point*>\* exitPoint =

findPoint(app, "main", BPatch_exit);

if (!exitPoint \|\| exitPoint->size() == 0) {

fprintf(stderr, "No exit points for main\n");

exit(1);

}

// Create and insert instrumentation snippet 2

if (!createAndInsertSnippet2(app, exitPoint)) {

fprintf(stderr, "createAndInsertSnippet2 failed\n");

exit(1);

}

// Finish instrumentation

const char\* progName2 = "InterestingProgram-rewritten";

finishInstrumenting(app, progName2);

}

.. _binary-analysis-1:

Binary Analysis
---------------

#include <stdio.h>

#include "BPatch.h"

#include "BPatch_addressSpace.h"

#include "BPatch_process.h"

#include "BPatch_binaryEdit.h"

#include "BPatch_function.h"

#include "BPatch_flowGraph.h"

using namespace std;

using namespace Dyninst;

// Create an instance of class BPatch

BPatch bpatch;

// Different ways to perform instrumentation

typedef enum {

create,

attach,

open

} accessType_t;

BPatch_addressSpace\* startInstrumenting(accessType_t accessType,

const char\* name,

int pid,

const char\* argv[]) {

BPatch_addressSpace\* handle = NULL;

switch(accessType) {

case create:

handle = bpatch.processCreate(name, argv);

if (!handle) { fprintf(stderr, "processCreate failed\n"); }

break;

case attach:

handle = bpatch.processAttach(name, pid);

if (!handle) { fprintf(stderr, "processAttach failed\n"); }

break;

case open:

// Open the binary file and all dependencies

handle = bpatch.openBinary(name, true);

if (!handle) { fprintf(stderr, "openBinary failed\n"); }

break;

}

return handle;

}

int binaryAnalysis(BPatch_addressSpace\* app) {

BPatch_image\* appImage = app->getImage();

int insns_access_memory = 0;

std::vector<BPatch_function*> functions;

appImage->findFunction("InterestingProcedure", functions);

if (functions.size() == 0) {

fprintf(stderr, "No function InterestingProcedure\n");

return insns_access_memory;

} else if (functions.size() > 1) {

fprintf(stderr, "More than one InterestingProcedure; using the first
one\n");

}

BPatch_flowGraph\* fg = functions[0]->getCFG();

std::set<BPatch_basicBlock*> blocks;

fg->getAllBasicBlocks(blocks);

for (auto block_iter = blocks.begin();

block_iter != blocks.end();

++block_iter) {

BPatch_basicBlock\* block = \*block_iter;

std::vector<InstructionAPI::Instruction::Ptr> insns;

block->getInstructions(insns);

for (auto insn_iter = insns.begin();

insn_iter != insns.end();

++insn_iter) {

InstructionAPI::Instruction::Ptr insn = \*insn_iter;

if (insn->readsMemory() \|\| insn->writesMemory()) {

insns_access_memory++;

}

}

}

return insns_access_memory;

}

int main() {

// Set up information about the program to be instrumented

const char\* progName = "InterestingProgram";

int progPID = 42;

const char\* progArgv[] = {"InterestingProgram", "-h", NULL};

accessType_t mode = create;

// Create/attach/open a binary

BPatch_addressSpace\* app =

startInstrumenting(mode, progName, progPID, progArgv);

if (!app) {

fprintf(stderr, "startInstrumenting failed\n");

exit(1);

}

int memAccesses = binaryAnalysis(app);

fprintf(stderr, "Found %d memory accesses\n", memAccesses);

}

.. _instrumenting-memory-accesses-1:

Instrumenting Memory Accesses
-----------------------------

#include <stdio.h>

#include "BPatch.h"

#include "BPatch_addressSpace.h"

#include "BPatch_process.h"

#include "BPatch_binaryEdit.h"

#include "BPatch_point.h"

#include "BPatch_function.h"

using namespace std;

using namespace Dyninst;

// Create an instance of class BPatch

BPatch bpatch;

// Different ways to perform instrumentation

typedef enum {

create,

attach,

open

} accessType_t;

// Attach, create, or open a file for rewriting

BPatch_addressSpace\* startInstrumenting(accessType_t accessType,

const char\* name,

int pid,

const char\* argv[]) {

BPatch_addressSpace\* handle = NULL;

switch(accessType) {

case create:

handle = bpatch.processCreate(name, argv);

if (!handle) { fprintf(stderr, "processCreate failed\n"); }

break;

case attach:

handle = bpatch.processAttach(name, pid);

if (!handle) { fprintf(stderr, "processAttach failed\n"); }

break;

   case open:

// Open the binary file; do not open dependencies

handle = bpatch.openBinary(name, false);

if (!handle) { fprintf(stderr, "openBinary failed\n"); }

break;

}

return handle;

}

bool instrumentMemoryAccesses(BPatch_addressSpace\* app) {

BPatch_image\* appImage = app->getImage();

// We're interested in loads and stores

BPatch_Set<BPatch_opCode> axs;

axs.insert(BPatch_opLoad);

axs.insert(BPatch_opStore);

// Scan the function InterestingProcedure

// and create instrumentation points

std::vector<BPatch_function*> functions;

appImage->findFunction("InterestingProcedure", functions);

std::vector<BPatch_point*>\* points =

functions[0]->findPoint(axs);

if (!points) {

fprintf(stderr, "No load/store points found\n");

return false;

}

// Create the printf function call snippet

std::vector<BPatch_snippet*> printfArgs;

BPatch_snippet\* fmt = new BPatch_constExpr("Access at: 0x%lx\n");

printfArgs.push_back(fmt);

BPatch_snippet\* eae = new BPatch_effectiveAddressExpr();

printfArgs.push_back(eae);

// Find the printf function

std::vector<BPatch_function*> printfFuncs;

appImage->findFunction("printf", printfFuncs);

if (printfFuncs.size() == 0) {

fprintf(stderr, "Could not find printf\n");

return false;

}

// Construct a function call snippet

BPatch_funcCallExpr printfCall(*(printfFuncs[0]), printfArgs);

// Insert the snippet at the instrumentation points

if (!app->insertSnippet(printfCall, \*points)) {

fprintf(stderr, "insertSnippet failed\n");

return false;

}

return true;

}

void finishInstrumenting(BPatch_addressSpace\* app, const char\*
newName) {

BPatch_process\* appProc = dynamic_cast<BPatch_process*>(app);

BPatch_binaryEdit\* appBin = dynamic_cast<BPatch_binaryEdit*>(app);

if (appProc) {

if (!appProc->continueExecution()) {

fprintf(stderr, "continueExecution failed\n");

}

while (!appProc->isTerminated()) {

bpatch.waitForStatusChange();

}

} else if (appBin) {

if (!appBin->writeFile(newName)) {

fprintf(stderr, "writeFile failed\n");

}

}

}

int main() {

// Set up information about the program to be instrumented

const char\* progName = "InterestingProgram";

int progPID = 42;

const char\* progArgv[] = {"InterestingProgram", "-h", NULL};

accessType_t mode = create;

// Create/attach/open a binary

BPatch_addressSpace\* app =

startInstrumenting(mode, progName, progPID, progArgv);

if (!app) {

fprintf(stderr, "startInstrumenting failed\n");

exit(1);

}

// Instrument memory accesses

if (!instrumentMemoryAccesses(app)) {

fprintf(stderr, "instrumentMemoryAccesses failed\n");

exit(1);

}

// Finish instrumentation

const char\* progName2 = "InterestingProgram-rewritten";

finishInstrumenting(app, progName2);

}

retee
-----

The final example is a program called “re-tee.” It takes three
arguments: the pathname of an executable program, the process id of a
running instance of the same program, and a file name. It adds code to
the running program that copies to the named file all output that the
program writes to its standard output file descriptor. In this way it
works like “tee,” which passes output along to its own standard out
while also saving it in a file. The motivation for the example program
is that you run a program, and it starts to print copious lines of
output to your screen, and you wish to save that output in a file
without having to re-run the program.

#include <stdio.h>

#include <fcntl.h>

#include <vector>

#include "BPatch.h"

#include "BPatch_point.h"

#include "BPatch_process.h"

#include "BPatch_function.h"

#include "BPatch_thread.h"

/\*

\* retee.C

\*

\* This program (mutator) provides an example of several facets of

\* Dyninst's behavior, and is a good basis for many Dyninst

\* mutators. We want to intercept all output from a target application

\* (the mutatee), duplicating output to a file as well as the

\* original destination (e.g., stdout).

\*

\* This mutator operates in several phases. In brief:

\* 1) Attach to the running process and get a handle (BPatch_process

\* object)

\* 2) Get a handle for the parsed image of the mutatee for function

\* lookup (BPatch_image object)

\* 3) Open a file for output

\* 3a) Look up the "open" function

\* 3b) Build a code snippet to call open with the file name.

\* 3c) Run that code snippet via a oneTimeCode, saving the returned

\* file descriptor

\* 4) Write the returned file descriptor into a memory variable for

\* mutatee-side use

\* 5) Build a snippet that copies output to the file

\* 5a) Locate the "write" library call

\* 5b) Access its parameters

\* 5c) Build a snippet calling write(fd, parameters)

\* 5d) Insert the snippet at write

\* 6) Add a hook to exit to ensure that we close the file (using

\* a callback at exit and another oneTimeCode)

\*/

void usage() {

fprintf(stderr, "Usage: retee <process pid> <filename>\n");

fprintf(stderr, " note: <filename> is relative to the application
process.\n");

}

// We need to use a callback, and so the things that callback requires

// are made global - this includes the file descriptor snippet (see
below)

BPatch_variableExpr \*fdVar = NULL;

// Before we add instrumentation, we need to open the file for

// writing. We can do this with a oneTimeCode - a piece of code run at

// a particular time, rather than at a particular location.

int openFileForWrite(BPatch_process \*app, BPatch_image \*appImage, char
\*fileName) {

// The code to be generated is:

// fd = open(argv[2], O_WRONLY|O_CREAT, 0666);

// (1) Find the open function

std::vector<BPatch_function \*>openFuncs;

appImage->findFunction("open", openFuncs);

if (openFuncs.size() == 0) {

fprintf(stderr, "ERROR: Unable to find function for open()\n");

return -1;

}

// (2) Allocate a vector of snippets for the parameters to open

std::vector<BPatch_snippet \*> openArgs;

// (3) Create a string constant expression from argv[3]

BPatch_constExpr fileNameExpr(fileName);

// (4) Create two more constant expressions \_WRONLY|O_CREAT and 0666

BPatch_constExpr fileFlagsExpr(O_WRONLY|O_CREAT);

BPatch_constExpr fileModeExpr(0666);

// (5) Push 3 & 4 onto the list from step 2, push first to last
parameter.

openArgs.push_back(&fileNameExpr);

openArgs.push_back(&fileFlagsExpr);

openArgs.push_back(&fileModeExpr);

// (6) create a procedure call using function found at 1 and

// parameters from step 5.

BPatch_funcCallExpr openCall(*openFuncs[0], openArgs);

// (7) The oneTimeCode returns whatever the return result from

// the BPatch_snippet is. In this case, the return result of

// open -> the file descriptor.

void \*openFD = app->oneTimeCode( openCall );

// oneTimeCode returns a void \*, and we want an int file handle

return (int) (long) openFD;

}

// We have used a oneTimeCode to open the file descriptor. However,

// this returns the file descriptor to the mutator - the mutatee has

// no idea what the descriptor is. We need to allocate a variable in

// the mutatee to hold this value for future use and copy the

// (mutator-side) value into the mutatee variable.

// Note: there are alternatives to this technique. We could have

// allocated the variable before the oneTimeCode and augmented the

// snippet to do the assignment. We could also write the file

// descriptor as a constant into any inserted instrumentation.

BPatch_variableExpr \*writeFileDescIntoMutatee(BPatch_process \*app,

BPatch_image \*appImage,

int fileDescriptor) {

// (1) Allocate a variable in the mutatee of size (and type) int

BPatch_variableExpr \*fdVar = app->malloc(*appImage->findType("int"));

if (fdVar == NULL) return NULL;

// (2) Write the value into the variable

// Like memcpy, writeValue takes a pointer

// The third parameter is for functionality called "saveTheWorld",

// which we don't worry about here (and so is false)

bool ret = fdVar->writeValue((void \*) &fileDescriptor, sizeof(int),

false);

if (ret == false) return NULL;

return fdVar;

}

// We now have an open file descriptor in the mutatee. We want to

// instrument write to intercept and copy the output. That happens

// here.

bool interceptAndCloneWrite(BPatch_process \*app,

BPatch_image \*appImage,

BPatch_variableExpr \*fdVar) {

// (1) Locate the write call

std::vector<BPatch_function \*> writeFuncs;

appImage->findFunction("write",

writeFuncs);

if(writeFuncs.size() == 0) {

fprintf(stderr, "ERROR: Unable to find function for write()\n");

return false;

}

// (2) Build the call to (our) write. Arguments are:

// ours: fdVar (file descriptor)

// parameter: buffer

// parameter: buffer size

// Declare a vector to hold these.

std::vector<BPatch_snippet \*> writeArgs;

// Push on the file descriptor

writeArgs.push_back(fdVar);

// Well, we need the buffer... but that's a parameter to the

// function we're implementing. That's not a problem - we can grab

// it out with a BPatch_paramExpr.

BPatch_paramExpr buffer(1); // Second (0, 1, 2) argument

BPatch_paramExpr bufferSize(2);

writeArgs.push_back(&buffer);

writeArgs.push_back(&bufferSize);

// And build the write call

BPatch_funcCallExpr writeCall(*writeFuncs[0], writeArgs);

// (3) Identify the BPatch_point for the entry of write. We're

// instrumenting the function with itself; normally the findPoint

// call would operate off a different function than the snippet.

std::vector<BPatch_point \*> \*points;

points = writeFuncs[0]->findPoint(BPatch_entry);

if ((*points).size() == 0) {

return false;

}

// (4) Insert the snippet at the start of write

return app->insertSnippet(writeCall, \*points);

// Note: we have just instrumented write() with a call to

// write(). This would ordinarily be a \_bad thing_, as there is

// nothing to stop infinite recursion - write -> instrumentation

// -> write -> instrumentation....

// However, Dyninst uses a feature called a "tramp guard" to

// prevent this, and it's on by default.

}

// This function is called as an exit callback (that is, called

// immediately before the process exits when we can still affect it)

// and thus must match the exit callback signature:

//

// typedef void (*BPatchExitCallback) (BPatch_thread \*,
BPatch_exitType)

//

// Note that the callback gives us a thread, and we want a process - but

// each thread has an up pointer.

void closeFile(BPatch_thread \*thread, BPatch_exitType) {

fprintf(stderr, "Exit callback called for process...\n");

// (1) Get the BPatch_process and BPatch_images

BPatch_process \*app = thread->getProcess();

BPatch_image \*appImage = app->getImage();

// The code to be generated is:

// close(fd);

// (2) Find close

std::vector<BPatch_function \*> closeFuncs;

appImage->findFunction("close", closeFuncs);

if (closeFuncs.size() == 0) {

fprintf(stderr, "ERROR: Unable to find function for close()\n");

return;

}

// (3) Allocate a vector of snippets for the parameters to open

std::vector<BPatch_snippet \*> closeArgs;

// (4) Add the fd snippet - fdVar is global since we can't

// get it via the callback

closeArgs.push_back(fdVar);

// (5) create a procedure call using function found at 1 and

// parameters from step 3.

BPatch_funcCallExpr closeCall(*closeFuncs[0], closeArgs);

// (6) Use a oneTimeCode to close the file

app->oneTimeCode( closeCall );

// (7) Tell the app to continue to finish it off.

app->continueExecution();

return;

}

BPatch bpatch;

// In main we perform the following operations.

// 1) Attach to the process and get BPatch_process and BPatch_image

// handles

// 2) Open a file descriptor

// 3) Instrument write

// 4) Continue the process and wait for it to terminate

int main(int argc, char \*argv[]) {

int pid;

if (argc != 3) {

usage();

exit(1);

}

pid = atoi(argv[1]);

// Attach to the program - we can attach with just a pid; the

// program name is no longer necessary

fprintf(stderr, "Attaching to process %d...\n", pid);

BPatch_process \*app = bpatch.processAttach(NULL, pid);

if (!app) return -1;

// Read the program's image and get an associated image object

BPatch_image \*appImage = app->getImage();

std::vector<BPatch_function*> writeFuncs;

fprintf(stderr, "Opening file %s for write...\n", argv[2]);

int fileDescriptor = openFileForWrite(app, appImage, argv[2]);

if (fileDescriptor == -1) {

fprintf(stderr, "ERROR: opening file %s for write failed\n",

argv[2]);

exit(1);

}

fprintf(stderr, "Writing returned file descriptor %d into"

"mutatee...\n", fileDescriptor);

// This was defined globally as the exit callback needs it.

fdVar = writeFileDescIntoMutatee(app, appImage, fileDescriptor);

if (fdVar == NULL) {

fprintf(stderr, "ERROR: failed to write mutatee-side variable\n");

exit(1);

}

fprintf(stderr, "Instrumenting write...\n");

bool ret = interceptAndCloneWrite(app, appImage, fdVar);

if (!ret) {

fprintf(stderr, "ERROR: failed to instrument mutatee\n");

exit(1);

}

fprintf(stderr, "Adding exit callback...\n");

bpatch.registerExitCallback(closeFile);

// Continue the execution...

fprintf(stderr, "Continuing execution and waiting for termination\n");

app->continueExecution();

while (!app->isTerminated())

bpatch.waitForStatusChange();

printf("Done.\n");

return 0;

}

Appendix C - Common pitfalls
============================

This appendix is designed to point out some common pitfalls that users
have reported when using the Dyninst system. Many of these are either
due to limitations in the current implementations, or reflect design
decisions that may not produce the expected behavior from the system.

**Attach followed by detach**

If a mutator attaches to a mutatee, and immediately exits, the current
behavior is that the mutatee is left suspended. To make sure the
application continues, call detach with the appropriate flags.

**Attaching to a program that has already been modified by Dyninst**

If a mutator attaches to a program that has already been modified by a
previous mutator, a warning message will be issued. We are working to
fix this problem, but the correct semantics are still being specified.
Currently, a message is printed to indicate that this has been
attempted, and the attach will fail.

**Dyninst is event-driven**

Dyninst must sometimes handle events that take place in the mutatee, for
instance when a new shared library is loaded, or when the mutatee
executes a fork or exec. Dyninst handles events when it checks the
status of the mutatee, so to allow this the mutator should periodically
call one of the functions BPatch::pollForStatusChange,
BPatch::wait­ForStatusChange, BPatch_thread::isStopped, or
BPatch_­thread::is­Termin­ated.
