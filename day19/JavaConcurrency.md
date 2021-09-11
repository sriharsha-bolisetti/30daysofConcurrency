#### Amdhal's Law
- q = Fraction of workload that is sequential
- Maximum speedup <= 1/q
- If q=0.5 Speedup is <= 2
- If q=0.1 speedup is <= 10
- Useful rule of thumb when trying to figure out how tto provision a system using a given workload.
- Span >= q * work
- We already know Speedup <= Work/Span
- Speedup <= Work/q*Work
- Speedup = 1/((1-P)+p/n)
- 1-P is the sequential piece.
- N = number of parallel processors  
- Speedup will stop going faster 

#### Futures - Tasks with Return Values
- The concept of asynchronous tasks can be extended to include `future tasks` and `future objects` (also known as `promise` objects).
- Future tasks are tasks with return values, and a future object is a `handle` for accessing a task's return value.
- There are two key operations that can be performed on a future object, A:
1. Assignement - A can be assigned a reference to a future object returned by a task of the form, `future{ <task-with-return-value>}`. The content of the future object is constrained to be `single assignment` and cannot be modified after the future task has returned.
2. Blocking Read -> the operation, `A.get()` waits until the task associated with future object A has completed, and then propagates the task's return value as the value returned by `A.get()`. Any statement, S, executed after A.get() can be assured that the task associated with future object A must have completed before S starts execution.

These operations are carefully defined to avoid the possibility of a race condition on a task's return value, which why futures are well suited for functional parallelis.

# Futures in Fork/Join
- Some key differences between future tasks and regular tasks in the FJ framework are as follows:
1. A future task extends Recursive Task class in Fork Join framework, instead of RecursiveAction as in regular tasks.
2. The compute() method of a future task must have a non-void return type, whereas it has a void return type for a regular tasks.
3. A method call like left.join() waits for the task referred to by object left in both cases, but also provides the tasks return value in case of future tasks

- Memoization
- Streams
#### Determinism
- Structural deterministic 
Same input -> Same comp graph.
#### Some other basic concepts review

#### Concurrency versus parallelism

- Concurrency -> when you have more than one task in a single processor with a single core. In this case, the operating system's task scheduler quickly switches from one task to another, so it seems that all the tasks run simultaneously.
- Parallelism -> when you have more than one task running simultaneously on different computers, processors, or cores inside a processor.

#### Synchronization

- Synchronization is the coordination of two or more tasks to get the desired results. We have two kinds of synchronization:

1. Control synchronization: When, for example, one task depends on the end of another task, the second task can't start before the first has finished
2. Data access synchronization: When two or more tasks have access to a shared variable and only one of the tasks can access the variable

- A `critical section` is a piece of code that can be only executed by one task at a time because of its access to a shared resource.
- Mutual exclusion is the mechanism used to guarantee this requirement and can be implemented in different ways.
- Synchronization helps you avoid some errors you might have with concurrent tasks, but it introduces some overhead to your algorithm. You have to calculate the number of tasks very carefully, which can be performed independently without intercommunication you will have in your parallel algorithm. It's the `granularity` of your concurrent algorithm
- If you have a `coarse-grained granularity` (big tasks with low intercommunication), the overhead due to synchronization will be low. However, maybe you won't benefit from all the cores of your system.
- If you have a `fine-grained granularity` (small tasks with high intercommunication), the overhead due to synchronization will be high, and perhaps the throughput of your algorithm won't be good.

There are different mechanisms to get synchronization in a concurrent system. The most popular mechanisms from a theoretical point of view are:

1. A `semaphore` is a mechanism that can be used to `control the access to one or more units of a resource`. It has a variable that stores the number of resources that can be used and two atomic operations to manage the value of the variable. A mutex (short for mutual exclusion) is a special kind of semaphore that can take only two values (resource is free and resource is busy), and only the process that sets the mutex to busy can release it. A mutex can help you to avoid race conditions by protecting a critical section.
2. A `monitor` is a mechanism to get `mutual exclusion over a shared resource`. It has a mutex, a condition variable, and two operations to wait for the condition and signal the condition. Once you signal the condition, only one of the tasks that are waiting for it continues with its execution.

- A piece of code (or a method or an object) is thread-safe if all the users of shared data are protected by synchronization mechanisms. A non-blocking, compare-and-swap (CAS) primitive of the data is immutable, so you can use that code in a concurrent application without any problems.
- An atomic operation is a kind of operation that appears to occur instantaneously to the rest of the tasks of the program. In a concurrent application, you can implement an atomic operation with a critical section to the whole operation using a synchronization mechanism.
- An atomic variable is a kind of variable that has atomic operations to set and get its value. You can implement an atomic variable using a synchronization mechanism or in a lock-free manner using CAS that doesn't need synchronization.

##### Shared memory versus message passing

- Tasks can use two different methods to communicate with each other.
- The first one is `shared memory` and, normally, it is used when the tasks are running on the same computer. The tasks use the same memory area where they write and read values. To avoid problems, `the access to this shared memory has to be in a critical section protected by a synchronization mechanism`.
- The other synchronization mechanism is `message passing` and, normally, it is used when the tasks are running on different computers. When tasks needs to communicate with another, it sends a message that follows a predefined protocol. This communication can be synchronous if the sender keeps it blocked waiting for a response or asynchronous if the sender continues with their execution after sending the message.

##### A methodology to design concurrent algorithms

- The starting point - a sequential version of the algorithm

###### Step 1 - Analysis

- In this step, we are going to analyze the sequential version of the algorithm to look for the parts of its code that can be executed in a parallel way.
- We should pay special attention to those parts that are executed most of the time or that execute more code because, by implementing a concurrent version of those parts, we're going to get a greater performance improvement.
- Good candidates for this process are loops, where one step is independent of the other steps, or portions of code are independent of other parts of the code (for example, an algorithm to initialize an application that opens the connections with the database, loads the configuration files, and initializes some objects; all these tasks are independent of each other).

##### Step 2 - Design

- Once you know what parts of the code you are going to parallelize, you have to decide how to do that parallelization.
- The changes in the code will affect two main parts of the application:

1. The structure of the code
2. The organization of the data structures

- You can take two different approaches to accomplish this task:
- `Task decomposition`: You do task decomposition when you split the code into two or more independent tasks that can be executed at once. Maybe some of these tasks have to be executed in a given order or have to wait at the same point. You must use synchronization mechanisms to get this behavior.
- `Data decomposition`: You do data decomposition when you have multiple instances of the same task that work with a subset of the dataset. This dataset will be a shared resource, so if the tasks need to modify the data, you have to protect access to it, implementing a critical section.
- Another important point to keep in mind is the granularity of your solution.
- The objective of implementing a parallel version of an algorithm is to achieve improved performance, so you should use all the available processors or cores.
- On the other hand, when you use a synchronization mechanism, you introduce some extra instructions that must be executed. If you split the algorithm into a lot of small tasks (fine-grained granularity), the extra code introduced by the synchronization can provoke performance degradation.
- If you split the algorithm into fewer tasks than cores (coarse-grained granularity), you are not taking advantage of all resources. Also, you must take into account the work every thread must do, especially if you implement a fine-grained granularity.
- If you have a task longer than the rest, that task will determine the execution time of the application. You have to find the equilibrium between these two points.

#### Step 3 - implementation

- The next step is to implement the parallel algorithm using a programming language and, if it's necessary, a thread library.

#### Step 4 - testing

- After finishing the implementation, you should test the parallel algorithm. If you have a sequential version of the algorithm, you can compare the results of both algorithms to verify that your parallel implementation is correct.
- Testing and debugging a parallel implementation are difficult tasks because the order of execution of the different tasks of the application is not guaranteed.

#### Step 5 - Tuning

- The last step is to compare the throughput of the parallel and the sequential algorithms. If the results are not as expected, you must review the algorithm, looking for the cause of the bad performance of the parallel algorithm.
- You can also test different parameters of the algorithm (for example, granularity, or number of tasks) to find the best configuration.
- There are different metrics to measure the possible performance improvement you can obtain parallelizing an algorithm.
  The three most popular metrics are:
- Speedup: This is a metric for relative performance improvements between the parallel and the sequential versions of the algorithm: Seedup = Tsequential/Tconcurrent
- Here, Tsequential is the execution time of the sequential version of the algorithm and Tconcurrent is the execution time of the parallel version.
- Amdahl's law: Used to calculate the maximum expected improvement obtained with the parallelization of an algorithm:
  Speedup <= 1/ (1-p)+ p/N
- Here, P is the percentage of code that can be parallelized and N is the number of cores of the computer where you're going to execute the algorithm.
- For example, if you can parallelize 75% of the code and you have four cores, the maximum speedup will be given by the following formula: Speedup <= 1/(1-0.75)+(0.75/4) <= 1/0.44 <= 2.29
- Gustafson-Barsis' law: Amdahl's law has a limitation. It supposes that you have the same input dataset when you increase the number of cores, but normally, when you have more cores, you want to process more data. Gustafson's law proposes that when you have more cores available, bigger problems can be solved at the same time using the following formula:
  `Speedup = P- alpha*(P-1)`
- Here, N is the number of cores and P is the percentage of parallelizable code.
- First of all, not every algorithm can be parallelized. For example, if you have to execute a loop where the result of iteration depends on the result of the previous iteration, you can't parallelize that loop. Recurrent algorithms are another example of algorithms that can be parallelized for a similar reason.
- Another important thing you have to keep in mind is that the sequential version with better performance of an algorithm can be a bad starting point to parallelize it. If you start parallelizing an algorithm and you find yourself in trouble because you cannot easily find independent portions of the code, you have to look for other versions of the algorithm and verify that the version can be parallelized in an easier way.
