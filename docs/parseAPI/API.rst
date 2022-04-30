.. _`sec:api`:

========
ParseAPI
========

Class CodeObject
----------------

**Defined in:** ``CodeObject.h``

The CodeObject class describes an individual binary code object, such as
an executable or library. It is the top-level container for parsing the
object as well as accessing that parse data. The following API routines
and data types are provided to support parsing and retrieving parsing
products.

.. code-block:: cpp
    
    typedef std::set<Function *, Function::less> funclist

Container for access to functions. Refer to Section
`4.12 <#sec:containers>`__ for details. Library users *must not* rely on
the underlying container type of std::set, as it is subject to change.

.. code-block:: cpp

    CodeObject(CodeSource * cs, CFGFactory * fact = NULL, ParseCallback *
    cb = NULL, bool defensiveMode = false)

Constructs a new CodeObject from the provided CodeSource and optional
object factory and callback handlers. Any parsing hints provided by the
CodeSource are processed, but the binary is not parsed when this
constructor returns. The passed CodeSource is **not** owned by this
object. However, it must have the same lifetime as the CodeObject.

The ``defensiveMode`` parameter optionally trades off coverage for
safety; this mode is not recommended for most applications as it makes
very conservative assumptions about control flow transfer instructions
(see Section `6 <#sec:defmode>`__).

.. code-block:: cpp
    
    void parse()

Recursively parses the binary represented by this CodeObject from all
known function entry points (i.e., the hints provided by the
CodeSource). This method and the following parsing methods may safely be
invoked repeatedly if new information about function locations is
provided through the CodeSource. Note that these parsing methods do not
automatically perform speculative gap parsing. parseGaps should be used
for this purpose.

.. code-block:: cpp

    void parse(Address target, bool recursive)

Parses the binary starting with the instruction at the provided target
address. If ``recursive`` is true, recursive traversal parsing is used
as in the default ``parse()`` method; otherwise only instructions
reachable through intraprocedural control flow are visited.

.. code-block:: cpp

    void parse(CodeRegion * cr, Address target, bool recursive)

Parses the specified core region of the binary starting with the
instruction at the provided target address. If ``recursive`` is true,
recursive traversal parsing is used as in the default ``parse()``
method; otherwise only instructions reachable through intraprocedural
control flow are visited.

.. code-block:: cpp

    struct NewEdgeToParse Block *source; Address target; EdgeTypeEnum type;
    bool parseNewEdges( vector<NewEdgeToParse> & worklist )

Parses a set of newly created edges specified in the worklist supplied
that were not included when the function was originally parsed.

ParseAPI is able to speculatively parse gaps (regions of binary that has
not been identified as code or data yet) to identify function entry
points and perform control flow traversal.

.. container:: center

   +------------------+--------------------------------------------------+
   | GapParsingType   | Technique description                            |
   +==================+==================================================+
   | PreambleMatching | If instruction patterns are matched at an        |
   |                  | adderss, the address is a function entry point   |
   +------------------+--------------------------------------------------+
   | IdiomMatching    | Based on a pre-trained model, this technique     |
   |                  | calculates the probability of an address to be a |
   |                  | function entry point and predicts whether which  |
   |                  | addresses are function entry points              |
   +------------------+--------------------------------------------------+


.. code-block:: cpp
   
    void parseGaps(CodeRegion *cr, GapParsingType type=IdiomMatching)

Speculatively parse the indicated region of the binary using the
specified technique to find likely function entry points, enabled on the
x86 and x86-64 platforms.

.. code-block:: cpp
    
    Function * findFuncByEntry(CodeRegion * cr, Address entry)

Find the function starting at address ``entry`` in the indicated
CodeRegion. Returns null if no such function exists.

.. code-block:: cpp

    int findFuncs(CodeRegion * cr, Address addr, std::set<Function*> & funcs)

Finds all functions spanning ``addr`` in the code region, adding each to
``funcs``. The number of results of this stabbing query are returned.

.. code-block:: cpp 

    int findFuncs(CodeRegion * cr, Address start, Address end,
    std::set<Function*> & funcs)

Finds all functions overlapping the range ``[start,end)`` in the code
region, adding each to ``funcs``. The number of results of this stabbing
query are returned.

.. code-block:: cpp

    const funclist & funcs()

Returns a const reference to a container of all functions in the binary.
Refer to Section `4.12 <#sec:containers>`__ for container access
details.

.. code-block:: cpp
    
    Block * findBlockByEntry(CodeRegion * cr, Address entry)

Find the basic block starting at address ``entry``. Returns null if no
such block exists.

.. code-block:: cpp

    int findBlocks(CodeRegion * cr, Address addr, std::set<Block*> & blocks)

Finds all blocks spanning ``addr`` in the code region, adding each to
``blocks``. Multiple blocks can be returned only on platforms with
variable-length instruction sets (such as IA32) for which overlapping
instructions are possible; at most one block will be returned on all
other platforms.

.. code-block:: cpp

    Block * findNextBlock(CodeRegion * cr, Address addr)

Find the next reachable basic block starting at address ``entry``.
Returns null if no such block exists.

.. code-block:: cpp
    
    CodeSource * cs()

Return a reference to the underlying CodeSource.

.. code-block:: cpp
    
    CFGFactory * fact()

Return a reference to the CFG object factory.

.. code-block:: cpp
    
    bool defensiveMode()

Return a boolean specifying whether or not defensive mode is enabled.

.. code-block:: cpp
    
    bool isIATcall(Address insn, std::string &calleeName)

Returns a boolean specifying if the address at ``addr`` is located at
the call named in ``calleeName``.

.. code-block:: cpp
    
    void startCallbackBatch()

Starts a batch of callbacks that have been registered.

.. code-block:: cpp
    
    void finishCallbackBatch()

Completes all callbacks in the current batch.

.. code-block:: cpp
    
    void registerCallback(ParseCallback *cb);

Register a callback ``cb``

.. code-block:: cpp
    
    void unregisterCallback(ParseCallback *cb);

Unregister an existing callback ``cb``

.. code-block:: cpp
    
    void finalize()

Force complete parsing of the CodeObject; parsing operations are
otherwise completed only as needed to answer queries.

.. code-block:: cpp
    
    void destroy(Edge *)

Destroy the edge listed.

.. code-block:: cpp
    
    void destroy(Block *)

Destroy the code block listed.

.. code-block:: cpp
    
    void destroy(Function *)

Destroy the function listed.

Class CodeRegion
----------------

**Defined in:** ``CodeSource.h``

The CodeRegion interface is an accounting structure used to divide
CodeSources into distinct regions. This interface is mostly of interest
to CodeSource implementors.

.. code-block:: cpp
    
    void names(Address addr, vector<std::string> & names)

Fills the provided vector with any names associated with the function at
a given address in the region, e.g. symbol names in an ELF or PE binary.

.. code-block:: cpp
    
    virtual bool findCatchBlock(Address addr, Address & catchStart)

Finds the exception handler associated with an address, if one exists.
This routine is only implemented for binary code sources that support
structured exception handling, such as the SymtabAPI-based
SymtabCodeSource provided as part of the ParseAPI.

.. code-block:: cpp
    
    Address low()

The lower bound of the interval of address space covered by this region.

.. code-block:: cpp
    
    Address high()

The upper bound of the interval of address space covered by this region.

.. code-block:: cpp
    
    bool contains(Address addr)

Returns true if
:math:`\small \texttt{addr} \in [\small \texttt{low()},\small \texttt{high()})`,
false otherwise.

.. code-block:: cpp
    
    virtual bool wasUserAdded() const

Return true if this region was added by the user, false otherwise.

Class Function
--------------

**Defined in:** ``CFG.h``

The Function class represents the portion of the program CFG that is
reachable through intraprocedural control flow transfers from the
function’s entry block. Functions in the ParseAPI have only a single
entry point; multiple-entry functions such as those found in Fortran
programs are represented as several functions that “share” a subset of
the CFG. Functions may be non-contiguous and may share blocks with other
functions.

.. container:: center

   ============ ==========================================
   FuncSource   Meaning
   ============ ==========================================
   RT           recursive traversal (default)
   HINT         specified in CodeSource hints
   GAP          speculative parsing heuristics
   GAPRT        recursive traversal from speculative parse
   ONDEMAND     dynamically discovered at runtime
   MODIFICATION Added via user modification
   ============ ==========================================

Return status of an function, which indicates whether this function will
return to its caller or not; see description below.

.. container:: center

   ================ ===============================
   FuncReturnStatus Meaning
   ================ ===============================
   UNSET            unparsed function (default)
   NORETURN         will not return
   UNKNOWN          cannot be determined statically
   RETURN           may return
   ================ ===============================

.. code-block:: cpp

    typedef boost::transform_iterator<selector, blockmap::iterator>
    bmap_iterator typedef boost::transform_iterator<selector,
    blockmap::const_iterator> bmap_const_iterator typedef
    boost::iterator_range<bmap_iterator> blocklist typedef
    boost::iterator_range<bmap_const_iterator> const_blocklist typedef
    std::set<Edge*> edgelist

Containers for block and edge access. Library users *must not* rely on
the underlying container type of std::set/std::vector lists, as it is
subject to change.

.. list-table:: 
   :widths: 30  35 35
   :header-rows: 1

   * - Method name
     - Return type
     - Method description
   * - name
     - string
     - Name of the function.
   * - addr
     - Address
     - Entry address of the function
   * - entry
     - Block *
     - Entry block of the function
   * - parsed
     - bool
     - Whether the function has been parsed.
   * - blocks
     - blocklist &
     - List of blocks contained by this function sorted by entry address.
   * - callEdges
     - const edgelist &
     - List of outgoing call edges from this function.
   * - returnBlocks
     - const blocklist &
     - List of all blocks ending in return edges
   * - exitBlocks
     - const blocklist &
     - List of all the blocks that end the function, including blocks with no out-edges.
   * - hasNoStackFrame
     - bool
     - True if the function does not create a stack frame
   * - saveFramePointer
     - bool
     - True if the function saves a frame pointer (e.g., %ebp)
   * - cleansOwnStack
     - bool
     - True if the function tears down stack-passed arguments upon return.
   * - region
     - CodeRegion *
     - Code region that contains the function
   * - isrc
     - InstructionSource *
     - The InstructionSource for this function
   * - obj
     - CodeObject *
     - CodeObject that contains this function.
   * - src
     - FuncSrc
     - The type of hint that identified this function's entry point
   * - restatus
     - FuncReturnStatus *
     - Returns the best-effort determination of whether this function may return or not. Return status cannot always be statically determined, and at most can guarantee that a function *may* return, not that it *will* return.
   * - getReturnType
     - Type *
     - Type representing the return type of the function


.. code-block:: cpp
    
    Function(Address addr, string name, CodeObject * obj, CodeRegion * region, InstructionSource * isource)

Creates a function at ``addr`` in the code region specified. Insructions
for this function are given in ``isource``.

.. code-block:: cpp
    
    LoopTreeNode* getLoopTree()

Return the nesting tree of the loops in the function. See class
``LoopTreeNode`` for more details

.. code-block:: cpp
    
    Loop* findLoop(const char *name)

Return the loop with the given nesting name. See class ``LoopTreeNode``
for more details about how loop nesting names are assigned.

.. code-block:: cpp
    
    bool getLoops(vector<Loop*> &loops);

Fill ``loops`` with all the loops in the function

.. code-block:: cpp
    
    bool getOuterLoops(vector<Loop*> &loops);

Fill ``loops`` with all the outermost loops in the function

.. code-block:: cpp
    
    bool dominates(Block* A, Block *B);

Return true if block ``A`` dominates block ``B``

.. code-block:: cpp
    
    Block* getImmediateDominator(Block *A);

Return the immediate dominator of block ``A``\ ，\ ``NULL`` if the block
``A`` does not have an immediate dominator.

.. code-block:: cpp
    
    void getImmediateDominates(Block *A, set<Block*> &imm);

Fill ``imm`` with all the blocks immediate dominated by block ``A``

.. code-block:: cpp
    
    void getAllDominates(Block *A, set<Block*> &dom);

Fill ``dom`` with all the blocks dominated by block ``A``

.. code-block:: cpp
    
    bool postDominates(Block* A, Block *B);

Return true if block ``A`` post-dominates block ``B``

.. code-block:: cpp
    
    Block* getImmediatePostDominator(Block *A);

Return the immediate post-dominator of block ``A``\ ，\ ``NULL`` if the
block ``A`` does not have an immediate post-dominator.

.. code-block:: cpp
    
    void getImmediatePostDominates(Block *A, set<Block*> &imm);

Fill ``imm`` with all the blocks immediate post-dominated by block ``A``

.. code-block:: cpp
    
    void getAllPostDominates(Block *A, set<Block*> &dom);

Fill ``dom`` with all the blocks post-dominated by block ``A``

.. code-block:: cpp
    
    std::vector<FuncExtent *> const& extents()

Returns a list of contiguous extents of binary code within the function.

.. code-block:: cpp
    
    void setEntryBlock(block * new_entry)

Set the entry block for this function to ``new_entry``.

.. code-block:: cpp
    
    void set_retstatus(FuncReturnStatus rs)

Set the return status for the function to ``rs``.

.. code-block:: cpp
    
    bool contains(Block *b)

Return true if this function contains the given block ``b``; otherwise
false.

.. code-block:: cpp
    
    void removeBlock(Block *)

Remove a basic block from the function.

Class Block
-----------

**Defined in:** ``CFG.h``

A Block represents a basic block as defined in Section
`2 <#sec:abstractions>`__, and is the lowest level representation of
code in the CFG.

.. code-block:: cpp
    
    typedef std::vector<Edge *> edgelist

Container for edge access. Refer to Section `4.12 <#sec:containers>`__
for details. Library users *must not* rely on the underlying container
type of std::vector, as it is subject to change.

.. list-table:: Title
   :widths: 30  35 35
   :header-rows: 1

   * - Method name
     - Return type
     - Method description
   * - start
     - Address
     - Address of the first instruction in the block
   * - end
     - Address
     - Address immediately following the last instruction in the block
   * - last
     - Address
     - Address of the last instruction in the block
   * - lastInsnAddr
     - Address
     - Alias of ``last``
   * - size
     - Address
     - Size of the block; ``end`` - ``start``.
   * - parsed
     - bool
     - Whether the block has been parsed
   * - obj
     - CodeObject *
     - CodeObject containing this block.
   * - region
     - CodeRegion *
     - CodeRegion containing this block.
   * - sources
     - const edgelist &
     - List of all in-edges to the block.
   * - targets
     - const edgelist &
     - List of all out-edges from the block.
   * - containingFuncs
     - int
     - Number of Functions that contain this block.


.. code-block:: cpp
    
    Block(CodeObject * o, CodeRegion * r, Address start, Function* f = NULL)

Creates a block at ``start`` in the code region and code object
specified. Optionally, one can specify the function that will parse the
block. This constructor is used by the ParseAPI parser, which will
update its end address during parsing.

.. code-block:: cpp
    
    Block(CodeObject * o, CodeRegion * r, Address start, Address end, Address last, Function* f = NULL)

Creates a block at ``start`` in the code region and code object
specified. The block has its last instruction at address ``last`` and
ends at address ``end``. This constructor allows external parsers to
construct their own blocks.

.. code-block:: cpp
    
    bool consistent(Address addr, Address & prev_insn)

Check whether address ``addr`` is *consistent* with this basic block. An
address is consistent if it is the boundary between two instructions in
the block. As long as ``addr`` is within the range of the block,
``prev_insn`` will contain the address of the previous instruction
boundary before ``addr``, regardless of whether ``addr`` is consistent
or not.

.. code-block:: cpp
    
    void getFuncs(std::vector<Function *> & funcs)

Fills in the provided vector with all functions that share this basic
block.

.. code-block:: cpp
    
    template <class OutputIterator> void getFuncs(OutputIterator result)

Generic version of the above; adds each Function that contains this
block to the provided OutputIterator. For example:

.. code-block:: cpp
    
    std::set<Function *> funcs;
    block->getFuncs(std::inserter(funcs, funcs.begin()));

    typedef std::map<Offset, InstructionAPI::Instruction::Ptr> Insns void
    getInsns(Insns &insns) const

Disassembles the block and stores the result in ``Insns``.

.. code-block:: cpp
    
    InstructionAPI::Instruction::Ptr getInsn(Offset o) const

Returns the instruction starting at offset ``o`` within the block.
Returns ``InstructionAPI::Instruction::Ptr()`` if ``o`` is outside the
block, or if an instruction does not begin at ``o``.

Parse API Class Edge
--------------------

**Defined in:** ``CFG.h``

Typed Edges join two blocks in the CFG, indicating the type of control
flow transfer instruction that joins the blocks to each other. Edges may
not correspond to a control flow transfer instruction at all, as in the
case of the fallthrough edge that indicates where straight-line control
flow is split by incoming transfers from another location, such as a
branch. While not all blocks end in a control transfer instruction, all
control transfer instructions end basic blocks and have outgoing edges;
in the case of unresolvable control flow, the edge will target a special
“sink” block (see ``sinkEdge()``, below).

.. container:: center

   ============== ==============================
   EdgeTypeEnum   Meaning
   ============== ==============================
   CALL           call edge
   COND_TAKEN     conditional branch–taken
   COND_NOT_TAKEN conditional branch–not taken
   INDIRECT       branch indirect
   DIRECT         branch direct
   FALLTHROUGH    direct fallthrough (no branch)
   CATCH          exception handler
   CALL_FT        post-call fallthrough
   RET            return
   ============== ==============================

.. list-table::
   :widths: 30  35 35
   :header-rows: 1

   * - Method name
     - Return type
     - Method description
   * - src
     - Block *
     - Source of the edge.
   * - trg
     - Block *
     - Target of the edge.
   * - type
     - EdgeTypeEnum
     - Type of the edge.
   * - sinkEdge
     - bool
     - True if the target is the sink block.
   * - interproc
     - bool
     - True if the edge should be interpreted as interprocedural (e.g. calls, returns, unconditional or conditional tail calls).

Class Loop
----------

**Defined in:** ``CFG.h``

The Loop class represents code that may execute repeatedly. We detect
both natural loops (loops that have a single entry block) and
irreducible loops (loops that have multiple entry blocks). A back edge
is defined as an edge that has its source in the loop and has its target
being an entry block of the loop. It represents the end of an iteration
of the loop. For all the loops detected in a function, we also build a
loop nesting tree to represent the nesting relations between the loops.
See class ``LoopTreeNode`` for more details.

.. code-block:: cpp
    
    Loop* parent

Returns the loop which directly encloses this loop. NULL if no such
loop.

.. code-block:: cpp
    
    bool containsAddress(Address addr)

Returns true if the given address is within the range of this loop’s
basic blocks.

.. code-block:: cpp
    
    bool containsAddressInclusive(Address addr)

Returns true if the given address is within the range of this loop’s
basic blocks or its children.

.. code-block:: cpp
    
    int getLoopEntries(vector<Block*>& entries);

Fills ``entries`` with the set of entry basic blocks of the loop. Return
the number of the entries that this loop has

.. code-block:: cpp
    
    int getBackEdges(vector<Edge*> &edges)

Sets ``edges`` to the set of back edges in this loop. It returns the
number of back edges that are in this loop.

.. code-block:: cpp
    
    bool getContainedLoops(vector<Loop*> &loops)

Returns a vector of loops that are nested under this loop.

.. code-block:: cpp
    
    bool getOuterLoops(vector<Loop*> &loops)

Returns a vector of loops that are directly nested under this loop.

.. code-block:: cpp
    
    bool getLoopBasicBlocks(vector<Block*> &blocks)

Fills ``blocks`` with all basic blocks in the loop

.. code-block:: cpp
    
    bool getLoopBasicBlocksExclusive(vector<Block*> &blocks)

Fills ``blocks`` with all basic blocks in this loop, excluding the
blocks of its sub loops.

.. code-block:: cpp
    
    bool hasBlock(Block *b);

Returns ``true`` if this loop contains basic block ``b``.

.. code-block:: cpp
    
    bool hasBlockExclusive(Block *b);

Returns ``true`` if this loop contains basic block ``b`` and ``b`` is
not in its sub loops.

.. code-block:: cpp
    
    bool hasAncestor(Loop *loop)

Returns true if this loop is a descendant of the given loop.

.. code-block:: cpp
    
    Function * getFunction();

Returns the function that this loop is in.

Class LoopTreeNode
------------------

**Defined in:** ``CFG.h``

The LoopTreeNode class provides a tree interface to a collection of
instances of class Loop contained in a function. The structure of the
tree follows the nesting relationship of the loops in a function. Each
LoopTreeNode contains a pointer to a loop (represented by Loop), and a
set of sub-loops (represented by other LoopTreeNode objects). The
``loop`` field at the root node is always ``NULL`` since a function may
contain multiple outer loops. The ``loop`` field is never ``NULL`` at
any other node since it always corresponds to a real loop. Therefore,
the outer most loops in the function are contained in the vector of
``children`` of the root.

Each instance of LoopTreeNode is given a name that indicates its
position in the hierarchy of loops. The name of each outermost loop
takes the form of ``loop_x``, where ``x`` is an integer from 1 to n,
where n is the number of outer loops in the function. Each sub-loop has
the name of its parent, followed by a ``.y``, where ``y`` is 1 to m,
where m is the number of sub-loops under the outer loop. For example,
consider the following C function:

.. code-block:: cpp
    
    void foo() {
     int x, y, z, i;
     for (x=0; x<10; x++) {
       for (y = 0; y<10; y++)
         ...
       for (z = 0; z<10; z++)
         ...
     }
     for (i = 0; i<10; i++) {
        ...
     }
   }

The ``foo`` function will have a root LoopTreeNode, containing a NULL
loop entry and two LoopTreeNode children representing the functions
outermost loops. These children would have names ``loop_1`` and
``loop_2``, respectively representing the ``x`` and ``i`` loops.
``loop_2`` has no children. ``loop_1`` has two child LoopTreeNode
objects, named ``loop_1.1`` and ``loop_1.2``, respectively representing
the ``y`` and ``z`` loops.

.. code-block:: cpp
    
    Loop *loop;

The Loop instance it points to.

.. code-block:: cpp
    
    std::vector<LoopTreeNode *> children;

The LoopTreeNode instances nested within this loop.

.. code-block:: cpp
    
    const char * name();

Returns the hierarchical name of this loop.

.. code-block:: cpp
    
    const char * getCalleeName(unsigned int i)

Returns the function name of the ith callee.

.. code-block:: cpp
    
    unsigned int numCallees()

Returns the number of callees contained in this loop’s body.

.. code-block:: cpp
    
    bool getCallees(vector<Function *> &v);

Fills ``v`` with a vector of the functions called inside this loop.

.. code-block:: cpp
    
    Loop * findLoop(const char *name);

Looks up a loop by the hierarchical name

.. _`sec:codesource`:

Class CodeSource
----------------

**Defined in:** ``CodeSource.h``

The CodeSource interface is used by the ParseAPI to retrieve binary code
from an executable, library, or other binary code object; it also can
provide hints of function entry points (such as those derived from
debugging symbols) to seed the parser. The ParseAPI provides a default
implementation based on the SymtabAPI that supports many common binary
formats. For details on implementing a custom CodeSource, see Appendix
`5 <#sec:extend>`__.

.. code-block:: cpp
    
    virtual bool nonReturning(Address func_entry) virtual bool
    nonReturning(std::string func_name)

Looks up whether a function returns (by name or location). This
information may be statically known for some code sources, and can lead
to better parsing accuracy.

.. code-block:: cpp
    
    virtual bool nonReturningSyscall(int /*number*/)

Looks up whether a system call returns (by system call number). This
information may be statically known for some code sources, and can lead
to better parsing accuracy.

.. code-block:: cpp
    
    virtual Address baseAddress() virtual Address loadAddress()

If the binary file type supplies non-zero base or load addresses (e.g.
Windows PE), implementations should override these functions.

.. code-block:: cpp
    
    std::map< Address, std::string > & linkage()

Returns a reference to the external linkage map, which may or may not be
filled in for a particular CodeSource implementation.

.. code-block:: cpp
    
    struct Hint Address _addr; CodeRegion *_region; std::string _name;
    Hint(Addr, CodeRegion *, std::string); std::vector< Hint > const&
    hints()

Returns a vector of the currently defined function entry hints.

.. code-block:: cpp
    
    std::vector<CodeRegion *> const& regions()

Returns a read-only vector of code regions within the binary represented
by this code source.

.. code-block:: cpp
    
    int findRegions(Address addr, set<CodeRegion *> & ret)

Finds all CodeRegion objects that overlap the provided address. Some
code sources (e.g. archive files) may have several regions with
overlapping address ranges; others (e.g. ELF binaries) do not.

.. code-block:: cpp
    
    bool regionsOverlap()

Indicates whether the CodeSource contains overlapping regions.

Class ParseCallback
-------------------

**Defined in:** ``ParseCallback.h``

The ParseCallback class allows ParseAPI users to be notified of various
events during parsing. For most users this notification is unnecessary,
and an instantiation of the default ParseCallback can be passed to the
CodeObject during initialization. Users who wish to be notified must
implement a class that inherits from ParseCallback, and implement one or
more of the methods described below to receive notification of those
events.

.. code-block:: cpp
    
    struct default_details default_details(unsigned char * b,size_t s, bool ib); unsigned char * ibuf; size_t isize; bool isbranch;

Details used in the ``unresolved_cf`` and ``abruptEnd_cf`` callbacks.

.. code-block:: cpp
    
    virtual void instruction_cb(Function *, Block *, Address, insn_details*)

Invoked for each instruction decoded during parsing. Implementing this
callback may incur significant overhead.

.. code-block:: cpp
    
    struct insn_details InsnAdapter::InstructionAdapter * insn;
    void interproc_cf(Function *, Address, interproc_details *)

Invoked for each interprocedural control flow instruction.

.. code-block:: cpp
    
    struct interproc_details typedef enum ret, call, branch_interproc, //
    tail calls, branches to plts syscall type_t; unsigned char * ibuf;
    size_t isize; type_t type; union struct Address target; bool
    absolute_address; bool dynamic_call; call; data;

Details used in the ``interproc_cf`` callback.

.. code-block:: cpp
    
    void overlapping_blocks(Block *, Block *)

Noification of inconsistent parse data (overlapping blocks).

Class FuncExtent
----------------

**Defined in:** ``CFG.h``

Function Extents are used internally for accounting and lookup purposes.
They may be useful for users who wish to precisely identify the ranges
of the address space spanned by functions (functions are often
discontiguous, particularly on architectures with variable length
instruction sets).

=========== =========== ===============================
Method name Return type Method description
=========== =========== ===============================
func        Function *  Function linked to this extent.
start       Address     Start of the extent.
end         Address     End of the extent (exclusive).
=========== =========== ===============================

.. _`sec:pred`:

Edge Predicates
---------------

**Defined in:** ``CFG.h``

Edge predicates control iteration over edges. For example, the provided
``Intraproc`` edge predicate can be used with filter iterators and
standard algorithms, ensuring that only intraprocedural edges are
visited during iteration. Two other examples of edge predicates are
provided: ``SingleContext`` only visits edges that stay in a single
function context, and ``NoSinkPredicate`` does not visit edges to the
*sink* block. The following code traverses all of the basic blocks
within a function:

.. code-block:: cpp
    
       #include <boost/filter_iterator.hpp>
       using boost::make_filter_iterator;
       struct target_block
       {
         Block* operator()(Edge* e) { return e->trg(); }
       };


       vector<Block*> work;
       Intraproc epred; // ignore calls, returns
      
       work.push_back(func->entry()); // assuming `func' is a Function*

       // do_stuff is a functor taking a Block* as its argument
       while(!work.empty()) {
           Block * b = work.back();
           work.pop_back();

           Block::edgelist & targets = block->targets();
           // Do stuff for each out edge
           std::for_each(make_filter_iterator(targets.begin(), epred), 
                         make_filter_iterator(targets.end(), epred),
                         do_stuff());
           std::transform(make_filter_iterator(targets.begin(), epred),
                          make_filter_iterator(targets.end(), epred), 
                          std::back_inserter(work), 
                          std::mem_fun(Edge::trg));
           Block::edgelist::const_iterator found_interproc =
                   std::find_if(targets.begin(), targets.end(), Interproc());
           if(interproc != targets.end()) {
                   // do something with the interprocedural edge you found
           }
       }

Anything that can be treated as a function from ``Edge*`` to a ``bool``
can be used in this manner. This replaces the beta interface where all
``EdgePredicate``\ s needed to descend from a common parent class. Code
that previously constructed iterators from an edge predicate should be
replaced with equivalent code using filter iterators as follows:

.. code-block:: cpp
    
     // OLD
     for(Block::edgelist::iterator i = targets.begin(epred); 
         i != targets.end(epred); 
         i++)
     {
       // ...
     }
     // NEW
     for_each(make_filter_iterator(epred, targets.begin(), targets.end()),
              make_filter_iterator(epred, targets.end(), targets,end()),
              loop_body_as_function);
     // NEW (C++11)
     for(auto i = make_filter_iterator(epred, targets.begin(), targets.end()); 
         i != make_filter_iterator(epred, targets.end(), targets.end()); 
         i++)
     {
       // ...
     }
     

.. _`sec:containers`:

Containers
----------

Several of the ParseAPI data structures export containers of CFG
objects; the CodeObject provides a list of functions in the binary, for
example, while functions provide lists of blocks and so on. To avoid
tying the internal storage for these structures to any particular
container type, ParseAPI objects export a ContainerWrapper that provides
an iterator interface to the internal containers. These wrappers and
predicate interfaces are designed to add minimal overhead while
protecting ParseAPI users from exposure to internal container storage
details. Users *must not* rely on properties of the underlying container
type (e.g. storage order) unless that property is explicity stated in
this manual.

ContainerWrapper containers export the following interface (``iterator``
types vary depending on the template parameters of the ContainerWrapper,
but are always instantiations of the PredicateIterator described below):

.. code-block:: cpp
    
    iterator begin() iterator begin(predicate *)

Return an iterator pointing to the beginning of the container, with or
without a filtering predicate implementation (see Section
`4.11 <#sec:pred>`__ for details on filter predicates).

.. code-block:: cpp
    
    iterator const& end()

Return the iterator pointing to the end of the container (past the last
element).

.. code-block:: cpp
    
    size_t size()

Returns the number of elements in the container. Execution cost may vary
depending on the underlying container type.

.. code-block:: cpp
    
    bool empty()

Indicates whether the container is empty or not.

The elements in ParseAPI containers can be accessed by iteration using
an instantiation of the PredicateIterator. These iterators can
optionally act as filters, evaluating a boolean predicate for each
element and only returning those elements for which the predicate
returns true. *Iterators with non-null predicates may return fewer
elements during iteration than their ``size()`` method indicates.*
Currently PredicateIterators only support forward iteration. The
operators ``++`` (prefix and postfix), ``==``, ``!=``, and ``*``
(dereference) are supported.