#### Blocking Queues and the Producer-Consumer Pattern
- Blocking queues support the `producer-consumer` design pattern. 
- Blocking queues provide the blocking `put` and `take` methods as well as the timed equivalents `offer` and `poll`. If the queue is full, `put` blocks until space becomes available; if the queue is empty, `take` blocks until an element is available.
- Queues can be bounded or unbounded; unbounded queues are never full, so a put on an unbounded queue never blocks.
- One of the most common producer-consumer designs is a thread pool coupled with a work queue - this pattern is embodied in the `Executor` task execution framework.
- While the producer-concumer pattern enables producer and consumer `code` to be decoupled from each other, their `behavior` is still `coupled indirectly through the shared work queue`.
- It is tempting to assume that the consumers will always keep up, so that you need not place any bounds on the size of work queues, but this is a prescription for rearchitecting system later.
- `Build resource management into your design early using blocking queues - it is a lot easier to do this up front than to retrofit it later`
- Blocking queues make this easy for a number of situations, but if blocking queues don't fit easily into your design, you can create other blocking data structures using Semaphore.
- For mutable objects, producer-consumer designs and blocking queues facilitate `serial thread confinement` for handing off ownership of objects from producers to consumers.
- A thread-confined object is owned exclusively by a single thread, but that ownership can be `transferred` by publishing it safely where `only one other thread will gain access to it` and ensuring that the publishing thread does not access it after handoff.
- The safe publication ensures that the object's state is visible to the new owner, and since the original owner will not touch it again, it is now confined to the new thread. The new owner may modify it freely since it has exclusive access.
- `Object pools exploit serial thread confinement`, `lending` an object to a requesting thread.
- As long as the pool contains sufficient internal synchronization to publish the pooled object safely, and as long as the `clients do not themselves publish the pooled object or use it after returning it to the pool,` `ownership` can be transferred `safely` from thread to thread.
- One could also use other publication mechanisms for transferring ownership of a mutable object, but it is necessary to ensure that only one thread receives the object being handed off. 
#### Deques and Work Stealing
- Java 6 adds another two collection types - Deque and BlockingDeque, that extend Queue and BlockingQueue.
- A Deque is a doubleended queu that allows efficient insertion and removal from both the head and the tail. Implementations include ArrayDeque and LinkedBlockingDeque.
- Just as blocking queues lend themselves to the produer-consumer pattern, deques lend themselves to a related pattern called `work stealing`
- A producer-consumer design has one shared work queue for all consumers; in a work stealing design, `every consumer has its own deque`
- If a consumer exhausts the work in its own deque, it can `steal work from the tail of someone else's deque`
- Work stealing can be more scalable than a traditional producer-consumer design because workers don't contend for a shared work queue; most of the time they access their own deque, reducing contention.
- When a worker has to access another's queue, it does so from the tail rather than from the head, further reducing contention. 
- Work stealing is well suited to problems in which consumers are also producers——when performing a unit of work is likely to result in the identification of more work.
- For example, processing a page in a web crawler usually results in the identification of new pages to be crawled. 
- Similarly, many graph-exploring algorithms, such as marking the heap during garbage colleciton, can be efficiently parallelized using work stealing.
- When a worker identifies a new unit of work, it places it at the end of its own deque (or alternatively, in a `work sharing` design,  on that of another worker); when the deque is empty, it looks for work at the end of someone else's deque, ensuring that each worker stays busy.
#### Synchronizers
- Blocking queues are unique among the collections classes - not only do they act as containers for objects, but they can `also coordinate the control flow` of producer and consumer threads because `take` and `put` block until the queue enters the desired state (not empty or not full).
- A `synchronizer` is any object that `coordinates the control flow of threads based on its state`.
- Blocking queues can act as synchronizers; other types of synchronizer includes semaphores, barriers, and latches.
- All synchronizers share certain properties - 
1. they encapsulate state that determines whether threads arriving at the synchronizer should be allowed to pass or forced to wait, 
2. provide methods to manipulate that state, and 
3. provide methods to wait efficiently for the synchronizer to enter the desired state.
##### Latches
- A `latch` is a synchronizer that can delay the progress of threads until it reaches its `terminal` state.
- A latch acts as a gate - until the latch reaches the terminal state the gate is closed and no thread can pass, and in the terminal state the gate opens, allowing all threads to pass.
- Once the `latch reaches the terminal state, it can not change state again`, so it remains open forever.
- Latches can be used to ensure that certain activities do not proceed until other one-time activities complete such as 
1. Ensuring that a computation `does not proceed until resources it needs have been initialized`. A simple binary (two-state) latch could be used to indicate "Resource R has been initialized", and any activity that requires R would wait first on this latch.
2. Ensuring that a service `does not start until other services on which it depends have started`. Each service would have an associated binary latch; starting service S would involve first waiting on the latches for other services on which S depends, and then releasing the S latch after startup completes so any services that depend on S can proceed.
3. Waiting until all the parties involved in an activity, for instance the players in a multi-player game, are ready to proceed. In this case, the latch reaches the terminal state after all the players are ready.
- `CountDownLatch` is a flexible latch implementation that can be used in any of these situations; it allows one or more threads to wait for a set of events to occur. The latch state consists of a counter initialized to a positive number, representing the number of events to wait for. The countDown method decrements the counter, indicating that an event has occurred, and the await methods wait for the counter to reach zero, which happens when all the events have occurred. 
##### FutureTask
- `FutureTask` also acts like a latch.
- A computation represented by a `FutureTask` is implemented with a `Callable`, the result-bearing equivalent of `Runnable`, and can be in one of the three states - waiting to run, running or completed.
- Completion subsumes all the ways a computation can complete, including normal completion, cancellation, and exception.
- Once a `FutureTask` enters the completed state, it stays in that state forever.
- The behavior of `Future.get` depends on the state of the task. If it is completed, `get` returns the result immediately, and otherwise blocks until the task transitions to the completed state and then returns results or throws an exception.
- `FutureTask` conveys the result from the thread executing the computation to the threads retrieving the result. The specification of FutureTask guarantees that this transfer constitutes a safe publication of the result.
- `FutureTask` is used by the Executor framework to represent asynchronous tasks, and can also be used to represent any potentially lengthy computation that can be started before the results are needed. 
##### Semaphores
- `Counting semaphores` are used to `control the number of activities that can access a certain resource or platform a given action at the same time`.
- Counting semaphores can be used to implement resource pools or to impose a bound on a collection. 
- A semaphore manages a set of virtual `permits`, the initial number of permits is passed to the `Semaphore` constructor.
- Activities can acquire permits and release permits when they are done with them.
- If no permit is available, `acquire` blocks until one is 
- The `release` method returns a permit to the semaphore.
- Semaphores are useful for implementing resource pools such as database connection pools. While it is easy to construct a fixed-sized pool that fails if you request a resource from an empty pool, what you really want is to `block` if the pool is empty and unblock when it becomes nonempty again.
- If you initialize a `Semaphore` to the pool size, `acquire` a permit before trying to fetch a resource from the pool, and `release` the permit after putting a resource back in the pool, `acquire` blocks until the pool becomes nonempty. 
- A semaphore can also be used to turn any collection into a blocking bounded collection.
##### Barriers
- `Barriers` are similar to latches in that they block a group of threads until some event has occurred.
- The key difference is that with a barrier, all the threads `must come together at a barrier point at the same time` in order to proceed.
- Latches are for waiting for `events`, barriers are for waiting for `other threads`
- `CyclicBarrier` allows a fixed number of parties to rendezvous repeatedly at a `barrier point` and is useful in parallel iterative algorithms that break down a problem into a fixed number of independent subproblems.
- Threads call `await` when they reach the barrier point, and `await` blocks until all the threads have reached the barrier point.
- If all threads meet at the barrier point, the barrier has been successfully passed, in which cases all threads are released and the barrier is reset so it can be used again.
- If a call to `await` times out or a thread blocked in await is interrupted, then the barrier is considered broken and all outstanding calls to await terminate with `BrokenBarrierException`.
- If the barrier is successfully passed, await returns a unique arrival index for each thread, which can be used to `elect` a leader that takes some special action in the next iteration.
- `Barriers` are often used in simulations, where the work to calcualte one step can be done in parallel but all the work associated with a given step must complete before advancing to the next step.