### Composing Objects
The design process for a thread-safe class should include these three basic elements -
1. Identify the variables that form the object's state
2. Identify the invariants that constrain the state variables
3. Establish a policy for managing concurrent access to the object's state
- The `synchronization policy` defines how an object coordinates access to its state without violating its invariants or postconditions.
- It specifies what combination of immutability, thread confinement, and locking is used to maintain thread safety, and which variables are guarded by which locks. 
###### Gathering Synchronization Requirements
- Making a class thread-safe means ensuring that its invariants hold under concurrent access; this requires reasoning about its state.
- A class can also have invariants that constrain multiple state variables. A number range class, like `NumberRange` typically maintains state variables for the lower and upper bounds of the range. These variables must obey the constraint that the lower bound be less than or equal to the upper bound. 
- Multivariable invariants like the above create atomicity requirements: related variables must be fetched or updated in a single atomic operation. You cannot update one, release and reacquire the lock, and then update th others, since this could involve leaving the object in an invalid state when the lock was released.
- When multiple variables participate in an invariant, the lock that guards them must be held for the duration of any operation that accesss the related variables. 
- You cannot ensure thread safety without understanding an object's invariants and postconditions. 
###### State-Dependent Operations
- Operations with state-based preconditions are called `state-dependent`
- In a single-threaded program, if a precondition does not hold, the operation has no choice but to fail. But in a concurrent program, the precondition may become true later due to the action of another thread. Concurrent programs add the possibility of waiting until the precondition becomes true, and then proceeding with the operations.
- The built-in mechanisms for efficiently waiting for a condition to become true - wait and notify -  are tightly bound to intrinsic locking, and can be `difficult to use correctly`. 
- Use library classes such as blocking queues or semaphores, to provide th edesired state-dependent behavior.

###### State Ownership
- Ownership and encapsulation go together - the object encapsulates the state it owns and owns the state it encapsulates.
- It is the owner of a given state variable that gets to decide on the locking protocol used to maintain the integrity of the variable's state.
- Ownership implies control, but once you publish a reference to mutalbe object, you no longer have exclusive control; at best, you might have `shared ownership`
- Collection classes often exhibit a form of `split ownership`, in which the collection owns the state of the collection infrastructure, but the client code owns the objects stored in the collection.

#### Instance Confinement
- If an object is not thread-safe, several techniques can still let it be used safely in a multithreaded program.
- You can ensure that it is only accessed from a single thread (thread confinement), or that all access to it is properly guarded by a lock.
- Encapuslation simplifies making classes thread-safe by promoting `instance confinement`, often just called `confinement`
- When an object is encapsulated within another object, all code paths that have access to the encapsulated object are known and can be therefore be analyzed more easily than if that object were accessible to the entire program.
- Combining confinement with an appropriate locking discipline can ensure that otherwise non-thread-safe objects are used in a thread-safe manner.
- Encapsulating data within an object confines access to the data to the object's methods, making it easier to ensure that the data is always accessed with the appropriate lock held.
- Confined objects must not escape their intended scope. An object may be confined to an class instance (such as a private class memember), a lexical scope(such as a local variable), or a thread(such as an object that is passed from method to method within a thread, but not supposed to be shared across threads). 
- Instance confinement is one of the easiest ways to build thread-safe classes. It also allows flexibility in the choice of locking strategy
- If an object is intended to be confined to a specific scope, then letting it escape from that scope is a bug. 
- Confined objects can also escape by publishing other objects such as iterators or inner class instances that may indirectly publish the confined objects.
##### Java Monitor Pattern
- An object following the Java monitor pattern encapsulates all its mutable state and guards it with the object's own intrinsic lock.
- The Java monitor pattern is merely a convention; any lock object could be used to guard an object's state so long as it is used consistently