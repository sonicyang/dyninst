.. _`sec:parseapi-intro`:

========
ParseAPI
========

A binary code parser converts the machine code representation of a
program, library, or code snippet to abstractions such as the
instructions, basic blocks, functions, and loops that the binary code
represents. The ParseAPI is a multi-platform library for creating such
abstractions from binary code sources. The current incarnation uses the
Dyninst SymtabAPI as the default binary code source; all platforms and
architectures handled by the SymtabAPI are supported. The ParseAPI has
cross-architecture binary analysis capabilities in analyzing ELF
binaries (parsing of ARM binaries on x86 and vice versa, for example).
The ParseAPI is designed to be easily extensible to other binary code
sources. Support for parsing binary code in memory dumps or other
formats requires only implementation of a small interface as described
in this document.

This API provides the user with a control flow-oriented view of a binary
code source. Each code object such as a program binary or library is
represented as a top-level collection containing the loops, functions,
basic blocks, and edges that represent the control flow graph. A simple
query interface is provided for retrieving lower level objects like
functions and basic blocks through address or other attribute lookups.
These objects can be used to navigate the program structure as described
below.

Since Dyninst 10.0, ParseAPI is officially supporting parallel binary
code analysis and parallel queries. We typically observe 4X speedup when
analyzing binaries with 8 threads. To control the number of threads used
during parallel parsing, please set environment variable
``OMP_NUM_THREADS``.

.. _`sec:parseapi-abstractions`:

Abstractions
============

The basic representation of code in this API is the control flow graph
(CFG). Binary code objects are represented as regions of contiguous
bytes that, when parsed, form the nodes and edges of this graph. The
following abstractions make up this CFG-oriented representation of
binary code:

.. container:: itemize

   block: Nodes in the CFG represent *basic blocks*: straight line
   sequences of instructions :math:`I_i \ldots I_j` where for each
   :math:`i < k
   \le j`, :math:`I_k` postdominates :math:`I_{k-1}`. Importantly, on
   some instruction set architectures basic blocks can *overlap* on the
   same address range—variable length instruction sets allow for
   multiple interpretations of the bytes making up the basic block.

   edge: Typed edges between the nodes in the CFG represent execution
   control flow, such as conditional and unconditional branches,
   fallthrough edges, and calls and returns. The graph therefore
   represents both *inter-* and *intraprocedural* control flow:
   traversal of nodes and edges can cross the boundaries of the higher
   level abstractions like *functions*.

   function: The *function* is the primary semantic grouping of code in
   the binary, mirroring the familiar abstraction of procedural
   languages like C. Functions represent the set of all basic blocks
   reachable from a *function entry point* through intraprocedural
   control flow only (that is, no calls or returns). Function entry
   points are determined in a variety of ways, such as hints from
   debugging symbols, recursive traversal along call edges and a machine
   learning based function entry point identification process.

   loop: The *loop* represents code in the binary that may execute
   repeatedly, corresponding to source language constructs like *for*
   loop or *while* loop. We use a formal definition of loops from
   “Nesting of Reducible and Irreducible Loops" by Paul Havlak. We
   support identifying both natural loops (single-entry loops) and
   irreducible loops (multi-entry loops).

.. container:: itemize

   code object: A collection of distinct code regions are represented as
   a single code object, such as an executable or library. Code objects
   can normally be thought of as a single, discontiguous unique address
   space. However, the ParseAPI supports code objects in which the
   different regions have overlapping address spaces, such as UNIX
   archive files containing unlinked code.

   instruction source: An instruction source describes a backing store
   containing binary code. A binary file, a library, a memory dump, or a
   process’s executing memory image can all be described as an
   instruction source, allowing parsing of a variety of binary code
   objects.

   code source: The code source implements the instruction source
   interface, exporting methods that can access the underlying bytes of
   the binary code for parsing. It also exports a number of additional
   helper methods that do things such as returning the location of
   structured exception handling routines and function symbols. Code
   sources are tailored to particular binary types; the ParseAPI
   provides a SymtabAPI-based code source that understands ELF, COFF and
   PE file formats.

.. _`sec:parseapi-usage`:

Usage
=====

Function Disassembly
--------------------

The following example uses ParseAPI and InstructionAPI to disassemble
the basic blocks in a function. As an example, it can be built with G++
as follows:
``g++ -std=c++0x -o code_sample code_sample.cc -L<library install path> -I<headers install path> -lparseAPI -linstructionAPI -lsymtabAPI -lsymLite -ldynDwarf -ldynElf -lcommon -L<libelf path> -lelf -L<libdwarf path> -ldwarf``.
Note: this example must be compiled with C++11x support; for G++ this is
enabled with ``-std=c++0x``, and it is on by default for Visual Studio.

.. code-block:: cpp

   /*
      Copyright (C) 2015 Alin Mindroc
      (mindroc dot alin at gmail dot com)

      This is a sample program that shows how to use InstructionAPI in order to
      6  print the assembly code and functions in a provided binary.


      This program is free software; you can redistribute it and/or
      modify it under the terms of the GNU Lesser General Public
      11  License as published by the Free Software Foundation; either
      version 2.1 of the License, or (at your option) any later version.
      */
   #include <iostream>
   #include "CodeObject.h"
   #include "InstructionDecoder.h"
   using namespace std;
   using namespace Dyninst;
   using namespace ParseAPI;

   using namespace InstructionAPI;
   int main(int argc, char **argv){
       if(argc != 2){
   	printf("Usage: %s <binary path>\n", argv[0]);
   	return -1;
       }
       char *binaryPath = argv[1];

       SymtabCodeSource *sts;
       CodeObject *co;
       Instruction::Ptr instr;
       SymtabAPI::Symtab *symTab;
       std::string binaryPathStr(binaryPath);
       bool isParsable = SymtabAPI::Symtab::openFile(symTab, binaryPathStr);
       if(isParsable == false){
   	const char *error = "error: file can not be parsed";
   	cout << error;
   	return - 1;
       }
       sts = new SymtabCodeSource(binaryPath);
       co = new CodeObject(sts);
       //parse the binary given as a command line arg
       co->parse();

       //get list of all functions in the binary
       const CodeObject::funclist &all = co->funcs();
       if(all.size() == 0){
   	const char *error = "error: no functions in file";
   	cout << error;
   	return - 1;
       }
       auto fit = all.begin();
       Function *f = *fit;
       //create an Instruction decoder which will convert the binary opcodes to strings
       InstructionDecoder decoder(f->isrc()->getPtrToInstruction(f->addr()),
   	    InstructionDecoder::maxInstructionLength,
   	    f->region()->getArch());
       for(;fit != all.end(); ++fit){
   	Function *f = *fit;
   	//get address of entry point for current function

   	Address crtAddr = f->addr();
   	int instr_count = 0;
   	instr = decoder.decode((unsigned char *)f->isrc()->getPtrToInstruction(crtAddr));
   	auto fbl = f->blocks().end();
   	fbl--;
   	Block *b = *fbl;
   	Address lastAddr = b->last();
   	//if current function has zero instructions, don’t output it
   	if(crtAddr == lastAddr)
   	    continue;
   	cout << "\n\n\"" << f->name() << "\" :";
   	while(crtAddr < lastAddr){
   	    //decode current instruction
   	    instr = decoder.decode((unsigned char *)f->isrc()->getPtrToInstruction(crtAddr));
   	    cout << "\n" << hex << crtAddr;
   	    cout << ": \"" << instr->format() << "\"";
   	    //go to the address of the next instruction
   	    crtAddr += instr->size();
   	    instr_count++;
   	}
       }
       return 0;
   }

Control flow graph traversal
----------------------------

The following complete example uses the ParseAPI to parse a binary and
dump its control flow graph in the Graphviz file format. As an example,
it can be built with G++ as follows:
``g++ -std=c++0x -o example example.cc -L<library install path> -I<headers install path> -lparseAPI -linstructionAPI -lsymtabAPI -lsymLite -ldynDwarf -ldynElf -lcommon -L<libelf path> -lelf -L<libdwarf path> -ldwarf``.
Note: this example must be compiled with C++11x support; for G++ this is
enabled with ``-std=c++0x``, and it is on by default for Visual Studio.

.. code-block:: cpp

   // Example ParseAPI program; produces a graph (in DOT format) of the
   // control flow graph of the provided binary. 
   //
   // Improvements by E. Robbins (er209 at kent dot ac dot uk)
   //

   #include <stdio.h>
   #include <map>
   #include <vector>
   #include <unordered_map>
   #include <sstream>
   #include "CodeObject.h"
   #include "CFG.h"

   using namespace std;
   using namespace Dyninst;
   using namespace ParseAPI;

   int main(int argc, char * argv[])
   {
      map<Address, bool> seen;
      vector<Function *> funcs;
      SymtabCodeSource *sts;
      CodeObject *co;
      
      // Create a new binary code object from the filename argument
      sts = new SymtabCodeSource( argv[1] );
      co = new CodeObject( sts );
      
      // Parse the binary
      co->parse();
      cout << "digraph G {" << endl;
      
      // Print the control flow graph
      const CodeObject::funclist& all = co->funcs();
      auto fit = all.begin();
      for(int i = 0; fit != all.end(); ++fit, i++) { // i is index for clusters
         Function *f = *fit;
         
         // Make a cluster for nodes of this function
         cout << "\t subgraph cluster_" << i 
              << " { \n\t\t label=\""
              << f->name()
              << "\"; \n\t\t color=blue;" << endl;
         
         cout << "\t\t\"" << hex << f->addr() << dec
              << "\" [shape=box";
         if (f->retstatus() == NORETURN)
            cout << ",color=red";
         cout << "]" << endl;
         
         // Label functions by name
         cout << "\t\t\"" << hex << f->addr() << dec
              << "\" [label = \""
              << f->name() << "\\n" << hex << f->addr() << dec
              << "\"];" << endl;

         stringstream edgeoutput;
         
         auto bit = f->blocks().begin();
         for( ; bit != f->blocks().end(); ++bit) {
            Block *b = *bit;
            // Don't revisit blocks in shared code
            if(seen.find(b->start()) != seen.end())
               continue;
            
            seen[b->start()] = true;
            
            cout << "\t\t\"" << hex << b->start() << dec << 
               "\";" << endl;
            
            auto it = b->targets().begin();
            for( ; it != b->targets().end(); ++it) {
               if(!*it) continue;
               std::string s = "";
               if((*it)->type() == CALL)
                  s = " [color=blue]";
               else if((*it)->type() == RET)
                  s = " [color=green]";

               // Store the edges somewhere to be printed outside of the cluster
               edgeoutput << "\t\"" 
                          << hex << (*it)->src()->start()
                          << "\" -> \""
                          << (*it)->trg()->start()
                          << "\"" << s << endl;
            }
         }
         // End cluster
         cout << "\t}" << endl;

         // Print edges
         cout << edgeoutput.str() << endl;
      }
      cout << "}" << endl;
   }

Loop analysis
-------------

The following code example shows how to get loop information using
ParseAPI once we have an parsed Function object.

.. code-block:: cpp

   void GetLoopInFunc(Function *f) {
       // Get all loops in the function
       vector<Loop*> loops;
       f->getLoops(loops);

       // Iterate over all loops
       for (auto lit = loops.begin(); lit != loops.end(); ++lit) {
           Loop *loop = *lit;

           // Get all the entry blocks of the loop
   	vector<Block*> entries;
   	loop->getLoopEntries(entries);

           // Get all the blocks in the loop
           vector<Block*> blocks;
   	loop->getLoopBasicBlocks(blocks);

   	// Get all the back edges in the loop
   	vector<Edge*> backEdges;
   	loop->getBackEdges(backEdges);
       }
   }

.. _`sec:extend`:

Extending ParseAPI
==================

The ParseAPI is design to be a low level toolkit for binary analysis
tools. Users can extend the ParseAPI in two ways: by extending the
control flow structures (Functions, Blocks, and Edges) to incorporate
additional data to support various analysis applications, and by adding
additional binary code sources that are unsupported by the default
SymtabAPI-based code source. For example, a code source that represents
a program image in memory could be implemented by fulfilling the
CodeSource and InstructionSource interfaces described in Section
`4.8 <#sec:codesource>`__ and below. Implementations that extend the CFG
structures need only provide a custom allocation factory in order for
these objects to be allocated during parsing.

Instruction and Code Sources
----------------------------

A CodeSource, as described above, exports its own and the
InstructionSource interface for access to binary code and other details.
In addition to implementing the virtual methods in the CodeSource base
class (Section `4.8 <#sec:codesource>`__), the methods in the
pure-virtual InstructionSource class must be implemented:

.. code-block:: cpp
    
    virtual bool isValidAddress(const Address)

Returns true if the address is a valid code location.

.. code-block:: cpp
    
    virtual void* getPtrToInstruction(const Address)

Returns pointer to raw memory in the binary at the provided address.

.. code-block:: cpp
    
    virtual void* getPtrToData(const Address)

Returns pointer to raw memory in the binary at the provided address. The
address need not correspond to an executable code region.

.. code-block:: cpp
    
    virtual unsigned int getAddressWidth()

Returns the address width (e.g. four or eight bytes) for the represented
binary.

.. code-block:: cpp
    
    virtual bool isCode(const Address)

Indicates whether the location is in a code region.

.. code-block:: cpp
    
    virtual bool isData(const Address)

Indicates whether the location is in a data region.

.. code-block:: cpp
    
    virtual Address offset()

The start of the region covered by this instruction source.

.. code-block:: cpp
    
    virtual Address length()

The size of the region.

.. code-block:: cpp
    
    virtual Architecture getArch()

The architecture of the instruction source. See the Dyninst manual for
details on architecture differences.

.. code-block:: cpp
    
    virtual bool isAligned(const Address)

For fixed-width instruction architectures, must return true if the
address is a valid instruction boundary and false otherwise; otherwise
returns true. This method has a default implementation that should be
sufficient.

CodeSource implementors need to fill in several data structures in the
base CodeSource class:

.. code-block:: cpp
    
    std::map<Address, std::string> _linkage

Entries in the linkage map represent external linkage, e.g. the PLT in
ELF binaries. Filling in this map is optional.

.. code-block:: cpp
    
    Address _table_of_contents

Many binary format have “table of contents” structures for position
independant references. If such a structure exists, its address should
be filled in.

.. code-block:: cpp
    
    std::vector<CodeRegion *> _regions Dyninst::IBSTree<CodeRegion> _region_tree

One or more contiguous regions of code or data in the binary object must
be registered with the base class. Keeping these structures in sync is
the responsibility of the implementing class.

.. code-block:: cpp
    
    std::vector<Hint> _hints

CodeSource implementors can supply a set of Hint objects describing
where functions are known to start in the binary. These hints are used
to seed the parsing algorithm. Refer to the CodeSource header file for
implementation details.

.. _`sec:factories`:

CFG Object Factories
--------------------

Users who which to incorporate the ParseAPI into large projects may need
to store additional information about CFG objects like Functions,
Blocks, and Edges. The simplest way to associate the ParseAPI-level CFG
representation with higher-level implementation is to extend the CFG
classes provided as part of the ParseAPI. Because the parser itself does
not know how to construct such extended types, implementors must provide
an implementation of the CFGFactory that is specialized for their CFG
classes. The CFGFactory exports the following simple interface:

.. code-block:: cpp
    
    virtual Function * mkfunc(Address addr, FuncSource src, std::string
    name, CodeObject * obj, CodeRegion * region,
    Dyninst::InstructionSource * isrc)

Returns an object derived from Function as though the provided
parameters had been passed to the Function constructor. The ParseAPI
parser will never invoke ``mkfunc()`` twice with identical ``addr``, and
``region`` parameters—that is, Functions are guaranteed to be unique by
address within a region.

.. code-block:: cpp
    
    virtual Block * mkblock(Function * func, CodeRegion * region, Address addr)

Returns an object derived from Block as though the provided parameters
had been passed to the Block constructor. The parser will never invoke
``mkblock()`` with identical ``addr`` and ``region`` parameters.

.. code-block:: cpp
    
    virtual Edge * mkedge(Block * src, Block * trg, EdgeTypeEnum type)

Returns an object derived from Edge as though the provided parameters
had been passed to the Edge constructor. The parser *may* invoke
``mkedge()`` multiple times with identical parameters.

.. code-block:: cpp
    
    virtual Block * mksink(CodeObject *obj, CodeRegion *r)

Returns a “sink” block derived from Block to which all unresolvable
control flow instructions will be linked. Implementors may return a
unique sink block per CodeObject or a single global sink.

Implementors of extended CFG classes are required to override the
default implementations of the *mk** functions to allocate and return
the appropriate derived types statically cast to the base type.
Implementors must also add all allocated objects to the following
internal lists:

.. code-block:: cpp
    
    fact_list<Edge> edges_ fact_list<Block> blocks_ fact_list<Function> funcs_

O(1) allocation lists for CFG types. See the CFG.h header file for list
insertion and removal operations.

Implementors *may* but are *not required to* override the deallocation
following deallocation routines. The primary reason to override these
routines is if additional action or cleanup is necessary upon CFG object
release; the default routines simply remove the objects from the
allocation list and invoke their destructors.

.. code-block:: cpp
    
    virtual void free_func(Function * f) virtual void free_block(Block *
    b) virtual void free_edge(Edge * e) virtual void free_all()

CFG objects should be freed using these functions, rather than delete,
to avoid leaking memory.

.. _`sec:defmode`:

Defensive Mode Parsing
======================

Binary code that defends itself against analysis may violate the
assumptions made by the the ParseAPI’s standard parsing algorithm.
Enabling defensive mode parsing activates more conservative assumptions
that substantially reduce the percentage of code that is analyzed by the
ParseAPI. For this reason, defensive mode parsing is best-suited for use
of ParseAPI in conjunction with dynamic analysis techniques that can
compensate for its limited coverage of the binary code.
