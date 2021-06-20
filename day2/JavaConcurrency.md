##### Reentrancy
- When a thread requests a lock that is already held by another thread, the requesting thread blocks. But because intrinsic locks are `reentrant`, if a `thread tries to acquire a lock that it already holds, the request succeeds.`
- Reentrancy means that locks are acquired on a `per-thread rather than per-invocation basis`
- When the count is zero, the lock is considered unheld. When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one. If that same thread acquires the lock again, the count is incremented, and when the owning thread exits the `synchronized` block, the count is decremented. When the count reaches zero, the lock is released.
- This differs from the default locking behavior for pthreads (POSIX threads) mutexes, which are granted on a per-invocation basis.
- Reentrancy facilitates encapsulation of locking behavior, and thus simplifies the devlopment of object-oriented concurrent code.

##### Guarding State with Locks
- Because locks enable `serialized` access to the code paths they guard, we can use them to construct protocols for guaranteeing exclusive access to shared state.
- Compound actions on shared state, such as incrementing a hit counter (read-modify-write) or lazy initialization (check-then-act), must be made atomic to avoid race conditions. However, just wrapping the compound action with a synchronized block is not sufficient; if synchronization is used to coordinate access to a variable, it is needed `everywhere that variable is accessed`. 
- Further, when using locks to coordinate access to a variable, the `same` lock must be used wherever that variable is accessed.
- For each mutable state variable that may be accessed by more than one thread, `all` accesses to that variable must be performed with the `same` lock held. In this case, we say that the variable is `guarded by` that lock.
- There is no inherent relationship between an object's intrinsic lock and its state; an object's fields need not be guarded by its intrinsic lock, though this is a perfectly valid locking convention that is used by many classes.
- Acquiring the lock associated with an object does `not` prevent other threads from accessing that object - the only thing that acquiring a lock prevents any other thread from doing is acquring that same lock.
- For every invariant that involves more than one variable, `all` the variables involved in that invariant must be guarded by the `same lock`

### Sharing Objects
- `Synchronized` is not only about atomicity or demarcating critical sections.
- It has another significant, and subtle, aspect: `memory visibility`
- We want not only to prevent one thread from modifying the state of an object when another is using it, but also to ensure that when a thread modifies the state of an object, other threads can actually `see` the changes that were made. But without synchronization, that may not happen. You can ensure that objects are published safely either by using explicit synchronization or by taking advantage of synchronization built into library classes.
##### Visibility
- In a single-threaded environment, if you write a value to a variable and later read that variable with no intervening writes, you can expect to get the same value back. 
- But when the reads and writes occur in different threads, this is simply not the case. In general, there is `no` guarantee that the reading thread will see a value written by another thread on a timely basis, or even at all. In order to ensure visibility of memory writes across threads, you must use synchronization.
- There is no guarantee that operations in one thread will be performed in the order given by the program, as long as the `reordering` is not detectable from within `that` thread - even if the reordering is apparent to other threads.
- eg. when main thread writes first to number and then to ready without synchronization, the reader thread could see those writes happen in the opposite order - or not at all.
- This may seem like broken design, but it is meant to allow JVMs to take full advantage of the performance of modern multiprocessor hardware. For example, in the absence of synchronization, the Java Memory Model permits the compiler to reorder operations and cache values in registers, and permits CPUs to reorder operations and cache values in processor-specific caches.
- In absense of synchronization, the compiler, processor, and runtime can do some downright wierd things to the order in which operations appear to execute.
- Attempts to reason about the order in which memory actions `must` happen in insufficiently synchronized multithreaded programs will almost certainly be incorrect.
- Easy way to avoid these complex issues - `always use the proper synchronization whenever data is shared across threads`
    