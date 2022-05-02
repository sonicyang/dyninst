.. _`sec:proccontrolapi-intro`:

==============
ProcControlAPI
==============

This document describes ProcControlAPI, an API and library for
controlling processes. ProcControlAPI runs as part of a *controller
process* and manages one or more *target processes*. It allows the
controller process to perform operations on target processes, such as
writing to memory, stopping and running threads, or receiving
notification when certain events occur. ProcControlAPI presents these
operations through a platform-independent API and high-level
abstractions. Users can describe what they want ProcControlAPI to do,
and ProcControlAPI handles the details.

An example use for ProcControlAPI would be as the underlying mechanism
for a debugger. A user writing a debugger could provide their own user
interface and debugging strategies, while using ProcControlAPI to
perform operations such as creating processes, running threads, and
handling breakpoints.

ProcControlAPI exposes a C++ interface. This document assumes some
familiarity with several concepts from C++, such as const types,
iterators, and inheritance.

The interface for ProcControlAPI can be generally divided into two
parts: an interface for managing a process (e.g., reading and writing to
target process memory, stopping and running threads), and an interface
for monitoring a target process for certain events (e.g., watching the
target process for fork or thread creation events). The manager
interface uses a set of C++ objects to represent a target process and
its threads, libraries, registers and other interesting aspects.
Operations performed on these C++ objects in the controller process are
translated into corresponding operations on the target process. The
event interface uses a callback system to notify the ProcControlAPI user
of interesting events in the target process.

Abstractions
============

This section focuses on some of the more important concepts in
ProcControlAPI and gives a high level overview before the detailed API
is presented in Section 3..

Processes and Threads
---------------------

There are two central classes to ProcControlAPI, Process and Thread.
Each class respectively represents a single target process or thread
running on the system. By performing operations on the Process and
Thread objects, a ProcControlAPI user is able to control the target
process and its threads.

Each Process is guaranteed to have at least one Thread associated with
it. A multi-threaded process may have a Process object with more than
one Thread. Each process has an address space associated with it, which
can be written or read through the Process object. Each thread has a set
of registers associated with it, which can be access through the Thread
object.

At any one time a Thread will be in either a *stopped state* or a
*running state*. A thread in a stopped state has had its execution
paused by ProcControlAPI—the OS will not schedule the thread to run. A
thread in a running state is allowed to execute as normal. A thread in a
running state may block for other reasons, e.g. blocking on IO calls,
but this does not affect ProcControlAPI’s view of the thread state. A
thread is only in the stopped state if ProcControlAPI has explicitly
stopped it.

A Process object is not considered to have a stopped or running
state—only its Thread objects are stopped or running. A stop operation
on a Process triggers a stop operation on each of its Threads, and
similarly a continue operation on a Process triggers continue operations
on each Thread.

Callbacks
---------

In addition to controlling a target process through the Process and
Thread objects, a ProcControlAPI user can also receive notification of
events that happen in that process. Examples of these events would be a
new thread being created, a breakpoint being executed, or a process
exiting.

The ProcControlAPI user receives notice of events through a callback
system. The user can register callback function that will be called by
ProcControlAPI whenever a particular type of event occurs. Details about
the event are passed to the callback function via an Event object.

Events
~~~~~~

Each event can be broken up into an EventType object and an Event
object. The EventType describes a type of event that can happen, and
Event describes a specific instance of an event happening. Each Event
will have one and only one EventType.

Each EventType has two primary fields: its time and its code. The code
field of describes what type of event occurred, e.g. EventType::Exit
represents a target process exiting. The time field of an EventType
represents whether the EventType is happening before or after will have
code and will have a value of EventType::Pre, EventType::Post, or
EventType::None.

For example, an EventType with time and code of EventType::Pre and
EventType::Exit will occur just before a target process exits, and a
code of EventType::Exec with a time of EventType::Post will occur after
an exec system call occurs. In this document we will abbreviate
EventTypes such as these as pre-exit and post-exec. Some EventTypes do
not have a time associated with them, for example EventType::Breakpoint
does not have an associated time and thus has a time value of
EventType::none.

An Event represents an instance of an EventType occurring. In addition
to an EventType, each Event also has pointer to the Process and Thread
that it occurred on. Certain events may also have event specific
information associated with them, which is represented in a sub-class of
Event. Each EventType is associated with a specific sub-class of Event.

For example, EventType::Library is used to signify a shared library
being loaded into the target process. When an EventType::Library occurs
ProcControlAPI will deliver an object of type EventLibrary, which is a
subclass of Event, to any registered callback functions. In addition to
the information inherited from Event, the EventLibrary will contain
extra information about the library that was loaded into the target
process.

Table 1 shows the Event subclass that is used for each EventType. Not
all EventTypes are available on every platform—a checkmark under the
specific OS column means that the EventType is available on that OS.

+----------+----------+----------+----------+----------+----------+
| **Eve    | **Event  | *        | **F      | **W      | **BG/Q** |
| ntType** | Su       | *Linux** | reeBSD** | indows** |          |
|          | bclass** |          |          |          |          |
+==========+==========+==========+==========+==========+==========+
| Stop     | E        |         |          |          |          |
|          | ventStop |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Br       | EventBr  |         |         |         |         |
| eakpoint | eakpoint |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Signal   | Eve      |         |         |         |         |
|          | ntSignal |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| UserThre | Ev       |         |         |          |         |
| adCreate | entNewUs |          |          |          |          |
|          | erThread |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| L        | Eve      |         |          |         |          |
| WPCreate | ntNewLWP |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Pre-U    | EventU   |         |         |          |         |
| serThrea | serThrea |          |          |          |          |
| dDestroy | dDestroy |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Post-U   | EventU   |         |         |          |          |
| serThrea | serThrea |          |          |          |          |
| dDestroy | dDestroy |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Pre-LW   | EventLW  |         |          |         |          |
| PDestroy | PDestroy |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Post-LW  | EventLW  |         |          |          |          |
| PDestroy | PDestroy |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Pre-Fork | E        |          |          |          |          |
|          | ventFork |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| P        | E        |         |          |          |          |
| ost-Fork | ventFork |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Pre-Exec | E        |          |          |          |          |
|          | ventExec |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| P        | E        |         |         |          |          |
| ost-Exec | ventExec |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| RPC      | EventRPC |         |         |         |         |
+----------+----------+----------+----------+----------+----------+
| Si       | EventSi  |         |         |         |         |
| ngleStep | ngleStep |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Br       | EventBr  |         |         |         |         |
| eakpoint | eakpoint |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Library  | Even     |         |         |         |         |
|          | tLibrary |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Pre-Exit | E        |         |          |          |          |
|          | ventExit |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| P        | E        |         |         |         |         |
| ost-Exit | ventExit |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| Crash    | Ev       |         |         |         |         |
|          | entCrash |          |          |          |          |
+----------+----------+----------+----------+----------+----------+
| ForceT   | Eve      |         |         |         |          |
| erminate | ntForceT |          |          |          |          |
|          | erminate |          |          |          |          |
+----------+----------+----------+----------+----------+----------+

Table 1 – EventTypes and Events

Details about specific events can be found in Section 3.14..

Callback Functions
~~~~~~~~~~~~~~~~~~

Events are delivered via a callback function. A ProcControlAPI user can
register callback functions for an EventType using the
Process::registerEventCallback function. All callback functions must be
declared using the signature:

.. container:: Definition

   Process::cb_ret_t callback_func_name(Event::ptr ev)

In order to prevent a class of race conditions, ProcControlAPI does not
allow a callback function to perform any operation that would require
another callback to be recursively delivered. At most one callback
function can be running at a time.

To enforce this, the event that is passed to a callback function
contains only const pointers to the triggering Process and Thread
objects. Any member function that could trigger callbacks is not marked
const, thus triggering a compilation error if they are called on an
object passed to a callback. If the ProcControlAPI user uses const_cast
or global variables to get around the const restriction it will result
in a runtime error. API functions that cannot be used from a callback
are mentioned in the API entries.

Operations such as Process::stopProc, Process::continueProc,
Thread::stopThread, and Thread::continueThread are not safe to call from
a callback function, but it would still be useful to perform these
operations. ProcControlAPI allows the user to use the return value from
a callback function to specify whether process or thread that triggered
the event should be stopped or continued. More details on this can be
found in the Process::cb_ret_t section of the API reference.

Callback Delivery
~~~~~~~~~~~~~~~~~

When ProcControlAPI needs to deliver a callback it must first gain
control of a user visible thread in the controller process. This thread
will be used to invoke the callback function. ProcControlAPI does not
use its internal threads for delivering callbacks, as this would expose
the ProcControlAPI user to race conditions.

Unfortunately, the user thread is not always accessible to
ProcControlAPI when it needs to invoke a callback function. For example,
the user visible thread may be performing network IO or waiting for
input from a GUI when an event occurs.

ProcControlAPI uses a notification system built around the EventNotify
class to alert the ProcControlAPI user that a callback is ready to be
delivered. Once the user is notified then they can call the
Process::handleEvents function, under which ProcControlAPI will invoke
any pending callback functions.

The EventNotify class has two mechanisms for notifying the
ProcControlAPI user that a callback is pending: writing to a file
descriptor and a light-weight callback function. The EventNotify::getFD
function returns a file descriptor that will have a byte written to it
when a callback is ready. This file descriptor can be added to a select
or poll to block a thread that handles ProcControlAPI events.
Alternatively, the ProcControlAPI user can register a light-weight
callback that is invoked when a callback is ready. This light-weight
callback provides no information about the Event and may occur on
another thread or from a signal handler—the ProcControlAPI user is
encouraged to keep this callback minimal.

It is important for a user to respond promptly to a callback
notification. A target process may remain blocked while a notification
is pending. If a target process is generating many events that need
callbacks, a long delay in notification could have a significant
performance impact.

Once the ProcControlAPI user knows that a callback is ready to be
delivered they can call Process::handleEvents, which will invoke all
callback functions. Alternatively, if the ProcControlAPI user does not
need to handle events outside of ProcControlAPI, they can continue to
block in Process::handleEvents without going through the notification
system.

iRPCs
=====

An iRPC (Inferior Remote Procedure Call) is a mechanism for executing
code in a target process. Despite the name, an iRPC does not necessarily
have to involve a procedure call—any piece of code can be executed.

A ProcControlAPI user can invoke an iRPC by providing ProcControlAPI
with a buffer of machine code and specifying a Process or Thread on
which to run the machine code. ProcControlAPI will insert the machine
code into the address space, save the register set, run the machine
code, and then remove the machine code after execution completes. When
the iRPC completes (but before the registers and memory are cleaned)
ProcControlAPI will deliver an EventIRPC to any registered callback
function. The ProcControlAPI user may use this callback to collect any
results from the registers or memory used by the iRPC.

Note that ProcControlAPI will preserve the registers of the thread
running the iRPC, and it will preserve the memory used by the machine
code. Other memory or system state changed by the iRPC may remain
visible to the target process after the iRPC completes.

The machine code for each iRPC must contain at least one trap
instruction (e.g., a 0xCC instruction on x86 family or a 0x7D821008
instruction on the PPC family). ProcControlAPI will stop executing the
iRPC upon invocation of the trap. Note that the trap instruction must
fall within the original machine code for the iRPC. If the iRPC calls or
jumps to another piece of code that executes a trap instruction then
ProcControlAPI will not treat it as the end of the iRPC.

Before an iRPC can be run it must be posted to a process or thread using
the Process::postIRPC or Thread::postIRPC API functions. The
Process::postIRPC function will select a thread to post the iRPC to.
Multiple iRPCs can be posted to the same thread, but only one iRPC will
run at a time—subsequent iRPCs will be queued and run after the
preceding iRPC completes. If multiple iRPCs are posted to different
threads in a multi-threaded process, then they may run in parallel.

An iRPC can be posted to a stopped or running thread. If posted to a
stopped thread, then the iRPC will run when the thread is continued. If
posted to a running thread, then the iRPC will run immediately or, if
posted from a callback function, when the callback function completes.

An iRPC may be blocking or non-blocking. If a blocking iRPC is posted to
any Process, then calls to Process::handleEvents will block until the
iRPC is completed.

Usage
=====

As an example, consider the code in Figure 1 that creates a target
process and prints a message whenever that target process creates a new
thread. Details on the API function used in this example can be found in
latter sections of this manual, but we will provide a high level
description of the operations here. Note that proper error handling and
checking have been left out for brevity.

1. We start by parsing the arguments passed to the controller process,
   turning them into arguments that will be passed to the new target
   process.

2. We ask ProcControlAPI to create a new Process using the given
   arguments. ProcControlAPI will spawn a new target process and leave
   it in a stopped state to prevent it from executing.

3. After creating the new target process we register a callback
   function. We ask ProcControlAPI to call our function,
   on_thread_create, when an event of type EventType::ThreadCreate
   occurs in the target process.

4. The on_thread_create function takes a pointer to an object of type
   Event and returns a Process::cb_ret_t. The Event describes the target
   process event that triggered this callback. In this case, it provides
   information about the new thread in the target process. It is worth
   noting that Event::const_ptr is a not a regular pointer, but a
   reference counted shared pointer. This means that we do not have to
   be concerned with cleaning the Event—it will be automatically cleaned
   when the last reference disappears. The Process::cb_ret_t describes
   what action should be taken on the process in response to this event,
   which is described in more detail in section 6.

5. The Event class has several child classes, one of which is
   EventNewThread. We start by casting the Event into an EventNewThread
   and then extract information about the new thread from the
   EventNewThread.

6. In step 6, we’ve finished handling the new thread event and need to
   tell ProcControlAPI what to do in response to this event. For
   example, we could choose to stop the process from further execution
   by returning a value of Process::cbProcStop. Instead, we choose let
   ProcControlAPI take its default action for an EventNewThread by
   returning Process::cbDefault, which is to continue the process and
   its new thread (which were both stopped before delivery of the
   callback).

7. The registering of our callback in step 3 did not actually trigger
   any calls to the callback function—the target process was created in
   a stopped state and has not yet been able to create any threads. We
   tell ProcControlAPI to continue the target process in this step,
   which allows it to execute and possibly start generating new events.

8. In this step we wait for the target process to finish executing and
   terminate. Calling Process::handleEvents blocks the controller
   process until an event occurs, allowing us to wait for events without
   needing to spin the controller process on the CPU.



