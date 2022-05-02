.. _`sec:dataflow-api`:

===========
DataflowAPI
===========

``Assignment``
--------------

**Defined in:** ``Absloc.h``

.. cpp:namespace:: Assignment

.. cpp:class:: Assignment
   
   
   An assignment represents data dependencies between an output abstract
   region that is modified by this instruction and several input abstract
   regions that are used by this instruction. An instruction may modify
   several abstract regions, so an instruction can correspond to multiple
   assignments.
   
   .. cpp:type:: boost::shared_ptr<Assignment> Ptr;
      
      Shared pointer for Assignment class.
      
   .. cpp:function:: const std::vector<AbsRegion> &inputs() const
      
   .. cpp:function:: std::vector<AbsRegion> &inputs()
      
      Return the input abstract regions.
      
   .. cpp:function:: const AbsRegion &out() const
      
   .. cpp:function:: AbsRegion &out()
      
      Return the output abstract region.
      
   .. cpp:function:: InstructionAPI::Instruction::Ptr insn() const
      
      Return the instruction that contains this assignment.
      
   .. cpp:function:: Address addr() const
      
      Return the address of this assignment.
      
   .. cpp:function:: ParseAPI::Function *func() const
      
      Return the function that contains this assignment.
      
   .. cpp:function:: ParseAPI::Block *block() const
      
      Return the block that contains this assignment.
      
   .. cpp:function:: const std::string format() const
      
      Return the string representation of this assignment.
      
``AssignmentConverter``
-----------------------

**Defined in:** ``AbslocInterface.h``

.. cpp:namespace:: AssignmentConverter

.. cpp:class:: AssignmentConverter
   
   
   This class should be used to convert instructions to assignments.
   
   .. cpp:function:: AssignmentConverter::AssignmentConverter(bool cache, bool stack = true)
      
      Construct an AssignmentConverter. When ``cache`` is ``true``, this
      object will cache the conversion results for converted instructions.
      When ``stack`` is ``true``, stack analysis is used to distinguish stack
      variables at different offset. When ``stack`` is ``false``, the stack is
      treated as a single memory region.
      
   .. cpp:function:: void convert( \
            InstructionAPI::Instruction::Ptr insn, \
            const Address &addr, \
            ParseAPI::Function *func, \
            ParseAPI::Block *blk, \
            std::vector<Assignment::Ptr> &assign)
      
      Convert instruction ``insn`` to assignments and return these assignments
      in ``assign``. The user also needs to provide the context of ``insn``,
      including its address ``addr``, function ``func``, and block ``blk``.
      
``Absloc``
----------

**Defined in:** ``Absloc.h``

.. cpp:namespace:: Absloc

.. cpp:class:: Absloc
   
   
   Absloc represents an abstract location. Abstract locations can
   have the following types
   
   .. container:: center
   
      ======== =================================================
      Type     Meaning
      ======== =================================================
      Register The abstract location represents a register
      Stack    The abstract location represents a stack variable
      Heap     The abstract location represents a heap variable
      Unknown  The default type of abstract location
      ======== =================================================
   
   .. cpp:function:: static Absloc makePC(Dyninst::Architecture arch)
   .. cpp:function:: static Absloc makeSP(Dyninst::Architecture arch)
   .. cpp:function:: static Absloc makeFP(Dyninst::Architecture arch)
      
      Shortcut interfaces for creating abstract locations representing PC, SP,
      and FP
      
   .. cpp:function:: bool isPC() const
   .. cpp:function:: bool isSP() const
   .. cpp:function:: bool isFP() const
      
      Check whether this abstract location represents a PC, SP, or FP.
      
   .. cpp:function:: Absloc::Absloc()
      
      Create an Unknown type abstract location.
      
      
   .. cpp:function:: Absloc::Absloc(MachRegister reg)
      
      Create a Register type abstract location, representing register ``reg``.
      
   .. cpp:function:: Absloc::Absloc(Address addr)
      
      Create a Heap type abstract location, representing a heap variable at
      address ``addr``.
      
   .. cpp:function:: Absloc::Absloc(int o, int r, ParseAPI::Function *f)
      
      Create a Stack type abstract location, representing a stack variable in
      the frame of function ``f``, within abstract region ``r``, and at offset
      ``o`` within the frame.

   .. cpp:function:: std::string format() const
      
      Return the string representation of this abstract location.
      
      
   .. cpp:function:: const Type& type() const
      
      Return the type of this abstract location.
      
   .. cpp:function:: bool isValid() const
      
      Check whether this abstract location is valid or not. Return ``true``
      when the type is not Unknown.
      
   .. cpp:function:: const MachRegister &reg() const
      
      Return the register represented by this abstract location. This method
      should only be called when this abstract location truly represents a
      register.
      
      
   .. cpp:function:: int off() const
      
      Return the offset of the stack variable represented by this abstract
      location. This method should only be called when this abstract location
      truly represents a stack variable.
      
   .. cpp:function:: int region() const
      
      Return the region of the stack variable represented by this abstract
      location. This method should only be called when this abstract location
      truly represents a stack variable.
      
   .. cpp:function:: ParseAPI::Function *func() const
      
      Return the function of the stack variable represented by this abstract
      location. This method should only be called when this abstract location
      truly represents a stack variable.
      
   .. cpp:function:: Address addr() const
      
      Return the address of the heap variable represented by this abstract
      location. This method should only be called when this abstract location
      truly represents a heap variable.
      
   .. cpp:function:: bool operator<(const Absloc &rhs) const
   .. cpp:function:: bool operator==(const Absloc &rhs) const
   .. cpp:function:: bool operator!=(const Absloc &rhs) const
      
      Comparison operators
      
.. _`sec:absregion`:
      
``AbsRegion``
-------------

**Defined in:** ``Absloc.h``

.. cpp:namespace:: AbsRegion

.. cpp:class:: AbsRegion
   
   
   AbsRegion represents a set of abstract locations of the same type.
   
   .. cpp:function:: AbsRegion::AbsRegion()
      
      Create a default abstract region.
      
   .. cpp:function:: AbsRegion::AbsRegion(Absloc::Type t)
      
      Create an abstract region representing all abstract locations with type ``t``.

   .. cpp:function:: AbsRegion::AbsRegion(Absloc a)
      
      Create an abstract region representing a single abstract location ``a``.
      
   .. cpp:function:: bool contains(const Absloc::Type t) const
   .. cpp:function:: bool contains(const Absloc &abs) const
   .. cpp:function:: bool contains(const AbsRegion &rhs) const
      
      Return ``true`` if this abstract region contains abstract locations of
      type ``t``, contains abstract location ``abs``, or contains abstract
      region ``rhs``.
      
   .. cpp:function:: bool containsOfType(Absloc::Type t) const
      
      Return ``true`` if this abstract region contains abstract locations in
      type ``t``.
      
   .. cpp:function:: bool operator==(const AbsRegion &rhs) const
   .. cpp:function:: bool operator!=(const AbsRegion &rhs) const
   .. cpp:function:: bool operator<(const AbsRegion &rhs) const
      
      Comparison operators
      
   .. cpp:function:: const std::string format() const
      
      Return the string representation of the abstract region.
      
   .. cpp:function:: Absloc absloc() const
      
      Return the abstract location in this abstract region.
      
   .. cpp:function:: Absloc::Type type() const
      
      Return the type of this abstract region.
      
   .. cpp:function:: AST::Ptr generator() const
      
      If this abstract region represents memory locations, this method returns
      address calculation of the memory access.
      
   .. cpp:function:: bool isImprecise() const
      
      Return ``true`` if this abstract region represents more than one
      abstract locations.
      
``AbsRegionConverter``
----------------------

**Defined in:** ``AbslocInterface.h``

.. cpp:namespace:: AbsRegionConverter

.. cpp:class:: AbsRegionConverter
   
   
   AbsRegionConverter converts instructions to abstract regions.
   
   AbsRegionConverter(bool cache, bool stack = true)
   
   Create an AbsRegionConverter. When ``cache`` is ``true``, this object
   will cache the conversion results for converted instructions. When
   ``stack`` is ``true``, stack analysis is used to distinguish stack
   variables at different offsets. When ``stack`` is ``false``, the stack
   is treated as a single memory region.

   .. cpp:function:: void convertAll( \
            InstructionAPI::Expression::Ptr expr, \
            Address addr, \
            ParseAPI::Function *func, \
            ParseAPI::Block *block, \
            std::vector<AbsRegion> &regions)
      
      Create all abstract regions used in ``expr`` and return them in
      ``regions``. All registers appear in ``expr`` will have a separate
      abstract region. If the expression represents a memory access, we will
      also create a heap or stack abstract region depending on where it
      accesses. ``addr``, ``func``, and ``blocks`` specify the contexts of the
      expression. If PC appears in this expression, we assume the expression
      is at address ``addr`` and replace PC with a constant value ``addr``.

   .. cpp:function:: void convertAll( \
            InstructionAPI::Instruction::Ptr insn, \
            Address addr, \
            ParseAPI::Function *func, \
            ParseAPI::Block *block, \
            std::vector<AbsRegion> &used, \
            std::vector<AbsRegion> &defined)
      
      Create abstract regions appearing in instruction ``insn``. Input
      abstract regions of this instructions are returned in ``used`` and
      output abstract regions are returned in ``defined``. If the expression
      represents a memory access, we will also create a heap or stack abstract
      region depending on where it accesses. ``addr``, ``func``, and
      ``blocks`` specify the contexts of the expression. If PC appears in this
      expression, we assume the expression is at address ``addr`` and replace
      PC with a constant value ``addr``.

   .. cpp:function:: AbsRegion convert(InstructionAPI::RegisterAST::Ptr reg)
      
      Create an abstract region representing the register ``reg``.
      
   .. cpp:function:: AbsRegion convert( \
            InstructionAPI::Expression::Ptr expr, \
            Address addr, \
            ParseAPI::Function *func, \
            ParseAPI::Block *block)
      
      Create and return the single abstract region represented by ``expr``.
      
``Graph``
---------

**Defined in:** ``Graph.h``

.. cpp:namespace:: Graph

.. cpp:class:: Graph
   
   
   We provide a generic graph interface, which allows users to add, delete,
   and iterate nodes and edges in a graph. Our slicing algorithms are
   implemented upon this graph interface, so users can inherit the defined
   classes for customization.
   
   .. cpp:type:: boost::shared_ptr<Graph> Ptr;
      
      Shared pointer for Graph
      
   .. cpp:function:: virtual void entryNodes(NodeIterator &begin, NodeIterator &end)
      
      The entry nodes (nodes without any incoming edges) of the graph.
      
   .. cpp:function:: virtual void exitNodes(NodeIterator &begin, NodeIterator &end)
      
      The exit nodes (nodes without any outgoing edges) of the graph.
      
   .. cpp:function:: virtual void allNodes(NodeIterator &begin, NodeIterator &end)
      
      Iterate all nodes in the graph.
      
   .. cpp:function:: bool printDOT(const std::string& fileName)
      
      Output the graph in dot format.
      
   .. cpp:function:: static Graph::Ptr createGraph()
      
      Return an empty graph.
      
   .. cpp:function:: void insertPair(NodePtr source, NodePtr target, EdgePtr edge = EdgePtr())
      
      Insert a pair of nodes into the graph and create a new edge ``edge``
      from ``source`` to ``target``.
      
   .. cpp:function:: virtual void insertEntryNode(NodePtr entry)
   .. cpp:function:: virtual void insertExitNode(NodePtr exit)
      
      Insert a node as an entry/exit node
      
   .. cpp:function:: virtual void markAsEntryNode(NodePtr entry)
   .. cpp:function:: virtual void markAsExitNode(NodePtr exit)
      
      Mark a node that has been added to this graph as an entry/exit node.
      
   .. cpp:function:: void deleteNode(NodePtr node)
   .. cpp:function:: void addNode(NodePtr node)
      
      Delete / Add a node.
      
   .. cpp:function:: bool isEntryNode(NodePtr node)
   .. cpp:function:: bool isExitNode(NodePtr node)
      
      Check whether a node is an entry / exit node
      
   .. cpp:function:: void clearEntryNodes()
   .. cpp:function:: void clearExitNodes()
      
      Clear the marking of entry / exit nodes. Note that the nodes are not
      deleted from the graph.
      
   .. cpp:function:: unsigned size() const
      
      Return the number of nodes in the graph.
      
``Node``
--------

**Defined in:** ``Node.h``

.. cpp:namespace:: Node

.. cpp:class:: Node
   
   
   .. cpp:type:: boost::shared_ptr<Node> Ptr;
      
      Shared pointer for Node
      
   .. cpp:function:: void ins(EdgeIterator &begin, EdgeIterator &end)
   .. cpp:function:: void outs(EdgeIterator &begin, EdgeIterator &end)
      
      Iterate over incoming/outgoing edges of this node.
      
   .. cpp:function:: void ins(NodeIterator &begin, NodeIterator &end)
   .. cpp:function:: void outs(NodeIterator &begin, NodeIterator &end)
      
      Iterate over adjacent nodes connected with incoming/outgoing edges of
      this node.
      
   .. cpp:function:: bool hasInEdges()
   .. cpp:function:: bool hasOutEdges()
      
      Return ``true`` if this node has incoming/outgoing edges.
      
   .. cpp:function:: void deleteInEdge(EdgeIterator e)
   .. cpp:function:: void deleteOutEdge(EdgeIterator e)
      
      Delete an incoming/outgoing edge.
      
   .. cpp:function:: virtual Address addr() const
      
      Return the address of this node.
      
   .. cpp:function:: virtual std::string format() const = 0;
      
      Return the string representation.
      
.. cpp:class:: NodeIterator
   
   Iterator for nodes. Common iterator operations including ``++``, ``–``,
   and dereferencing are supported.
   
``Edge``
--------

**Defined in:** ``Edge.h``

.. cpp:namespace:: Edge

.. cpp:class:: Edge
   
   
   .. cpp:type:: boost::shared_ptr<Edge> Edge::Ptr;
      
      Shared pointer for ``Edge``.
      
   .. cpp:function:: static Edge::Ptr Edge::createEdge(const Node::Ptr source, const Node::Ptr target)
      
      Create a new directed edge from ``source`` to ``target``.
      
   .. cpp:function:: Node::Ptr Edge::source() const
   .. cpp:function:: Node::Ptr Edge::target() const
      
      Return the source / target node.
      
   .. cpp:function:: void Edge::setSource(Node::Ptr source)
   .. cpp:function:: void Edge::setTarget(Node::Ptr target)
      
      Set the source / target node.
      
.. cpp:namespace:: EdgeIterator

.. cpp:class:: EdgeIterator
   
   Iterator for edges. Common iterator operations including ``++``, ``–``,
   and dereferencing are supported.
   
   .. _`sec:slicer`:
   
``Slicer``
----------

**Defined in:** ``slicing.h``

.. cpp:namespace:: Slicer

.. cpp:class:: Slicer
   
   
   Slicer is the main interface for performing forward and backward
   slicing. The slicing algorithm starts with a user provided Assignment
   and generates a graph as the slicing results. The nodes in the generated
   Graph are individual assignments that affect the starting assignment
   (backward slicing) or are affected by the starting assignment (forward
   slicing). The edges in the graph are directed and represent either data
   flow dependencies or control flow dependencies.
   
   We provide call back functions and allow users to control when to stop
   slicing. In particular, class ``Slicer::Predicates`` contains a
   collection of call back functions that can control the specific
   behaviors of the slicer. Users can inherit from the Predicates class to
   provide customized stopping criteria for the slicer.
   
   .. cpp:function:: Slicer::Slicer( \
            AssignmentPtr a, \
            ParseAPI::Block *block, \
            ParseAPI::Function *func, \
            bool cache = true, \
            bool stackAnalysis = true)
      
      Construct a slicer, which can then be used to perform forward or
      backward slicing starting at the assignment ``a``. ``block`` and
      ``func`` represent the context of assignment ``a``. ``cache`` specifies
      whether the slicer will cache the results of conversions from
      instructions to assignments. ``stackAnalysis`` specifies whether the
      slicer will invoke stack analysis to distinguish stack variables.

   .. cpp:function:: GraphPtr forwardSlice(Predicates &predicates)
   .. cpp:function:: GraphPtr backwardSlice(Predicates &predicates)
      
      Perform forward or backward slicing and use ``predicates`` to control
      the stopping criteria and return the slicing results as a graph
      
      A slice is represented as a Graph. The nodes and edges are defined as
      below:
      
.. cpp:namespace:: SliceNode

.. cpp:class:: SliceNode : public Node
   
   The default node data type in a slice graph.
   
   .. cpp:type:: boost::shared_ptr<SliceNode> Ptr
   .. cpp:function:: static SliceNode::Ptr SliceNode::create( \
            AssignmentPtr ptr, \
            ParseAPI::Block *block, \
            ParseAPI::Function *func)
      
      Create a slice node, which represents assignment ``ptr`` in basic block
      ``block`` and function ``func``.

SliceNode has the following methods to retrieve information
associated the node:

.. list-table:: Class SlideNode Methods
   :widths: 30  35 35
   :header-rows: 1

   * - Method name
     - Return type
     - Method description
   * - block
     - ParseAPI::Block*
     - Basic block of this SliceNode.
   * - func
     - ParseAPI::Function*
     - Function of this SliceNode. 
   * - addr
     - Address
     - Address of this SliceNode.
   * - assign
     - Assignment::Ptr
     - Assignment of this SliceNode.
   * - format
     - std::string
     - String representation of this SliceNode. 

.. cpp:namespace:: SliceEdge

.. cpp:class:: SliceEdge : public Edge
   
   The default edge data type in a slice graph.
   
   .. cpp:type:: boost::shared_ptr<SliceEdge> Ptr
   .. cpp:function:: static SliceEdge::Ptr create(SliceNode::Ptr source, SliceNode::Ptr target, AbsRegion const&data)
      
      Create a slice edge from ``source`` to ``target`` and the edge presents
      a dependency about abstract region ``data``.
      
   .. cpp:function:: const AbsRegion &data() const
      
      Get the data annotated on this edge.
      
      .. _`sec:slicing`:
      
``Slicer::Predicates``
----------------------

**Defined in:** ``slicing.h``

.. cpp:namespace:: Slicer::Predicates

.. cpp:class:: Slicer::Predicates
   
   
   Predicates abstracts the stopping criteria of slicing. Users can
   inherit this class to control slicing in various situations, including
   whether or not to perform inter-procedural slicing, whether or not to
   search for control flow dependencies, and whether or not to stop slicing
   after discovering certain assignments. We provide a set of call back
   functions that allow users to dynamically control the behavior of the
   Slicer.
   
   .. cpp:function:: Predicates()
      
      Construct a default predicate, which will only search for
      intraprocedural data flow dependencies.
      
   .. cpp:function:: bool searchForControlFlowDep()
      
      Return ``true`` if this predicate will search for control flow
      dependencies. Otherwise, return ``false``.
      
   .. cpp:function:: void setSearchForControlFlowDep(bool cfd)
      
      Change whether or not to search for control flow dependencies according
      to ``cfd``.
      
   .. cpp:function:: virtual bool widenAtPoint(AssignmentPtr)
      
      The default behavior is to return ``false``.
      
   .. cpp:function:: virtual bool endAtPoint(AssignmentPtr)
      
      In backward slicing, after we find a match for an assignment, we pass it
      to this function. This function should return ``true`` if the user does
      not want to continue searching for this assignment. Otherwise, it should
      return ``false``. The default behavior of this function is to always
      return ``false``.
      
   .. cpp:type:: std::pair<ParseAPI::Function *, int> StackDepth_t
   .. cpp:type:: std::stack<StackDepth_t> CallStack_t
   .. cpp:function:: virtual bool followCall(ParseAPI::Function * callee, CallStack_t & cs, AbsRegion argument)
      
      This predicate function is called when the slicer reaches a direct call
      site. If it returns ``true``, the slicer will follow into the callee
      function ``callee``. This function also takes input ``cs``, which
      represents the call stack of the followed callee functions from the
      starting point of the slicing to this call site, and ``argument``, which
      represents the variable to slice with in the callee function. This
      function defaults to always returning ``false``. Note that as Dyninst
      currently does not try to resolve indirect calls, the slicer will NOT
      call this function at an indirect call site.
      
   .. cpp:function:: virtual std::vector<ParseAPI::Function *> followCallBackward( \
            ParseAPI::Block * caller, \
            CallStack_t & cs, \
            AbsRegion argument)
      
      This predicate function is called when the slicer reaches the entry of a
      function in the case of backward slicing or reaches a return instruction
      in the case of forward slicing. It returns a vector of caller functions
      that the user wants the slicer to continue to follow. This function
      takes input ``caller``, which represents the call block of the caller,
      ``cs``, which represents the caller functions that have been followed to
      this place, and ``argument``, which represents the variable to slice
      with in the caller function. This function defaults to always returning
      an empty vector.

   .. cpp:function:: virtual bool addPredecessor(AbsRegion reg)
      
      In backward slicing, after we match an assignment at a location, the
      matched AbsRegion ``reg`` is passed to this predicate function. This
      function should return ``true`` if the user wants to continue to search
      for dependencies for this AbsRegion. Otherwise, this function should
      return ``true``. The default behavior of this function is to always
      return ``true``.
      
   .. cpp:function:: virtual bool addNodeCallback(AssignmentPtr assign, std::set<ParseAPI::Edge*> &visited)
      
      In backward slicing, this function is called when the slicer adds a new
      node to the slice. The newly added assignment ``assign`` and the set of
      control flow edges ``visited`` that have been visited so far are passed
      to this function. This function should return ``true`` if the user wants
      to continue slicing. If this function returns ``false``, the Slicer will
      not continue to search along the path. The default behavior of this
      function is to always return ``true``.
      
      .. _`sec:stackanalysis`:
      
``StackAnalysis``
-----------------

The StackAnalysis interface is used to determine the possible stack

.. cpp:namespace:: StackAnalysis

.. cpp:class:: StackAnalysis
   
   heights of abstract locations at any instruction in a function. Due to
   there often being many paths through the CFG to reach a given
   instruction, abstract locations may have different stack heights
   depending on the path taken to reach that instruction. In other cases,
   StackAnalysis is unable to adequately determine what is contained in an
   abstract location. In both situations, StackAnalysis is conservative in
   its reported stack heights. The table below explains what the reported
   stack heights mean.
   
   +-----------------------+---------------------------------------------+
   | Reported stack height | Meaning                                     |
   +=======================+=============================================+
   | TOP                   | On all paths to this instruction, the       |
   |                       | specified abstract location contains a      |
   |                       | value that does not point to the stack.     |
   +-----------------------+---------------------------------------------+
   |                       |                                             |
   +-----------------------+---------------------------------------------+
   | *x* (some number)     | On at least one path to this instruction,   |
   |                       | the specified abstract location has a stack |
   |                       | height of *x*. On all other paths, the      |
   |                       | abstract location either has a stack height |
   |                       | of *x* or doesn’t point to the stack.       |
   +-----------------------+---------------------------------------------+
   |                       |                                             |
   +-----------------------+---------------------------------------------+
   | BOTTOM                | There are three possible meanings:          |
   |                       |                                             |
   |                       | #. On at least one path to this             |
   |                       | instruction, StackAnalysis was unable to    |
   |                       | determine whether or not the specified      |
   |                       | abstract location points to the stack.      |
   |                       |                                             |
   |                       | #. On at least one path to this             |
   |                       | instruction, StackAnalysis determined       |
   |                       | that the specified abstract location        |
   |                       | points to the stack but could not           |
   |                       | determine the exact stack height.           |
   |                       |                                             |
   |                       | #. On at least two paths to this            |
   |                       | instruction, the specified abstract         |
   |                       | location pointed to different parts of      |
   |                       | the stack.                                  |
   +-----------------------+---------------------------------------------+
   
   .. cpp:function:: StackAnalysis::StackAnalysis(ParseAPI::Function *f)
      
      Constructs a StackAnalysis object for function ``f``.
      
      
   .. cpp:function:: StackAnalysis::StackAnalysis( \
            ParseAPI::Function *f, \
            const std::map<Address, Address> &crm, \
            const std::map<Address, TransferSet> &fs)
      
      Constructs a StackAnalysis object for function ``f`` with
      interprocedural analysis activated. A call resolution map is passed in
      ``crm`` mapping addresses of call sites to the resolved inter-module
      target address of the call. Generally the call resolution map is created
      with DyninstAPI where PLT resolution is done. Function summaries are
      passed in ``fs`` which maps function entry addresses to summaries. The
      function summaries are then used at all call sites to those functions.

   .. cpp:function:: StackAnalysis::Height find(ParseAPI::Block *b, Address addr, Absloc loc)
      
      Returns the stack height of abstract location ``loc`` before execution
      of the instruction with address ``addr`` contained in basic block ``b``.
      The address ``addr`` must be contained in block ``b``, and block ``b``
      must be contained in the function used to create this StackAnalysis
      object.
      
   .. cpp:function:: StackAnalysis::Height findSP(ParseAPI::Block *b, Address addr)
          StackAnalysis::Height findFP(ParseAPI::Block *b, Address addr)
      
      Returns the stack height of the stack pointer and frame pointer,
      respectively, before execution of the instruction with address ``addr``
      contained in basic block ``b``. The address ``addr`` must be contained
      in block ``b``, and block ``b`` must be contained in the function used
      to create this StackAnalysis object.
      
   .. cpp:function:: void findDefinedHeights( \
            ParseAPI::Block *b, \
            Address addr, \
            std::vector<std::pair<Absloc, StackAnalysis::Height>> &heights)
      
      Writes to the vector ``heights`` all defined <abstract location, stack
      height> pairs before execution of the instruction with address ``addr``
      contained in basic block ``b``. Note that abstract locations with stack
      heights of TOP (i.e. they do not point to the stack) are not written to
      ``heights``. The address ``addr`` must be contained in block ``b``, and
      block ``b`` must be contained in the function used to create this
      StackAnalysis object.

   .. cpp:function:: bool canGetFunctionSummary()
      
      Returns true if the function associated with this StackAnalysis object
      returns on some execution path.
      
   .. cpp:function:: bool getFunctionSummary(TransferSet &summary)
      
      Returns in ``summary`` a summary for the function associated with this
      StackAnalysis object. Function summaries can then be passed to the
      constructors for other StackAnalysis objects to enable interprocedural
      analysis. Returns true on success.
      
``StackAnalysis::Height``
-------------------------

**Defined in:** ``stackanalysis.h``

.. cpp:namespace:: StackAnalysis::Height

.. cpp:class:: StackAnalysis::Height
   
   
   The Height class is used to represent the abstract notion of stack
   heights. Every Height object represents a stack height of either TOP,
   BOTTOM, or *x*, where *x* is some integral number. The Height class also
   defines methods for comparing, combining, and modifying stack heights in
   various ways.
   
   .. cpp:type:: signed long Height_t
      
      The underlying data type used to convert between Height objects and
      integral values.
      
      =========== =========== =======================================
      Method name Return type Method description
      =========== =========== =======================================
      height      Height_t    This stack height as an integral value.
      format      std::string This stack height as a string.
      isTop       bool        True if this stack height is TOP.
      isBottom    bool        True if this stack height is BOTTOM.
      =========== =========== =======================================
      
   .. cpp:function:: Height(const Height_t h)
      
      Creates a Height object with stack height ``h``.
      
   .. cpp:function:: Height()
      
      Creates a Height object with stack height TOP.
      
   .. cpp:function:: bool operator<(const Height &rhs)
   .. cpp:function:: const bool operator>(const Height &rhs)
   .. cpp:function:: const bool operator<=(const Height &rhs)
   .. cpp:function:: const bool operator>=(const Height &rhs)
   .. cpp:function:: const bool operator==(const Height &rhs)
   .. cpp:function:: const bool operator!=(const Height &rhs) const
      
      Comparison operators for Height objects. Compares based on the integral
      stack height treating TOP as MAX_HEIGHT and BOTTOM as MIN_HEIGHT.
      
      Height &operator+=(const Height &rhs) Height &operator+=(const signed
      long &rhs) const Height operator+(const Height &rhs) const const Height
      operator+(const signed long &rhs) const const Height operator-(const
      Height &rhs) const
      
      Returns the result of basic arithmetic on Height objects according to
      the following rules, where *x* and *y* are integral stack heights and
      *S* represents any stack height:
      
      -  :math:`TOP + TOP = TOP`
      
      -  :math:`TOP + x = BOTTOM`
      
      -  :math:`x + y = (x+y)`
      
      -  :math:`BOTTOM + S = BOTTOM`
      
      Note that the subtraction rules can be obtained by replacing all + signs
      with - signs.
      
      The ``operator+`` and ``operator-`` methods leave this Height object
      unmodified while the ``operator+=`` methods update this Height object
      with the result of the computation. For the methods where ``rhs`` is a
      ``const signed long``, it is not possible to set ``rhs`` to TOP or
      BOTTOM.

.. _`sec:ast`:

``AST``
-------

**Defined in:** ``DynAST.h``

.. cpp:namespace:: AST

.. cpp:class:: AST
   
   
   We provide a generic AST framework to represent tree structures. One
   example use case is to represent instruction semantics with symbolic
   expressions. The AST framework includes the base class definitions for
   tree nodes and visitors. Users can inherit tree node classes to create
   their own AST structure and AST visitors to write their own analyses for
   the AST.
   
   All AST node classes should be derived from the AST class. Currently we
   have the following types of AST nodes.
   
   .. container:: center
   
      ============= ======================
      AST::ID       Meaning
      ============= ======================
      V_AST         Base class type
      V_BottomAST   Bottom AST node
      V_ConstantAST Constant AST node
      V_VariableAST Variable AST node
      V_RoseAST     ROSEOperation AST node
      V_StackAST    Stack AST node
      ============= ======================
   
   .. cpp:type:: boost::shared_ptr<AST> Ptr;
      
      Shared pointer for class AST.
      
   .. cpp:type:: std::vector<AST::Ptr> Children;
      
      The container type for the children of this AST.
      
   .. cpp:function:: bool operator==(const AST &rhs) const
   .. cpp:function:: bool equals(AST::Ptr rhs)
      
      Check whether two AST nodes are equal. Return ``true`` when two nodes
      are in the same type and are equal according to the ``==`` operator of
      that type.
      
   .. cpp:function:: virtual unsigned numChildren() const
      
      Return the number of children of this node.
      
   .. cpp:function:: virtual AST::Ptr child(unsigned i) const
      
      Return the ``i``\ th child.
      
   .. cpp:function:: virtual const std::string format() const = 0;
      
      Return the string representation of the node.
      
   .. cpp:function:: static AST::Ptr substitute(AST::Ptr in, AST::Ptr a, AST::Ptr b)
      
      Substitute every occurrence of ``a`` with ``b`` in AST ``in``. Return a
      new AST after the substitution.
      
   .. cpp:function:: virtual AST::ID AST::getID() const
      
      Return the class type ID of this node.
      
   .. cpp:function:: virtual Ptr accept(ASTVisitor *v)
      
      Apply visitor ``v`` to this node. Note that this method will not
      automatically apply the visitor to its children.
      
   .. cpp:function:: virtual void AST::setChild(int i, AST::Ptr c)
      
      Set the ``i``\ th child of this node to ``c``.
      
      .. _`sec:symeval`:
      
``SymEval``
-----------

**Defined in:** ``SymEval.h``

.. cpp:namespace:: SymEval

.. cpp:class:: SymEval
   
   
   SymEval provides interfaces for expanding an instruction to its
   symbolic expression and expanding a slice graph to symbolic expressions
   for all abstract locations defined in this slice.
   
   .. cpp:type:: std::map<Assignment::Ptr, AST::Ptr, AssignmentPtrValueComp> Result_t;
      
      This data type represents the results of symbolic expansion of a slice.
      Each assignment in the slice has a corresponding AST.
      
   .. cpp:function:: static std::pair<AST::Ptr, bool> expand( \
            const Assignment::Ptr &assignment, \
            bool applyVisitors = true)
      
      This interface expands a single assignment given by ``assignment`` and
      returns a ``std::pair``, in which the first element is the AST after
      expansion and the second element is a bool indicating whether the
      expansion succeeded or not. ``applyVisitors`` specifies whether or not
      to perform stack analysis to precisely track stack variables.
      
   .. cpp:function:: static bool expand( \
            Result_t &res, \
            std::set<InstructionPtr> &failedInsns, \
            bool applyVisitors = true)
      
      This interface expands a set of assignment prepared in ``res``. The
      corresponding ASTs are written back into ``res`` and all instructions
      that failed during expansion are inserted into ``failedInsns``.
      ``applyVisitors`` specifies whether or not to perform stack analysis to
      precisely track stack variables. This function returns ``true`` when all
      assignments in ``res`` are successfully expanded.

.. container:: center

   ================== ==================
   Retval_t           Meaning
   ================== ==================
   FAILED             failed
   WIDEN_NODE         widen
   FAILED_TRANSLATION failed translation
   SKIPPED_INPUT      skipped input
   SUCCESS            success
   ================== ==================

   .. cpp:function:: static Retval_t expand(Dyninst::Graph::Ptr slice, DataflowAPI::Result_t &res)
      
      This interface expands a slice and returns an AST for each assignment in
      the slice. This function will perform substitution of ASTs.
      
      We use an AST to represent the symbolic expressions of an assignment. A
      symbolic expression AST contains internal node type ``RoseAST``, which
      abstracts the operations performed with its child nodes, and two leave
      node types: ``VariableAST`` and ``ConstantAST``.
      
      ``RoseAST``, ``VariableAST``, and ``ConstantAST`` all extend class
      ``AST``. Besides the methods provided by class ``AST``, ``RoseAST``,
      ``VariableAST``, and ``ConstantAST`` each have a different data
      structure associated with them.

   .. cpp:function:: Variable& VariableAST::val() const
   .. cpp:function:: Constant& ConstantAST::val() const
   .. cpp:function:: ROSEOperation & RoseAST::val() const
      
      We now describe data structure ``Variable``, ``Constant``, and ``ROSEOperation``.

.. cpp:namespace:: Variable

.. cpp:class:: Variable
   
   A ``Variable`` represents an abstract region at a particular address.
   
   .. cpp:function:: Variable::Variable()
   .. cpp:function:: Variable::Variable(AbsRegion r)
   .. cpp:function:: Variable::Variable(AbsRegion r, Address a)
      
      The constructors of class Variable.
      
   .. cpp:function:: bool Variable::operator==(const Variable &rhs) const
   .. cpp:function:: bool Variable::operator<(const Variable &rhs) const
      
      Two Variable objects are equal when their AbsRegion are equal and their
      addresses are equal.
      
   .. cpp:function:: const std::string Variable::format() const
      
      Return the string representation of the Variable.
      
   .. cpp:function:: AbsRegion Variable::reg()
   .. cpp:function:: Address Variable::addr()
      
      The abstraction region and the address of this Variable.
      
.. cpp:namespace::  Constant

.. cpp:class::  Constant
   
   A ``Constant`` object represents a constant value in code.
   
   .. cpp:function:: Constant::Constant()
   .. cpp:function:: Constant::Constant(uint64_t v)
   .. cpp:function:: Constant::Constant(uint64_t v, size_t s)
      
      Construct Constant objects.
      
   .. cpp:function:: bool Constant::operator==(const Constant &rhs) const
   .. cpp:function:: bool Constant::operator<(const Constant &rhs) const
      
      Comparison operators for Constant objects. Comparison is based on the
      value and size.
      
   .. cpp:function:: const std::string Constant::format() const
      
      Return the string representation of the Constant object.
      
   .. cpp:function:: uint64_t Constant::val()
   .. cpp:function:: size_t Constant::size()
      
      The numerical value and bit size of this value.
      
   .. cpp:struct:: ROSEOperation
   
   ``ROSEOperation`` defines the following operations and we represent the
   semantics of all instructions with these operations.

.. container:: center

   ================= ==========================================
   ROSEOperation::Op Meaning
   ================= ==========================================
   nullOp            No operation
   extractOp         Extract bit ranges from a value
   invertOp          Flip every bit
   negateOp          Negate the value
   signExtendOp      Sign-extend the value
   equalToZeroOp     Check whether the value is zero or not
   generateMaskOp    Generate mask
   LSBSetOp          LSB set op
   MSBSetOp          MSB set op
   concatOp          Concatenate two values to form a new value
   andOp             Bit-wise and operation
   orOp              Bit-wise or operation
   xorOp             Bit-wise xor operation
   addOp             Add operation
   rotateLOp         Rotate to left operation
   rotateROp         Rotate to right operation
   shiftLOp          Shift to left operation
   shiftROp          Shift to right operation
   shiftRArithOp     Arithmetic shift to right operation
   derefOp           Dereference memory operation
   writeRepOp        Write rep operation
   writeOp           Write operation
   ifOp              If operation
   sMultOp           Signed multiplication operation
   uMultOp           Unsigned multiplication operation
   sDivOp            Signed division operation
   sModOp            Signed modular operation
   uDivOp            Unsigned division operation
   uModOp            Unsigned modular operation
   extendOp          Zero extend operation
   extendMSBOp       Extend the most significant bit operation
   ================= ==========================================

   .. cpp:function:: ROSEOperation::ROSEOperation(Op o)
   .. cpp:function:: ROSEOperation::ROSEOperation(Op o, size_t s)
      
      Constructors for ROSEOperation
      
   .. cpp:function:: bool ROSEOperation::operator==(const ROSEOperation &rhs) const
      
      Equal operator
      
   .. cpp:function:: const std::string ROSEOperation::format() const
      
      Return the string representation.
      
   .. cpp:function:: ROSEOperation::Op ROSEOperation::op()
   .. cpp:function:: size_t ROSEOperation::size()
      
``ASTVisitor``
--------------

The ASTVisitor class defines callback functions to apply during visiting

.. cpp:namespace:: ASTVisitor

.. cpp:class:: ASTVisitor
   
   an AST for each AST node type. Users can inherit from this class to
   write customized analyses for ASTs.
   
   .. cpp:type:: boost::shared_ptr<AST> ASTVisitor::ASTPtr
   .. cpp:function:: virtual ASTVisitor::ASTPtr ASTVisitor::visit(AST *)
   .. cpp:function:: virtual ASTVisitor::ASTPtr ASTVisitor::visit(DataflowAPI::BottomAST *)
   .. cpp:function:: virtual ASTVisitor::ASTPtr ASTVisitor::visit(DataflowAPI::ConstantAST *)
   .. cpp:function:: virtual ASTVisitor::ASTPtr ASTVisitor::visit(DataflowAPI::VariableAST *)
   .. cpp:function:: virtual ASTVisitor::ASTPtr ASTVisitor::visit(DataflowAPI::RoseAST *)
   .. cpp:function:: virtual ASTVisitor::ASTPtr ASTVisitor::visit(StackAST *)
      
      Callback functions for visiting each type of AST node. The default
      behavior is to return the input parameter.
