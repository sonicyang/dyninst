.. _sec:patchapi-api:

PatchAPI
########

This section describes public interfaces in PatchAPI. The API is
organized as a collection of C++ classes. The classes in PatchAPI fall
under the C++ namespace Dyninst::PatchAPI. To access them, programmers
should refer to them using the “Dyninst::PatchAPI::” prefix, e.g.,
Dyninst::PatchAPI::Point. Alternatively, programmers can add the C++
*using* keyword above any references to PatchAPI objects, e.g.,\ *using
namespace Dyninst::PatchAPI* or *using Dyninst::PatchAPI::Point*.

Classes in PatchAPI use either the C++ raw pointer or the boost shared
pointer (*boost::shared_ptr<T>*) for memory management. A class uses a
raw pointer whenever it is returning a handle to the user that is
controlled and destroyed by the PatchAPI runtime library. Classes that
use a raw pointer include the CFG objects, a Point, and various plugins,
e.g., AddrSpace, CFGMaker, PointMaker, and Instrumenter. A class uses a
shared_pointer whenever it is handing something to the user that the
PatchAPI runtime library is not controlling and destroying. Classes that
use a boost shared pointer include a Snippet, PatchMgr, and Instance,
where we typedef a class’s shared pointer by appending the Ptr to the
class name, e.g., PatchMgrPtr for PatchMgr.

CFG Interface
*************

.. _sec-3.2.8:

PatchObject
===========

**Declared in**: PatchObject.h

The PatchObject class is a wrapper of ParseAPI’s CodeObject class
(has-a), which represents an individual binary code object, such as an
executable or a library.

.. code-block:: cpp
    
    static PatchObject* create(ParseAPI::CodeObject* co, Address base,
    CFGMaker* cm = NULL, PatchCallback *cb = NULL);

Creates an instance of PatchObject, which has *co* as its on-disk
representation (ParseAPI::CodeObject), and *base* as the base address
where this object is loaded in the memory. For binary rewriting, base
should be 0. The *cm* and *cb* parameters are for registering plugins.
If *cm* or *cb* is NULL, then we use the default implementation of
CFGMaker or PatchCallback.

.. code-block:: cpp
    
    static PatchObject* clone(PatchObject* par_obj, Address base,
    CFGMaker* cm = NULL, PatchCallback *cb = NULL);

Returns a new object that is copied from the specified object *par_obj*
at the loaded address *base* in the memory. For binary rewriting, base
should be 0. The *cm* and *cb* parameters are for registering plugins.
If *cm* or *cb* is NULL, then we use the default implementation of
CFGMaker or PatchCallback.

.. code-block:: cpp
    
    Address codeBase();

Returns the base address where this object is loaded in memory.

.. code-block:: cpp
    
    PatchFunction *getFunc(ParseAPI::Function *func, bool create = true);

Returns an instance of PatchFunction in this object, based on the *func*
parameter. PatchAPI creates a PatchFunction on-demand, so if there is
not any PatchFunction created for the ParseAPI function *func*, and the
*create* parameter is false, then no any instance of PatchFunction will
be created.

It returns NULL in two cases. First, the function *func* is not in this
PatchObject. Second, the PatchFunction is not yet created and the
*create* is false. Otherwise, it returns a PatchFunction.

.. code-block:: cpp
    
    template <class Iter> void funcs(Iter iter);

Outputs all instances of PatchFunction in this PatchObject to the STL
inserter *iter*.

.. code-block:: cpp
    
    PatchBlock *getBlock(ParseAPI::Block* blk, bool create = true);

Returns an instance of PatchBlock in this object, based on the *blk*
parameter. PatchAPI creates a PatchBlock on-demand, so if there is not
any PatchBlock created for the ParseAPI block *blk*, and the *create*
parameter is false, then no any instance of PatchBlock will be created.

It returns NULL in two cases. First, the ParseAPI block *blk* is not in
this PatchObject. Second, the PatchBlock is not yet created and the
*create* is false. Otherwise, it returns a PatchBlock.

.. code-block:: cpp
    
    template <class Iter> void blocks(Iter iter);

Outputs all instances of PatchBlock in this object to the STL inserter
*iter*.

.. code-block:: cpp
    
     PatchEdge *getEdge(ParseAPI::Edge* edge, PatchBlock* src,
     PatchBlock* trg, bool create = true);

Returns an instance of PatchEdge in this object, according to the
parameters ParseAPI::Edge *edge*, source PatchBlock *src*, and target
PatchBlock *trg*. PatchAPI creates a PatchEdge on-demand, so if there is
not any PatchEdge created for the ParseAPI *edge*, and the *create*
parameter is false, then no any instance of PatchEdge will be created.

It returns NULL in two cases. First, the ParseAPI *edge* is not in this
PatchObject. Second, the PatchEdge is not yet created and the *create*
is false. Otherwise, it returns a PatchEdge.

.. code-block:: cpp
    
    template <class Iter> void edges(Iter iter);

Outputs all instances of PatchEdge in this object to the STL inserter
*iter*.

.. code-block:: cpp
    
   PatchCallback *cb() const;

Returns the PatchCallback object associated with this PatchObject.

.. _sec-3.2.9:

PatchFunction
=============

**Declared in**: PatchCFG.h

The PatchFunction class is a wrapper of ParseAPI’s Function class
(has-a), which represents a function.

.. code-block:: cpp
    
    const string &name();

Returns the function’s mangled name.

.. code-block:: cpp
    
    Address addr() const;

Returns the address of the first instruction in this function.

.. code-block:: cpp
    
    ParseAPI::Function *function();

Returns the ParseAPI::Function associated with this PatchFunction.

.. code-block:: cpp
    
    PatchObject* obj();

Returns the PatchObject associated with this PatchFunction.

.. code-block:: cpp
    
    typedef std::set<PatchBlock *> PatchFunction::Blockset;
    const Blockset &blocks();

Returns a set of all PatchBlocks in this PatchFunction.

.. code-block:: cpp
    
    PatchBlock *entry();

Returns the entry block of this PatchFunction.

.. code-block:: cpp
    
    const Blockset &exitBlocks();

Returns a set of exit blocks of this PatchFunction.

.. code-block:: cpp
    
    const Blockset &callBlocks();

Returns a set of all call blocks of this PatchFunction.

.. code-block:: cpp
    
    PatchCallback *cb() const;

Returns the PatchCallback object associated with this PatchFunction.

.. code-block:: cpp
    
    PatchLoopTreeNode* getLoopTree()

Return the nesting tree of the loops in the function. See class
``PatchLoopTreeNode`` for more details

.. code-block:: cpp
    
    PatchLoop* findLoop(const char *name)

Return the loop with the given nesting name. See class
``PatchLoopTreeNode`` for more details about how loop nesting names are
assigned.

.. code-block:: cpp
    
    bool getLoops(vector<PatchLoop*> &loops);

Fill ``loops`` with all the loops in the function

.. code-block:: cpp
    
    bool getOuterLoops(vector<PatchLoop*> &loops);

Fill ``loops`` with all the outermost loops in the function

.. code-block:: cpp
    
    bool dominates(PatchBlock* A, PatchBlock *B);

Return true if block ``A`` dominates block ``B``

.. code-block:: cpp
    
    PatchBlock* getImmediateDominator(PatchBlock *A);

Return the immediate dominator of block ``A``\ ，\ ``NULL`` if the block
``A`` does not have an immediate dominator.

.. code-block:: cpp
    
    void getImmediateDominates(PatchBlock *A, set<PatchBlock*> &imm);

Fill ``imm`` with all the blocks immediate dominated by block ``A``

.. code-block:: cpp
    
    void getAllDominates(PatchBlock *A, set<PatchBlock*> &dom);

Fill ``dom`` with all the blocks dominated by block ``A``

.. code-block:: cpp
    
    bool postDominates(PatchBlock* A, PatchBlock *B);

Return true if block ``A`` post-dominates block ``B``

.. code-block:: cpp
    
    PatchBlock* getImmediatePostDominator(PatchBlock *A);

Return the immediate post-dominator of block ``A``\ ，\ ``NULL`` if the
block ``A`` does not have an immediate post-dominator.

.. code-block:: cpp
    
    void getImmediatePostDominates(PatchBlock *A, set<PatchBlock*> &imm);

Fill ``imm`` with all the blocks immediate post-dominated by block ``A``

.. code-block:: cpp
    
    void getAllPostDominates(PatchBlock *A, set<PatchBlock*> &dom);

Fill ``dom`` with all the blocks post-dominated by block ``A``

.. _sec-3.2.10:

PatchBlock
==========

**Declared in**: PatchCFG.h

The PatchBlock class is a wrapper of ParseAPI’s Block class (has-a),
which represents a basic block.

.. code-block:: cpp
    
    Address start() const;

Returns the lower bound of this block (the address of the first
instruction).

.. code-block:: cpp
    
    Address end() const;

Returns the upper bound (open) of this block (the address immediately
following the last byte in the last instruction).

.. code-block:: cpp
    
    Address last() const;

Returns the address of the last instruction in this block.

.. code-block:: cpp
    
    Address size() const;

Returns end() - start().

.. code-block:: cpp
    
    bool isShared();

Indicates whether this block is contained by multiple functions.

.. code-block:: cpp
    
    int containingFuncs() const;

Returns the number of functions that contain this block.

.. code-block:: cpp
    
    typedef std::map<Address, InstructionAPI::Instruction::Ptr> Insns; void getInsns(Insns &insns) const;

This function outputs Instructions that are in this block to *insns*.

.. code-block:: cpp
    
    InstructionAPI::Instruction::Ptr getInsn(Address a) const;

Returns an Instruction that has the address *a* as its starting address.
If no any instruction can be found in this block with the starting
address *a*, it returns InstructionAPI::Instruction::Ptr().

.. code-block:: cpp
    
    std::string disassemble() const;

Returns a string containing the disassembled code for this block. This
is mainly for debugging purpose.

.. code-block:: cpp
    
    bool containsCall();

Indicates whether this PatchBlock contains a function call instruction.

.. code-block:: cpp
    
    bool containsDynamicCall();

Indicates whether this PatchBlock contains any indirect function call,
e.g., via function pointer.

.. code-block:: cpp
    
    PatchFunction* getCallee();

Returns the callee function, if this PatchBlock contains a function
call; otherwise, NULL is returned.

.. code-block:: cpp
    
    PatchFunction *function() const;

Returns a PatchFunction that contains this PatchBlock. If there are
multiple PatchFunctions containing this PatchBlock, then a random one of
them is returned.

.. code-block:: cpp
    
    ParseAPI::Block *block() const;

Returns the ParseAPI::Block associated with this PatchBlock.

.. code-block:: cpp
    
    PatchObject* obj() const;

Returns the PatchObject that contains this block.

.. code-block:: cpp
    
    typedef std::vector<PatchEdge*> PatchBlock::edgelist;
    const edgelist &sources();

Returns a list of the source PatchEdges. This PatchBlock is the target
block of the returned edges.

.. code-block:: cpp
    
    const edgelist &targets();

Returns a list of the target PatchEdges. This PatchBlock is the source
block of the returned edges.

.. code-block:: cpp
    
    template <class OutputIterator> void getFuncs(OutputIterator result);

Outputs all functions containing this PatchBlock to the STL inserter
*result*.

.. code-block:: cpp
    
    PatchCallback *cb() const;

Returns the PatchCallback object associated with this PatchBlock.

.. _sec-3.2.11:

PatchEdge
=========

**Declared in**: PatchCFG.h

The PatchEdge class is a wrapper of ParseAPI’s Edge class (has-a), which
joins two PatchBlocks in the CFG, indicating the type of control flow
transfer instruction that joins the basic blocks to each other.

.. code-block:: cpp
    
    ParseAPI::Edge *edge() const;

Returns a ParseAPI::Edge associated with this PatchEdge.

.. code-block:: cpp
    
    PatchBlock *src();

Returns the source PatchBlock.

.. code-block:: cpp
    
    PatchBlock *trg();

Returns the target PatchBlock.

.. code-block:: cpp
    
    ParseAPI::EdgeTypeEnum type() const;

Returns the edge type (ParseAPI::EdgeTypeEnum, please see `ParseAPI
Manual <ftp://ftp.cs.wisc.edu/paradyn/releases/release7.0/doc/parseapi.pdf>`__).

.. code-block:: cpp
    
    bool sinkEdge() const;

Indicates whether this edge targets the special sink block, where a sink
block is a block to which all unresolvable control flow instructions
will be linked.

.. code-block:: cpp
    
    bool interproc() const;

Indicates whether the edge should be interpreted as interprocedural
(e.g., calls, returns, direct branches under certain circumstances).

.. code-block:: cpp
    
    PatchCallback *cb() const;

Returns a Patchcallback object associated with this PatchEdge.

.. _sec-3.2.12:

PatchLoop
=========

**Declared in**: PatchCFG.h

The PatchLoop class is a wrapper of ParseAPI’s Loop class (has-a). It
represents code structure that may execute repeatedly.

.. code-block:: cpp
    
    PatchLoop* parent

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
    
    int getLoopEntries(vector<PatchBlock*>& entries);

Fills ``entries`` with the set of entry basic blocks of the loop. Return
the number of the entries that this loop has

.. code-block:: cpp
    
    int getBackEdges(vector<PatchEdge*> &edges)

Sets ``edges`` to the set of back edges in this loop. It returns the
number of back edges that are in this loop.

.. code-block:: cpp
    
    bool getContainedLoops(vector<PatchLoop*> &loops)

Returns a vector of loops that are nested under this loop.

.. code-block:: cpp
    
    bool getOuterLoops(vector<PatchLoop*> &loops)

Returns a vector of loops that are directly nested under this loop.

.. code-block:: cpp
    
    bool getLoopBasicBlocks(vector<PatchBlock*> &blocks)

Fills ``blocks`` with all basic blocks in the loop

.. code-block:: cpp
    
    bool getLoopBasicBlocksExclusive(vector<PatchBlock*> &blocks)

Fills ``blocks`` with all basic blocks in this loop, excluding the
blocks of its sub loops.

.. code-block:: cpp
    
    bool hasBlock(PatchBlock *b);

Returns ``true`` if this loop contains basic block ``b``.

.. code-block:: cpp
    
    bool hasBlockExclusive(PatchBlock *b);

Returns ``true`` if this loop contains basic block ``b`` and ``b`` is
not in its sub loops.

.. code-block:: cpp
    
    bool hasAncestor(PatchLoop *loop)

Returns true if this loop is a descendant of the given loop.

.. code-block:: cpp
    
    PatchFunction * getFunction();

Returns the function that this loop is in.

.. _sec-3.2.13:

PatchLoopTreeNode
=================

**Declared in**: PatchCFG.h

The PatchLoopTreeNode class provides a tree interface to a collection of
instances of class PatchLoop contained in a function. The structure of
the tree follows the nesting relationship of the loops in a function.
Each PatchLoopTreeNode contains a pointer to a loop (represented by
PatchLoop), and a set of sub-loops (represented by other
PatchLoopTreeNode objects). The ``loop`` field at the root node is
always ``NULL`` since a function may contain multiple outer loops. The
``loop`` field is never ``NULL`` at any other node since it always
corresponds to a real loop. Therefore, the outer most loops in the
function are contained in the vector of ``children`` of the root.

Each instance of PatchLoopTreeNode is given a name that indicates its
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

The ``foo`` function will have a root PatchLoopTreeNode, containing a
NULL loop entry and two PatchLoopTreeNode children representing the
functions outermost loops. These children would have names ``loop_1``
and ``loop_2``, respectively representing the ``x`` and ``i`` loops.
``loop_2`` has no children. ``loop_1`` has two child PatchLoopTreeNode
objects, named ``loop_1.1`` and ``loop_1.2``, respectively representing
the ``y`` and ``z`` loops.


.. code-block:: cpp
    
    PatchLoop *loop;

The PatchLoop instance it points to.

.. code-block:: cpp
    
    std::vector<PatchLoopTreeNode *> children;

The PatchLoopTreeNode instances nested within this loop.

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
    
    bool getCallees(vector<PatchFunction *> &v);

Fills ``v`` with a vector of the functions called inside this loop.

.. code-block:: cpp
    
    PatchLoop * findLoop(const char *name);

Looks up a loop by the hierarchical name

.. _sec-3.1:

Point/Snippet Interface
***********************

.. _sec-3.1.1:

PatchMgr
========

**Declared in**: PatchMgr.h

The PatchMgr class is the top-level class for finding instrumentation
**Points**, inserting or deleting **Snippets**, and registering
user-provided plugins.

.. code-block:: cpp
    
    static PatchMgrPtr create(AddrSpace* as, Instrumenter* inst = NULL, PointMaker* pm = NULL);

This factory method creates a new PatchMgr object that performs binary
code patching. It takes input three plugins, including AddrSpace *as*,
Instrumenter *inst*, and PointMaker *pm*. PatchAPI uses default plugins
for PointMaker and Instrumenter, if *pm* and *inst* are not specified
(NULL by default).

This method returns PatchMgrPtr() if it was unable to create a new
PatchMgr object.

.. code-block:: cpp
    
    Point *findPoint(Location loc, Point::Type type, bool create = true);

This method returns a unique Point according to a Location *loc* and a
Type *type*. The Location structure is to specify a physical location of
a Point (e.g., at function entry, at block entry, etc.), details of
Location will be covered in Section `4.2.2 <#sec-3.1.2>`__. PatchAPI
creates Points on demand, so if a Point is not yet created, the *create*
parameter is to indicate whether to create this Point. If the Point we
want to find is already created, this method simply returns a pointer to
this Point from a buffer, no matter whether *create* is true or false.
If the Point we want to find is not yet created, and *create* is true,
then this method constructs this Point and put it in a buffer, and
finally returns a Pointer to this Point. If the Point creation fails,
this method also returns false. If the Point we want to find is not yet
created, and *create* is false, this method returns NULL. The basic
logic of finding a point can be found in the
Listing `[findpt] <#findpt>`__.

.. code-block:: cpp
    
   if (point is in the buffer) {
     return point;
   } else {
     if (create == true) {
       create point
       if (point creation fails) return NULL;
       put the point in the buffer
     } else {
       return NULL;
     }
   }

.. code-block:: cpp
    

    template <class OutputIterator> bool findPoint(Location loc, Point::Type type, OutputIterator outputIter, bool create = true);

This method finds a Point at a physical Location *loc* with a *type*. It
adds the found Point to *outputIter* that is a STL inserter. The point
is created on demand. If the Point is already created, then this method
outputs a pointer to this Point from a buffer. Otherwise, the *create*
parameter indicates whether to create this Point.

This method returns true if a point is found, or the *create* parameter
is false; otherwise, it returns false.

.. code-block:: cpp
    
    template <class OutputIterator> bool findPoints(Location loc,
    Point::Type types, OutputIterator outputIter, bool create = true);

This method finds Points at a physical Location *loc* with composite
*types* that are combined using the overloaded operator “\|”. This
function outputs Points to the STL inserter *outputIter*. The point is
created on demand. If the Point is already created, then this method
outputs a pointer to this Point from a buffer. Otherwise, the *create*
parameter indicates whether to create this Point.

This method returns true if a point is found, or the *create* parameter
is false; otherwise, it returns false.

.. code-block:: cpp
    
    template <class FilterFunc, class FilterArgument, class OutputIterator>
    bool findPoints(Location loc, Point::Type types, FilterFunc filter_func,
    FilterArgument filter_arg, OutputIterator outputIter, bool create = true);

This method finds Points at a physical Location *loc* with composite
*types* that are combined using the overloaded operator “\|”. Then, this
method applies a filter functor *filter_func* with an argument
*filter_arg* on each found Point. The method outputs Points to the
inserter *outputIter*. The point is created on demand. If the Point is
already created, then this method returns a pointer to this Point from a
buffer. Otherwise, the *create* parameter indicates whether to create
this Point.

If no any Point is created, then this method returns false; otherwise,
true is returned. The code below shows the prototype of an example
functor.

.. code-block:: cpp
    
   template <class T>
   class FilterFunc {
     public:
       bool operator()(Point::Type type, Location loc, T arg) {
         // The logic to check whether this point is what we need
         return true;
       }
   };

In the functor FilterFunc above, programmers check each candidate Point
by looking at the Point::Type, Location, and the user-specified
parameter *arg*. If the return value is true, then the Point being
checked will be put in the STL inserter *outputIter*; otherwise, this
Point will be discarded.

.. code-block:: cpp
    
    struct Scope Scope(PatchBlock *b); Scope(PatchFunction *f, PatchBlock *b); Scope(PatchFunction *f);;

The Scope structure specifies the scope to find points, where a scope
could be a function, or a basic block. This is quite useful if
programmers don’t know the exact Location, then they can use Scope as a
wildcard. A basic block can be contained in multiple functions. The
second constructor only specifies the block *b* in a particular function
*f*.

.. code-block:: cpp
    
    template <class FilterFunc, class FilterArgument, class OutputIterator>
    bool findPoints(Scope scope, Point::Type types, FilterFunc filter_func,
    FilterArgument filter_arg, OutputIterator output_iter, bool create = true);

This method finds points in a *scope* with certain *types* that are
combined together by using the overloaded operator “\|”. Then, this
method applies the filter functor *filter_func* on each found Point. It
outputs Points where *filter_func* returns true to the STL inserter
*output_iter*. Points are created on demand. If some points are already
created, then this method outputs pointers to them from a buffer.
Otherwise, the *create* parameter indicates whether to create Points.

If no any Point is created, then this function returns false; otherwise,
true is returned.

.. code-block:: cpp
    
    template <class OutputIterator> bool findPoints(Scope scope, Point::Type types, OutputIterator output_iter, bool create = true);

This method finds points in a *scope* with certain *types* that are
combined together by using the overloaded operator “\|”. It outputs the
found points to the STL inserter *output_iter*. If some points are
already created, then this method outputs pointers to them from a
buffer. Otherwise, the *create* parameter indicates whether to create
Points.

If no any Point is created, then this method returns false; otherwise,
true is returned.

.. code-block:: cpp
    
    bool removeSnippet(InstancePtr);

This method removes a snippet Instance.

It returns false if the point associated with this Instance cannot be
found; otherwise, true is returned.

.. code-block:: cpp
    
    template <class FilterFunc, class FilterArgument> bool
    removeSnippets(Scope scope, Point::Type types, FilterFunc filter_func,
    FilterArgument filter_arg);

This method deletes ALL snippet instances at certain points in certain
*scope* with certain *types*, and those points pass the test of
*filter_func*.

If no any point can be found, this method returns false; otherwise, true
is returned.

.. code-block:: cpp
    
    bool removeSnippets(Scope scope, Point::Type types);

This method deletes ALL snippet instances at certain points in certain
*scope* with certain *types*.

If no any point can be found, this method returns false; otherwise, true
is returned.

.. code-block:: cpp
    
    void destroy(Point *point);

This method is to destroy the specified *Point*.

.. code-block:: cpp
    
    AddrSpace* as() const; PointMaker* pointMaker() const; Instrumenter* instrumenter() const;

The above three functions return the corresponding plugin: AddrSpace,
PointMaker, Instrumenter.

.. _sec-3.1.2:

Point
=====

**Declared in**: Point.h

The Point class is in essence a container of a list of snippet
instances. Therefore, the Point class has methods similar to those in
STL.

.. code-block:: cpp
    
    struct Location static Location Function(PatchFunction *f); static
    Location Block(PatchBlock *b); static Location
    BlockInstance(PatchFunction *f, PatchBlock *b, bool trusted = false);
    static Location Edge(PatchEdge *e); static Location
    EdgeInstance(PatchFunction *f, PatchEdge *e); static Location
    Instruction(PatchBlock *b, Address a); static Location
    InstructionInstance(PatchFunction *f, PatchBlock *b, Address a);
    static Location InstructionInstance(PatchFunction *f, PatchBlock *b,
    Address a, InstructionAPI::Instruction::Ptr i, bool trusted = false);
    static Location EntrySite(PatchFunction *f, PatchBlock *b, bool
    trusted = false); static Location CallSite(PatchFunction *f, PatchBlock
    *b); static Location ExitSite(PatchFunction *f, PatchBlock *b);;

The Location structure uniquely identifies the physical location of a
point. A Location object plus a Point::Type value uniquely identifies a
point, because multiple Points with different types can exist at the
same physical location. The Location structure provides a set of static
functions to create an object of Location, where each function takes the
corresponding CFG structures to identify a physical location. In
addition, some functions above (e.g., InstructionInstance) takes input
the *trusted* parameter that is to indicate PatchAPI whether the CFG
structures passed in is trusted. If the *trusted* parameter is false,
then PatchAPI would have additional checking to verify the CFG
structures passed by users, which causes nontrivial overhead.

.. code-block:: cpp

    enum Point::Type PreInsn, PostInsn, BlockEntry, BlockExit, BlockDuring, FuncEntry, FuncExit, FuncDuring, EdgeDuring, PreCall, PostCall, OtherPoint, None, InsnTypes = PreInsn | PostInsn, BlockTypes = BlockEntry | BlockExit | BlockDuring, FuncTypes = FuncEntry | FuncExit | FuncDuring, EdgeTypes = EdgeDuring, CallTypes = PreCall | PostCall;

The enum Point::Type specifies the logical point type. Multiple enum
values can be OR-ed to form a composite type. For example, the composite
type of “PreCall \| BlockEntry \| FuncExit” is to specify a set of
points with the type PreCall, or BlockEntry, or FuncExit.

.. code-block:: cpp
    
    typedef std::list<InstancePtr>::iterator instance_iter; instance_iter
    begin(); instance_iter end();

The method begin() returns an iterator pointing to the beginning of the
container storing snippet Instances, while the method end() returns an
iterator pointing to the end of the container (past the last element).

.. code-block:: cpp
    
    InstancePtr pushBack(SnippetPtr); InstancePtr pushFront(SnippetPtr);

Multiple instances can be inserted at the same Point. We maintain the
instances in an ordered list. The pushBack method is to push the
specified Snippet to the end of the list, while the pushFront method is
to push to the front of the list.

Both methods return the Instance that uniquely identifies the inserted
snippet.

.. code-block:: cpp
    
    bool remove(InstancePtr instance);

This method removes the given snippet *instance* from this Point.

.. code-block:: cpp
    
    void clear();

This method removes all snippet instances inserted to this Point.

.. code-block:: cpp
    
    size_t size();

Returns the number of snippet instances inserted at this Point.

.. code-block:: cpp
    
    Address addr() const;

Returns the address associated with this point, if it has one;
otherwise, it returns 0.

.. code-block:: cpp
    
    Type type() const;

Returns the Point type of this point.

.. code-block:: cpp
    
    bool empty() const;

Indicates whether the container of instances at this Point is empty or
not.

.. code-block:: cpp
    
    PatchFunction* getCallee();

Returns the function that is invoked at this Point, which should have
Point::Type of Point::PreCall or Point::PostCall. It there is not a
function invoked at this point, it returns NULL.

.. code-block:: cpp
    
    const PatchObject* obj() const;

Returns the PatchObject where the Point resides.

.. code-block:: cpp
    
    const InstructionAPI::Instruction::Ptr insn() const;

Returns the Instruction where the Point resides.

.. code-block:: cpp
    
    PatchFunction* func() const;

Returns the function where the Point resides.

.. code-block:: cpp
    
    PatchBlock* block() const;

Returns the PatchBlock where the Point resides.

.. code-block:: cpp
    
    PatchEdge* edge() const;

Returns the Edge where the Point resides.

.. code-block:: cpp
    
    PatchCallback *cb() const;

Returns the PatchCallback object that is associated with this Point.

.. code-block:: cpp
    
    static bool TestType(Point::Type types, Point::Type type);

This static method tests whether a set of *types* contains a specific
*type*.

.. code-block:: cpp
    
    static void AddType(Point::Type& types, Point::Type type);

This static method adds a specific *type* to a set of *types*.

.. code-block:: cpp
    
    static void RemoveType(Point::Type& types, Point::Type trg);

This static method removes a specific *type* from a set of *types*.

.. _sec-3.1.3:

Instance
========

**Declared in**: Point.h

The Instance class is a representation of a particular snippet inserted
at a particular point. If a Snippet is inserted to N points or to the
same point for N times (N :math:`>` 1), then there will be N Instances.

.. code-block:: cpp
    
    bool destroy();

This method destroys the snippet Instance itself.

.. code-block:: cpp
    
    Point* point() const;

Returns the Point where the Instance is inserted.

.. code-block:: cpp
    
    SnippetPtr snippet() const;

Returns the Snippet. Please note that, the same Snippet may have
multiple instances inserted at different Points or the same Point.


Callback Interface
******************

.. _sec-3.2.7:

PatchCallback
=============

**Declared in**: PatchCallback.h

The PatchAPI CFG layer may change at runtime due to program events
(e.g., a program loading additional code or overwriting its own code
with new code). The ``PatchCallback`` interface allows users to specify
callbacks they wish to occur whenever the PatchAPI CFG changes.

.. code-block:: cpp
    
    virtual void destroy_cb(PatchBlock *); virtual void
    destroy_cb(PatchEdge *); virtual void destroy_cb(PatchFunction *);
    virtual void destroy_cb(PatchObject *);

Programmers implement the above virtual methods to handle the event of
destroying a PatchBlock, a PatchEdge, a PatchFunction, or a PatchObject
respectively. All the above methods will be called before corresponding
object destructors are called.

.. code-block:: cpp
    
    virtual void create_cb(PatchBlock *); virtual void create_cb(PatchEdge
    *); virtual void create_cb(PatchFunction *); virtual void
    create_cb(PatchObject *);

Programmers implement the above virtual methods to handle the event of
creating a PatchBlock, a PatchEdge, a PatchFunction, or a PatchObject
respectively. All the above methods will be called after the objects are
created.

.. code-block:: cpp
    
    virtual void split_block_cb(PatchBlock *first, PatchBlock *second);

Programmers implement the above virtual method to handle the event of
splitting a PatchBlock as a result of a new edge being discovered. The
above method will be called after the block is split.

.. code-block:: cpp
    
    virtual void remove_edge_cb(PatchBlock *, PatchEdge *, edge_type_t);
    virtual void add_edge_cb(PatchBlock *, PatchEdge *, edge_type_t);

Programmers implement the above virtual methods to handle the events of
removing or adding an PatchEdge respectively. The method remove_edge_cb
will be called before the event triggers, while the method add_edge_cb
will be called after the event triggers.

.. code-block:: cpp
    
    virtual void remove_block_cb(PatchFunction *, PatchBlock *); virtual
    void add_block_cb(PatchFunction *, PatchBlock *);

Programmers implement the above virtual methods to handle the events of
removing or adding a PatchBlock respectively. The method remove_block_cb
will be called before the event triggers, while the method add_block_cb
will be called after the event triggers.

.. code-block:: cpp
    
    virtual void create_cb(Point *pt); virtual void destroy_cb(Point *pt);

Programmers implement the create_cb method above, which will be called
after the Point *pt* is created. And, programmers implement the
destroy_cb method, which will be called before the point *pt* is
deleted.

.. code-block:: cpp
    
    virtual void change_cb(Point *pt, PatchBlock *first, PatchBlock *second);

Programmers implement this method, which is to be invoked after a block
is split. The provided Point belonged to the first block and is being
moved to the second.

.. _sec-modification-api:

Modification API Reference
**************************

This section describes the modification interface of PatchAPI. While
PatchAPI’s main goal is to allow users to insert new code into a
program, a secondary goal is to allow safe modification of the original
program code as well.

To modify the binary, a user interacts with the ``PatchModifier`` class
to manipulate a PatchAPI CFG. CFG modifications are then instantiated as
new code by the PatchAPI. For example, if PatchAPI is being used as part
of Dyninst, executing a ``finalizeInsertionSet`` will generate modified
code.

The three key benefits of the PatchAPI modification interface are
abstraction, safety, and interactivity. We use the CFG as a mechanism
for transforming binaries in a platform-independent way that requires no
instruction-level knowledge by the user. These transformations are
limited to ensure that the CFG can always be used to instantiate code,
and thus the user can avoid unintended side-effects of modification.
Finally, modifications to the CFG are represented in that CFG, allowing
users to iteratively combine multiple CFG transformations to achieve
their goals.

Since modification can modify the CFG, it may invalidate any analyses
the user has performed over the CFG. We suggest that users take
advantage of the callback interface described in Section
`4.3.1 <#sec-3.2.7>`__ to update any such analysis information.

The PatchAPI modification capabilities are currently in beta; if you
experience any problems or bugs, please contact ``bugs@dyninst.org``.

Many of these methods return a boolean type; true indicates a successful
operation, and false indicates a failure. For methods that return a
pointer, a ``NULL`` return value indicates a failure.

.. code-block:: cpp
    
    bool redirect(PatchEdge *edge, PatchBlock *target);

Redirects the edge specified by ``edge`` to a new target specified by
``target``. In the current implementation, the edge may not be indirect.

.. code-block:: cpp
    
    PatchBlock *split(PatchBlock *orig, Address addr, bool trust = false,
    Address newlast = (Address) -1);

Splits the block specified by ``orig``, creating a new block starting at
``addr``. If ``trust`` is true, we do not verify that ``addr`` is a
valid instruction address; this may be useful to reduce overhead. If
``newlast`` is not -1, we use it as the last instruction address of the
first block. All Points are updated to belong to the appropriate block.
The second block is returned.

.. code-block:: cpp
    
    bool remove(std::vector<PatchBlock *> &blocks, bool force = true)

Removes the blocks specified by ``blocks`` from the CFG. If ``force`` is
true, blocks are removed even if they have incoming edges; this may
leave the CFG in an unsafe state but may be useful for reducing
overhead.

.. code-block:: cpp
    
    bool remove(PatchFunction *func)

Removes ``func`` and all of its non-shared blocks from the CFG; any
shared blocks remain.

.. code-block:: cpp
    
    class InsertedCode typedef boost::shared_ptr<...> Ptr; PatchBlock
    *entry(); const std::vector<PatchEdge *> &exits(); const
    std::set<PatchBlock *> &blocks();

    InsertedCode::Ptr insert(PatchObject *obj, SnippetPtr snip, Point
    *point); InsertedCode::Ptr insert(PatchObject *obj, void *start,
    unsigned size);

Methods for inserting new code into a CFG. The ``InsertedCode``
structure represents a CFG subgraph generated by inserting new code; the
graph has a single entry point and multiple exits, represented by edges
to the sink node. The first ``insert`` call takes a PatchAPI Snippet
structure and a Point that is used to generate that Snippet; the point
is only passed through to the snippet code generator and thus may be
``NULL`` if the snippet does not use Point information. The second
``insert`` call takes a raw code buffer.

.. _sec-plugin-api:

Plugin API Reference
********************

This section describes the various plugin interfaces for extending
PatchAPI. We expect that most users should not have to ever explicitly
use an interface from this section; instead, they will use plugins
previously implemented by PatchAPI developers.

As with the public interface, all objects and methods in this section
are in the “Dyninst::PatchAPI” namespace.

.. _sec-3.2.1:

AddrSpace
=========

**Declared in**: AddrSpace.h

The AddrSpace class represents the address space of a **Mutatee**, where
it contains a collection of **PatchObjects** that represent shared
libraries or a binary executable. In addition, programmers implement
some memory management interfaces in the AddrSpace class to determine
the type of the code patching - 1st party, 3rd party, or binary
rewriting.

.. code-block:: cpp
    
    virtual bool write(PatchObject* obj, Address to, Address from, size_t size);

This method copies *size*-byte data stored at the address *from* on the
**Mutator** side to the address *to* on the **Mutatee** side. The
parameter *to* is the relative offset for the PatchObject *obj*, if the
instrumentation is for binary rewriting; otherwise *to* is an absolute
address.

If the write operation succeeds, this method returns true; otherwise,
false.

.. code-block:: cpp
    
    virtual Address malloc(PatchObject* obj, size_t size, Address near);

This method allocates a buffer of *size* bytes on the **Mutatee** side.
The address *near* is a relative address in the object *obj*, if the
instrumentation is for binary rewriting; otherwise, *near* is an
absolute address, where this method tries to allocate a buffer near the
address *near*.

If this method succeeds, it returns a non-zero address; otherwise, it
returns 0.

.. code-block:: cpp
    
    virtual Address realloc(PatchObject* obj, Address orig, size_t size);

This method reallocates a buffer of *size* bytes on the **Mutatee**
side. The original buffer is at the address *orig*. This method tries to
reallocate the buffer near the address *orig*, where *orig* is a
relative address in the PatchObject *obj* if the instrumentation is for
binary rewriting; otherwise, *orig* is an absolute address.

If this method succeeds, it returns a non-zero address; otherwise, it
returns 0.

.. code-block:: cpp
    
    virtual bool free(PatchObject* obj, Address orig);

This method deallocates a buffer on the **Mutatee** side at the address
*orig*. If the instrumentation is for binary rewriting, then the
parameter *orig* is a relative address in the object *obj*; otherwise,
*orig* is an absolute address.

If this method succeeds, it returns true; otherwise, it returns false.

.. code-block:: cpp
    
    virtual bool loadObject(PatchObject* obj);

This method loads a PatchObject into the address space. If this method
succeeds, it returns true; otherwise, it returns false.

.. code-block:: cpp
    
    typedef std::map<const ParseAPI::CodeObject*, PatchObject*> AddrSpace::ObjMap;
    ObjMap& objMap();

Returns a set of mappings from ParseAPI::CodeObjects to PatchObjects,
where PatchObjects in all mappings represent all binary objects (either
executable or libraries loaded) in this address space.

.. code-block:: cpp
    
    PatchObject* executable();

Returns the PatchObject of the executable of the **Mutatee**.

.. code-block:: cpp
    
    PatchMgrPtr mgr();

Returns the PatchMgr’s pointer, where the PatchMgr contains this address
space.

.. _sec-3.2.2:

Snippet
=======

**Declared in**: Snippet.h

The Snippet class allows programmers to customize their own snippet
representation and the corresponding mini-compiler to translate the
representation into the binary code.

.. code-block:: cpp
    
    static Ptr create(Snippet* a);

Creates an object of the Snippet.

.. code-block:: cpp
    
    virtual bool generate(Point *pt, Buffer &buf);

Users should implement this virtual function for generating binary code
for the snippet.

Returns false if code generation failed catastrophically. Point *pt* is
an in-param that identifies where the snippet is being generated. Buffer
*buf* is an out-param that holds the generated code.

.. _sec-3.2.3:

Command
=======

**Declared in**: Command.h

The Command class represents an instrumentation request (e.g., snippet
insertion or removal), or an internal logical step in the code patching
(e.g., install instrumentation).

.. code-block:: cpp
    
    virtual bool run() = 0;

Executes the normal operation of this Command.

It returns true on success; otherwise, it returns false.

.. code-block:: cpp
    
    virtual bool undo() = 0;

Undoes the operation of this Command.

.. code-block:: cpp
    
    virtual bool commit();

Implements the transactional semantics: all succeed, or all fail.
Basically, it performs such logic:

.. code-block:: cpp
   
   if (run()) {
     return true;
   } else {
     undo();
     return false;
   }

.. _sec-3.2.4:

BatchCommand
============

**Declared in**: Command.h

The BatchCommand class inherits from the Command class. It is actually a
container of a list of Commands that will be executed in a transaction:
all Commands will succeed, or all will fail.

.. code-block:: cpp
    
    typedef std::list<CommandPtr> CommandList;
    CommandList to_do_; CommandList done_;

This class has two protected members *to_do\_* and *done\_*, where
*to_do\_* is a list of Commands to execute, and *done\_* is a list of
Commands that are executed.

.. code-block:: cpp
    
    virtual bool run(); virtual bool undo();

The method run() of BatchCommand invokes the run() method of each
Command in *to_do\_* in order, and puts the finished Commands in
*done\_*. The method undo() of BatchCommand invokes the undo() method of
each Command in *done \_* in order.

.. code-block:: cpp
    
    void add(CommandPtr command);

This method adds a Command into *to_do\_*.

.. code-block:: cpp
    
    void remove(CommandList::iterator iter);

This method removes a Command from *to_do\_*.

.. _sec-3.2.5:

Instrumenter
============

**Declared in**: Command.h

The Instrumenter class inherits BatchCommand to encapsulate the core
code patching logic, which includes binary code generation. Instrumenter
would contain several logical steps that are individual Commands.

    ``CommandList user_commands_;``

This class has a protected data member *user_commands\_* that contains
all Commands issued by users, e.g., snippet insertion. This is to
facilitate the implementation of the instrumentation engine.

.. code-block:: cpp
    
    static InstrumenterPtr create(AddrSpacePtr as);

Returns an instance of Instrumenter, and it takes input the address
space *as* that is going to be instrumented.

.. code-block:: cpp
    
    virtual bool replaceFunction(PatchFunction* oldfunc, PatchFunction* newfunc);

Replaces a function *oldfunc* with a new function *newfunc*.

It returns true on success; otherwise, it returns false.

.. code-block:: cpp
    
    virtual bool revertReplacedFunction(PatchFunction* oldfunc);

Undoes the function replacement for *oldfunc*.

It returns true on success; otherwise, it returns false.

.. code-block:: cpp
    
    typedef std::map<PatchFunction*, PatchFunction*> FuncModMap;

The type FuncModMap contains mappings from an PatchFunction to another
PatchFunction.

.. code-block:: cpp
    
    virtual FuncModMap& funcRepMap();

Returns the FuncModMap that contains a set of mappings from an old
function to a new function, where the old function is replaced by the
new function.

.. code-block:: cpp
    
    virtual bool wrapFunction(PatchFunction* oldfunc, PatchFunction* newfunc, string name);

Replaces all calls to *oldfunc* with calls to wrapper *newfunc* (similar
to function replacement). However, we create a copy of original using
the *name* that can be used to call the original. The wrapper code would
look like follows:

.. code-block:: cpp

   void *malloc_wrapper(int size) {
     // do stuff
     void *ret = malloc_clone(size);
     // do more stuff
     return ret;
   }

This interface requires the user to give us a name (as represented by
clone) for the original function. This matches current techniques and
allows users to use indirect calls (function pointers).

.. code-block:: cpp
    
    virtual bool revertWrappedFunction(PatchFunction* oldfunc);

Undoes the function wrapping for *oldfunc*.

It returns true on success; otherwise, it returns false.

.. code-block:: cpp
    
    virtual FuncModMap& funcWrapMap();

The type FuncModMap contains mappings from the original PatchFunction to
the wrapper PatchFunction.

.. code-block:: cpp
    
    bool modifyCall(PatchBlock *callBlock, PatchFunction *newCallee, PatchFunction *context = NULL);

Replaces the function that is invoked in the basic block *callBlock*
with the function *newCallee*. There may be multiple functions
containing the same *callBlock*, so the *context* parameter specifies in
which function the *callBlock* should be modified. If *context* is NULL,
then the *callBlock* would be modified in all PatchFunctions that
contain it. If the *newCallee* is NULL, then the *callBlock* is removed.

It returns true on success; otherwise, it returns false.

.. code-block:: cpp
    
    bool revertModifiedCall(PatchBlock *callBlock, PatchFunction *context = NULL);

Undoes the function call modification for *oldfunc*. There may be
multiple functions containing the same *callBlock*, so the *context*
parameter specifies in which function the *callBlock* should be
modified. If *context* is NULL, then the *callBlock* would be modified
in all PatchFunctions that contain it.

It returns true on success; otherwise, it returns false.

.. code-block:: cpp
    
    bool removeCall(PatchBlock *callBlock, PatchFunction *context = NULL);

Removes the *callBlock*, where a function is invoked. There may be
multiple functions containing the same *callBlock*, so the *context*
parameter specifies in which function the *callBlock* should be
modified. If *context* is NULL, then the *callBlock* would be modified
in all PatchFunctions that contain it.

It returns true on success; otherwise, it returns false.

.. code-block:: cpp
    
    typedef map<PatchBlock*, // B : A call block map<PatchFunction*, // F_c:
    Function context PatchFunction*> // F : The function to be replaced >
    CallModMap;

The type CallModMap maps from B -> F\ :math:`_c` -> F, where B
identifies a call block, and F\ :math:`_c` identifies an (optional)
function context for the replacement. If F\ :math:`_c` is not specified,
we use NULL. F specifies the replacement callee; if we want to remove
the call entirely, we use NULL.

.. code-block:: cpp
    
    CallModMap& callModMap();

Returns the CallModMap for function call replacement / removal.

.. code-block:: cpp
    
    AddrSpacePtr as() const;

Returns the address space associated with this Instrumenter.

.. _sec-3.2.6:

Patcher
=======

**Declared in**: Command.h

The class Patcher inherits from the class BatchCommand. It accepts
instrumentation requests from users, where these instrumentation
requests are Commands (e.g., snippet insertion). Furthermore, Patcher
implicitly adds an instance of Instrumenter to the end of the Command
list to generate binary code and install the instrumentation.

.. code-block:: cpp
    
    Patcher(PatchMgrPtr mgr)

The constructor of Patcher takes input the relevant PatchMgr *mgr*.

.. code-block:: cpp
    
    virtual bool run();

Performs the same logic as BatchCommand::run(), except that this
function implicitly adds an internal Command – Instrumenter, which is
executed after all other Commands in the *to_do\_*.

CFGMaker
========

**Declared in**: CFGMaker.h

The CFGMaker class is a factory class that constructs the above CFG
structures (PatchFunction, PatchBlock, and PatchEdge). The methods in
this class are used by PatchObject. Programmers can extend
PatchFunction, PatchBlock and PatchEdge by annotating their own data,
and then use this class to instantiate these CFG structures.

.. code-block:: cpp
    
    virtual PatchFunction* makeFunction(ParseAPI::Function* func,
    PatchObject* obj); virtual PatchFunction* copyFunction(PatchFunction*
    func, PatchObject* obj);

    virtual PatchBlock* makeBlock(ParseAPI::Block* blk, PatchObject*
    obj); virtual PatchBlock* copyBlock(PatchBlock* blk, PatchObject*
    obj);

    virtual PatchEdge* makeEdge(ParseAPI::Edge* edge, PatchBlock* src,
    PatchBlock* trg, PatchObject* obj); virtual PatchEdge*
    copyEdge(PatchEdge* edge, PatchObject* obj);

Programmers implement the above virtual methods to instantiate a CFG
structure (either a PatchFunction, a PatchBlock, or a PatchEdge) or to
copy (e.g., when forking a new process).

PointMaker
==========

**Declared in**: Point.h

The PointMaker class is a factory class that constructs instances of the
Point class. The methods of the PointMaker class are invoked by
PatchMgr’s findPoint methods. Programmers can extend the Point class,
and then implement a set of virtual methods in this class to instantiate
the subclasses of Point.

.. code-block:: cpp
    
    PointMaker(PatchMgrPtr mgr);

The constructor takes input the relevant PatchMgr *mgr*.

.. code-block:: cpp
    
    virtual Point *mkFuncPoint(Point::Type t, PatchMgrPtr m, PatchFunction
    *f); virtual Point *mkFuncSitePoint(Point::Type t, PatchMgrPtr m,
    PatchFunction *f, PatchBlock *b); virtual Point
    *mkBlockPoint(Point::Type t, PatchMgrPtr m, PatchBlock *b,
    PatchFunction *context); virtual Point *mkInsnPoint(Point::Type t,
    PatchMgrPtr m, PatchBlock *, Address a,
    InstructionAPI::Instruction::Ptr i, PatchFunction *context); virtual
    Point *mkEdgePoint(Point::Type t, PatchMgrPtr m, PatchEdge *e,
    PatchFunction *context);

Programmers implement the above virtual methods to instantiate the
subclasses of Point.

.. _sec-3.3:

Default Plugin
**************

.. _sec-3.3.1:

PushFrontCommand and PushBackCommand
====================================

**Declared in**: Command.h

The class PushFrontCommand and the class PushBackCommand inherit from
the Command class. They are to insert a snippet to a point. A point
maintains a list of snippet instances. PushFrontCommand would add the
new snippet instance to the front of the list, while PushBackCommand
would add to the end of the list.

.. code-block:: cpp
    
    static Ptr create(Point* pt, SnippetPtr snip);

This static method creates an object of PushFrontCommand or
PushBackCommand.

.. code-block:: cpp
    
    InstancePtr instance();

Returns a snippet instance that is inserted at the point.

.. _sec-3.3.2:

RemoveSnippetCommand
====================

**Declared in**: Command.h

The class RemoveSnippetCommand inherits from the Command class. It is to
delete a snippet Instance.

.. code-block:: cpp
    
    static Ptr create(InstancePtr instance);

This static function creates an instance of RemoveSnippetCommand.

.. _sec-3.3.3:

RemoveCallCommand
=================

**Declared in**: Command.h

The class RemoveCallCommand inherits from the class Command. It is to
remove a function call.

.. code-block:: cpp
    
    static Ptr create(PatchMgrPtr mgr, PatchBlock* call_block,
    PatchFunction* context = NULL);

This static method takes input the relevant PatchMgr *mgr*, the
*call_block* that contains the function call to be removed, and the
PatchFunction *context*. There may be multiple PatchFunctions containing
the same *call_block*. If the *context* is NULL, then the *call_block*
would be deleted from all PatchFunctions that contains it; otherwise,
the *call_block* would be deleted only from the PatchFuncton *context*.

.. _sec-3.3.4:

ReplaceCallCommand
==================

**Declared in**: Command.h

The class ReplaceCallCommand inherits from the class Command. It is to
replace a function call with another function.

.. code-block:: cpp
    
    static Ptr create(PatchMgrPtr mgr, PatchBlock* call_block,
    PatchFunction* new_callee, PatchFunction* context);

This Command replaces the *call_block* with the new PatchFunction
*new_callee*. There may be multiple functions containing the same
*call_block*, so the *context* parameter specifies in which function the
*call_block* should be replaced. If *context* is NULL, then the
*call_block* would be replaced in all PatchFunctions that contains it.

.. _sec-3.3.5:

ReplaceFuncCommand
==================

**Declared in**: Command.h

The class ReplaceFuncCommand inherits from the class Command. It is to
replace an old function with the new one.

.. code-block:: cpp
    
    static Ptr create(PatchMgrPtr mgr, PatchFunction* old_func,
    PatchFunction* new_func);

This Command replaces the old PatchFunction *old_func* with the new
PatchFunction *new_func*.
