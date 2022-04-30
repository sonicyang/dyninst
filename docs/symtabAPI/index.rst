.. _`sec:symtab-intro`:

=========
SymtabAPI
=========

SymtabAPI is a multi-platform library for parsing symbol tables, object
file headers and debug information. SymtabAPI currently supports the ELF
(IA-32, AMD-64, ARMv8-64, and POWER) and PE (Windows) object file
formats. In addition, it also supports the DWARF debugging format.

The main goal of this API is to provide an abstract view of binaries and
libraries across multiple platforms. An abstract interface provides two
benefits: it simplifies the development of a tool since the complexity
of a particular file format is hidden, and it allows tools to be easily
ported between platforms. Each binary object file is represented in a
canonical platform independent manner by the API. The canonical format
consists of four components: a header block that contains general
information about the object (e.g., its name and location), a set of
symbol lists that index symbols within the object for fast lookup, debug
information (type, line number and local variable information) present
in the object file and a set of additional data that represents
information that may be present in the object (e.g., relocation or
exception information). Adding a new format requires no changes to the
interface and hence will not affect any of the tools that use the
SymtabAPI.

Our other design goal with SymtabAPI is to allow users and tool
developers to easily extend or add symbol or debug information to the
library through a platform-independent interface. Often times it is
impossible to satify all the requirements of a tool that uses SymtabAPI,
as those requirements can vary from tool to tool. So by providing
extensible structures, SymtabAPI allows tools to modify any structure to
fit their own requirements. Also, tools frequently use more
sophisticated analyses to augment the information available from the
binary directly; it should be possible to make this extra information
available to the SymtabAPI library. An example of this is a tool
operating on a stripped binary. Although the symbols for the majority of
functions in the binary may be missing, many can be determined via more
sophisticated analysis. In our model, the tool would then inform the
SymtabAPI library of the presence of these functions; this information
would be incorporated and available for subsequent analysis. Other
examples of such extensions might involve creating and adding new types
or adding new local variables to certain functions.

.. _`sec:symtab-abstractions`:

Abstractions
============

SymtabAPI provides a simple set of abstractions over complicated data
structures which makes it straight-forward to use. The SymtabAPI
consists of five classes of interfaces: the symbol table interface, the
type interface, the line map interface, the local variable interface,
and the address translation interface.

Figure `[fig:object-ownership] <#fig:object-ownership>`__ shows the
ownership hierarchy for the SymtabAPI classes. Ownership here is a
“contains” relationship; if one class owns another, then instances of
the owner class maintain an exclusive instance of the other. For
example, each Symtab class instance contains multiple instances of class
Symbol and each Symbol class instance belongs to one Symtab class
instance. Each of four interfaces and the classes belonging to these
interfaces are described in the rest of this section. The API functions
in each of the classes are described in detail in Section
`6 <#sec:symtabAPI>`__.

Symbol Table Interface
----------------------

The symbol table interface is responsible for parsing the object file
and handling the look-up and addition of new symbols. It is also
responsible for the emit functionality that SymtabAPI supports. The
Symtab and the Module classes inherit from the LookupInterface class, an
abstract class, ensuring the same lookup function signatures for both
Module and Symtab classes.

Symtab
   A Symtab class object represents either an object file on-disk or
   in-memory that the SymtabAPI library operates on.

Symbol
   A Symbol class object represents an entry in the symbol table.

Module
   A Module class object represents a particular source file in cases
   where multiple files were compiled into a single binary object; if
   this information is not present, we use a single default module.

Archive
   An Archive class object represents a collection of binary objects
   stored in a single file (e.g., a static archive).

ExceptionBlock
   An ExceptionBlock class object represents an exception block which
   contains the information necessary for run-time exception handling.

In addition, we define two symbol aggregates, Function and Variable.
These classes collect multiple symbols with the same address and type
but different names; for example, weak and strong symbols for a single
function.

.. _`subsec:typeInterface`:

Type Interface
--------------

The Type interface is responsible for parsing type information from the
object file and handling the look-up and addition of new type
information. Figure `[fig:class-inherit] <#fig:class-inherit>`__ shows
the class inheritance diagram for the type interface. Class Type is the
base class for all of the classes that are part of the interface. This
class provides the basic common functionality for all the types, such as
querying the name and size of a type. The rest of the classes represent
specific types and provide more functionality based on the type.

= [rectangle, draw, rounded corners, fill=yellow!100] = [rectangle,
draw, rounded corners, fill=yellow!100, node distance=.65cm] =
[rectangle, draw, rounded corners, pattern=north west lines, pattern
color=yellow] = []

Some of the types inherit from a second level of type classes, each
representing a separate category of types.

fieldListType
   - This category of types represent the container types that contain a list of fields. Examples of this category include structure and the union types.

derivedType
   - This category of types represent types derived from a base type. Examples of this category include typedef, pointer and reference types.

rangedType
   - This category represents range types. Examples of this category include the array and the sub-range types.

The enum, function, common block and scalar types do not fall under any
of the above category of types. Each of the specific types is derived
from Type.

Line Number Interface
---------------------

The Line Number interface is responsible for parsing line number
information from the object file debug information and handling the
look-up and addition of new line information. The main classes for this
interface are LineInformation and LineNoTuple.

LineInformation
   A LineInformation class object represents a mapping of line numbers
   to address range within a module (source file).

Statement/LineNoTuple
   A Statement class object represents a location in source code with
   a source file, line number in that source file and start column in
   that line. For backwards compatibility, Statements may also be
   referred to as LineNoTuples.

Local Variable Interface
------------------------

The Local Variable Interface is responsible for parsing local variable
and parameter information of functions from the object file debug
information and handling the look-up and addition of new add new local
variables. All the local variables within a function are tied to the
Symbol class object representing that function.

localVar
   A localVar class object represents a local variable or a parameter
   belonging to a function.

Dynamic Address Translation
---------------------------

The AddressLookup class is a component for mapping between absolute
addresses found in a running process and SymtabAPI objects. This is
useful because libraries can load at different addresses in different
processes. Each AddressLookup instance is associated with, and provides
mapping for, one process.

.. _symtabapi-usage:

Usage
=====

To illustrate the ideas in the API, this section presents several short
examples that demonstrate how the API can be used. SymtabAPI has the
ability to parse files that are on-disk or present in memory. The user
program starts by requesting SymtabAPI to parse an object file.
SymtabAPI returns a handle if the parsing succeeds, whcih can be used
for further interactions with the SymtabAPI library. The following
example shows how to parse a shared object file on disk.

.. code-block:: cpp

   using namespace Dyninst;
   using namespace SymtabAPI;

   //Name the object file to be parsed:
   std::string file = "libfoo.so";

   //Declare a pointer to an object of type Symtab; this represents the file.
   Symtab *obj = NULL;

   // Parse the object file
   bool err = Symtab::openFile(obj, file);

Once the object file is parsed successfully and the handle is obtained,
symbol look up and update operations can be performed in the following
way:

.. code-block:: cpp

   using namespace Dyninst;
   using namespace SymtabAPI;
   std::vector <Symbol *> syms;
   std::vector <Function *> funcs;

   // search for a function with demangled (pretty) name "bar".
   if (obj->findFunctionsByName(funcs, "bar")) {
          // Add a new (mangled) primary name to the first function
          funcs[0]->addMangledName("newname", true);
   }

   // search for symbol of any type with demangled (pretty) name "bar".
   if (obj->findSymbol(syms, "bar", Symbol::ST_UNKNOWN)) { 

       // change the type of the found symbol to type variable(ST_OBJECT)
       syms[0]->setType(Symbol::ST_OBJECT);

       // These changes are automatically added to symtabAPI; no further
       // actions are required by the user.
   }

New symbols, functions, and variables can be created and added to the
library at any point using the handle returned by successful parsing of
the object file. When possible, add a function or variable rather than a
symbol directly.

.. code-block:: cpp

   using namespace Dyninst;
   using namespace SymtabAPI;

   //Module for the symbol
   Module *mod;

   // obj represents a handle to a parsed object file.
   // Lookup module handle for "DEFAULT_MODULE"
   obj->findModuleByName(mod, "DEFAULT_MODULE");

   // Create a new function symbol
   Variable *newVar = mod->createVariable("newIntVar",  // Name of new variable
                                          0x12345,      // Offset from data section
                                          sizeof(int)); // Size of symbol 

SymtabAPI gives the ability to query type information present in the
object file. Also, new user defined types can be added to SymtabAPI. The
following example shows both how to query type information after an
object file is successfully parsed and also add a new structure type.

.. code-block:: cpp

   // create a new struct Type
   // typedef struct{
   //int field1,
   //int field2[10]
   // } struct1;

   using namespace Dyninst;
   using namespace SymtabAPI;

   // Find a handle to the integer type; obj represents a handle to a parsed object file
   Type *lookupType;
   obj->findType(lookupType, "int");

   // Convert the generic type object to the specific scalar type object
   typeScalar *intType = lookupType->getScalarType();

   // container to hold names and types of the new structure type
   vector<pair<string, Type *> >fields;

   //create a new array type(int type2[10])
   typeArray *intArray = typeArray::create("intArray",intType,0,9, obj);

   //types of the structure fields
   fields.push_back(pair<string, Type *>("field1", intType));
   fields.push_back(pair<string, Type *>("field2", intArray));

   //create the structure type
   typeStruct *struct1 = typeStruct::create("struct1", fields, obj);

Users can also query line number information present in an object file.
The following example shows how to use SymtabAPI to get the address
range for a line number within a source file.

.. code-block:: cpp

   using namespace Dyninst;
   using namespace SymtabAPI;

   // obj represents a handle to a parsed object file using symtabAPI
   // Container to hold the address range
   vector< pair< Offset, Offset > > ranges;

   // Get the address range for the line 30 in source file foo.c
   obj->getAddressRanges(ranges, "foo.c", 30);

Local variable information can be obtained using symtabAPI. You can
query for a local variable within the entire object file or just within
a function. The following example shows how to find local variable foo
within function bar.

.. code-block:: cpp

   using namespace Dyninst;
   using namespace SymtabAPI;

   // Obj represents a handle to a parsed object file using symtabAPI
   // Get the Symbol object representing function bar
   vector<Symbol *> syms;
   obj->findSymbol(syms, "bar", Symbol::ST_FUNCTION);

   // Find the local var foo within function bar
   vector<localVar *> *vars = syms[0]->findLocalVarible("foo");

The rest of this document describes the class hierarchy and the API in
detail.

Symtab Definitions and Basic Types
==================================

The following definitions and basic types are referenced throughout the
rest of this document.

Symtab Definitions
------------------

Offset
   Offsets represent an address relative to the start address(base) of
   the object file. For executables, the Offset represents an absolute
   address. The following definitions deal with the symbol table
   interface.

Object File
   An object file is the representation of code that a compiler or
   assembler generates by processing a source code file. It represents
   .o’s, a.out’s and shared libraries.

Region
   A region represents a contiguous area of the file that contains
   executable code or readable data; for example, an ELF section.

Symbol
   A symbol represents an entry in the symbol table, and may identify a
   function, variable or other structure within the file.

Function
   A function represents a code object within the file represented by
   one or more symbols.

Variable
   A variable represents a data object within the file represented by
   one or more symbols.

Module
   A module represents a particular source file in cases where multiple
   files were compiled into a single binary object; if this information
   is not present, or if the binary object is a shared library, we use a
   single default module.

Archive
   An archive represents a collection of binary objects stored in a
   single file (e.g., a static archive).

Relocations
   These provide the necessary information for inter-object references
   between two object files.

Exception Blocks
   These contain the information necessary for run-time exception
   handling The following definitions deal with members of the Symbol
   class.

Mangled Name
   A mangled name for a symbol provides a way of encoding additional
   information about a function, structure, class or another data type
   in a symbol name. It is a technique used to produce unique names for
   programming entities in many modern programming languages. For
   example, the method *foo* of class C with signature *int C::foo(int,
   int)* has a mangled name *\_ZN1C3fooEii* when compiled with gcc.
   Mangled names may include a sequence of clone suffixes (begins with
   ‘.’ that indicate a compiler synthesized function), and this may be
   followed by a version suffix (begins with ‘@’).

Pretty Name
   A pretty name for a symbol is the demangled user-level symbolic name
   without type information for the function parameters and return
   types. For non-mangled names, the pretty name is the symbol name. Any
   function clone suffixes of the symbol are appended to the result of
   the demangler. For example, a symbol with a mangled name
   *\_ZN1C3fooEii* for the method *int C::foo(int, int)* has a pretty
   name *C::foo*. Version suffixes are removed from the mangled name
   before conversion to the pretty name. The pretty name can be obtained
   by running the command line tool ``c++filt`` as
   ``c++filt -i -p name``, or using the libiberty library function
   ``cplus_demangle`` with options of ``DMGL_AUTO | DMGL_ANSI``.

Typed Name
   A typed name for a symbol is the demangled user-level symbolic name
   including type information for the function parameters. Typically,
   but not always, function return type information is not included. Any
   function clone information is also included. For non-mangled names,
   the typed name is the symbol name. For example, a symbol with a
   mangled name *\_ZN1C3fooEii* for the method *int C::foo(int, int)*
   has a typed name *C::foo(int, int)*. Version suffixes are removed
   from the mangled name before conversion to the typed name. The typed
   name can be obtained by running the command line tool ``c++filt`` as
   ``c++filt -i name``, or using the libiberty library function
   ``cplus_demangle`` with options of
   ``DMGL_AUTO | DMGL_ANSI | DMGL_PARAMS``.

Symbol Linkage
   The symbol linkage for a symbol gives information on the visibility
   (binding) of this symbol, whether it is visible only in the object
   file where it is defined (local), if it is visible to all the object
   files that are being linked (global), or if its a weak alias to a
   global symbol.

Symbol Type
   Symbol type for a symbol represents the category of symbols to which
   it belongs. It can be a function symbol or a variable symbol or a
   module symbol. The following definitions deal with the type and the
   local variable interface.

Type
   A type represents the data type of a variable or a parameter. This
   can represent language pre-defined types (e.g. int, float),
   pre-defined types in the object (e.g., structures or unions), or
   user-defined types.

Local Variable
   A local variable represents a variable that has been declared within
   the scope of a sub-routine or a parameter to a sub-routine.

Symtab Basic Types
------------------

.. code-block:: cpp

    typedef unsigned long Offset

An integer value that contains an offset from base address of the object
file.

.. code-block:: cpp

    typedef int typeId_t

A unique handle for identifying a type. Each of types is assigned a
globally unique ID. This way it is easier to identify any data type of a
variable or a parameter.

.. code-block:: cpp

    typedef ... PID

A handle for identifying a process that is used by the dynamic
components of SymtabAPI. On UNIX platforms PID is a int, on Windows it
is a HANDLE that refers to a process.

.. code-block:: cpp

    typedef unsigned long Address

An integer value that represents an address in a process. This is used
by the dynamic components of SymtabAPI.

Building SymtabAPI
==================

This appendix describes how to build SymtabAPI from source code, which
can be downloaded from http://www.paradyn.org or http://www.dyninst.org.

Building on Unix
----------------

Building SymtabAPI on UNIX platforms is a four step process that
involves: unpacking the SymtabAPI source, installing any SymtabAPI
dependencies, configuring paths in make.config.local, and running the
build.

SymtabAPI’s source code is packaged in a tar.gz format. If your
SymtabAPI source tarball is called ``symtab_src_1.0.tar.gz``, then you
could extract it with the command
``gunzip symtab_src_1.0.tar.gz; tar -xvf symtab_src_1.0.tar``. This will
create two directories: core and scripts.

SymtabAPI has several dependencies, depending on what platform you are
using, which must be installed before SymtabAPI can be built. Note that
for most of these packages Symtab needs to be able to access the
package’s include files, which means that development versions are
required. If a version number is listed for a packaged, then there are
known bugs that may affect Symtab with earlier versions of the package.

.. container:: center

   ============ ==================
   Linux/x86    libdwarf-200120327
   \            libelf
   Linux/x86-64 libdwarf-200120327
   \            libelf
   Windows/x86  <none>
   ============ ==================

At the time of this writing the Linux packages could be found at:

-  libdwarf - http://reality.sgiweb.org/davea/dwarf.html

-  libelf - http://www.mr511.de/software/english.html

Once the dependencies for SymtabAPI have been installed, SymtabAPI must
be configured to know where to find these packages. This is done through
SymtabAPI’s ``core/make.config.local`` file. This file must be written
in GNU Makefile syntax and must specify directories for each dependency.
Specifically, LIBDWARFDIR, LIBELFDIR and LIBXML2DIR variables must be
set. LIBDWARFDIR should be set to the absolute path of libdwarf library
where ``dwarf.h`` and ``libdw.h`` files reside. LIBELFDIR should be set
to the absolute path where ``libelf.a`` and ``libelf.so`` files are
located. Finally, LIBXML2DIR to the absolute path where libxml2 is
located.

The next thing is to set DYNINST_ROOT, PLATFORM, and LD_LIBRARY_PATH
environment variables. DYNINST_ROOT should be set to the path of the
directory that contains core and scripts subdirectories.

PLATFORM should be set to one of the following values depending upon
what operating system you are running on:

.. container:: description

   i386-unknown-linux2.4Linux 2.4/2.6 on an Intel x86 processor

   x86_64-unknown-linux2.4Linux 2.4/2.6 on an AMD-64 processor

LD_LIBRARY_PATH variable should be set in a way that it includes
libdwarf home directory/lib and $DYNINST_ROOT/$PLATFORM/lib directories.

Once ``make.config.local`` is set you are ready to build SymtabAPI.
Change to the core directory and execute the command make SymtabAPI.
This will build the SymtabAPI library. Successfully built binaries will
be stored in a directory named after your platform at the same level as
the core directory.

Building on Windows
-------------------

SymtabAPI for Windows is built with Microsoft Visual Studio 2003 project
and solution files. Building SymtabAPI for Windows is similar to UNIX in
that it is a four step process: unpack the SymtabAPI source code,
install SymtabAPI’s package dependencies, configure Visual Studio to use
the dependencies, and run the build system.

SymtabAPI’s source code is distributed as part of a tar.gz package. Most
popular unzipping programs are capable of handling this format.
Extracting the Symtab tarball results in two directories: core and
scripts.

Symtab for Windows depends on Microsoft’s Debugging Tools for Windows,
which could be found at
http://www.microsoft.com/whdc/devtools/debugging/default.mspx at the
time of this writing. Download these tools and install them at an
appropriate location. Make sure to do a custom install and install the
SDK, which is not always installed by default. For the rest of this
section, we will assume that the Debugging Tools are installed at
``C:\ Program Files\Debugging Tools for Windows``. If this is not the
case, then adjust the following instruction appropriately.

Once the Debugging Tools are installed, Visual Studio must be configured
to use them. We need to add the Debugging Tools include and library
directories to Visual Studios search paths. In Visual Studio 2003 select
``Options...`` from the tools menu. Next select Projects and VC++
Directories from the pane on the left. You should see a list of
directories that are sorted into categories such as ‘Executable files’,
‘Include files’, etc. The current category can be changed with a drop
down box in the upper right hand side of the Dialog.

| First, change to the ‘Library files’ category, and add an entry that
  points to
| ``C:\Program Files\Debugging Tools for Windows\sdk\lib\i386``. Make
  sure that this entry is above Visual Studio’s default search paths.

Next, Change to the ‘Include files’ category and make a new entry in the
list that points to
``C:\Program Files\Debugging Tools for Windows\sdk\inc``. Also make sure
that this entry is above Visual Studio’s default search paths. Some
users have had a problem where Visual Studio cannot find the
``cvconst.h`` file. You may need to add the directory containing this
file to the include search path. We have seen it installed at
``$(VCInstallDir)\ ..\Visual Studio SDKs\DIA SDK\include``, although you
may need to search for it. You also need to add the libxml2 include path
depending on the where the libxml2 is installed on the system.

Once you have installed and configured the Debugging Tools for Windows
you are ready to build Symtab. First, you need to create the directories
where Dyninst will install its completed build. From the core directory
you need to create the directories ``..\i386-unknown-nt4.0\bin`` and
``..\i386-unknown-nt4.0\lib``. Next open the solution file
core/SymtabAPI.sln with Visual Studio. You can then build SymtabAPI by
select ‘Build Solution’ from the build menu. This will build the
SymtabAPI library.
