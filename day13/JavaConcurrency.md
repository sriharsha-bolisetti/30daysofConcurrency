#### Concurrency Utilities
- Threads API lets you execute runnable tasks via expressions such as `new java.lang.Thread(new RunnableTask()).start()`-> These expressions `tightly couple task submission with the task's execution mechanics` (run on current thread, new thread, or a thread arbitrarily chosen from a pool of threads)
- A `task` is an object whose class implements the Runnable interface or Callable interface.
- The concurrency utilities include executors as a high-level alternative to low-level thread expressions for executing runnable tasks. 
- Executor is an object whose class directly or indirectly implements the Executor interface, which decouples task submission from task-execution mechanics.
```
public class CalculateE {
    final static int LASTITER = 17;

    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(1);
        Callable<BigDecimal> callable;
        callable = new Callable<BigDecimal>() {
            @Override
            public BigDecimal call()  {
                MathContext mc =
                        new MathContext(100, RoundingMode.HALF_UP);
                BigDecimal result = BigDecimal.ZERO;
                for (int i = 0; i <= LASTITER; i++)
                {
                    BigDecimal factorial =
                            factorial(new BigDecimal(i));
                    BigDecimal res = BigDecimal.ONE.divide(factorial,
                            mc);
                    result = result.add(res);
                }
                return result;
            }
            public BigDecimal factorial(BigDecimal n){
                if (n.equals(BigDecimal.ZERO))
                    return BigDecimal.ONE;
                else
                    return n.multiply(factorial(n.
                            subtract(BigDecimal.ONE)));
            }
        };
        Future<BigDecimal> taskFuture = executor.submit(callable);
        try
        {
            while (!taskFuture.isDone())
                System.out.println("waiting");
            System.out.println(taskFuture.get());
        }
        catch(ExecutionException ee)
        {
            System.err.println("task threw an exception");
            System.err.println(ee);
        }
        catch(InterruptedException ie)
        {
            System.err.println("interrupted while waiting");
        }
        executor.shutdownNow();
    }
}
```
#### Cancellation and Shutdown
- Getting tasks and threads to stop safely, quickly and reliably is not always easy. Java does not provide any mechanism for safely forcing a thread to stop what it is doing.
- Instead, it provides `interruption`, a cooperative mechanism that lets one thread ask another to stop what it is doing.
- The cooperative approach is required because we rarely want a task, thread, or service to stop `immediately` since that could leave shared data structures in an inconsistent state.
- Instead, tasks and services can be coded so that, when requested, they clean up any work currently in progress and then terminate.
- This provides greater flexibility, since the task code itself is usually better able to assess the cleanup required than is the code requesting cancellation.
- End-of-lifecycle issues can complicate the design and implementation of tasks, services, and applications, and this important element of program design is too often ignored.
- `Dealing well with failure, shutdown, and cancellation is one of the characteristics that distinguishes a well-behaved application from one that merely works`.
- An activity is `cancellable` if external code can move it to completion before its normal completion.
- There is no safe way to preemptively stop a thread in Java, and therefore no safe way to preemptively stop a task. There is only cooperative mechanisms, by which the task and the code requesting cancellation follow an agreed-upon protocol.
- One such cooperative mechanism is a `cancellation requested` flag that the task checks periodically; if it finds the flag set, the task terminates early.
- Eg. `PrimeGenerator` which enumerates prime numbers until it is cancelled
- `cancel` method sets the `cancelled` flag and the main loop polls this flag before searching for the next prime number.
```
public class PrimeGenerator implements Runnable{
    private static ExecutorService exec = Executors.newCachedThreadPool();
    private final List<BigInteger> primes = new ArrayList<>();
    private volatile boolean cancelled;
    @Override
    public void run() {
        BigInteger p = BigInteger.ONE;
        while (!cancelled){
            p = p.nextProbablePrime();
            synchronized (this){
                primes.add(p);
            }
        }
    }
    public void cancel(){
        cancelled = true;
    }
    public synchronized List<BigInteger> get(){
        return new ArrayList<BigInteger>(primes);
    }

    static List<BigInteger> aSecondOfPrimes() throws InterruptedException {
        PrimeGenerator generator = new PrimeGenerator();
        exec.execute(generator);
        try {
            SECONDS.sleep(1);
        } finally {
            generator.cancel();
        }
        return generator.get();
    }


    public static void main(String[] args) throws InterruptedException {
        List<BigInteger> bigIntegers = aSecondOfPrimes();
        bigIntegers.forEach(System.out::println);
    }
}
```
- `PrimeGenerator` uses a simple cancellation policy: `client code requests cancellation by calling cancel`
- `PrimeGenerator` checks for cancellation once per prime found and exits when it detects cancellation has been requested.
##### Interruption
- The cancellation mechanism in PrimeGenerator will eventually cause the primeseeking task to exit, but it might take a while.
- If, however, a task uses this approach calls a blocking method such as `BlockingQueue.put`, we could have a more serious problem——`the task might never check the cancellation flag and therefore might never terminate`
- Eg. `BrokenPrimeProducer` -> The producer thread generates primes and places them on a blocking queue. If the producer gets ahead of the consumer, the queue will fill up and put will block. What happens if the consumer tries to cancel the producer task while it is blocked in put? It can call cancel, which will set the cancelled flag——but the producer will never check the flag because it will never emerge from the blocking put(because the consumer has stopped retrieving primes from the queue)
- Certain blocking library methods support `interruption`
- In Practice, `using interruption for anything but cancellation is fragile and difficult to sustain in larger applications`.
- Each thread has a boolean `interrupted status`, interrupting a thread sets its interrupted status to `true`
- `Thread` contains methods for interrupting a thread and querying the interrupted status of a thread.
- The `interrupt` method interrupts the target thread, and `isInterrupted` returns the interrupted status of the target thread.
- The poorly named static `interrupted` method `clears` the interrupted status of the current thread and returns its previous value; this is the only way to clear the interrupted status.
- Blocking library methods like `Thread.sleep` and `Object.wait` try to detect when a thread has been interrupted and return early. They respond to interruption by clearing the interrupted status and throwing `InterruptedException`, indicating that the blocking operation completed early due to interruption.
- If a thread is `interrupted when it is not blocked, its interrupted status is set`, and it is up to the activity being cancelled to poll the interrupted status to detect interruption.
- In this way `interruption is sticky`——if it doesn't trigger an `InterruptedException`, evidence of interruption persists until someone deliberately clears the interrupted status.
- A good way to think about interruption is that it does not actually interrupt a running thread; it just `requests the thread interrupt itself at the next convinient oppurtunity`. These oppurtunities are callec `cancellation points`
- Some methods, such as `wait, sleep, and join`, take such requests seriously, throwing an exception when they receive an interrupt request or encounter an already set interrupt status upon entry.
-  The static interrupted method should be used with caution, because it clears the current thread's interrupted status.
- If you call `interrupted` and it returns `true`, unless you are planning to swallow the interruption you should do something with it——either `throw InterruptedException` or `restore the interrupted status by calling interrupt` again.
- Interruption is usually the most sensible way to implement cancellation.
- Eg. `BrokenPrimeProducer` can be easily fixed and simplified by using interruption instead of a boolean flag to request cancellation. 
- There are two points in each loop iteration where interruption may be detected: in the blocking put call, and by explicitly polling the interrupted status in the loop header.
##### Interruption Policies
- Just as `tasks should have a cancellation policy, threads should have an interruption policy`
- An interruption policy determines `how a thread interprets an interruption request`——what it does when one is detected, what units of work are considered atomic with respect to interruption, and how quickly it reacts to interruption.
- The most sensible interruption policy is some form of thread-level or servicelevel cancellation: exit as quickly as practical, cleaning up if necessary, and possibly notifying some owning entity that the thread is exiting. It is possible to establish other interruption policies, such as pausing or resuming a service, but threads or thread pools with nonstandard interruption policies may need to be restricted to tasks that have been written with an awareness of the policy.
- It is important to distinguish between how `tasks` and `threads` should react to interruption.
- A single interrupt request may have more than one desired recipient——interrupting a worker thread in a thread pool can mean both cancel the current task and shut down the worker thread.
- Tasks do not execute in threads they own, they borrow threads owned by a service such as a thread pool.
- `Code that doesn't own the thread should be careful to preserve the interrupted status` so that the owning code can eventually act on it, even if the guest code acts on interruption as well.
- This is why most blocking library methods simply throw InterruptedException in response to an interrupt.
- A task should not assume anything about the interruption policy of its executing thread unless it is explicitly designed to run within a service that has a specific interruption policy.
- Just as task code should not make assumptions about what interruption means to its executing thread, cancellation code should not make assumptions about the interrpution policy of arbitrary threads.
- A thread should be, interrupted only by its owner, the owner can encapsulate knowledge of thread's interruption policy in an appropriate cancellation mechanism such as a shutdown method.
- Because each thread has its own interruption policy, `you should not interrupt a thread unless you now what interruption means to a thread` 