##### Stale Data
- Stale data can cause serious and confusing failures such as unexpected exceptions, corrupted data structures, inaccurate computations, and infinite loops.
- Reading data without synchronization is analogous to using the READ_UNCOMMITTED isolation level in a database, where you are willing to trade accuracy for performance. However, in the case of unsynchronized reads, you are trading away a greater degree of accuracy, since the visible value for a shared variable can be arbitrarily stale.
- `Out of thin air safety`
##### Locking and Visibility
- Intrinsic locking can be used to guarantee that `one thread sees the effects of another in a predictable manner`
- For example, when thread A executes a `synchronized` block, and subsequently thread B enters a `synchronized` block guarded by the same lock, the values of variables that were visible to A prior to releasing the lock are guaranteed to be visible to B when it executes a `synchronized` block guarded by the same lock. `Without synchronization there would be no such guarantee`
- Another reason for the rule requiring all threads to synchronize on the `same` lock when accessing a shared mutable variable - to guarantee that values written by one thread are made visible to other threads. Otherwise, if a thread reads a variable without holding appropriate lock, it might see a stale value.
- Locking is not just about mutual exclusion, it's about `memory visibility`. To `ensure that all threads see the most up-to-date values of shared mutable variables`, the reading and writing threads must synchronize on a common lock.

##### Volatile Variables
- Java language also provides an alternative, weaker form of synchronization, `volatile variables`, to ensure that updates to a variable are propagated predictably to other threads. 
- When a field is declared `volatile`, the compiler and run time are put on notice that this `variable is shared and that operations on it should not be reordered` with other memory operations.
- Volatile variables are not cached in registers or in caches where they are hidden from other processors, `so a read of a volatile variables always returns the most recent write by any thread`.
- Accessing a volatile variable performs no locking and so cannot cause the executing thread to block, making `volatile variables a lighter-weight synchronization` mechanism than synchronized.
- Locking can guarantee both visibility and atomicity; volatile variables can only guarantee visiblity.
#### Publication and Escape
- `Publishing` an object means making it available to code outside of the current scope.
- An object that is published when it should not have been is said to have `escaped`
- Publishing an object also publishes any objects referred to by its nonprivate fields.
- More generally, any object that is `reachable` from a published object by following some chain of nonprivate field references and method calls has also been published.
- Whether another thread actually does something with a published reference doesn't really matter, because the risk of misuse is still present. Once an object escapes, you have to assume that another class or thread may, maliciously or carelessly, misuse it. This is a compelling reason to use `encapsulation` -> it makes it practical to analyze programs for correctness and harder to violate design constraints accidentally.

#### Safe Construction Practice 
- An object is in a predictable, consistent state only after its constructor returns, so publishing an object from within its constructor can publish an incompletely constructed object.
- This is true `even if the publication is the last statement in the constructor` -> if the `this` reference escapes during construction, the object is considered `not properly constructed.`
- Do not allow the `this` reference to escape during construction.
- A common mistake that can let the `this` reference escape during construction is to start a thread from a constructor. When an object creates a thread from its constructor, it almost always share its `this` reference with the new thread, either explicitly (by passing it to the constructor) or implicitly (because the `Thread` or `Runnable` is an inner class of the owning object).
- The new thread might then be able to see the owning object before it is fully constructed.
- There's nothing wrong with `creating` a thread in a constructor but it is best not to `start` the thread immediately.
- Instead, expose a start or initialize method that starts the owned thread.
- Calling an overrideable instance method from the constructor can also allow the `this` reference to escape.
#### Thread Confinement
- Accessing shared, mutable data requires using synchronization; one way to avoid this requirement is to `not share`.
- This technique, `thread confinement` is one of the simplest ways to achieve thread safety.
- When an object is confined to a thread, such usage is automatically thread-safe even if the confined object itself is not.
##### ThreadLocal
- A more formal means of maintaining thread confinement is `ThreadLocal`, which allows you to associate a per-thread value with a `value-holding object.`
- Thread-Local provides get and set accessor methods that maintain a separate copy of the value for each thread that uses it, so a get returns the most recent value passed to set `from the currently executing thread.`
#### Immutability
- The other end-run around the need to synchronize is to use `immutable` objects.
An object is `immutable` if:
1. Its state cannot be modified after construction.
2. All its fields are final.
3. It is properly constructed.

##### Final fields
- Final fields can't be modified, but they also have special semantics under the Java Memory Model.
- It is the use of final fields that make possible the guarantee of `initialization safety` that lets immutable objects be freely accessed and shared without synchronizatin.
- Immutable objects can be safely accessed `even when synchronization is not used to publish the object reference`
- This guarantee extends to the values of all final fields of properly constructed objects - final fields can be safely accessed without additional synchrnozation. 
- However, if final fields refer to mutable objects, synchronization is still required to access the state of the objects they refer to.
##### Safe Publication Idioms
- Objects that are not immutable must be `safely published` 
- To publish an object safely, both the reference to the object and the object's state must be made visible to other threads at the same time.
- A properly constructed object can be safely published by
1. Initializing an object reference from a static initializer;
2. Storing a reference to it into a volatile field or AtomicReference;
3. Storing a reference to it into a final field of a properly constructed object; or
4. Storing a reference to it into a field that is properly guarded by a lock.

###### Publication requirements for an object depend on its mutability - 
1. `Immutable objects` can be published through any mechanism.
2. `Effectively Immutable objects` must be safely published.
3. `Mutable objects` must be safely published, and must be either threadsafe or guarded by a lock.

###### Useful policies for using and sharing objects in a concurrent program
1. Thread confined -> A thread-confined object is `owned exclusively by and confined to one thread`, and can be modified by its owning thread.
2. Shared read-only -> A shared read-only object can be `accessed concurrently by multiple threads without additional synchronization, but cannot be modified by any thread`. Shared read-only objects include immutable and effectively immutable objects.
3. Shared thread-safe -> A thread-safe object `performs synchronization internally`, so multiple threads can freely access it through its public interface without further synchronizatin.
4. Guarded -> A guarded object can be accessed only with a specific lock held. Guarded objects include those `that are encapsulated within other thread-safe objects` and published objects that are known to be `guarded by a specific lock`