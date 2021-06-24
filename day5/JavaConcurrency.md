#### Delegating Thread Safety
- An example class as NumberRange can be made thread-safe by using locking to maintain its invariants, such as guarding lower and upper with a common lock.
- It must also avoid publishing lower and upper to prevent clients from subverting its invariants.
- If a class has compound actions, delegation alone is not suitable approach for thread safety. In these cases, the class must provide `its own locking to ensure that compound actions are atomic`, unless the entire compound action can also be delegated to the underlying state variables.
- If a class is composed of multiple `independent` thread-safe state variables and has no operations that have any invalid state transitions, then it can delegate thread safety to the underlying state variables.
###### Publishing underlying state variables
- An example Counter class has value field which could take any integer value, however Counter constrains it to take on only positive values, and the increment operation constrains the set of valid next states given any current state.
- If you were to make value field public, clients could change it to an invalid value, so publishing it would render the class incorrect. 
- On the other hand, if a variable represents the current temparature or the ID of the last user to log on, then having another class modify this value at any time probably would not violate any invariants, so publishing this variable might be acceptable.
- If a state variable is thread-safe, does not participate in any invariants that constrain its value, and has no prohibited state transitions for any of its operations, the it can safely be published.

#### Adding Functionality to existing thread-safe classes
- As an example, let's say we need a thread-safe List with an atomic put-ifabsent operation. The synchronized List implementations nearly do the job, since they provide the contains and add methods from which we can construct a put-if-abset operation.
- The requirement that the class be thread-safe implicitly adds another requirement-that operations like `put-if-absent` be `atomic`
- Safest way to add a new atomic operation is to modify the original class to support the desired operation.
- Extension of class is more fragile than adding code directly to class, because the implementation of the synchronization policy is now distributed over multiple, separately maintained source files.
- If the underlying class were to change its synchronization policy by choosing a different lock to guard its state variables, the subclass would subtly and silently break, because it is no longer used the right lock to control concurrent access to the base class's state.
##### Client-side Locking
- For an `ArrayList` wrapped with a `Collections.synchronizedList` wrapper, neither of these approaches - adding a method to the original class or extending the class - works because the client code does not even know the class of the List object returned from the synchronized wrapper factories.
- A third startegy is to extend the functionality of the class without extending the class itself by placing extension code in a `helper` class.

A naive implementation would be 

```
public synchronized boolean putIfAbsent(E x) {
    boolean absent = !list.contains(x);
    if(absent)
        list.add(x)
    return absent;
}
```
`putIfAbsent` synchronizes on the `wrong lock`.
- Whatever lock the List uses to guard its state, it sure isn't the lock on the ListHelper method above. 
- This method only provides an `illusion of synchronization` - the various list operations, while all `synchronized`, use different locks, which means that `putIfAbsent` is not atomic relative to other operations on the List.
- So there is no guarantee that another thread won't modify the list while `putIfAbsent` is executing.
- To make this approach work, we have to use the `same lock that the List uses` by using client-side locking or external locking.
```
public  boolean putIfAbsent(E x) {
    synchronized(list){
    boolean absent = !list.contains(x);
    if(absent)
        list.add(x)
    return absent;
    }
}
```
- If extending a class to add another atomic operation is fragile because it distributes the locking code for a class over multiple classes in an object hierarchy, `client-side locking is even more fragile because it entails putting locking code for class C into classes that are totally unrelated to C`.
##### Composition
- There is a less fragile alternative for adding an atomic operation to an existing class: composition
