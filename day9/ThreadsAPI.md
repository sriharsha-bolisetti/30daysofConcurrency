##### Beware of Liveness Problems
- The term `liveness` refers to something beneficial happening eventually.
- A liveness failure occurs when an application reaches a state in which `it can make no further progress`.
- In a single-threaded application, an infinite loop would be an example. Multithreaded applications face the additional liveness challenges of deadlock, livelock, and starvation.
1. Deadlock
2. Livelock -> Thread x keeps retrying an operation that `will always fail`. It `cannot make progress` for this reason.
3. Starvation -> Thread x is continuially denied (by the scheduler) access to a needed resource in order to make progress. Perhaps the scheduler executes higher-priority threads before lower-priority threads and there is always a higher-priority thread available for execution. Starvation is also commonly referred to as `indefinite postponement`
- Synchronization exhibits two properties - mutual exclusion and visibility. The `synchronized` keyword is associated with both properties.
- Java also provides a weaker form of synchronization involving visibility only, and associates only this property with the `volatile` keyword.
#### Waiting and Notification
- Java provides a small API that supports communication between threads.
- Using this API, one thread waits for a `condition` (a prerequisite for continued execution) to exist.
- The `wait()` methods wait for a condition to exist; the `notify()` and `notifyAll()` methods notify waiting threads when the condition exists.
- `wait()` -> cause the current thread to wait until another thread invokes `notify()` or `notifyAll()` method for this object, or for some other thread to interrupt the current thread while waiting.
- `notifyAll()` -> Wake up all threads that are waiting on this Object's monitor. The awakened threads will not be able to proceed until the current thread reliquishes the lock on this object. The awakened threads will compete in the usual manner with any other threads that might be actively competing to synchronize on this object; for example, the awakened threads enjoy no reliable privilege or disadvantage in being the next thread to lock this object.
- This API leveragees an object's `condition queue`, which is a data structure that stores threads waiting for a condition to exist.
- The waiting threads are known as the `wait set`
- Because the condition queue is tightly bound to an object's lock, all five methods must be called from within a synchronized context (the current thread must be owner of the object's monitor); otherwise `java.lang.IllegalMonitorStateException` is thrown.
- Following code/pseudocode fragment demonstrates the noargument `wait()` method:
```
synchronized(obj){
    while(<condition does not holdv>)
    obj.wait();
    //perform an action that's approproate to condition
}
```
- The wait() method is called from within a synchronized block that synchronizes on the same object as the object on which `wait()` is called (obj).
- Because of the possibility of `spurious wakeups` (a thread wakes up without being notified, interrupted, or timing out), `wait()` is called from within a `while` loop that tests for the condition holding and reexecutes `wait()` when the condition still doesn't hold.
- After the `while` loop exits, the condition exists and an action appropriate to the condition can be performed.
- The following code fragment demonstrates the notify() method, which notifies the waiting thread in the previous example:
```
synchronized(obj){
    obj.notify();
}
```
- `notify()` is called from a critical section guarded by the same object (obj) as the critical section for wait() method.
- Also, notify() is called using the same obj reference. 

###### Producers and Consumers
- A classic example of thread communication involving conditions is the relationship between a producer thread and a consumer thread.
- The producer thread produces data items to be consumed by the consumer thread.
- Each produced data item is stored in a shared variable.
- Imagine that the threads are running at different speeds. The producer might produce a new data item and record it in the shared variable before the consumer retrieves the previous data item for processing. Also, the consumer might retrieve the contents of the shared variable before a new data item is produced.
- To overcome those problems, the producer thread must wait until it's notified that the previously produced data item has been consumed, and the consumer thread must wait until it's notified that a new data item has been produced.
