#### Introduction to Threads
- Threads allow multiple streams of program control flow to coexist within a process.
- They share process-wide resources such as memory and file handlers, but each thread has its own program counter, stack, and local variables.
- Threads are sometimes called `lightweight processes`, and most modern operating systems treat threads, not processes, as the `basic units of scheduling`.
- A server application that accepts socket connections from multiple remote clients may be easier to develop when each connection is allocated its own thread and allowed to use synchronous I/O.
- If an application goes to read from a socket when no data is available, `read` blocks until some data is available.
- In a single-threaded application, this means that not only does processing the corresponding request stall, but processing of `all` requests stall while the single thread is blocked.
- To avoid this problem, singlethreaded server applications are forced to use nonblocking I/O, which can be far more complicated and error prone than synhcronous I/O. However, if each request has its own thread, then blocking does not affect the processing of other requests.
- A `liveness` failure occurs when an activity gets into a state such that it is permanently unable to make forward progress. One form of liveness failure that can occur in sequential programs is an `inadvertent infinite loop`, where the code that follows the loop never gets executed.
- Like most concurrency bugs, bugs that cause `liveness failures` can be elusive because they depend on the relative timing of events in different threads, and therefore do not always manifest themselves in development or testing.
- Even if your program never explicitly creates a thread, frameworks may create threads on your behalf, and code called from these threads must be `thread-safe`
- Every Java application uses threads. When the JVM starts, it creates threads for JVM housekeeping tasks (garbage collection, finalization) and a `main thread` for running the main method. Component frameworks, such as servlets and RMI create pools of threads and invoke component methods in these threads.
- If you use these facilities - as many developers do - you have to be familiar with concurrency and thread safety, because these frameworks create threads and call your components from them. It would be nice to believe that concurrency is an `optional` or `advanced` language features, but reality is that nearly all Java applications are `multithreaded` and these frameworks `do not insulate you from the need to properly coordinate access to application state`.
- When concurrency is introduced into an application by a framework, it is usually `impossible to restrict the concurrency-awareness to the framework code`, because frameworks by nature make callbacks to application components that in turn access application state. 
- Similarly, the need for thread safety does not end with the components called by the framework - it extends to all code paths that access the program state accessed by those components. Thus, the need for `thread safety` is contagious.

### Thread Safety
- Writing thread-safe code is, at its core, about managing access to `state`, and in particular to `shared, mutable state`.
- Informally, an object's `state` is its data, stored in `state variables` such as `instance or static fields`. An object's state may include fields from other, dependent objects; a HashMap's state is partially stored in the HashMap object itself, but also in many `Map.Entry` objects.
- An object's state encompasses any data that can affect its externally visible behavior.
- By `shared`, we mean that a variable could be accessed by multiple threads; by `mutable`, we mean that its value could change during its lifetime.
- Thread safety is about trying to protect `data` from uncontrolled concurrent access.
- Whether an object needs to be thread-safe depends on whether it will be accessed from multiple threads. This is a property of how the object is `used` in a program, not what it `does`.
- Making an object thread-safe requires using synchronication to coordinate access to its mutable state; failing to do so could result in data corruption and other undesirable consequences.
- If multiple threads access the same mutable state variable without appropriate synchronization, `your program is broken`. There are three ways to fix it -
1. Don't share the state variable across threads.
2. Make the state variable `immuatable`
3. Use `synchronization` whenever accessing the state variable.
- Correctness means that a class `conforms to its specification.`
- A good specification defines `invariants` constraining an object's state and `postconditions` describing the effects of its operations.
- A class is `thread-safe`, if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of the execution of those threads by the runtime environment, with no additional synchronization or other coordination on the part of the calling code.
- No set of operations performed sequentially or concurrently on instances of a thread safe class can cause an instance to be in an invalid state.
- Thread-safe classes encapsulate any needed synchronization so that clients need not provide their own.
- `read-modify-write` operation -> result state is derived from the previous state.
- The possibility of incorrect results in the presence of unlucky timing is so important in concurrent programming that it has a name - `race condition`

##### Race Conditions
- A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime; in other words, when getting the right answer relies on lucky timing.
- The most common type of race condition is `check-then-act`, where potentially stale observation is used to make a decision on what to do next.
- The term `race condition` is often confused with the related term `data race`, which arises when synchronization is not used to cooridnate all access to a shared nonfinal field. 
- You risk a `data race` whenever a thread writes a variable that might next be read by another thread or reads a variable that might have last been written by another thread if both threads do not use synchronization.
- Not all race conditions are data races, and not all data races are race conditions, but they both can cause concurrent programs to fail in unpredictable ways.
##### Compound Actions
- To avoid race conditions, there must be a way to prevent other threads from using a variable while we're in the middle of modifying it, so we can ensure that other threads can observe or modify the state only before we start or after we finish, but not in the middle.
- Operations A and B are `atomic` with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has.
- An `atomic operation` is one that is atomic with respect to all operations, including itself, that operate on the same state.
- To ensure thread safety, `check-then-act` operations (like lazy initialization) and `read-modify-write` operations (like increment) must always be atomic. 
- We refer collectivey to `check-then-act` and `read-modify-write` sequences as `compound actions` - sequences of operations that must be executed atomically in order to remain thread-safe.

##### Intrinsic Locks
- Java provides a built-in locking mechanism for enforcing atomicity: the `synchronized` block.
- A synchronized block has two parts
1. A reference to an object that will serve as the `lock`,
2. A block of code to be guaraded by that lock.
- A `synchronized` method is a shorthand for a `synchronized` block that spans an entire method body, whose lock is the object on which the method is being invoked.
- Every Java object can implicitly act as a lock for purposes of synchronization - these built-in locks are called `intrinsic locks` or `monitor locks`. 
- The lock is automatically acquired by the executing thread before entering a `synchronized` block and automatically released when control exits the `synchronized` block, whether by the normal control path or by throwing an exception out of the block. The only way to acquire an intrinsic lock is to enter a synchronized block or method guarded by that lock.
- Intrinsic locks in Java act as `mutexes` (mutually exclusion locks), which mean that at most one thread may own the lock. When thread A attempts to acquire a lock held by thread B, A must wait, or `block` until B releases it. If B never releases the lock, A waits forever.
