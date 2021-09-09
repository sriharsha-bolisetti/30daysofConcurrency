### INTRO AGAIN
- You can avoid confusion by remembering that two players are always involved in running a thread: a Java language Thread object that represents the thread itself and an arbitrary target object that contains the method that the thread is to execute.
- All execution in Java is associated with a Thread object, beginning with a “main” thread that is started by the Java VM to launch your application.
- A new thread is born when we create an instance of the java.lang.Thread class.
- The `Thread` object represents a real thread in the Java interpreter and `serves as a handle for controlling and coordinating its execution`.
- With it, we can start the thread, wait for it to complete, cause it to sleep for a time, or interrupt its activity
- The constructor for the Thread class accepts information about where the thread should begin its execution.
- Conceptually, we would like to simply tell it what method to run. There are a number of ways to do this; Java 8 allows method references that would do the trick. Here we will take a short detour and use the java.lang.Runnable interface to create or mark an object that contains a “runnable” method
- Runnable defines a single, general-purpose run() method:

```
   public interface Runnable {
         abstract public void run();
    }
```

- Every thread begins its life by executing the `run()` method in a `Runnable` object, which is the "target object" that was passed to the thread's constructor.
- The `run()` method can contain any code, but it must be public, take no arguments, have no return value, and throw no checked exceptions.
- Any class that contains an appropriate run() method can declare that it implements the Runnable interface
- An instance of this class is then a runnable object that can serve as the target of a new thread.

#### Creating and Starting Threads

- A newly born thread remains idle until we give it a figurative slap on the bottom by calling its start() method.
- The thread then wakes up and proceeds to execute the run() method of its target object. start() can be called only once in the lifetime of a thread.
- Once a thread starts, it continues running until the target object’s run() method returns (or throws an unchecked exception of some kind).
- The Runnable interface lets us make an arbitrary object the target of a thread, as we did in the previous example. This is the most important general usage of the Thread class.
- In most situations in which you need to use threads, you’ll create a class (possibly a simple adapter class) that implements the Runnable interface.

#### Controlling Threads

- The static Thread.sleep() method causes the currently executing thread to wait for a designated period of time (give or take), without consuming much (or possibly any) CPU time.
- The methods wait() and join() coordinate the execution of two or more threads.
- The interrupt() method wakes up a thread that is sleeping in a sleep() or wait() operation or is otherwise blocked on a long I/O operation

#### Sleep()

- The sleep() method may throw an InterruptedException if it is interrupted by another thread via the interrupt() method. The thread can catch this exception and take the opportunity to perform some action—such as checking a variable to determine whether or not it should exit—or perhaps just perform some housekeeping and then go back to sleep.

#### Join()

- If you need to coordinate your activities with another thread by waiting for it to complete its task, you can use the join() method.
- Calling a thread’s join() method causes the caller to block until the target thread completes.
- Alternatively, you can poll the thread by calling join() with a number of milliseconds to wait.
- This is a very coarse form of thread synchronization.
- Java supports more general and powerful mechanisms for coordinating thread activity including the wait() and notify() methods, as well as higher-level APIs in the java.util.concurrent package.

#### Interrupt()

- `interrupt()` method as a way to wake up a thread that is idle in a sleep(), wait(), or lengthy I/O operation. Any thread that is not running continuously (not a “hard loop”) must enter one of these states periodically and so this is intended to be a point where the thread can be flagged to stop.
- When a thread is interrupted, its `interrupt status` flag is set. This can happen at any time, whether the thread is idle or not.
- The thread can test this status with the isInterrupted() method.
- isInterrupted(boolean), another form, accepts a Boolean value indicating whether or not to clear the interrupt status. In this way, a thread can use the interrupt status as a flag and a signal.

#### Death of a Thread

- A thread continues to execute until one of the following happens:

1. It explicitly returns from its target run() method.
2. It encounters an uncaught runtime exception.
3. The evil and nasty deprecated stop() method is called.

- What happens if none of these things occurs, and the run() method for a thread never terminates? The answer is that the thread can live on, even after what is ostensibly the part of the application that created it has finished.
- This means we have to be aware of how our threads eventually terminate, or an application can end up leaving orphaned threads that unnecessarily consume resources or keep the application alive when it would otherwise quit.
- In many cases, we really want to create background threads that do simple, periodic tasks in an application. The setDaemon() method can be used to mark a thread as a daemon thread that should be killed and discarded when no other nondaemon application threads remain.
- Normally, the Java interpreter continues to run until all threads have completed. But when daemon threads are the only threads still alive, the interpreter will exit.
