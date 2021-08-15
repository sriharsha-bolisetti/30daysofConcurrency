#### Phaser
- A `phaser` is a more flexible cyclic barrier.
- Like a cyclic barrier, a phaser lets a group of threads wait on a barrier; these threads continue after the last thread arrives.
- A phaser also offers the equivalent of a barrier action.
- Unlike a cyclic barrier, which coordinates a fixed number of threads, a phaser can coordinate a variable number of threads which can register anytime.
- To implement this capability, a phaser uses phases and phase numbers.
- A `phase` is the phaser's current state, and this state is identified by an integer based `phase number`
- When the last of the registered threads arrives at the phaser barrier, a phaser advances to the next phase and increments its phase number by 1.
## Parallel Programming In Java
#### Task Creation and Termination
- Concepts of task creation and task termination in parallel programs, using array-sum as an illustrative example.
- Async notation for task creation:- `async (stmt1)` -> causes the parent task i.e. the task executing the async statement to create a new child task to execute the body of the async, (stmt1), asynchronously (i.e. before, after or in parallel) with the remainder of the parent task.
-  There's finish notation for task termination: `finish(stmt2)` causes the parent task to execute `stmt2`, and then wait until `stmt2 and all asynch tasks created within stmt2` have completed.
- Async and finish constructs may be arbitrarily nested.

finish{
    async S1; //asynchronously compute sum of the lower half of the array
    S2; // compute sum of the upper half of the array in paralled with S1
}
S3; // combine the two partial sums after both S1 and S3 have finished.
- Asynchronous method invocation (AMI) - is a design pattern in which the call site is not blocked while waiting for the calling code to finish. Instead, the calling thread is notified when the reply arrives. Polling for a reply is undesired option.
- In most programming languages a called method is executed synchronously i.e in the thread of execution from which it is invoked. 
- If the method takes a long time to compute - e.g. because it is loading data over internet, the calling thread is blocked until the method has finished.
- When this is not desired, it is possible to start a `worker thread` and invoke the method from there.
- In most programming environments this require many lines of code, especially if care is taken to avoid the overhead that may be caused by creating many threads.
- Asynchronous Method Invocation solves this problem in that it augments a potentially long-running object method with an asynchronous variant that return immediately, along with additional methods that make it easy to receive notification of completion or to wait for completion at a later time.
- One common use of AMI is in the active object design pattern.
- Alternatives are synchronous method invocation and future objects.
##### Active Object
- The active object design pattern decouples method execution from method invocation for objects that each reside in their own thread of control.
- The pattern consists of six elements
1. Proxy -> which provides an interface towards clients with publicly accessible methods.
2. Interface which defines the method request on an active object.
3. List of pending requests from the clients
4. Scheduler which decides which request to execute next
5. The implementation of the active object method.
6. A callback or variable for the client to receive the result.

```
public class Myclass{
    private double val;
    //container for tasks
    // decides which request to execute next
    // asyncMode=true means our worker thread processes its local task queue in the FIFO order
    //only single thread may modify internal state
    private final ForkJoinPool fj = new ForkJoinPool(1, ForkJoinPool.defaultForkJoinWorkerThreadFactory, null, true);
      // implementation of active object method
    public void doSomething() throws InterruptedException {
        fj.execute(() -> { val = 1.0; });
    }
 
    // implementation of active object method
    public void doSomethingElse() throws InterruptedException {
        fj.execute(() -> { val = 2.0; });
    }
}
```
#### Fork-Join
- Implement the `async` and `finish` functionality using Java's standard Fork/Join framework.
- In this framework, a task can be specified in the compute() method of a user-defined class that extends the standard RecursiveAction class in FJ framework.
- In the Array Sum example, a class ASum with fields A for the input array, LO, HI for the subrange for which the sum is to be computed, and SUM for the result for that subrange.
- For an instance of this user-defined class, the method call L.fork(), creates a new task that executes L.join() then waits until the computation created by L.fork() has completed.
- Note that `join()` is a lower-level primitive than `finish` because `join()` waits for a specific task, whereas `finish` implicitly waits for all tasks created in its scope.
- To implement the `finish` construct using `join()` operations, you have to be sure to call `join()` on every task created in the finish scope.

```
private static class ASum extends RecursiveAction {
  int[] A; // input array
  int LO, HI; // subrange
  int SUM; // return value
  . . .
  @Override
  protected void compute() {
    SUM = 0;
    for (int i = LO; i <= HI; i++) SUM += A[i];
  } // compute()
}
```
- FJ tasks are executed in a ForkJoinPool which is a pool of Java threads. 
- This pool supports the `invokeAll` method that combines both the `fork` and `join` operations by executing a set of tasks in parallel, and waiting for their completion.
- For example, ForkJoinTask.invokeAll(left, right) implicitly performs `fork()` operations on left and right, followed by join() operations on both objects.