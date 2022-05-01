
==============
ProcControlAPI
==============

This section gives an API reference for all classes, functions and types
in ProcControlAPI. Everything defined in this section is under the
namespaces Dyninst and ProcControlAPI. These types can be accessed by
prepending a Dyninst::ProcControlAPI:: in-front of them (e.g.,
Dyninst::ProcControlAPI::Process) or by adding a using namespace
directive before the references (e.g., using namespace Dyninst; using
namespace ProcControlAPI; )

Process
-------

The Process class is the primary handle for operating on a single target
process. Process objects may be created by calls to the static functions
Process::createProcess or Process::attachProcess, or in response to
certain types of events (e.g, fork on UNIX systems).

The static functions of the Process class serve as a central location
for performing general ProcControlAPI operations, such as handleEvents
and registerEventCallback when dealing with callbacks.

Process Declared In:

.. container:: Definition

   PCProcess.h

Process Types:

.. container:: Definition

   Process::ptr

.. container:: Definition

   Process::const_ptr

The Process::ptr and Process::const_ptr respectively represent a pointer
and a const pointer to a Process object. Both pointer types are
reference counted and will cause the underlying Process object to be
cleaned when there are no more references. ProcControlAPI will maintain
internal references to any Process it actively controls, relinquishing
those references when the process either exits or is detached.

.. container:: Definition

   enum Process::cb_action_t {

.. container:: Definition

   cbDefault,

.. container:: Definition

   cbThreadContinue,

.. container:: Definition

   cbThreadStop,

.. container:: Definition

   cbProcContinue,

.. container:: Definition

   cbProcStop

.. container:: Definition

   }

.. container:: Definition

   struct Process::cb_ret_t {

.. container:: Definition

   cb_ret_t(cb_action_t p) : parent(p), child(cbDefault) {}

.. container:: Definition

      | cb_ret_t(cb_action_t p, cb_action_t c) : parent(p), child(c)
      | {}

.. container:: Definition

      cb_action_t parent;

.. container:: Definition

      cb_action_t child;

.. container:: Definition

   }

The cb_ret_t enum is used as the return type for callback functions
registered through Process::registerEventCallback(). A callback function
can specify whether the thread or process associated with its event
should be stopped or continued by respectively returning
cbThreadContinue, cbThreadStop, cbProcContinue, or cbProcStop. The
cbDefault return value returns a Process and Thread to the original
state before the event occurred.

Some events, such as process spawn or thread create involve two
processes or threads. In this case the ProcControlAPI user can specify a
cb_action_t value for both the parent and child using the two parameter
constructor for cb_ret_t.

.. container:: Definition

   typedef Process::cb_ret_t(*cb_func_t)(Event::const_ptr)

The cb_func_t type is a function pointer type for functions that can
handle event callbacks. The callback function gets an Event::const_ptr
as input, which points to the Event that triggered the callback. The
cb_func_t function should return a cb_ret_t describing what to do with
the process after handling the event.

.. container:: Definition

   typedef enum {

.. container:: Definition

   OSNone,

.. container:: Definition

   Linux,

.. container:: Definition

   FreeBSD,

.. container:: Definition

   Windows

.. container:: Definition

   VxWorks

.. container:: Definition

   BlueGeneL

.. container:: Definition

   BlueGeneP

.. container:: Definition

   BlueGeneQ

.. container:: Definition

   } Dyninst::OSType

A value from this enum is returned from Process::getOS and signifies the
current OS on which the target process is running.

This type is used by Process, but it is declared in the Dyninst
namespace in dyntypes.h.

.. container:: Definition

   typedef std::pair<Dyninst::Address, Dyninst::Address> MemoryRegion;

The MemoryRegion type represents a region of allocated memory, and the
first part of the pair is the start address, the second, the end.

Process Static Member Functions:

.. container:: Definition

   static Process::ptr createProcess(

.. container:: Definition

   std::string executable,

.. container:: Definition

   const std::vector<std::string> &argv,

.. container:: Definition

   const std::vector<std::string> &envp = emptyEnv,

.. container:: Definition

   const std::map<int,int> &fds = emptyFDs)

This function creates a new process by launching an executable file
named by executable with the arguments specified by argv, the
environment specified in envp, and it returns a pointer to the new
Process object upon success. The new process will be created with its
initial thread in the stopped state.

It is an error to call this function from a callback.

If the fds map is not empty, then the new process will be created with
the file descriptors from the fds’ first elements dup2 mapped to the
file descriptors in fds’ second elements.

If envp is empty, the environment will be inherited from the calling
process.

ProcControlAPI may deliver callbacks when this function is called.

This function returns Process::ptr() on error, and a subsequent call to
getLastError returns details on the error.

.. container:: Definition

   static Process::ptr attachProcess(

.. container:: Definition

   Dyninst::PID pid,

.. container:: Definition

   std::string executable = "")

This function creates a new Process object by attaching to the PID
specified by pid. The new Process object will be returned from this
function upon success. The executable argument is optional, and can be
used to assist ProcControlAPI in finding the process’ executable on
operating systems where this cannot be easily determined (currently on
AIX). The new process will be returned with all of its threads in the
stopped state.

It is an error to call this function from a callback.

ProcControlAPI may deliver callbacks when this function is called.

This function return Process::ptr() on error, and a subsequent call to
getLastError returns details on the error.

.. container:: Definition

   static bool handleEvents(bool block)

This function causes ProcControlAPI to handle any pending debug events
and deliver callbacks. When an event requires a callback ProcControlAPI
needs control of the main thread in order to deliver the callback. This
function gives control of the main thread to ProcControlAPI for callback
delivery. A user can know when to call handleEvents by using the
EventNotify interface; See Sections 2.2.3. and for more details on
EventNotify.

If the block parameter is true, then handleEvents will block until at
least one debug event has been handled. If block is false then
handleEvents returns immediately if no events are ready to be handled.

This function returns true if it handled at least one event and false
otherwise.

It is an error to call this function from a callback.

.. container:: Definition

   static bool registerEventCallback(

.. container:: Definition

   EventType evt,

.. container:: Definition

   cb_func_t cbfunc)

This function registers a new callback function with ProcControlAPI.
Upon receiving an event with type evt, ProcControlAPI will deliver a
callback with that event to the cbfunc function. Multiple functions can
be registered to receive callbacks for a single EventType, and a single
function can be registered with multiple EventTypes.

If multiple callback functions are registered with a single EventType,
then it is undefined what order those callback functions will be invoked
in. In this case the cb_ret_t result of the last callback function
called will be used to determine what stop or continue operations should
be performed on the process. If a single callback function is registered
for the same EventType multiple times, then ProcControlAPI will only
invoke one call to the callback function for each instance of the
EventType.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   static bool removeEventCallback(

.. container:: Definition

   EventType evt,

.. container:: Definition

      cb_func_t cbfunc)

This function un-registers a callback that was registered with
registerEventCallback. After a successful call to this function the
callback function cbfunc will stop being called for events with
EventType evt. Other callback functions registered for evt will not be
affected. Other instances of cbfunc registered for different EventTypes
will not be affected.

This function returns true if a callback was successfully removed and
false otherwise. Upon an error a subsequent call to getLastError returns
details on the error.

.. container:: Definition

   static bool removeEventCallback(EventType evt)

This function unregisters all callback functions associated with the
EventType evt. After a successful call to this function ProcControlAPI
will stop delivering callbacks for evt until a new callback function is
registered.

This function returns true if a callback was successfully removed and
false otherwise. Upon an error a subsequent call to getLastError returns
details on the error.

.. container:: Definition

   static bool removeEventCallback(cb_func_t func)

This function unregisters all instances of callback function func from
any callback with any EventType.

This function returns true if a callback was successfully removed and
false otherwise. Upon an error a subsequent call to getLastError returns
details on the error.

Process Member Functions:

.. container:: Definition

   Dyninst::PID getPid() const

This function returns an OS handle referencing the process. On UNIX
systems this is the pid of the process.

.. container:: Definition

   Dyninst::Architecture getArchitecture() const

This function returns an enum that describes the architecture of the
target process. See Appendix A. for the definition of
Dyninst::Architecture.

.. container:: Definition

   Dyninst::OSType getOS () const

This function returns an enum that describes the OS of the target
process. See the beginning of this section for the definition of
Dyninst::OSType.

.. container:: Definition

   bool supportsLWPEvents () const

This function returns true if the target process can throw LWP create
and destroy events and false otherwise.

.. container:: Definition

   bool supportsUserThreadEvents () const

This function returns true if the target process can throw user thread
create and destroy events and false otherwise.

.. container:: Definition

   bool supportsFork () const

This function returns true if the fork system call is supported in the
target process and false otherwise.

.. container:: Definition

   bool supportsExec () const

This function returns true if the exec system call is supported in the
target process and false otherwise.

.. container:: Definition

   bool isTerminated() const

This function returns true if the target process has terminated (either
via a crash or normal exit) or if the ProcControlAPI has detached from
the target process. It returns false otherwise.

.. container:: Definition

   bool isExited() const

This function returns true of the target process exited via a normal
exit (e.g, calling the exit function or returning from main). It returns
false otherwise.

.. container:: Definition

   int getExitCode() const

If a target process exited normally then this function returns its exit
code. The return result of this function is undefined if the Process’
isExited function returns false.

.. container:: Definition

   bool isCrashed() const

This function returns true if the target process exited because of a
crash. It returns false otherwise.

.. container:: Definition

   int getCrashSignal() const

If a target process exited because of a crash, then this function
returns the signal that caused the target process to crash. The return
result of this function is undefined if the Process’ isCrashed function
returns false.

.. container:: Definition

   bool hasStoppedThread() const

This function returns true if the target process has at least one thread
in the stopped state. It returns false otherwise or if an error occurs.
In the event of an error a call to getLastError returns details on the
error.

.. container:: Definition

   bool hasRunningThread() const

This function returns true if the target process has at least one thread
in the running state. It returns false otherwise or if an error occurs.
In the event of an error a call to getLastError returns details on the
error.

.. container:: Definition

   bool allThreadsStopped() const

This function returns true if all threads in the target process are in
the stopped state. It returns false otherwise or if an error occurs. In
the event of an error a call to getLastError returns details on the
error.

.. container:: Definition

   bool allThreadsRunning() const

This function returns true if all threads in the target process are in
the running state. It returns false otherwise or if an error occurs. In
the event of an error a call to getLastError returns details on the
error.

.. container:: Definition

   bool allThreadsRunningWhenAttached() const

This function returns true if all threads were running when the
controller process attached to this process. It returns false if any
threads were stopped. If the target process was created instead of
attached, this function returns true.

.. container:: Definition

   bool continueProc()

This function will move all threads in the target process into the
running state. This function returns true if at least one thread was
continued as part of the call, and false otherwise.

It is an error to call this function from a callback.

ProcControlAPI may deliver callbacks when this function is called.

This function return false on error, and a subsequent call to
getLastError returns details on the error.

.. container:: Definition

   bool stopProc()

This function will move all threads in the target process into the
stopped state. This function returns true if at least one thread was
stopped as part of the call, and false otherwise.

It is an error to call this function from a callback.

ProcControlAPI may deliver callbacks when this function is called.

This function return false on error, and a subsequent call to
getLastError returns details on the error.

.. container:: Definition

   bool detach(bool leaveStopped = false)

This function will detach ProcControlAPI from the target process.
ProcControlAPI will no longer be able to control or receive events from
the target process. All breakpoints will be removed from the target.
This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

If the leaveStopped parameter is set to true, and the process is in a
stopped state, then the target process will be left in a stopped state
after the detach.

It is an error to call this function from a callback.

.. container:: Definition

   bool temporaryDetach()

This function temporarily detaches from the target process, but leaves
the Process data structure intact. This functionality is commonly called
detach-on-the-fly. The target process will not report new events nor be
controllable or able to be queried by the user. Breakpoints are removed
from the process. The reAttach function will reconnect the process after
this call.

This function returns true on success and false upon error.

It is an error to call this function from a callback.

.. container:: Definition

   bool reAttach()

This function reconnects to the target process after a temporaryDetach
call. Any breakpoints will be re-inserted back into the function, and if
threads have been created or destroyed during the time detached new
events will be thrown for them.

This function returns true on success and false upon error.

It is an error to call this function from a callback.

.. container:: Definition

   bool terminate()

This function forcefully terminated the target process. Upon a
successful call to this function the target process will end execution.
The Process object will record the target process as having crashed.
This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

It is an error to call this function from a callback.

.. container:: Definition

   const ThreadPool &threads() const

.. container:: Definition

   ThreadPool &threads()

These functions respectively return a const reference or a reference to
the Process’ ThreadPool. The ThreadPool object can be used to iterate
over and query the Process’ Thread objects—see the Section 3.6. for more
details on ThreadPool.

.. container:: Definition

   const LibraryPool &libraries() const

.. container:: Definition

   LibraryPool &libraries()

These functions respectively return a const reference or a reference to
the Process’ LibraryPool. The LibraryPool object can be used to iterate
over and query the Process’ Library objects—see the Section 3.7. for
more details on LibraryPool.

.. container:: Definition

   bool addLibrary(std::string libname)

This function causes the specified library to be loaded into the
process. It will trigger an event (and thus a user callback) for each
library loaded (including dependencies).

.. container:: Definition

   void \*getData () const

.. container:: Definition

   void setData (void \*p) const

These functions respectively get and set an opaque data object that can
be associated with this process. The data is not interpreted by
ProcControlAPI, but remains associated with the process.

.. container:: Definition

   unsigned getMemoryPageSize() const

This function returns memory page size for the current OS on which the
target process is running.

.. container:: Definition

   Dyninst::Address mallocMemory(size_t long size)

.. container:: Definition

   Dyninst::Address mallocMemory(

.. container:: Definition

   size_t size,

.. container:: Definition

   Dyninst::Address addr)

These functions allocate a region of memory in the target process’
address space of size size. Upon a successful call these functions will
map an area of memory in the target process that is readable, writeable
and executable. The mallocMemory(size_t) function will allocate memory
at any available address. The mallocMemory(size_t, Dyninst::Address)
function will only allocate memory at the specified address, addr.

It is an error to call this function from a callback.

ProcControlAPI may deliver callbacks when this function is called.

Upon success these functions return the start address of memory that was
allocated and 0 otherwise. Upon an error a subsequent call to
getLastError returns details on the error.

.. container:: Definition

   bool freeMemory(Dyninst::Address addr)

This function will free a region of memory that was allocated by the
mallocMemory function. Upon a successful call to this function, the area
of memory starting at addr will be unmapped and no longer accessible to
the target process. It is an error to call this function with an address
that was not returned by mallocMemory.

It is an error to call this function from a callback.

ProcControlAPI may deliver callbacks when this function is called.

Upon success this function returns true, otherwise it returns false.
Upon an error a subsequent call to getLastError returns details on the
error.

.. container:: Definition

   bool writeMemory(

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   void \*buffer,

.. container:: Definition

   size_t size) const

This function writes to the target process’s memory. The addr parameter
specifies an address in the target process to which ProcControlAPI
should write. The buffer and size parameters specify a region of
controller process memory that will be copied into the target process.

It is an error to call this function on a Process that does not have at
least one Thread in a stopped state.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool readMemory(

.. container:: Definition

   void \*buffer,

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   size_t size) const

This function reads from the target process’ memory. The addr and size
parameters specify an address in the target process from which
ProcControlAPI should read. The buffer parameter specifies an address in
the controller process where ProcControlAPI should write the copied
bytes.

It is an error to call this function on a Process that does not have at
least one Thread in a stopped state.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool getMemoryAccessRights(

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   mem_perm& rights)

.. container:: Definition

   bool setMemoryAccessRights(

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   size_t size,

.. container:: Definition

   mem_perm rights,

.. container:: Definition

   mem_perm& oldRights)

These functions respectively get and set memory permission at the
specified address, addr. The setMemoryAccessRights function also affects
a region of memory in the target process’s address space of size size.

.. container:: Definition

   bool findAllocatedRegionAround(

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   MemoryRegion& memRegion)

This function finds a region of allocated memory memRegion contains
address addr, and returns true on success, otherwise false.

.. container:: Definition

   bool addBreakpoint(

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   Breakpoint::ptr bp) const

This function inserts the Breakpoint specified by bp into the target
process at address addr. See the Section 3.4. for more details on
Breakpoint.

It is an error to call this function on a Process that does not have at
least one Thread in a stopped state.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool rmBreakpoint(

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   Breakpoint::ptr bp) const

This function removes the Breakpoint specified by bp at address addr
from the target process. See the section 3.4. on Breakpoint for more
details.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool postIRPC(IRPC::ptr irpc) const

This function posts the given irpc to the Process. ProcControlAPI
selects a Thread from the Process to run the iRPC and put irpc into that
Thread’s queue of posted IRPCs. See Sections 2.3. and 3.5. for more
information on iRPCs.

Each instance of an IRPC object can be posted at most once. It is an
error to attempt to post a single IRPC object multiple times.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool runIRPCSync(IRPC::ptr irpc)

This function posts an irpc, similar to Process::postIRPC; continues the
thread the irpc was posted to; and returns when the irpc has completed
running. The thread will be returned to its original running state when
this function returns.

This function returns true if the irpc was successfully run, and false
otherwise. Note that stopping the thread that is running the irpc while
this function waits for irpc completion causes this function to return
an error.

It is an error to call this function from a callback.

.. container:: Definition

   bool runIRPCAsync(IRPC::ptr irpc)

This function posts an irpc, similar to Process::postIRPC, and then
continues the thread the irpc was posted to.

This function returns true if the irpc was successfully posted and run,
and false otherwise.

It is an error to call this function from a callback.

.. container:: Definition

   bool getPostedIRPCs(std::vector<IRPC::ptr> &rpcs) const

This function returns all IRPCs posted to this Process in the rpcs
vector. This list does not include any IRPCs currently running—see
Thread::getRunningIRPC() for this functionality.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   LibraryTracking \*getLibraryTracking()

.. container:: Definition

   ThreadTracking \*getThreadTracking()

.. container:: Definition

   LWPTracking \*getLWPTracking()

.. container:: Definition

   FollowFork \*getFollowFork()

.. container:: Definition

   SignalMask \*getSignalMask()

These functions return pointers to configuration objects for
platform-specific features associated with this Process object.

These functions return NULL if the specified feature is unsupported on
the current platform.

mem_perm
--------

The mem_perm nested class, which defined within Process class,
represents general memory page permission for the given memory page in
the process.

mem_perm Declared In:

.. container:: Definition

   PCProcess.h

mem_perm Types:

.. container:: Definition

   Process::mem_perm::read

.. container:: Definition

   Process::mem_perm::write

.. container:: Definition

   Process::mem_perm::execute

The Process::mem_perm::read, Process::mem_perm::write, and
Process::mem_perm::execute, just as their names imply, respectively
represent read, write, and execution permission of given memory page.

mem_perm Member Functions:

.. container:: Definition

   mem_perm() : read(false), write(false), execute(false) {}

.. container:: Definition

   mem_perm(const mem_perm& p) : read(p.read), write(p.write),
   execute(p.execute) {}

.. container:: Definition

   mem_perm(bool r, bool w, bool x) : read(r), write(w), execute(x) {}

These constructors provide a convenient way to create the specific
memory permission for the given page.

.. container:: Definition

   bool getR() const

.. container:: Definition

   bool getW() const

.. container:: Definition

   bool getX() const

These functions return true if the given memory page has read, write,
and execution permission, respectively, and false otherwise.

.. container:: Definition

   bool isNone() const

.. container:: Definition

   bool isR()    const

.. container:: Definition

   bool isX()    const

.. container:: Definition

   bool isRW()   const

.. container:: Definition

   bool isRX()   const

.. container:: Definition

   bool isRWX()  const

These functions return true if the permission of given memory page is
NO_ACCESS, READ_ONLY, EXECUTE, READ_WRITE, READ_EXECUTE, and
READ_WRITE_EXECUTE, respectively, and false otherwise.

.. container:: Definition

   Process::mem_perm& setR()

.. container:: Definition

   Process::mem_perm& setW()

.. container:: Definition

   Process::mem_perm& setX()

These functions enable read, write, and execution permission for the
given page, respectively, and return this mem_perm.

.. container:: Definition

   Process::mem_perm& clrR()

.. container:: Definition

   Process::mem_perm& clrW()

.. container:: Definition

   Process::mem_perm& clrX()

These functions disable read, write, and execution permission for the
given page, respectively, and return this mem_perm.

.. container:: Definition

   bool operator==(const mem_perm& p) const

This function returns true if memory permission p is the same as this
mem_perm and false otherwise.

.. container:: Definition

   bool operator!=(const mem_perm& p) const

This function returns true if memory permission p is different from this
mem_perm and false otherwise.

.. container:: Definition

   bool operator<(const mem_perm& p) const

This function returns true if this mem_perm is less than p according to
the notation that read permission encodes to 4, write, 2, and execute,
1, and false otherwise.

.. container:: Definition

   std::string getPermName()

Return the memory permission name for this mem_perm.

Thread
------

The Thread class represents a single thread of execution in the target
process. Any Process has at least one Thread, and multi-threaded target
processes may have more. Each Thread has an associated integral value
known as its LWP, which serves as a handle for communicating with the OS
about the thread (e.g., a PID value on Linux). On some systems,
depending on availability, a Thread may have information from the user
space threading library.

Thread Declared In:

.. container:: Definition

   PCProcess.h

Thread Types:

.. container:: Definition

   Thread::ptr

.. container:: Definition

   Thread::const_ptr

The Thread::ptr and Thread::const_ptr respectively represent a pointer
and a const pointer to a Thread object. Both pointer types are reference
counted and cause the underlying Thread object to be cleaned when there
are no more references. ProcControlAPI maintains internal references to
any Thread it actively controls, relinquishing those references when the
thread exits or is detached.

Thread Member Functions:

.. container:: Definition

   Dyninst::LWP getLWP() const

This function returns an OS handle for this thread. On Linux this
returns a pid_t for this thread. On FreeBSD, this returns a lwpid_t.

.. container:: Definition

   Process::ptr getProcess()

.. container:: Definition

   Process::const_ptr getProcess() const

These functions return a pointer to the Process object that contains
this thread.

.. container:: Definition

   bool isStopped() const

This function returns true if this thread is in a stopped state and
false otherwise.

.. container:: Definition

   bool isRunning() const

This function returns true if this thread is in a running state and
false otherwise.

.. container:: Definition

   bool isLive() const

This function returns true if this thread is alive, and it returns false
if this thread has been destroyed.

.. container:: Definition

   bool isDetached() const

This function returns true if this thread has been detached via
Process::temporaryDetach and false otherwise.

.. container:: Definition

   bool isInitialThread() const

This function returns true if this thread is the initial thread for the
process and false otherwise.

.. container:: Definition

   bool stopThread()

This function moves the thread to into a stopped state. Upon a
successful call to this function the Thread object will be paused and
will not resume execution until the Thread is continued. It is an error
to call this function from a callback. Instead of calling this function,
a callback can stop a thread by returning Process::cbThreadStop or
Process::cbProcStop.

ProcControlAPI may deliver callbacks when this function is called.

Upon success this function returns true, otherwise it returns false.
Upon an error a subsequent call to getLastError returns details on the
error.

.. container:: Definition

   bool continueThread()

This function moves the thread into a running state. It is an error to
call this function from a callback. Instead of calling this function, a
callback can stop a thread by returning Process::cbThreadContinue or
Process::cbProcContinue.

ProcControlAPI may deliver callbacks when this function is called.

Upon success this function returns true, otherwise it returns false.
Upon an error a subsequent call to getLastError returns details on the
error.

.. container:: Definition

   bool getRegister(

.. container:: Definition

   Dyninst::MachRegister reg,

.. container:: Definition

   Dyninst::MachRegisterVal &val) const

This function gets the value of a single register from this thread. The
register is specified by the reg parameter, and the value of the
register is returned by the val parameter. See Appendix A. for an
explanation of the MachRegister class.

It is an error to call this function on a thread that is not in the
stopped state.

Upon success this function returns true, otherwise it returns false.
Upon an error a subsequent call to getLastError returns details on the
error.

.. container:: Definition

   bool getAllRegisters(RegisterPool pool) const

This function reads the values of every register in the thread and
returns them as part of the RegisterPool object pool. Depending on the
OS, this call may be more efficient that calling Thread::getRegister
multiple times. See Section 3.8. for a discussion of the RegisterPool
class.

It is an error to call this function on a thread that is not in the
stopped state.

Upon success this function returns true, otherwise it returns false.
Upon an error a subsequent call to getLastError returns details on the
error.

.. container:: Definition

   bool setRegister(

.. container:: Definition

   Dyninst::MachRegister reg,

.. container:: Definition

   Dyninst::MachRegisterVal val) const

This function writes the value of a single register in this thread. The
register is specified by the reg parameter, and the value that should be
written is specified by the val parameter. See Appendix A. for an
explanation of the MachRegister class.

It is an error to call this function on a thread that is not in the
stopped state.

Upon success this function returns true, otherwise it returns false.
Upon an error a subsequent call to getLastError returns details on the
error.

.. container:: Definition

   bool setAllRegisters(RegisterPool &pool) const

This function sets the values of every register in this thread to the
values specified in the RegisterPool object pool. Depending on the OS,
this call may be more efficient that calling Thread::setRegister
multiple times. See Section 3.8. for a discussion of the RegisterPool
class.

It is an error to call this function on a thread that is not in the
stopped state.

Upon success this function returns true, otherwise it returns false.
Upon an error a subsequent call to getLastError returns details on the
error.

.. container:: Definition

   bool haveUserThreadInfo() const;

This function returns true if information about this Thread’s underlying
user-level thread is available.

.. container:: Definition

   Dyninst::THR_ID getTID() const;

This function returns the unique identifier for the user-level thread.
This value is only valid if haveUserThreadInfo returns true.

.. container:: Definition

   Dyninst::Address getStartFunction() const;

This function returns the address of the initial function for the
user-level thread. This value is only valid if haveUserThreadInfo
returns true.

.. container:: Definition

   Dyninst::Address getStackBase() const;

This function returns the address of the bottom of the user-level
thread’s stack. This value is only valid if haveUserThreadInfo returns
true.

.. container:: Definition

   unsigned long getStackSize() const;

This function returns the size in bytes of the user-level thread’s
allocated stack. This value is only valid if haveUserThreadInfo returns
true.

.. container:: Definition

   Dyninst::Address getTLS() const;

This function returns the address of the user-level thread’s thread
local storage area. This value is only valid if haveUserThreadInfo
returns true.

.. container:: Definition

   bool readThreadLocalMemory(

.. container:: Definition

   void \*buffer,

.. container:: Definition

   Library::const_ptr lib,

.. container:: Definition

   Dyninst::Offset tls_symbol_offset,

.. container:: Definition

   size_t size) const

This function reads from a symbol in thread local storage (TLS) memory.
TLS is memory that is local to a thread and has a lifetime matching the
thread. The tls_symbol_offset is the TLS symbol’s offset in lib, and can
be found by reading a TLS symbol’s value. The lib parameter can point to
a library or the executable. The buffer parameter specifies an address
in the controller process where ProcControlAPI should write the copied
bytes.

It is an error to call this function on a Thread that is not in a
stopped state. It is also an error to call this function on a Thread
that has not have user-level thread information, which can be tested
with haveUserThreadInfo.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool writeThreadLocalMemory(

.. container:: Definition

   Library::const_ptr lib,

.. container:: Definition

   Dyninst::Offset tls_symbol_offset,

.. container:: Definition

   const void \*buffer,

.. container:: Definition

   size_t size) const

This function writes to a symbol in thread local storage (TLS) memory.
TLS is memory that is local to a thread and has a lifetime matching the
thread. The tls_symbol_offset is the TLS symbol’s offset in lib, and can
be found by reading a TLS symbol’s value. The lib parameter can point to
a library or the executable. The buffer parameter specifies an address
in the controller process where ProcControlAPI should read the bytes to
be copied.

It is an error to call this function on a Thread that is not in a
stopped state. It is also an error to call this function on a Thread
that has not have user-level thread information, which can be tested
with haveUserThreadInfo.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool getThreadLocalAddress(

.. container:: Definition

   Library::const_ptr lib,

.. container:: Definition

   Dyninst::Offset tls_symbol_offset,

.. container:: Definition

   Dyninst::Address &result_addr) const

This function looks up the address of a symbol in thread local storage
(TLS) memory. The tls_symbol_offset is the TLS symbol’s offset in lib,
and can be found by reading a TLS symbol’s value. The lib parameter can
point to a library or the executable. The result_addr parameter will be
set to the target address for the TLS symbol in this Thread.

It is an error to call this function on a Thread that is not in a
stopped state. It is also an error to call this function on a Thread
that has not have user-level thread information, which can be tested
with haveUserThreadInfo.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool postIRPC(IRPC::ptr irpc) const

This function posts the given irpc to the Thread. The IRPC is put irpc
into the Thread’s queue of posted IRPCs and will be run when ready. See
Section 2.3. for more information on posting IRPCs.

Each instance of an IRPC object can be posted at most once. It is an
error to attempt to post a single IRPC object multiple times.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   bool getPostedIRPCs(std::vector<IRPC::ptr> &rpcs) const

This function returns all IRPCs posted to this Thread in the vector
rpcs. This does not include any running IRPC.

This function returns true on success and false on error. Upon an error
a subsequent call to getLastError returns details on the error.

.. container:: Definition

   IRPC::const_ptr getRunningIRPC() const

This function returns a const pointer to any IRPC that is actively
running on this Thread. If there is no IRPC actively running, then this
function returns IRPC::const_ptr().

.. container:: Definition

   void setSingleStepMode(bool mode) const

This function sets whether a Thread is in single-step mode. If called
with a mode of true, then the Thread is put in single-step mode. If
called with a mode of false, then the Thread is taken out of single-step
mode.

A Thread in single-step mode will pause execution at each instruction
and trigger an EventSingleStep event. After each EventSingleStep is
handled (and presuming the Thread is still running and in single-step
mode) the Thread will execute one more instruction and trigger another
EventSingleStep.

.. container:: Definition

   bool getSingleStepMode() const

This function returns true if the Thread is in single-step mode and
false otherwise.

.. container:: Definition

   void \*getData() const

.. container:: Definition

   void setData(void \*p) const

These functions respectively get and set an opaque data object that can
be associated with this Thread. The data is not interpreted by
ProcControlAPI, but remains associated with the Thread.

Library
-------

A Library represents a single shared library (frequently referred to as
a DLL or DSO, depending on the OS) that has been loaded into the target
process. In addition, a Library will be used to represent the process’
executable. Process’ with statically linked executables will only
contain the single Library that represents the executable.

Each Library contains a *load address* and a file name. The load address
is the address at which the OS loaded the library, and the file name is
the path to the library’s file. Note that on some operating systems
(Linux, Solaris, BlueGene, FreeBSD) the load address does not
necessarily represent the beginning of the library in memory; instead it
is a value that can be added to a library’s symbol offsets to compute
the dynamic address of a symbol.

Libraries may be loaded and unloaded by the process during execution. A
library load or unload can trigger a callback with an EventLibrary
parameter. The current list of libraries loaded into a process can be
accessed via a Process’ LibraryPool object (see Section 3.7.).

Library Types

.. container:: Definition

   | Library::ptr
   | Library::const_ptr

The Library::ptr and Library::const_ptr types are respective typedefs
for a pointer and a const pointer to a library.

These pointers are not shared pointers—ProcControl will automatically
clean a Library object when it is unloaded. It is not recommended that
the user maintains copies of pointers to Library objects after an
EventLibrary delivers notice of a library unload.

Library Member Functions

.. container:: Definition

   std::string getName() const

Returns the file name for this Library.

.. container:: Definition

   std::string getAbsoluteName() const

Returns a file name for this Library that does not contain symlinks or a
relative path.

.. container:: Definition

   Dyninst::Address getLoadAddress() const

Returns the load address for this Library.

.. container:: Definition

   Dyninst::Address getDataLoadAddress() const

The AIX operating system can have two load addresses for a library: one
for the code region and one for the data region. On AIX
Library::getLoadAddress returns the load address of the code region and
Library::getDataLoadAddress returns the load address of the data region.
On non-AIX systems this function returns 0.

.. container:: Definition

   Dyninst::Address getDynamicAddress() const

On ELF based systems (FreeBSD, Linux, BlueGene) this function will
return the address of the Library’s dynamic section. On other systems
this function returns 0.

.. container:: Definition

   bool isSharedLib() const

This function returns true of this Library object is refers to a shared
library and false if it refers to an executable.

.. container:: Definition

   void \*getData() const

This function returns an opaque data object that user code can associate
with this library. Use setData to set this opaque value.

.. container:: Definition

   void setData(void \*p) const

This function sets associates an opaque data object with the Library.
ProcControlAPI does not try to interpret this value, but will return it
with the getData function.

Breakpoint
----------

A breakpoint is a point in the code region of a target process that,
when executed, stops the process execution and notifies ProcControlAPI.
Upon being continued the process will resume execution at the point. A
Breakpoint object is a handle that can represents one or more
breakpoints in one or more processes. Upon receiving notification that a
breakpoint has executed, ProcControlAPI will deliver a callback with an
EventBreakpoint, (see Section 3.15.7.).

Some Breakpoint objects can be created as *control-transfer
breakpoints*. When a process is continued after executing a
control-transfer the process will resume at an alternate location,
rather than at the breakpoint’s installation point.

A single Breakpoint can be inserted into multiple locations within a
target process. This can be useful when a user has wants to perform a
single action at multiple locations in a target process. For example, if
a user wants to insert a breakpoint at the entry to function foo, and
foo has multiple instantiations in a process, then a single Breakpoint
can be inserted at each instance of foo.

A single Breakpoint object can be inserted into multiple target
processes at the same time. When a process does an operation that copies
an address space, such as fork on UNIX, the child process will receive
all Breakpoint objects that were installed in the parent process.

Multiple Breakpoint objects can be inserted into the same location
with-in the same process. When this location is executed in the target
process a single callback will be delivered, and the EventBreakpoint
object will contain a reference to each Breakpoint inserted at the
location. At most one control-transfer breakpoint can be inserted at any
one point in a process.

Due to the many-to-many nature of Breakpoints and Processes, a single
installation of a Breakpoint can be identified by a Breakpoint, Process,
Address triple. The functions for inserting and removing breakpoints
(Process::addBreakpoint and Process::rmBreakpoint) need all three pieces
of information.

A Breakpoint can be a hardware breakpoint or a software breakpoint. A
hardware breakpoint is typically implemented by setting special debug
register in the process and can trigger on code execution, data reads or
data write. A software breakpoint is typically implemented by writing a
special instruction into a code sequence and can only be triggered by
code execution. There are typically a limited number of hardware
breakpoints available at the same time.

Breakpoint Types

.. container:: Definition

   Breakpoint::ptr

.. container:: Definition

   Breakpoint::const_ptr

The Breakpoint::ptr and Breakpoint::const_ptr types are respectively a
pointer and a const pointer to a Breakpoint object. These pointers are
shared pointers, and the underlying Breakpoint object will be
automatically clean when there are no more references to it.
ProcControlAPI will automatically maintain at least one reference to any
Breakpoint that is installed in a target process.

Breakpoint Constant Values

static const int BP_X = 1

static const int BP_W = 2

static const int BP_R = 4

These constant values are used to set execute, write and read bits on
hardware breakpoints.

Breakpoint Static Functions

.. container:: Definition

   Breakpoint::ptr newBreakpoint()

This function creates a new software Breakpoint object and returns it.
The Breakpoint is not inserted into a Process until it is passed to
Process::addBreakpoint().

.. container:: Definition

   Breakpoint::ptr newTransferBreakpoint(Dyninst::Address ctrl_to)

This function creates a new control transfer software breakpoint. Upon
resumption after executing this Breakpoint, control will resume at the
address specified by the ctrl_to parameter.

.. container:: Definition

   Breakpoint::ptr newHardwareBreakpoint(unsigned int mode, unsigned int
   size)

This function creates a new hardware breakpoint. The mode parameter is a
bitfield that contains an OR combination the values BP_X, BP_W and BP_R.
These control whether the breakpoint will trigger when its target
address is executed, written or read.

The size parameter specifies a range of memory that this breakpoint
monitors. If memory is accessed between the target address and target
address + size, then the breakpoint will trigger.

Breakpoint Member Functions

.. container:: Definition

   bool isCtrlTransfer() const

This function returns true if the Breakpoint is a control transfer
breakpoint, and false if it is a regular Breakpoint.

.. container:: Definition

   Dyninst::Address getToAddress() const

If this Breakpoint is a control transfer breakpoint, then this function
returns the address to which it transfers control. If this Breakpoint is
not a control transfer breakpoint, then this function returns 0.

.. container:: Definition

   void setData(void \*data) const

This function sets the value of an opaque handle that is associated with
each Breakpoint. The opaque handle can be any value, and it can be
retrieved with the getData function.

.. container:: Definition

   void \*getData() const

This function returns the value of the opaque handled that is associated
with this Breakpoint.

.. container:: Definition

   void setSuppressCallbacks(bool val)

This function can be used to suppress callbacks stemming from a specific
breakpoint when called with val set to true value. All other effects
from this breakpoint will still occur, but it will not generate a
callback. By default callbacks occur from every breakpoint.

.. container:: Definition

   bool suppressCallbacks() const

This function returns true if callbacks have been suppressed for this
breakpoint from Breakpoint::setSuppressCallbacks and false otherwise.

IRPC
----

IRPC is a class representing an Inferior Remote Procedure Call that can
be run in a target process. See Section 2.3. for a high level discussion
of iRPCs. Also see Process::postIRPC and Thread::postIRPC for
information about posting an IRPC.

IRPC Declared In:

.. container:: Definition

   PCProcess.h

IRPC Types:

.. container:: Definition

   IRPC::ptr

.. container:: Definition

   IRPC::const_ptr

The IRPC::ptr and IRPC::const_ptr respectively represent a pointer and a
const pointer to an IRPC object. Both pointer types are reference
counted and will cause the underlying IRPC object to be cleaned when
there are no more references. ProcControlAPI will maintain internal
references to any IRPC currently posted or executing.

IRPC Static Member Functions:

.. container:: Definition

   IRPC::ptr createIRPC(

.. container:: Definition

   void \*binary_blob,

.. container:: Definition

   unsigned int size,

.. container:: Definition

   bool non_blocking = false)

.. container:: Definition

   IRPC::ptr createIRPC(

.. container:: Definition

   void \*binary_blob,

.. container:: Definition

   unsigned int size,

.. container:: Definition

   Dyninst::Address addr,

.. container:: Definition

   bool non_blocking = false)

The createIRPC static function creates and returns a new IRPC object.
The binary_blob and size parameters specify a buffer of machine code
bytes that this IRPC should execute when invoked. ProcControlAPI will
maintain its own copy of the binary_blob buffer, the ProcControlAPI user
can free the buffer once this function completes.

If the non_blocking parameter is true then calls to
Process::handleEvents will block until this IRPC is completed.

If the addr parameter is given, then ProcControlAPI will write and run
the binary code at addr. Otherwise ProcControlAPI will select a location
at which to run the IRPC.

IRPC Member Functions:

.. container:: Definition

   Dyninst::Address getAddress() const

The getAddress function returns the address at which the IRPC will be
run. If the IRPC was not given an address at construction and has not
yet started running, then this function may return 0.

.. container:: Definition

   void \*getBinaryCodeBlob() const

The getBinaryCodeBlob returns a pointer to memory that contains the
binary code for this IRPC.

.. container:: Definition

   unsigned int getBinaryCodeSize() const

The getBinaryCodeSize function returns the size of the binary code blob
buffer.

.. container:: Definition

   unsigned long getID() const

The getID function returns an integer identifier that uniquely
identifies this IRPC.

.. container:: Definition

   void setStartOffset(unsigned long off)

By default an IRPC will start executing its code blob at the beginning
of the blob. This function can be used to tell ProcControlAPI to start
execution of the code blob at some byte offset, off, into the blob.

This function should be called before the IRPC is posted.

.. container:: Definition

   unsigned long getStartOffset() const

If a start offset has been set for this IRPC, then getStartOffset
returns it. Otherwise this function returns 0.

ThreadPool
----------

A ThreadPool object is a collection for holding the Threads that make up
a Process. Each Process object has one ThreadPool object, and each
ThreadPool object has one or more Thread objects. A ThreadPool is
typically used to iterate over or search the set of Threads.

Note that it is not safe to make assumptions about having consistent
contents of a ThreadPool for a running target process. As the target
process runs Thread objects may be inserted or removed from the
ThreadPool. It is generally safer to stop a Process before operating on
its ThreadPool. When used on a running process the ThreadPool iterator
methods guarantee that they will not return invalid Thread objects (e.g,
nothing that would lead to a segfault), but they do not guarantee that
the Thread objects will refer to live threads or that they return all
Threads.

ThreadPool Declared In:

.. container:: Definition

   PCProcess.h

ThreadPool Types:

.. container:: Definition

   class iterator {

.. container:: Definition

   public:

.. container:: Definition

   iterator();

.. container:: Definition

   ~iterator();

.. container:: Definition

   Thread::ptr operator*() const;

.. container:: Definition

   bool operator==(const iterator &i);

.. container:: Definition

   bool operator!=(const iterator &i);

.. container:: Definition

   ThreadPool::iterator operator++();

.. container:: Definition

   ThreadPool::iterator operator++(int);

.. container:: Definition

   };

.. container:: Definition

   class const_iterator {

.. container:: Definition

   public:

.. container:: Definition

   const_iterator();

.. container:: Definition

   ~const_iterator();

.. container:: Definition

   Thread::const_ptr operator*() const;

.. container:: Definition

   bool operator==(const const_interator &i);

.. container:: Definition

   bool operator!=(const const_iterator &i);

.. container:: Definition

   ThreadPool::const_iterator operator++();

.. container:: Definition

   ThreadPool::const_iterator operator++(int);

.. container:: Definition

   };

The iterator and const_iterator types of ThreadPool are respectively C++
iterators and const iterators over the set of threads represented by the
ThreadPool. The behavior of operator*, operator==, operator!=,
operator++, and operator++(int) match the standard behavior of C++
iterators.

ThreadPool Member Functions:

.. container:: Definition

   ThreadPool::iterator begin()

.. container:: Definition

   ThreadPool::const_iterator begin() const

These functions respectively return an iterator and a const_iterator
that point to the beginning of the set of Thread objects.

.. container:: Definition

   ThreadPool::iterator end()

.. container:: Definition

   ThreadPool::const_iterator end() const

These functions respectively return an iterator and a const_iterator
that point to the iterator element after the end of the set of Thread
objects.

.. container:: Definition

   ThreadPool::iterator find(Dyninst::LWP lwp)

.. container:: Definition

   ThreadPool::const_iterator find(Dyninst::LWP lwp) const

The functions respectively return an iterator and a const_iterator that
points to the Thread with a LWP of lwp. If lwp is not found in the
thread list, then this function returns end().

.. container:: Definition

   size_t size() const

This function returns the number of Threads in the Process.

.. container:: Definition

   Process::ptr getProcess()

.. container:: Definition

   Process::const_ptr getProcess() const

These functions respectively return a pointer or a const pointer to the
Process that owns this ThreadPool.

.. container:: Definition

   Thread::ptr getInitialThread()

.. container:: Definition

   Thread::const_ptr getInitialThread() const

These functions respectively return a pointer or a const pointer to the
initial Thread in a Process. The initial thread is the thread that
started execution of the process (i.e., the thread that called main).

LibraryPool
-----------

A LibraryPool is a container representing the executable and set shared
libraries (e.g., .dll and .so libraries) loaded into the target process’
address space. A statically linked target process will only have a
single executable, while a dynamically linked target process will have
an executable and zero or more shared libraries.

The LibraryPool class contains iterators and search functions for
operating on the set of libraries.

LibraryPool Declared In:

.. container:: Definition

   PCProcess.h

LibraryPool Types:

.. container:: Definition

   class iterator {

.. container:: Definition

   public:

.. container:: Definition

   iterator();

.. container:: Definition

   ~iterator();

.. container:: Definition

   Library::ptr operator*() const;

.. container:: Definition

   bool operator==(const iterator &i);

.. container:: Definition

   bool operator!=(const iterator &i);

.. container:: Definition

   LibraryPool::iterator operator++();

.. container:: Definition

   LibraryPool::iterator operator++(int);

.. container:: Definition

   };

.. container:: Definition

   class const_iterator {

.. container:: Definition

   const_iterator();

.. container:: Definition

   ~const_iterator();

.. container:: Definition

   Library::const_ptr operator*() const;

.. container:: Definition

   bool operator==(const const_interator &i);

.. container:: Definition

   bool operator!=(const const_iterator &i);

.. container:: Definition

   LibraryPool::const_iterator operator++();

.. container:: Definition

   LibraryPool::const_iterator operator++(int);

.. container:: Definition

   };

The iterator and const_iterator types of LibraryPool are respectively
C++ iterators and const iterators over the set of libraries represented
by the LibraryPool. The behavior of operator*, operator==, operator!=,
operator++, and operator++(int) match the standard behavior of C++
iterators.

LibraryPool Member Functions:

.. container:: Definition

   LibraryPool::iterator begin()

.. container:: Definition

   LibraryPool::const_iterator begin() const

These functions respectively return an iterator and a const_iterator
that point to the beginning of the set of Library objects.

.. container:: Definition

   LibraryPool::iterator end()

.. container:: Definition

   LibraryPool::const_iterator end() const

These functions respectively return an iterator and a const_iterator
that point to the iterator element after the end of the set of Library
objects.

.. container:: Definition

   size_t size() const

This function returns the number of elements in the library set.

.. container:: Definition

   Library::ptr getExecutable()

.. container:: Definition

   Library::const_ptr getExecutable() const

These functions respectively return a pointer or a const pointer to the
Library object that represents the target process’ executable.

.. container:: Definition

   Library::ptr getLibraryByName(std::string name)

.. container:: Definition

   Library::const_ptr getLibraryByName(std::string name) const

These functions respectively return a pointer or a const pointer to the
Library object that with a file name equal to name. If no library is
found then these functions respectively return Library::ptr() or
Library::const_ptr().

RegisterPool
------------

The RegisterPool object represents a set of registers. It can be used to
get or set all registers in a Thread at once. See the
Thread::getAllRegisters and Thread::setAllRegisters functions. See
Appendix A. for more information about MachRegister and MachRegisterVal.

RegisterPool Declared In:

.. container:: Definition

   PCProcess.h

RegisterPool Types:

.. container:: Definition

   class iterator {

.. container:: Definition

   public:

.. container:: Definition

   iterator();

.. container:: Definition

   ~iterator();

.. container:: Definition

   std::pair<MachRegister, MachRegisterVal> operator*() const;

.. container:: Definition

   bool operator==(const iterator &i);

.. container:: Definition

   bool operator!=(const iterator &i);

.. container:: Definition

   RegisterPool::iterator operator++();

.. container:: Definition

   RegisterPool::iterator operator++(int);

.. container:: Definition

   };

This is a C++ iterator over the set of registers contained in the
registerPool. The behavior of operator*, operator==, operator!=,
operator++, and operator++(int) match the standard behavior of C++
iterators.

RegisterPool Member Functions:

.. container:: Definition

   LibraryPool::iterator begin()

This function returns an iterator that points to the beginning of the
set of registers.

.. container:: Definition

   LibraryPool::iterator end()

This function returns an iterator that points element after the end of
the set of registers.

.. container:: Definition

   LibraryPool::iterator find(Dyninst::MachRegister r)

This function returns an iterator that points to the element in the
register pool that equals register r. If r is not found, then this
function returns end().

.. container:: Definition

   size_t size() const

This function returns the number of elements in the register set.

.. container:: Definition

   Dyninst::MachRegisterVal& operator[](Dyninst::MachRegister r)

This function returns a reference to the value associated with the
register r in this register pool. It can be used to efficiently get and
set the values of registers in this pool, or to fill the pool with new
MachRegister objects.

AddressSet
----------

The AddressSet class is a set container of Process and Dyninst::Address
tuples, with each set element a std::pair<Address, Process::ptr>.
AddressSet is used by the ProcessSet and ThreadSet classes for
performing group operations on large numbers of processes. An AddressSet
might, for example, represent the location of a symbol across numerous
processes, or the location of a buffer in each process where data can be
written or read.

The iteration interfaces of AddressSet resemble a C++ STL
std::multimap<Address, Process::ptr>. When iterating all Addresses will
appear in sequential order from smallest to largest.

AddressSet Declared In:

.. container:: Definition

   ProcessSet.h

AddressSet Types:

.. container:: Definition

   AddressSet::ptr

.. container:: Definition

   AddressSet::const_ptr

The AddressSet::ptr and AddressSet::const_ptr respectively represent a
pointer and a const pointer to an AddressSet object. Both pointer types
are reference counted and will cause the underlying AddressSet object to
be cleaned when there are no more references.

.. container:: Definition

   typedef std::pair<Dyninst::Address, Process::ptr> value_type

.. container:: Definition

   class iterator {

.. container:: Definition

   public:

.. container:: Definition

   iterator();

.. container:: Definition

   ~iterator();

.. container:: Definition

   value_type operator*() const;

.. container:: Definition

   bool operator==(const iterator &i);

.. container:: Definition

   bool operator!=(const iterator &i);

.. container:: Definition

   AddressSet::iterator operator++();

.. container:: Definition

   AddressSet::iterator operator++(int);

.. container:: Definition

   };

.. container:: Definition

   class const_iterator {

.. container:: Definition

   public:

.. container:: Definition

   const_iterator();

.. container:: Definition

   ~const_iterator();

.. container:: Definition

   value_type operator*() const;

.. container:: Definition

   bool operator==(const const_interator &i);

.. container:: Definition

   bool operator!=(const const_iterator &i);

.. container:: Definition

   AddressSet::const_iterator operator++();

.. container:: Definition

   AddressSet::const_iterator operator++(int);

.. container:: Definition

   };

These are C++ iterators over the Address and Process pairs contained in
the AddressSet. The behavior of operator*, operator==, operator!=,
operator++, and operator++(int) match the standard behavior of C++
iterators.

AddressSet Static Member Functions:

.. container:: Definition

   static AddressSet::ptr newAddressSet()

This function returns a new AddressSet that is empty.

.. container:: Definition

   | static AddressSet::ptr newAddressSet(ProcessSet::const_ptr ps,
   | Dyninst::Address addr)

This function returns a new AddressSet initialized with the elements
from ps paired with the Address addr.

.. container:: Definition

   | static AddressSet::ptr newAddressSet(ProcessSet::const_ptr ps,
   | std::string library_name,
   | Dyninst::Offset off)

This function returns a new AddressSet initialized with the elements
from ps. The Address element for each process is calculated by looking
up the load address of library_name in each Process and adding it to
off.

AddressSet Member Functions

.. container:: Definition

   iterator begin()

.. container:: Definition

   const_iterator begin() const

These functions return an iterator that points to the first element in
the AddressSet, or end() if the AddressSet is empty.

.. container:: Definition

   iterator end()

.. container:: Definition

   const_iterator end() const

These functions return an iterator that points to the element that comes
after the final element in the AddressSet.

.. container:: Definition

   iterator find(Dyninst::Address addr)

.. container:: Definition

   const_iterator find(Dyninst::Address addr) const

These functions return an iterator that points to the first element in
the AddressSet with an address of addr. They return end() if no element
matches addr.

.. container:: Definition

   iterator find(Dyninst::Address addr, Process::const_ptr proc)

.. container:: Definition

   | const_iterator find(Dyninst::Address addr,
   | Process::const_ptr proc) const

These functions return an iterator that points to any element that has a
process and address of proc and addr. It returns end() if no element
matches.

.. container:: Definition

   size_t count(Dyninst::Address addr) const

This function returns the number of elements with address addr.

.. container:: Definition

   size_t size() const

This function returns the number of elements in the AddressSet.

.. container:: Definition

   bool empty() const

This function returns true if the AddressSet has zero elements and false
otherwise.

.. container:: Definition

   | std::pair<iterator, bool> insert(Dyninst::Address addr,
   | Process::const_ptr proc)

This function inserts a new element into the AddressSet with addr and
proc as its values. If another element with those values already exists,
then no new element will be inserted. It returns an iterator that points
to the new or existing element and a boolean value that is true if a new
element was inserted and false otherwise.

.. container:: Definition

   size_t insert(Dyninst::Address addr, ProcessSet::const_ptr ps)

For every element in ps, this function inserts it and addr into the
AddressSet. It returns the number of new elements created.

.. container:: Definition

   void erase(iterator pos)

This function removes the element pointed to by pos from the AddressSet.

.. container:: Definition

   size_t erase(Process::const_ptr proc)

This function removes every element with a process of proc from the
AddressSet. It returns the number of elements removed.

.. container:: Definition

   size_t erase(Dyninst::Address addr, Process::const_ptr proc)

This function removes any element that has and address and process of
addr and proc from the AddressSet. It returns the number of elements
removed.

.. container:: Definition

   void clear()

This function erases all elements from the AddressSet leaving an
AddressSet of size zero.

.. container:: Definition

   iterator lower_bound(Dyninst::Address addr)

This function returns an iterator pointing to the first element in the
AddressSet that has an address greater than or equal to addr.

.. container:: Definition

   iterator upper_bound(Dyninst::Address addr)

This function returns an iterator pointing to the first element in the
AddressSet that has an address greater than addr.

.. container:: Definition

   std::pair<iterator, iterator> equal_range(Address addr) const

This function returns a pair of iterators. The first iterator has the
same value as the return of lower_bound(addr) and the second iterator
has the same value as the return of upper_bound(addr).

.. container:: Definition

   AddressSet::ptr set_union(AddressSet::const_ptr aset)

This function returns a new AddressSet whose elements are the set union
of this AddressSet and aset.

.. container:: Definition

   AddressSet::ptr set_intersection(AddressSet::const_ptr aset)

This function returns a new AddressSet whose elements are the set
intersection of this AddressSet and aset.

.. container:: Definition

   AddressSet::ptr set_difference(AddressSet::const_ptr aset)

This function returns a new AddressSet whose elements are the set
difference of this AddressSet minus aset.

ProcessSet
----------

The ProcessSet class is a set container for multiple Process objects. It
shares many of the same operations as the Process class, but when an
operation is performed on a ProcessSet it is done on every Process in
the ProcessSet. On some systems, such as Blue Gene/Q, a ProcessSet can
achieve better performance when repeating an operation across many
target processes.

ProcessSet Declared In:

.. container:: Definition

   ProcessSet.h

ProcessSet Types

.. container:: Definition

   ProcessSet::ptr

.. container:: Definition

   ProcessSet::const_ptr

The ptr and const_ptr types are smart pointers to a ProcessSet object.
When the last smart pointer to the ProcessSet is cleaned, then the
underlying ProcessSet is cleaned.

.. container:: Definition

   ProcessSet::weak_ptr

.. container:: Definition

   ProcessSet::const_weak_ptr

The weak_ptr and const_weak_ptr are weak smart pointers to a ProcessSet
object. Unlike regular smart pointers, weak pointers are not counted as
references when determining whether to clean the ProcessSet object.

.. container:: Definition

   struct CreateInfo {

.. container:: Definition

   std::string executable;

.. container:: Definition

   std::vector<std::string> argv;

.. container:: Definition

   std::vector<std::string> envp;

.. container:: Definition

   std::map<int, int> fds;

.. container:: Definition

   ProcControlAPI::err_t error_ret;

.. container:: Definition

   Process::ptr proc;

.. container:: Definition

   }

.. container:: Definition

   struct AttachInfo {

.. container:: Definition

   Dyninst::PID pid;

.. container:: Definition

   std::string executable;

.. container:: Definition

   ProcControlAPI::err_t error_ret;

.. container:: Definition

   Process::ptr proc;

.. container:: Definition

   }

The CreateInfo and AttachInfo types are used by the
ProcessSet::createProcessSet and ProcessSet::attachProcessSet functions
when creating groups of processes.

.. container:: Definition

   class iterator {

.. container:: Definition

   public:

.. container:: Definition

   iterator()

.. container:: Definition

   ~iterator()

.. container:: Definition

   Process::ptr operator*() const

.. container:: Definition

   bool operator==(const iterator &i) const

.. container:: Definition

   bool operator!=(const iterator &i) const

.. container:: Definition

   ProcessSet::iterator operator++();

.. container:: Definition

   ProcessSet::iterator operator++(int);

.. container:: Definition

   }

.. container:: Definition

   class const_iterator {

.. container:: Definition

   public:

.. container:: Definition

   const_iterator()

.. container:: Definition

   ~const_iterator()

.. container:: Definition

   Process::const_ptr operator*() const

.. container:: Definition

   bool operator==(const const_iterator &i) const

.. container:: Definition

   bool operator!=(const const_iterator &i) const

.. container:: Definition

   ProcessSet::const_iterator operator++();

.. container:: Definition

   ProcessSet::const_iterator operator++(int);

.. container:: Definition

   }

These are C++ iterators over the Process pointers contained in the
ProcessSet. The behavior of operator*, operator==, operator!=,
operator++, and operator++(int) match the standard behavior of C++
iterators.

.. container:: Definition

   struct write_t {

.. container:: Definition

   void \*buffer

.. container:: Definition

   Dyninst::Address addr

.. container:: Definition

   size_t size

.. container:: Definition

   err_t err

.. container:: Definition

   bool operator<(const write_t &w)

.. container:: Definition

   }

.. container:: Definition

   struct read_t {

.. container:: Definition

   Dyninst::Address addr

.. container:: Definition

   void \*buffer

.. container:: Definition

   size_t size

.. container:: Definition

   err_t err

.. container:: Definition

   bool operator<(const read_t &r)

.. container:: Definition

   }

The write_t and read_t types are used by ProcessSet::readMemory and
ProcessSet::writeMemory.

ProcessSet Static Member Functions

.. container:: Definition

   static ProcessSet::ptr newProcessSet()

This function creates a new ProcessSet that is empty.

.. container:: Definition

   static ProcessSet::ptr newProcessSet(Process::const_ptr proc)

This function creates a new ProcessSet containing proc.

.. container:: Definition

   static ProcessSet::ptr newProcessSet(ProcessSet::const_ptr pset)

This function creates a new ProcessSet that is a copy of pset.

.. container:: Definition

   | static ProcessSet::ptr newProcessSet(
   | const std::set<Process::const_ptr> &procs)

This function creates a new ProcessSet containing every element from
procs.

.. container:: Definition

   | static ProcessSet newProcessSet(AddressSet::const_iterator ab,
   | AddressSet::const_iterator ae)

This function creates a new ProcessSet containing the processes that are
found within [ab, ae) of an AddressSet.

.. container:: Definition

   | static ProcessSet::ptr createProcessSet(
   | std::vector<CreateInfo> &cinfo)

This function creates a new ProcessSet by launching new processes. Each
element in cinfo specifies an executable, arguments, environment and
file descriptor mappings (with similar semantics to
Process::createProcess), which are used to launch a new process.

Every successfully created Process will be added to a new ProcessSet
that is returned by this function.

In addition, the cinfo vector will be updated so that each entry’s proc
field points to the Process created by that entry, and the error_ret
entry will contain an error code for any process launch that failed.

.. container:: Definition

   | static ProcessSet::ptr attachProcessSet(
   | std::vector<AttachInfo> &ainfo)

This function creates a new ProcessSet by attaching to existing
processes. Each element in ainfo specifies a PID and executable (with
similar semantics to Process::attachProcess), which are used to attach
to the processes.

Every successfully attached Process will be added to a new ProcessSet
that is returned by this function.

In addition, the ainfo vector will be updated so that each entry’s proc
field points to the Process attached by that entry, and the error_ret
entry will contain an error code any process attach that failed.

ProcessSet Member Functions

.. container:: Definition

   ProcessSet::ptr set_union(ProcessSet::ptr pset) const

This function returns a new ProcessSet whose elements are a set union of
this ProcessSet and pset.

.. container:: Definition

   ProcessSet::ptr set_intersection(ProcessSet::ptr pset) const

This function returns a new ProcessSet whose elements are a set
intersection of this ProcessSet and pset.

.. container:: Definition

   ProcessSet::ptr set_difference(ProcessSet::ptr pset) const

This function returns a new ProcessSet whose elements are a set
difference of this ProcessSet minus pset.

.. container:: Definition

   iterator begin()

.. container:: Definition

   const_iterator begin() const

These functions return iterators to the first element in the ProcessSet.

.. container:: Definition

   iterator end()

.. container:: Definition

   const_iterator end() const

These functions return iterators that come after the last element in the
ProcessSet.

.. container:: Definition

   iterator find(Process::const_ptr proc)

.. container:: Definition

   const_iterator find(Process::const_ptr proc) const

These functions search a ProcessSet for the Process pointed to by proc
and returns an iterator that points to that element. It returns
ProcessSet::end() if no element is found.

.. container:: Definition

   iterator find(Dyninst::PID pid)

.. container:: Definition

   const_iterator find(Dyninst::PID pid) const

These functions search a ProcessSet for the Process pointed to by proc
and returns an iterator that points to that element. It returns
ProcessSet::end() if no element is found.

.. container:: Definition

   bool empty() const

This function returns true if the ProcessSet has zero elements, false
otherwise.

.. container:: Definition

   size_t size() const

This function returns the number of elements in the ProcessSet.

.. container:: Definition

   std::pair<iterator, bool> insert(Process::const_ptr proc)

This function inserts proc into the ProcessSet. If proc already exists
in the ProcessSet, then no change will occur. This function returns an
iterator pointing to either the new or existing element and a boolean
that is true if an insert happened and false otherwise.

.. container:: Definition

   void erase(iterator pos)

This function removes the element pointed to by pos from the ProcessSet.

.. container:: Definition

   size_t erase(Process::const_ptr proc)

This function searches the ProcessSet for proc, then erases that element
from the ProcessSet. It returns 1 if it erased an element and 0
otherwise.

.. container:: Definition

   void clear()

This function erases all elements in the ProcessSet.

.. container:: Definition

   ProcessSet::ptr getErrorSubset() const

This function returns a new ProcessSet containing every Process from
this ProcessSet that has a non-zero error code. Error codes are reset
upon every ProcessSet API call, so this function shows which Processes
had an error on the last ProcessSet operation.

.. container:: Definition

   void getErrorSubsets(std::map<ProcControlAPI::err_t, ProcessSet::ptr>
   &err_sets) const

This function returns a set of new ProcessSets containing every Process
from this ProcessSet that has non-zero error codes, and grouped by error
code. For each error code generated by the last ProcessSet API operation
an element will be added to err_sets, and every Process that has the
same error code will be added to the new ProcessSet associated with that
error code.

.. container:: Definition

   bool anyTerminated() const;

.. container:: Definition

   bool allTerminated() const;

These functions respectively return true if any or all processes in this
ProcessSet are terminated, and false otherwise.

.. container:: Definition

   bool anyExited() const;

.. container:: Definition

   bool allExited() const;

These functions respectively return true if any or all processes in this
ProcessSet have exited normally, and false otherwise.

.. container:: Definition

   bool anyCrashed() const

.. container:: Definition

   bool allCrashed() const

These functions respectively return true if any or all processes in this
ProcessSet have crashed normally, and false otherwise.

.. container:: Definition

   bool anyDetached();

.. container:: Definition

   bool allDetached();

These functions respectively return true if any or all processes in this
ProcessSet have been detached, and false otherwise.

.. container:: Definition

   bool anyThreadStopped();

.. container:: Definition

   bool allThreadStopped();

These functions respectively return true if any or all threads in this
ProcessSet are stopped, and false otherwise.

.. container:: Definition

   bool anyThreadRunning();

.. container:: Definition

   bool allThreadRunning();

These functions respectively return true if any or all threads in this
ProcessSet are running, and false otherwise.

.. container:: Definition

   ProcessSet::ptr getTerminatedSubset() const

This function returns a new ProcessSet, which is a subset of this
ProcessSet, and contains every Process that is terminated.

.. container:: Definition

   ProcessSet::ptr getExitedSubset() const

This function returns a new ProcessSet, which is a subset of this
ProcessSet, and contains every Process that has exited normally.

.. container:: Definition

   ProcessSet::ptr getCrashedSubset() const

This function returns a new ProcessSet, which is a subset of this
ProcessSet, and contains every Process that has crashed.

.. container:: Definition

   ProcessSet::ptr getDetachedSubset() const

This function returns a new ProcessSet, which is a subset of this
ProcessSet, and contains every Process that is detached.

.. container:: Definition

   ProcessSet::ptr getAllThreadRunningSubset() const

.. container:: Definition

   ProcessSet::ptr getAnyThreadRunningSubset() const

This function returns a new ProcessSet, which is a subset of this
ProcessSet, and contains every Process that respectively has any or all
threads running.

.. container:: Definition

   ProcessSet::ptr getAllThreadStoppedSubset() const

.. container:: Definition

   ProcessSet::ptr getAnyThreadStoppedSubset() const

This function returns a new ProcessSet, which is a subset of this
ProcessSet, and contains every Process that respectively has any or all
threads stopped.

.. container:: Definition

   bool continueProcs()

This function continues every thread in every process of this
ProcessSet, similar to Process::continueProc. It returns true if every
process was successfully continued and false otherwise.

.. container:: Definition

   bool stopProcs()

This function stops every thread in every process of this ProcessSet,
similar to Process::stopProc. It returns true if every process was
successfully stopped and false otherwise.

.. container:: Definition

   bool detach(bool leaveStopped = true)

This function detaches from every process in this ProcessSet, similar to
Process::detach. It returns true if every process detach was successful
and false otherwise.

If the leaveStopped parameter is set to true, and the processes in this
ProcessSet are stopped, then the processes will be left in a stopped
state after the detach.

.. container:: Definition

   bool terminate()

This function terminates every process in this ProcessSet, similar to
Process::terminate. It returns true if every process was successfully
terminated and false otherwise.

.. container:: Definition

   bool temporaryDetach()

This function does a temporary detach from every process in this
ProcessSet, similar to Process::temporaryDetach. It returns true if
every process was successfully detached and false otherwise.

.. container:: Definition

   bool reAttach()

This function reattaches to every process in this ProcessSet, similar to
Process::reAttach. It returns true if every process was successfully
reAttached and false otherwise.

.. container:: Definition

   AddressSet::ptr mallocMemory(size_t sz) const

This function allocates a block of memory of size sz in each process in
this ProcessSet. The addresses of the allocations are returned in a new
AddressSet object.

.. container:: Definition

   bool mallocMemory(size_t size, AddressSet::ptr location)

This function allocates a block of memory of size sz in each process in
this ProcessSet. The memory will be allocated in each process based on
the Process/Address pairs in location.

This function’s behavior is undefined if location contains processes not
included in this ProcessSet.

This function returns true if every allocation happened without error
and false otherwise.

.. container:: Definition

   bool freeMemory(AddressSet::ptr addrs) const

This function frees memory allocated by Process::mallocMemory or
ProcessSet::mallocMemory. The AddressSet addrs should contain a list of
Process/Address pairs that point to the memory that should be freed.

This function’s behavior is undefined if addrs contains processes not
included in this ProcessSet.

This function returns true if every free happened without error and
false otherwise.

.. container:: Definition

   | bool readMemory(AddressSet::ptr addrs,
   | std::multimap<Process::ptr, void \*> &result,

.. container:: Definition

   size_t size) const

This function reads memory from processes in this ProcessSet. addrs
should contain the addresses to read memory from. size should be the
amount of memory read from each process. The results of the memory reads
will be returned by filling in the result multimap. Each process that is
read from will have an entry in result along with a malloc allocated
buffer containing the results of the read.

It is the ProcControlAPI user’s responsibility to free the memory
buffers returned by this function.

This function’s behavior is undefined if addrs contains processes not
included in this ProcessSet.

This function returns true if every read happened without error, and
false otherwise.

.. container:: Definition

   | bool readMemory(AddressSet::ptr addrs,
   | std::map<void \*, ProcessSet::ptr> &result,
   | size_t size)

This function reads memory from processes in this ProcessSet. addrs
should contain the addresses to read memory from. size should be the
amount of memory to read from each process. The results of the memory
reads will be aggregated together into the result map. If any two
processes read equivalent byte-for-byte data, then those processes are
grouped together in a new ProcessSet associated with a common malloc
allocated buffer containing their memory contents.

It is the ProcControlAPI user’s responsibility to free the memory
buffers returned by this function.

This function’s behavior is undefined if addrs contains processes not
included in this ProcessSet.

This function returns true if every read happened without error, and
false otherwise.

.. container:: Definition

   bool readMemory(std::multimap<Process::const_ptr, read_t> &addr)

This function reads memory from processes in this ProcessSet. The
processes to read from are specified in the indexes of addr. The remote
address, read size and local buffer are specified in the read_t elements
of addr.

This function’s behavior is undefined if addr contains processes not
included in this ProcessSet.

This function returns true if every read happened without error, and
false otherwise. If any read results in an error, then the error_ret
field of the associated addr element will be set.

.. container:: Definition

   | bool writMemory(AddressSet::ptr addrs,
   | const void \*buffer,
   | size_t sz) const

This function will write the contents of buffer of size sz into the
memory of each process at addrs.

This function’s behavior is undefined if addrs contains processes not
included in this ProcessSet.

This function returns true if every write happened without error, and
false otherwise.

.. container:: Definition

   | bool writeMemory(
   | std::multimap<Process::const_ptr, write_t> &addrs) const

This function writes to the memory of each process in addrs. The
processes to write to are specified as the indexes of addrs. The local
memory buffer, buffer size, and target location are specified in the
write_t element of addrs.

This function’s behavior is undefined if addrs contains processes not
included in this ProcessSet.

This function returns true if every write happened without error, and
false otherwise. If any write results in an error, then the error_ret
field of the associated addr element will be set.

.. container:: Definition

   bool addBreakpoint(AddressSet::ptr as, Breakpoint::ptr bp) const

This function inserts the Breakpoint bp into every process and address
specified by as. It is similar to Process::addBreakpoint.

This function’s behavior is undefined if addrs contains processes not
included in this ProcessSet.

This function returns true if every breakpoint add happened without
error, and false otherwise.

.. container:: Definition

   bool rmBreakpoint(AddressSet::ptr as, Breakpoint::ptr bp) const

The function removes the Breakpoint bp from each process at the
locations specified in as. It is similar to Process::rmBreakpoint.

This function’s behavior is undefined if as contains processes not
included in this ProcessSet.

This function returns true if every breakpoint remove happened without
error, and false otherwise.

.. container:: Definition

   bool postIRPC(const std::multimap<Process::const_ptr, IRPC::ptr>
   &rpcs) const

This function posts the IRPC objects specified in rpcs to their
associated processes in the multimap. It is similar to
Process::postIRPC.

This function’s behavior is undefined if rpcs contains processes not
included in this ProcessSet.

This function returns true if every post happened without error, and
false otherwise.

.. container:: Definition

   | bool postIRPC(IRPC::ptr irpc,
   | std::multimap<Process::ptr, IRPC::ptr> \*result = NULL)

This function makes a copy of irpc for each Process in this ProcessSet
and posts it to that Process. If result is non-NULL, then the multimap
will be filled with each newly created IRPC and the Process to which it
was posted. It is similar to Process::postIRPC.

This function returns true if every post happened without error, and
false otherwise.

.. container:: Definition

   | bool postIRPC(IRPC::ptr irpc
   | AddressSet::ptr addrs,
   | std::multimap<Process::ptr, IRPC::ptr> \*result = NULL)

This function makes a copy of irpc and posts it to each Process in addrs
at the given Address. If result is non-NULL, then the multimap will be
filled with each newly created IRPC and the Process to which it was
posted. It is similar to Process::postIRPC.

This function’s behavior is undefined if rpcs contains processes not
included in this ProcessSet.

This function returns true if every post happened without error, and
false otherwise.

ThreadSet
---------

The ThreadSet class is a set container for Thread pointers. It has
similar operations as Thread, and operations done on a ThreadSet affect
every Thread in that ThreadSet. One some system, such as Blue Gene Q,
using a ThreadSet is more efficient when doing the same operation across
a large number of Threads.

ThreadSet Declared In:

ProcessSet.h

ThreadSet Types:

.. container:: Definition

   ThreadSet::ptr

.. container:: Definition

   ThreadSet::const_ptr

The ptr and const_ptr types are smart pointers to a ThreadSet object.
When the last smart pointer to the ThreadSet is cleaned, then the
underlying ThreadSet is cleaned. The const_ptr type is a const smart
pointer.

.. container:: Definition

   ThreadSet::weak_ptr

.. container:: Definition

   ThreadSet::const_weak_ptr

The weak_ptr and const_weak_ptr are weak smart pointers to a ThreadSet
object. Unlike regular smart pointers, weak pointers are not counted as
references when determining whether to clean the ThreadSet object. The
const_weak_ptr type is a const weak smart pointer.

.. container:: Definition

   class iterator {

.. container:: Definition

   public:

.. container:: Definition

   iterator()

.. container:: Definition

   ~iterator()

.. container:: Definition

   Thread::ptr operator*() const

.. container:: Definition

   bool operator==(const iterator &i) const

.. container:: Definition

   bool operator!=(const iterator &i) const

.. container:: Definition

   ThreadSet::iterator operator++();

.. container:: Definition

   ThreadSet::iterator operator++(int);

.. container:: Definition

   }

.. container:: Definition

   class const_iterator {

.. container:: Definition

   public:

.. container:: Definition

   const_iterator()

.. container:: Definition

   ~const_iterator()

.. container:: Definition

   Thread::const_ptr operator*() const

.. container:: Definition

   bool operator==(const const_iterator &i) const

.. container:: Definition

   bool operator!=(const const_iterator &i) const

.. container:: Definition

   ThreadSet::const_iterator operator++();

.. container:: Definition

   ThreadSet::const_iterator operator++(int);

.. container:: Definition

   }

These are C++ iterators over the Thread pointers contained in the
ThreadSet. The behavior of operator*, operator==, operator!=,
operator++, and operator++(int) match the standard behavior of C++
iterators.

ThreadSet Static Member Functions

.. container:: Definition

   static ThreadSet::ptr newThreadSet()

This function creates a new ThreadSet that is empty.

.. container:: Definition

   static ThreadSet::ptr newThreadSet(Thread::ptr thr)

This function creates a new ThreadSet that contains thr.

.. container:: Definition

   static ThreadSet::ptr newThreadSet(const ThreadPool &threadp)

This function creates a new ThreadSet that contains all of the Threads
currently in threadp.

.. container:: Definition

   | static ThreadSet::ptr newThreadSet (
   | const std::set<Thread::const_ptr> &thrds)

This function creates a new ThreadSet that contains all of the threads
in thrds.

.. container:: Definition

   static ThreadSet::ptr newThreadSet(ProcessSet::ptr pset)

This function creates a new ThreadSet that contains every live thread
currently in every process in pset.

ThreadSet Member Functions

.. container:: Definition

   ThreadSet::ptr set_union(ThreadSet::ptr tset) const

This function returns a new ThreadSet whose elements are a set union of
this ThreadSet and tset.

.. container:: Definition

   ThreadSet::ptr set_intersection(ThreadSet::ptr tset) const

This function returns a new ThreadSet whose elements are a set
intersection of this ThreadSet and tset.

.. container:: Definition

   ThreadSet::ptr set_difference(ThreadSet::ptr tset) const

This function returns a new ThreadSet whose elements are a set
difference of this ThreadSet minus tset.

.. container:: Definition

   iterator begin()

.. container:: Definition

   const_iterator begin() const

These functions return iterators to the first element in the ThreadSet.

.. container:: Definition

   iterator end()

.. container:: Definition

   const_iterator end() const

These functions return iterators that come after the last element in the
ThreadSet.

.. container:: Definition

   iterator find(Thread::const_ptr thr)

.. container:: Definition

   const_iterator find(Thread::const_ptr thr) const

These functions search a ThreadSet for thr and returns an iterator
pointing to that element. It returns ThreadSet::end() if no element is
found

.. container:: Definition

   bool empty() const

This function returns true if the ThreadSet has zero elements and false
otherwise.

.. container:: Definition

   size_t size() const

This function returns the number of elements in the ThreadSet.

.. container:: Definition

   std::pair<iterator, bool> insert(Thread::const_ptr thr)

This function inserts thr into the ThreadSet. If thr already exists in
the ThreadSet, then no change will occur. This function returns an
iterator pointing to either the new or existing element and a boolean
that is true if an insert happened and false otherwise.

.. container:: Definition

   void erase(iterator pos)

This function removes the element pointed to by pos from the ThreadSet.

.. container:: Definition

   size_t erase(Thread::const_ptr thr)

This function searches the ThreadSet for thr, then erases that element
from the ThreadSet. It returns 1 if it erased an element and 0
otherwise.

.. container:: Definition

   void clear()

This function erases all elements in the ThreadSet.

.. container:: Definition

   ThreadSet::ptr getErrorSubset() const

This function returns a new ThreadSet containing every Thread from this
ThreadSet that has a non-zero error code. Error codes are reset upon
every ThreadSet API call, so this function shows which Threads had an
error on the last ThreadSet operation.

.. container:: Definition

   | void getErrorSubsets(
   | std::map<ProcControlAPI::err_t, ThreadSet::ptr> &err) const

This function returns a set of new ThreadSets containing every Thread
from this ThreadSet that has non-zero error codes, and grouped by error
code. For each error code generated by the last ThreadSet API operation
an element will be added to err, and every Thread that has that error
code will be added to the new ThreadSet associated with that error code.

.. container:: Definition

   bool allStopped() const

.. container:: Definition

   bool anyStopped() const

These functions respectively return true if any or all threads in this
ThreadSet are stopped and false otherwise.

.. container:: Definition

   bool allRunning() const

.. container:: Definition

   bool anyRunning() const

These functions respectively return true if any or all threads in this
ThreadSet are running and false otherwise.

.. container:: Definition

   bool allTerminated() const

.. container:: Definition

   bool anyTerminated() const

These functions respectively return true if any or all threads in this
ThreadSet are terminated and false otherwise.

.. container:: Definition

   bool allSingleStepMode() const

.. container:: Definition

   bool anySingleStepMode() const

These functions respectively return true if any or all threads in this
ThreadSet are running in single step mode, and false otherwise.

.. container:: Definition

   bool allHaveUserThreadInfo() const

.. container:: Definition

   bool anyHaveUserThreadInfo() const

These functions respectively return true if any or all threads in this
ThreadSet have user thread information available and false otherwise.

.. container:: Definition

   ThreadSet::ptr getStoppedSubset() const

This function returns a new ThreadSet, which is a subset of this
ThreadSet, and contains every Thread that is stopped.

.. container:: Definition

   ThreadSet::ptr getRunningSubset() const

This function returns a new ThreadSet, which is a subset of this
ThreadSet, and contains every Thread that is running.

.. container:: Definition

   ThreadSet::ptr getTerminatedSubset() const

This function returns a new ThreadSet, which is a subset of this
ThreadSet, and contains every Thread that is terminated.

.. container:: Definition

   ThreadSet::ptr getSingleStepSubset() const

This function returns a new ThreadSet, which is a subset of this
ThreadSet, and contains every Thread that is in single step mode.

.. container:: Definition

   ThreadSet::ptr getHaveUserThreadInfoSubset() const

This function returns a new ThreadSet, which is a subset of this
ThreadSet, and contains every Thread that has user thread information
available.

.. container:: Definition

   bool getStartFunctions(AddressSet::ptr result) const

This function fills in the AddressSet pointed to by result with the
address of every start function of each Thread in this ThreadSet. This
information is only available on threads that have user thread
information available.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   bool getStackBases(AddressSet::ptr result) const

This function fills in the AddressSet pointed to by result with the
address of every stack base of each Thread in this ThreadSet. This
information is only available on threads that have user thread
information available.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   bool getTLSs(AddressSet::ptr result) const

This function fills in the AddressSet pointed to by result with the
address of every thread-local storage region of each Thread in this
ThreadSet. This information is only available on threads that have user
thread information available.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   bool stopThreads() const

This function stops every Thread in this ThreadSet. It is similar to
Thread::stopThread.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   bool continueThreads() const

This function stops every Thread in this ThreadSet. It is similar to
Thread::continueThread.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   bool setSingleStepMode(bool v) const

This function puts every Thread in this ThreadSet into single step mode
if v is true. It clears single step mode if v is false. It is similar to
Thread::setSingleStepMode.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   | bool getRegister(Dyninst::MachRegister reg,
   | std::map<Thread::ptr, Dyninst::MachRegisterVal> &res) const

This function gets the value of register reg in every Thread in this
ThreadSet. The collected values are returned in the res map, with each
Thread mapped to the value of reg in that thread. It is similar to
Thread::getRegister.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   bool getRegister(Dyninst::MachRegister reg,

.. container:: Definition

      | std::map<Dyninst::MachRegisterVal, ThreadSet::ptr> &res)
      | const

This function gets the value of register reg in every Thread in this
ThreadSet and then aggregates all identical values together. The res map
will contain an entry for each unique register value, and map that value
to a new ThreadSet that contains every Thread that produced that
register value. It is similar to Thread::getRegister.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   bool setRegister(Dyninst::MachRegister reg,

.. container:: Definition

   | const std::map<ThreadSet::const_ptr,
   | Dyninst::MachRegisterVal> &vals) const

This function sets the value of register reg in each Thread in this
ThreadSet. The value set in each thread is looked up in the vals map. It
is similar to Thread::setRegister.

This function’s behavior is undefined if it is passed a Thread that is
not in this ThreadSet.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   | bool setRegister(Dyninst::MachRegister reg,
   | Dyninst::MachRegisterVal val) const

This function sets the register reg to val in each Thread in this
ThreadSet. It is similar to Thread::setRegister.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   | bool getAllRegisters(
   | std::map<Thread::ptr, RegisterPool> &results) const

This function gets the values of every register in each Thread in this
ThreadSet. The register values are returned as RegisterPools in the
results map, with each Thread mapped to its RegisterPool. It is similar
to Thread::getAllRegisters.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   | bool setAllRegisters(
   | const std::map<Thread::const_ptr, RegisterPool> &val) const

This function sets the values of every register in each Thread in this
ThreadSet. The register values are extracted from the val map, with each
Thread specifying its register values via the map. This function is
similar to Thread::setAllRegisters.

This function’s behavior is undefined if it is passed a Thread that is
not in this ThreadSet.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   | bool postIRPC(const std::multimap<Thread::const_ptr,
   | IRPC::ptr> &rpcs) const

This function posts an IRPC to every Thread in this ThreadSet. The IRPC
to post to each Thread is specified in the rpcs multimap. This function
is similar to Thread::postIRPC.

This function return true if it succeeded for every Thread, and false
otherwise.

.. container:: Definition

   | bool postIRPC(IRPC::ptr irpc,
   | std::multimap<Thread::ptr, IRPC::ptr> \*result = NULL)

This function posts a copy of irpc to every Thread in this ThreadSet. If
result is non-NULL, then the new IRPC objects are returned in the result
multimap, with the Thread mapped to the IRPC that was posted there. This
function is similar to Thread::postIRPC.

This function return true if it succeeded for every Thread, and false
otherwise.

EventNotify
-----------

The EventNotify class is used to notify the user when ProcControlAPI is
ready to deliver a callback function. EventNotify is a singleton class,
which means only one instance of it is ever instantiated. See Section
2.2.3. for a high level description of notification.

EventNotify Declared In:

.. container:: Definition

   PCProcess.h

EventNotify Types:

.. container:: Definition

   typedef void (*notify_cb_t)()

This function signature is used for light-weight notification callback.

EventNotify Related Global Functions:

.. container:: Definition

   EventNotify \*evNotify()

This function returns the singleton instance of the EventNotify class.

EventNotify Member Functions:

.. container:: Definition

   int getFD()

This function returns a file descriptor. ProcControlAPI will write a
byte that will be available for reading on this file descriptor when a
callback function is ready to be invoked. Upon seeing that a byte has
been written to this file descriptor (likely via select or poll) the
user should call the Process::handleEvents function. The user should
never actually read the byte from this file descriptor; ProcControlAPI
will handle clearing the byte after the callback function is invoked.

This function returns -1 on error. Upon an error a subsequent call to
getLastError returns details on the error.

.. container:: Definition

   void registerCB(notify_cb_t cb)

This function registers a light-weight callback function that will be
invoked when a ProcControlAPI wishes to notify the user when a callback
function is ready to be invoked. This light-weight callback may be
called by a ProcControlAPI internal thread or from a signal handler; the
user is encouraged to keep its implementation appropriately safe for
these circumstances.

.. container:: Definition

   void removeCB(notify_cb_t cb)

This function removes a light-weight callback that was previously
registered with EventNotify::registerCB. ProcControlAPI will no longer
invoke the cb function after this function completes.

EventType
---------

The EventType class represents a type of event. Each instance of an
Event happening has one associated EventType, and callback functions can
be registered against EventTypes. All EventTypes have an associate
code—an integral value that identifies the EventType. Some EventTypes
also have a time associated with them (Pre, Post, or None)—describing
when an Event may occur relative to the Code. For example, an EventType
with a code of Exit and a time of Pre (written as pre-exit) would be
associated with an Event that occurs just before a process exits and its
address space is cleaned. An EventType with code Exit and a time of Post
would be associated with an Event that occurs after the process exits
and the address space is cleaned.

When using EventTypes to register for callback functions a special time
value of Any can be used. This signifies that the callback function
should trigger for both Pre and Post time events. ProcControlAPI will
never deliver an Event that has an EventType with time code Any.

More details on Events and EventTypes can be found in Section 2.2.1..

EventType Types:

.. container:: Definition

   typedef enum {

.. container:: Definition

   Pre = 0,

.. container:: Definition

   Post,

.. container:: Definition

   None,

.. container:: Definition

   Any

.. container:: Definition

   } Time;

.. container:: Definition

   typedef int Code;

The Time and Code types are respectively used to describe the time and
code values of an EventType.

EventType Constants:

.. container:: Definition

   static const int Error = -1

.. container:: Definition

   static const int Unset = 0

.. container:: Definition

   static const int Exit = 1

.. container:: Definition

   static const int Crash = 2

.. container:: Definition

   static const int Fork = 3

.. container:: Definition

   static const int Exec = 4

.. container:: Definition

   static const int ThreadCreate = 5

.. container:: Definition

   static const int ThreadDestroy = 6

.. container:: Definition

   static const int Stop = 7

.. container:: Definition

   static const int Signal = 8

.. container:: Definition

   static const int LibraryLoad = 9

.. container:: Definition

   static const int LibraryUnload = 10

.. container:: Definition

   static const int Bootstrap = 11

.. container:: Definition

   static const int Breakpoint = 12

.. container:: Definition

   static const int RPC = 13

.. container:: Definition

   static const int SingleStep = 14

.. container:: Definition

   static const int Library = 15

.. container:: Definition

   static const int MaxProcCtrlEvent = 1000

These constants describe possible values for an EventType’s code. The
Error and Unset codes are for handling error cases and should not be
used for callback functions or be associated with Events.

The EventType codes were implemented as an integer (rather than an enum)
to allow users to create custom EventTypes. Any custom EventType should
begin at the MaxProcCtrlEvent value, all smaller values are reserved by
ProcControlAPI.

EventType Related Types:

.. container:: Definition

   struct eventtype_cmp {

.. container:: Definition

   bool operator()(const EventType &a, const EventType &b);

.. container:: Definition

   }

This type defines a less-than comparison function for EventTypes. While
a comparison of EventTypes does not have a semantic meaning, this can be
useful for inserting EventTypes into maps or other STL data structures.

EventType Member Functions:

.. container:: Definition

   EventType(Code e)

Constructs an EventType with the given code and a time of Any.

.. container:: Definition

   EventType(Time t, Code e)

Constructs an EventType with the given time and code values.

.. container:: Definition

   EventType()

Constructs an EventType with an Unset code and None time value.

.. container:: Definition

   Code code() const

Returns the code value of the EventType.

.. container:: Definition

   Time time() const

Returns the time value of the EventType.

.. container:: Definition

   std::string name() const

Returns a human readable name for this EventType.

Event
-----

The Event class represents an instance of an event happening. Each Event
has an EventType that describes the event and pointers to the Process
and Thread that the event occurred on.

The Event class is an abstract class that is never instantiated.
Instead, ProcControlAPI will instantiate children of the Event class,
each of which add information specific to the EventType. For example, an
Event representing a thread creation will have an EventType of
ThreadCreate and can be cast into an EventNewThread for specific
information about the new thread. The specific events that are
instantiated from Event are described in the Section 3.15..

An event that occurs on a running thread may cause the process, thread,
or neither to stop running until the event has been handled. The
specifics of what is stopped can change between different event types
and operating systems. Each Event describes whether it stopped the
associated process or thread with a SyncType field. The values of this
field can be async (the event stopped neither the process nor thread),
sync_thread (the event stopped its thread), or sync_process (the event
stopped all threads in the process). A callback function can choose how
to resume or stop a process or thread using its return value (see
Section 2.2.2.).

More details on Event can be found in Section 2.2.1..

Event Declared In:

.. container:: Definition

   Event.h

Event Types:

.. container:: Definition

   typedef enum {

.. container:: Definition

   unset,

.. container:: Definition

   async,

.. container:: Definition

   sync_thread,

.. container:: Definition

   | sync_process
   | } SyncType

The SyncType type is used to describe how a process or thread is stopped
by an Event. See the above explanation for more details.

Event Member Functions:

.. container:: Definition

   Thread::const_ptr getThread() const

This function returns a const pointer to the Thread object that
represents the thread this event occurred on.

.. container:: Definition

   Process::const_ptr getProcess() const

This function returns a const pointer to the Process object that
represents the process this event occurred on.

.. container:: Definition

   EventType getEventType() const

This function returns the EventType associated with this Event.

.. container:: Definition

   SyncType getSyncType() const

This function returns the SyncType associated with this Event.

.. container:: Definition

   std::string name() const

This function returns a human readable name for this Event.

.. container:: Definition

   EventTerminate::ptr getEventTerminate()

.. container:: Definition

   EventTerminate::const_ptr getEventTerminate() const

.. container:: Definition

   EventExit::ptr getEventExit()

.. container:: Definition

   EventExit::const_ptr getEventExit() const

.. container:: Definition

   EventCrash::ptr getEventCrash()

.. container:: Definition

   EventCrash::const_ptr getEventCrash() const

.. container:: Definition

   EventForceTerminate::ptr getEventForceTerminate()

.. container:: Definition

   EventForceTerminate::const_ptr getEventForceTerminate() const

.. container:: Definition

   EventExec::ptr getEventExec()

.. container:: Definition

   EventExec::const_ptr getEventExec() const

.. container:: Definition

   EventStop::ptr getEventStop()

.. container:: Definition

   EventStop::const_ptr getEventStop() const

.. container:: Definition

   EventBreakpoint::ptr getEventBreakpoint()

.. container:: Definition

   EventBreakpoint::const_ptr getEventBreakpoint() const

.. container:: Definition

   EventNewThread::ptr getEventNewThread()

.. container:: Definition

   EventNewThread::const_ptr getEventNewThread() const

.. container:: Definition

   EventNewUserThread::ptr getEventNewUserThread()

.. container:: Definition

   EventNewUserThread::const_tr getEventNewUserThread() const

.. container:: Definition

   EventNewLWP::ptr getEventNewLWP()

.. container:: Definition

   EventNewLWP::const_ptr getEventNewLWP() const

.. container:: Definition

   EventThreadDestroy::ptr getEventThreadDestroy()

.. container:: Definition

   EventThreadDestroy::const_ptr getEventThreadDestroy() const

.. container:: Definition

   EventUserThreadDestroy::ptr getEventUserThreadDestroy()

.. container:: Definition

   EventUserThreadDestroy::const_ptr getEventUserThreadDestroy() const

.. container:: Definition

   EventLWPDestroy::ptr getEventLWPDestroy()

.. container:: Definition

   EventLWPDestroy::const_ptr getEventLWPDestroy() const

.. container:: Definition

   EventFork::ptr getEventFork()

.. container:: Definition

   EventFork::const_ptr getEventFor() const

.. container:: Definition

   EventSignal::ptr getEventSignal()

.. container:: Definition

   EventSignal::const_ptr getEventSignal() const

.. container:: Definition

   EventRPC::ptr getEventRPC()

.. container:: Definition

   EventRPC::const_ptr getEventRPC() const

.. container:: Definition

   EventSingleStep::ptr getEventSingleStep()

.. container:: Definition

   EventSingleStep::const_ptr getEventSingleStep() const

.. container:: Definition

   EventLibrary::ptr getEventLibrary()

.. container:: Definition

   EventLibrary::const_ptr getEventLibrary() const

These functions serve as a form of dynamic_cast. They cast the Event
into a child type and return the result of that cast. If the Event
object is not of the appropriate type for the given function, then they
return a shared pointer NULL equivalent (ptr() or const_ptr()).

For example, if an Event was an instance of an EventRPC, then the
getEventRPC() function would cast it to EventRPC and return the
resulting value.

Event Child Classes
-------------------

The Event class is an abstract parent class, while the classes listed in
this section are the child classes that are actually instantiated. Given
an Event object passed to a callback function, a ProcControlAPI user can
inspect the Event’s EventType and cast it to the appropriate child class
listed below.

Note that each child class inherits the member functions described in
the Event class in Section 3.14..

Common Types:

.. container:: Definition

   <EventChildClassHere>::ptr

.. container:: Definition

   <EventChildClassHere>::const_ptr

These types are common to all Event children classes. Rather than repeat
them for each class, they are listed once here for brevity.

The ptr and const_ptr respectively represent a pointer and a const
pointer to an Event child class. Both pointer types are reference
counted and will cause the underlying object will be cleaned when there
are no more references.

EventTerminate
~~~~~~~~~~~~~~

The EventTerminate class is a parent class for EventExit and EventCrash.
It is never instantiated by ProcControlAPI and simply serves as a
place-holder type for a user to deal with process termination without
dealing with the specifics of whether a process exited properly or
crashed.

Associated EventType Codes:

.. container:: Definition

   Exit, Crash and ForceTerminate

EventExit
~~~~~~~~~

An EventExit triggers when a process performs a normal exit (e.g.,
calling the exit function or returning from main). The process that
exited is referenced with Event’s getProcess function.

An EventExit may be associated with an EventType of pre-exit or
post-exit. Pre-exit means the process has not yet cleaned up its address
space, and thus memory can still be read or written. Post-exit means the
process has cleaned up its address space, memory is no longer
accessible.

Associated EventType Code:

.. container:: Definition

   Exit

EventExit Member Functions:

.. container:: Definition

   int getExitCode() const

This function returns the process’ exit code.

EventCrash
~~~~~~~~~~

An EventCrash triggers when a process performs an abnormal exit (e.g.,
crashing on a memory violation). The process that crashed is referenced
with Event’s getProcess function.

An EventCrash may be associated with an EventType of pre-crash or
post-crash. Pre-crash means the process has not yet cleaned up its
address space, and thus memory can still be read or written. Post-crash
means the process has cleaned up its address space, memory is no longer
accessible.

Associated EventType Code:

.. container:: Definition

   Crash

EventCrash Member Functions:

.. container:: Definition

   int getTermSignal() const

This function returns the signal that caused the process to crash.

EventForceTerminate
~~~~~~~~~~~~~~~~~~~

An EventForceTerminate triggers when a process is forcefully terminated
via the Process::terminate function. When the callback is delivered for
this event, the address space of the corresponding process will no
longer be available.

Associated EventType Code:

.. container:: Definition

   ForceTerminate

EventForceTerminate Member Functions:

.. container:: Definition

   int getTermSignal() const

This function returns the signal that was used to terminate the process.

EventExec
~~~~~~~~~

An EventExec triggers when a process performs a UNIX-style exec
operation. An EventType of post-Exec means the process has completed the
exec and setup its new address space. An EventType of pre-Exec means the
process has not yet torn down its old address space.

Associated EventType Code:

.. container:: Definition

   Exec

EventExec Member Functions:

.. container:: Definition

   std::string getExecPath() const

This function returns the file path to the process’ new executable.

EventStop
~~~~~~~~~

An EventStop is triggered when a process is stopped by a
non-ProcControlAPI source. On UNIX based systems, this is triggered by
receipt of a SIGSTOP signal.

Unlike most other events, an EventStop will explicitly move the
associated thread or process (see the Event’s SyncType to tell which) to
a stopped state. Returning cbDefault from a callback function that has
received EventStop will leave the target process in a stopped state
rather than restore it to the pre-event state.

Associated EventType Code:

.. container:: Definition

   Stop

EventBreakpoint
~~~~~~~~~~~~~~~

An EventBreakpoint triggers when the target process encounters a
breakpoint inserted by the ProcControlAPI (see Section 3.4.).

Similar to EventStop, EventBreakpoint will explicitly move the thread or
process to a stopped state. Returning cbDefault from a callback function
that has received EventBreakpoint will leave the target process in a
stopped state rather than restore it to the pre-event state.

Associated EventType Code:

.. container:: Definition

   Breakpoint

EventBreakpoint Member Functions:

.. container:: Definition

   Dyninst::Address getAddress() const

This function returns the address at which the breakpoint was hit.

.. container:: Definition

   void getBreakpoints(std::vector<Breakpoint::const_ptr> &b) const

This function returns a vector of pointers to the Breakpoints that were
hit. Since it is possible to insert multiple Breakpoints at the same
location, it is possible for this function to return more than one
Breakpoint.

EventNewThread
~~~~~~~~~~~~~~

An EventNewThread triggers when a process spawns a new thread. The Event
class’ getThread function returns the original Thread that performed the
spawn operation, while EventNewThread’s getNewThread returns the newly
created Thread.

This event is never instantiated by ProcControlAPI and simply serves as
a place-holder type for a user to deal with thread creation without
having to deal with the specifics of LWP and user thread creation.

A callback function that receives an EventNewThread can use the two
field form of Process::cb_ret_t (see Sections 2.2.2. and 3.1.) to
control the parent and child thread.

Associated EventType Codes:

.. container:: Definition

   ThreadCreate, UserThreadCreate, LWPCreate

EventNewThread Member Functions:

.. container:: Definition

   Thread::const_ptr getNewThread() const

This function returns a const pointer to the Thread object that
represents the newly spawned thread.

EventNewUserThread
~~~~~~~~~~~~~~~~~~

An EventNewUserThread triggers when a process spawns a new user-level
thread. The Event class’ getThread function returns the original Thread
that performed the spawn operation. This thread may have already been
created if the platform supports the EventNewLWP event. If not, the
getNewThread function returns the newly created Thread.

A callback function that receives an EventNewThread can use the two
field form of Process::cb_ret_t (see Sections 2.2.2. and 3.1.) to
control the parent and child thread.

Associated EventType Code:

.. container:: Definition

   UserThreadCreate

EventNewThread Member Functions:

.. container:: Definition

   Thread::const_ptr getNewThread() const

This function returns a const pointer to the Thread object that
represents the newly spawned thread or the corresponding thread, if the
thread has already been created.

EventNewLWP
~~~~~~~~~~~

An EventNewLWP triggers when a process spawns a new LWP. The Event
class’ getThread function returns the original Thread that performed the
spawn operation, while EventNewThread’s getNewThread returns the newly
created Thread.

A callback function that receives an EventNewThread can use the two
field form of Process::cb_ret_t (see Sections 2.2.2. and 3.1.) to
control the parent and child thread.

Associated EventType Code:

.. container:: Definition

   LWPCreate

EventNewThread Member Functions:

.. container:: Definition

   Thread::const_ptr getNewThread() const

This function returns a const pointer to the Thread object that
represents the newly spawned thread.

EventThreadDestroy
~~~~~~~~~~~~~~~~~~

An EventThreadDestroy triggers when a thread exits. Event’s getThread
member function returns the thread that exited.

This event is never instantiated by ProcControlAPI and simply serves as
a place-holder type for a user to deal with thread destruction without
having to deal with the specifics of LWP and user thread destruction.

Associated EventType Codes:

.. container:: Definition

   ThreadDestroy, UserThreadDestroy, LWPDestroy

EventUserThreadDestroy
~~~~~~~~~~~~~~~~~~~~~~

An EventUserThreadDestroy triggers when a thread exits. Event’s
getThread member function returns the thread that exited.

If the platform also supports EventLWPDestroy events, this event will
precede an EventLWPDestroy event.

Associated EventType Code:

.. container:: Definition

   UserThreadDestroy

EventLWPDestroy
~~~~~~~~~~~~~~~

An LWPThreadDestroy triggers when a thread exits. Event’s getThread
member function returns the thread that exited.

Associated EventType Code:

.. container:: Definition

   LWPDestroy

EventFork
~~~~~~~~~

An EventFork triggers when a process performs a UNIX-style fork
operation. The process that performed the initial fork is returned via
Event’s getProcess member function, while the newly created process can
be found via EventFork’s getChildProcess member function.

Associated EventType Code:

.. container:: Definition

   Fork

EventFork Member Functions:

.. container:: Definition

   Process::const_ptr getChildProcess() const

This function returns a pointer to the Process object that represents
the newly created child process.

EventSignal
~~~~~~~~~~~

An EventSignal triggers when a process receives a UNIX style signal.

Associated EventType Code:

.. container:: Definition

   Signal

EventSignal Member Functions:

.. container:: Definition

   int getSignal() const

This function returns the signal number that triggered the EventSignal.

EventRPC
~~~~~~~~

An EventRPC triggers when a process or thread completes a ProcControlAPI
iRPC (see Sections 2.3. and 3.5.). When a callback function receives an
EventRPC, the memory and registers that were used by the iRPC can still
be found in the address space and thread that the iRPC ran on. Once the
callback function completes, the registers and memory are restored to
their original state.

Associated EventType Code:

.. container:: Definition

   RPC

EventRPC Member Functions:

.. container:: Definition

   IRPC::const_ptr getIRPC() const

This function returns a const pointer to the IRPC object that completed.

EventSingleStep
~~~~~~~~~~~~~~~

An EventSingleStep triggers when a thread, which was put in single-step
mode by Thread::setSingleStep, completes a single step operation. The
Thread will remain in single-step mode after completion of this event
(presuming it has not be explicitly disabled by Thread::setSingleStep).

Associated EventType Code:

.. container:: Definition

   SingleStep

EventLibrary
~~~~~~~~~~~~

An EventLibrary triggers when the process either loads or unloads a
shared library. ProcControlAPI will not trigger an EventLibrary for
library unloads associated with a Process being terminated, and it will
not trigger EventLibrary for library loads that happened before a
ProcControlAPI attach operation.

It is possible for multiple libraries to be loaded or unloaded at the
same time. In this case, an EventLibrary will contain multiple libraries
in its load and unload sets.

Associated EventType Code:

.. container:: Definition

   Library

EventLibrary Member Functions:

.. container:: Definition

   const std::set<Library::ptr> &libsAdded() const

This function returns the set of Library objects that were loaded into
the target process’ address space. The set will be empty if no libraries
were loaded.

.. container:: Definition

   const std::set<Library::ptr> &libsRemoved() const

This function returns the set of libraries that were unloaded from the
target process’ address space. The set will be empty if no libraries
were unloaded.

EventPreSyscall, EventPostSyscall
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An EventPreSyscall is triggered when a thread enters a system call,
provided that the thread is in system call tracing mode. An
EventPostSyscall is triggered when a system call returns. These are both
children of EventSyscall, which provides all the relevant methods.

Associated EventType Code:

.. container:: Definition

   Syscall

EventPreSyscall and EventPostSyscall Member Functions:

.. container:: Definition

   Dyninst::Address getAddress() const

This function returns the address where the system call occurred.

.. container:: Definition

   MachSyscall getSyscall() const

This function returns information about the system call. See Appendix B.
for information about the MachSyscall class.

Platform-Specific Features
==========================

The classes described in this section are all used to configure
platform-specific features for Process objects. The three tracking
classes (LibraryTracking, ThreadTracking, LWPTracking) all contain
member functions to set either interrupt-driven or polling-driven
handling for different events associated with Process objects. When
interrupt-driven handling is enabled, the associated process may be
modified to accommodate timely handling (e.g., inserting breakpoints).
When polling-driven handling is enabled, the associated process is not
modified and events are handled on demand by calling the appropriate
“refresh” member function. All of these classes are defined in
PlatFeatures.h.

LibraryTracking
---------------

The LibraryTracking class is used to configure the handling of library
events for its associated Process.

LibraryTracking Declared In:

.. container:: Definition

   PlatFeatures.h

LibraryTracking Static Member Functions:

.. container:: Definition

   static void setDefaultTrackLibraries(bool b)

Sets the default handling mechanism for library events across all
Process objects to interrupt-driven (b = true) or polling-driven (b =
false).

.. container:: Definition

   static bool getDefaultTrackLibraries()

Returns the current default handling mechanism for library events across
all Process objects. A return value of true indicates interrupt-driven
while false indicates polling-driven.

LibraryTracking Member Functions:

.. container:: Definition

   bool setTrackLibraries(bool b) const

Sets the library event handling mechanism for the associated Process
object to interrupt-driven (b = true) or polling-driven (b = false).

Returns true on success and false on failure.

.. container:: Definition

   bool getTrackLibraries() const

Returns the current library event handling mechanism for the associated
Process object. A return value of true indicates interrupt-driven while
false indicates polling-driven.

.. container:: Definition

   bool refreshLibraries()

Manually polls for queued library events to handle.

Returns true on success and false on failure.

ThreadTracking
--------------

The ThreadTracking class is used to configure the handling of thread
events for its associated Process.

ThreadTracking Declared In:

.. container:: Definition

   PlatFeatures.h

ThreadTracking Static Member Functions:

.. container:: Definition

   static void setDefaultTrackThreads(bool b)

Sets the default handling mechanism for thread events across all Process
objects to interrupt-driven (b = true) or polling-driven (b = false).

.. container:: Definition

   static bool getDefaultTrackThreads()

Returns the current default handling mechanism for thread events across
all Process objects. A return value of true indicates interrupt-driven
while false indicates polling-driven.

ThreadTracking Member Functions:

.. container:: Definition

   bool setTrackThreads(bool b) const

Sets the thread event handling mechanism for the associated Process
object to interrupt-driven (b = true) or polling-driven (b = false).

Returns true on success and false on failure.

.. container:: Definition

   bool getTrackThreads() const

Returns the current thread event handling mechanism for the associated
Process object. A return value of true indicates interrupt-driven while
false indicates polling-driven.

.. container:: Definition

   bool refreshThreads()

Manually polls for queued thread events to handle.

Returns true on success and false on failure.

LWPTracking
-----------

The LWPTracking class is used to configure the handling of LWP events
for its associated Process.

LWPTracking Declared In:

.. container:: Definition

   PlatFeatures.h

LWPTracking Static Member Functions:

.. container:: Definition

   static void setDefaultTrackLWPs(bool b)

Sets the default handling mechanism for LWP events across all Process
objects to interrupt-driven (b = true) or polling-driven (b = false).

.. container:: Definition

   static bool getDefaultTrackLWPs()

Returns the current default handling mechanism for LWP events across all
Process objects. A return value of true indicates interrupt-driven while
false indicates polling-driven.

LWPTracking Member Functions:

.. container:: Definition

   bool setTrackLWPs(bool b) const

Sets the LWP event handling mechanism for the associated Process object
to interrupt-driven (b = true) or polling-driven (b = false).

Returns true on success and false on failure.

.. container:: Definition

   bool getTrackLWPs() const

Returns the current LWP event handling mechanism for the associated
Process object. A return value of true indicates interrupt-driven while
false indicates polling-driven.

.. container:: Definition

   bool refreshLWPs()

Manually polls for queued LWP events to handle.

Returns true on success and false on failure.

FollowFork
----------

The FollowFork class is used to configure ProcControlAPI’s behavior when
the associated Process forks.

FollowFork Declared In:

.. container:: Definition

   PlatFeatures.h

FollowFork Types:

.. container:: Definition

   typedef enum {

.. container:: Definition

   None,

.. container:: Definition

   ImmediateDetach,

.. container:: Definition

   DisableBreakpointsDetach,

.. container:: Definition

   Follow

.. container:: Definition

   } follow_t

The follow_t type represents the configurable behaviors under forking.

None is specified when fork tracking is not available for the current
platform.

ImmediateDetach means that forked children are never attached to.

DisableBreakpointsDetach means that inherited breakpoints are removed
from forked children, and then the children are detached.

Follow is the default behavior, and it means that forked children are
attached to and remain under full control of ProcControlAPI.

FollowFork Static Member Functions:

.. container:: Definition

   static void setDefaultFollowFork(follow_t f)

This function sets the default forking behavior across all Process
objects to f.

.. container:: Definition

   static follow_t getDefaultFollowFork()

This function returns the current default forking behavior across all
Process objects.

FollowFork Member Functions:

.. container:: Definition

   bool setFollowFork(follow_t f) const

This function sets the forking behavior for the associated Process
object to f.

This function returns true on success and false on failure.

.. container:: Definition

   follow_t getFollowFork() const

This function returns the current forking behavior for the associated
Process object.

SignalMask
----------

The SignalMask class is used to configure the signal mask for its
associated Process.

SignalMask Declared In:

.. container:: Definition

   PlatFeatures.h

SignalMask Types:

.. container:: Definition

   dyn_sigset_t

On POSIX systems, this type is equivalent to sigset_t.

SignalMask Static Member Functions:

.. container:: Definition

   static void setDefaultSigMask(dyn_sigset_t s)

This function sets the default signal mask across all Process objects to
s.

.. container:: Definition

   static dyn_sigset_t getDefaultSigMask()

This function returns the current default signal mask across all Process
objects.

SignalMask Member Functions:

.. container:: Definition

   bool setSigMask(dyn_sigset_t s)

This function sets the signal mask for the associated Process object to
s.

This function returns true on success and false on failure.

.. container:: Definition

   dyn_sigset_t getSigMask() const

This function returns the current signal mask for the associated Process
object.

Registers
=========

This appendix describes the MachRegister interface, which is used for
accessing registers in ProcControlAPI. The entire definition of
MachRegister contains more register names than are listed here; this
appendix only lists the registers that can be accessed through
ProcControlAPI.

An instance of MachRegister is defined for each register ProcControlAPI
can name. These instances live inside a namespace that represents the
register’s architecture. For example, we can name a register from an
AMD64 machine with Dyninst::x86_64::rax or a register from a Power
machine with Dyninst::ppc32::r1.

All functions, types, and objects listed below are part of the C++
namespace Dyninst.

Declared In:

.. container:: Definition

   dyn_regs.h

Related Types:

.. container:: Definition

   typedef unsigned long MachRegisterVal

The MachRegisterVal type is used to represent the contents of a
register. If a register’s contents are smaller than MachRegisterVal,
then it will be up cast into a MachRegisterVal.

.. container:: Definition

   typedef enum {

.. container:: Definition

   Arch_none,

.. container:: Definition

   Arch_x86,

.. container:: Definition

   Arch_x86_64,

.. container:: Definition

   Arch_ppc32,

.. container:: Definition

   Arch_ppc64

.. container:: Definition

   } Architecture

The Architecture enum represents a system’s architecture.

Related Global Functions

.. container:: Definition

   unsigned int getArchAddressWidth(Architecture arch)

Returns the size of a pointer, in bytes, on the given architecture,
arch.

MachRegister Static Member Functions

.. container:: Definition

   MachRegister getPC(Dyninst::Architecture arch)

.. container:: Definition

   MachRegister getFramePointer(Dyninst::Architecture arch)

.. container:: Definition

   MachRegister getStackPointer(Dyninst::Architecture arch)

These functions respectively return the register that represents the
program counter, frame pointer, or stack pointer for the given
architecture. If an architecture does not support a frame pointer (ppc32
and ppc64) then getFramePointer returns an invalid register.

MachRegister Member Functions

.. container:: Definition

   MachRegister getBaseRegister() const

This function returns the largest register that may alias with the given
register. For example, getBaseRegister on x86_64::ah returns
x86_64::rax.

.. container:: Definition

   Architecture getArchitecture() const

This function returns the Architecture for this register.

.. container:: Definition

   bool isValid() const

This function returns true if this register is valid. Some API functions
may return invalid registers upon error.

.. container:: Definition

   MachRegisterVal getSubRegVal(

.. container:: Definition

   MachRegister subreg,

.. container:: Definition

   MachRegisterVal orig)

Given a value for this register, orig, and a smaller aliased register,
subreg, then this function returns the value of the aliased register.
For example, if this function were called on x86::eax with subreg as
x86::al and an orig value of 0x11223344, then it would return 0x44.

.. container:: Definition

   const char \*name() const

This function returns a human readable name that identifies this
register.

.. container:: Definition

   unsigned int size() const

This function returns the size of the register in bytes.

.. container:: Definition

   signed int val() const

This function returns a unique integer that represents this register.
This can be useful for writing switch statements that take a
MachRegister. The unique integers for a MachRegister can be found by
preceding the register object name with an ‘i'. For example, a switch
statement for MachRegister, reg, could be written as:

.. container:: Definition

      switch (reg.val()) {

.. container:: Definition

      case x86_64::irax:

.. container:: Definition

      case x86_64::irbx:

.. container:: Definition

      case x86_64::ircx:

.. container:: Definition

      …

.. container:: Definition

      }

.. container:: Definition

   bool isPC() const

.. container:: Definition

   bool isFramePointer() const

.. container:: Definition

   bool isStackPointer() const

These functions respectively return true if the register is the program
counter, frame pointer, or stack pointer for its architecture. They
return false otherwise.

MachRegister Objects

The following list describes instances of MachRegister that can be
passed to ProcControlAPI. These can be named by prepending the namespace
to the listed names, e.g., x86::eax.

.. container:: Definition

   namespace x86

eax

ebx

ecx

edx

ebp

esp

esi

edi

oeax

eip

flags

cs

ds

es

fs

gs

ss

fsbase

gsbase

.. container:: Definition

   namespace x86_64

rax

rbx

rcx

rdx

rbp

rsp

rsi

rdi

r8

r9

r10

r11

r12

r13

r14

r15

orax

rip

flags

cs

ds

es

fs

gs

ss

fsbase

gsbase

.. container:: Definition

   namespace ppc32

r0

r1

r2

r3

r4

r5

r6

r7

r8

r9

r10

r11

r12

r13

r14

r15

r16

r17

r18

r19

r20

r21

r22

r23

r24

r25

r26

r27

r28

r29

r30

r31

fpscw

lr

cr

xer

ctr

pc

msr

.. container:: Definition

   namespace ppc64

r0

r1

r2

r3

r4

r5

r6

r7

r8

r9

r10

r11

r12

r13

r14

r15

r16

r17

r18

r19

r20

r21

r22

r23

r24

r25

r26

r27

r28

r29

r30

r31

fpscw

lr

cr

xer

ctr

pcmsr

.. container:: Definition

   namespace aarch64

x0

x1

x2

x3

x4

x5

x6

x7

x8

x9

x10

x11

x12

x13

x14

x15

x16

x17

x18

x19

x20

x21

x22

x23

x24

x25

x26

x27

x28

x29

x30

q0

q1

q2

q3

q4

q5

q6

q7

q8

q9

q10

q11

q12

q13

q14

q15

q16

q17

q18

q19

q20

q21

q22

q23

q24

q25

q26

q27

q28

q29

q30

q31

sp

pc

pstate

fpcr

fpsr

System Calls
============

The MachSyscall class, found in MachSyscall.h, represents system calls
in a platform-independent manner. Currently, syscall events are only
supported on Linux.

MachSyscall Methods

.. container:: Definition

   SyscallIDPlatform num() const

Returns the platform-specific syscall number

.. container:: Definition

   SyscallName name() const

Returns the name of the system call (e.g. “getpid”)

.. container:: Definition

   bool operator==(const MachSyscall&) const

Equality operator. Two system calls are equal if they are for the same
platform and have the same syscall number.

.. container:: Definition

   static MachSyscall makeFromPlatform(Platform, SyscallIDIndependent)

Constructs a MachSyscall from a Platform and a platform-independent ID
(e.g. dyn_getpid). The platform-independent syscall IDs may be found in
dyn_syscalls.h.
