#### Building Blocks
- Where practical, delegation is one of the most effective strategies for creating thread-safe classes: just let `existing thread-safe classes manage all the state`.
- The platform libraries include a rich set of concurrent building blocks, such as `thread-safe collections` and a variety of `synchronizers` that can coordinate the `control flow of cooperating threads`. 
##### Synchronized Collections
- These classes achieve thread safety by encapsulating their state and synchronizing every public method so that only one thread at a time can access the collection state.
- The synchronized collections are thread-safe, but you may sometimes need to use additional client-side locking to guard compound actions. Common compound actions on collections include iteration (repeatedly fetch elements until the collection is exhausted), navigation (find the next element after this one according to some order), and conditional operations such as put-if-absent (check if a Map has a mapping for key K, and if not, add the mapping (K,V)).
- With a synchronized collection, these compound actions are still technically thread-safe even without client-side locking, but they may not behave as you might expect with other threads can concurrently modify the collection.
- Because the synchronized collections commit to a synchronization policy that supports client-side locking, it is possible to create new operations that are atomic with respect to other collection operations as long as we know which lock to use. The synchronized collection classes guard each method with the lock on the synchronized collection object itself.

###### Iterators and ConcurrentModificationException
- The standard way to iterate a `Collection` is with an `Iterator`, wither explicitly or through the for-each loop syntax, but using iterators does not obviate the need to lock the collection during iteration if other threads can concurrently modify it.
- The iterators returned by the synchronized collections are not designed to deal with concurrent modification, and the are `fail-fast`——meaning that if they detect that the collection has changed since iteration began, they throw the unchecked `ConcurrentModificationException`
- The fai-fast iterators are not designed to be foolprood. They are implemented by associating a modification count with the collection: if the modification count changes during iteration, `hasNext` or `next` throws `ConcurrentModificationException`
- Alternative to locking the collection during the iteration is to clone the collection and iterate the copy instead. 
###### Hidden Iterators
- The greater the distance between the state and the synchronization that guards it, the more likely that someone will forget to use proper synchronization when accessing that state.
- Just as encapsualting an object's state makes it easier to preserve its invariants, encapsulating its synchronization makes it easier to enforce its synchronization policy.

##### Concurrent Collections
- Synchronized collections achieve their thread safety by serializing all access to collection's state. The cost of this approach is poor concurrency.
- The concurrent collections, on the other hand, are designed for concurrent access from multiple threads.
###### ConcurrentHashMap
- The synchronized collections classes hold a lock for the duration of each operation. Some operations, such as HashMap.get or List.contains, may involve mroe work than it is initially obvious: traversing a hash bucket or list to find a specific object entails calling equals on number of candidate objects.
- In a hash-based collection, if hashCode does not spread out hash values well, elements may be unevenly distributed among buckets; in the degenerate case, a poor hash function will a hash table into a linked list. Traversing a long list and calling equals on some or all of the elements can take a long time, and during that time no other thread can access the collection.
- ConcurrentHashMap is a hash-based Map like HashMap, but it uses an entirely different locking strategy that offers better scalability and concurrency. 
- Instead of synchronizing every method on a common lock, restricting access to a single thread at a time, it uses a finer-grained locking mechanisms called `lock striping`
- ConcurrentHashMap, along with the other concurrent collections, further improve on the synchronized collection classses by providing iterators that do not throw `ConcurrentModificationException`, thus eliminating the need to lock the collection during iteration.
- Iterators returned by ConcurrentHashMap are `weakly consistent` instead of fail-fast.
- A weakly consistent iterator can tolerate concurrent modification, traverses elements as they existed when the iterator was constructed, and may (but not guaranteed to) reflect modifications to the collection after the construction of the iterator.
###### CopyOnWriteArrayList
- Copy on write collections derive their thread safety from the fact that as long as an effectively immutable object is properly published, no further synchronization is required when accessing it.
- They implement mutability by `creating and republishing a new copy of the collection every time it is modified`
- Iterators for the copy-on-write collections retain a reference to the backing array that was current at the start of iteration, and since this will never change, they need to synchronize only briefly to ensure visiblity of the array contents.
- As a result, multiple threads can iterate the collection without interference from one another or from threads wanting to modify the collection. 
- The iterators returned by the copy-on-write collections do not throw ConcurrentModificationException and return the elements exactly as they were at the time the iterator was created, regardless of subsequent modifications.
- The copy-on-write collections are reasonable to use only when iteration is far more common than modification.