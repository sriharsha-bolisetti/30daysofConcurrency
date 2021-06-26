### Threads and Runnables
- Java applications execute via `threads`, which are independent paths of execution through an application's code.
- Each Java application has a `default main thread` that executes the `main()` method. 
- The application can also create threads to perform time-intensive tasks in background so that it can remain responsive to its users.
- These threads execute code sequences encapsulated in objects that are known as `runnables`
- JVM gives each thread its own JVM stack to prevent threads from interfering with each other.
- Sperate stacks let threads keep track of their next instruction to execute, which can differ from thread to thread.
- The stack also provides a thread with its own copy of method parameters, local variables, and return value.
- Java supports threads primarily through its java.lang.Thread class and java.lang.Runnable interface. 
- A single operating system thread is associated with a Thread object.
- The `Runnable` interface supplies the code to be executed by the thread that's associated with a `Thread` object. This code is located in Runnable's void run() method——a thread receives no arguments and returns no value, although it might throw an exceptin.
- Two ways to create a `Runnable` object. First way is to create an anonymous class that implements `Runnable`, as follows
```
Runnable r = new Runnable() {
    @Override
    public void run()
    {
        System.out.println("Hello from thread");
    }
};
```
Java 8 introcuded the lambda expression to more conveniently create a runnable
```
Runnable r = ()  -> System.out.println("Hello from thread");
```
- After creating the `Runnable` object, you can pass it to a `Thread` constructor that receives a `Runnable` argument. 
```
Thread t = new Thread(r);
```  
- A few constructors don't take `Runnable` arguments. For example, `Thread()` doesn't initialize `Thread` to a `Runnable` argument. You must extend `Thread` and override its `run()` method to supply the code to run
```
class MyThread extends Thread
{
   @Override
   public void run()
   {
      // perform some work
      System.out.println("Hello from thread");
   }
}
MyThread mt = new MyThread();
```
###### Getting and Setting Thread State, Thread's Alive status, Execution state
- A Thread object associates state with a thread.
- This state consists of a name, an indication of whether the thread is alive or dead, the execution state of the thread (is it runnable?), the thread's priority, and an indication of whether the thread is daemon or nondaemon.
- You can determine if a thread is alive or dead by calling Thread's boolean `isAlive()` method.

###### Concurrency vs Parallelism
- Parallelism is `a condition that arises when at least two threads are executing simultaneously`
- Concurrency is `a condition that exists when at least two threads are making progress`
- It is a more generalized form of parallelism that can include time-slicing as a form of virtual parallelism.

###### Daemon Threads
- Java lets you classify threads as daemon threads or nondaemon threads.
- A daemon thread is a thread that acts as a `helper to a nondaemon thread and dies automatically when the application's last nondaemon thread dies` so that the application can terminate.
- An application will not terminate when the nondaemon default main thread terminates until all background nondaemon threads terminate. 
- If the background threads are daemon threads, the `application will terminate as soon as the default main thread terminates`.

###### Starting a Thread
- After creating a Thread or Thread subclass object, you start the thread associated with this object by calling Thread's void start() method.
```
Thread t = new Thread(r);
t.start();
```
- Calling `start()` results in the runtime creating the underlying thread and scheduling it for subsequent execution in which the runnable's `run()` method is invoked.
- When execution leaves `run()`, the thread is destroyed and the `Thread` object on which `start()` was called is no longer viable, which is why calling `start()` results in `IllegalThreadStateExecption`
```
public class ThreadDemo {
    public static void main(String[] args) {
        boolean isDaemon = args.length !=0;
        Runnable r = new Runnable() {
            @Override
            public void run() {
                Thread thd = Thread.currentThread();
                while (true){
                    System.out.printf("%s is %s alive and in %s " +
                                    "state%n",
                            thd.getName(),
                            thd.isAlive() ? "" : "not ",
                            thd.getState());
                }
            }
        };

        Thread t1 = new Thread(r, "thd1");
        if(isDaemon)
            t1.setDaemon(true);
        System.out.printf("%s is %salive and in %s state%n",
                t1.getName(),
                t1.isAlive() ? "" : "not ",
                t1.getState());
        Thread t2 = new Thread(r);
        t2.setName("thd2");
        if (isDaemon)
            t2.setDaemon(true);
        System.out.printf("%s is %salive and in %s state%n",
                t2.getName(),
                t2.isAlive() ? "" : "not ",
                t2.getState());
        t1.start();
        t2.start();
    }
}
```

##### Performing More advanced Tasks
- Thread class also supports more advanced tasks, which include interrupting another thread, joining one thread to another thread, and causing a thread to go to sleep.
###### Interrupting Threads
- The Thread class provides an interruption mechanism in which one thread can interrupt another thread. When a thread is interrupted, it throws `java.lang.InterruptedException`
####### void interrupt() 
- Interrupt the thread identified by the Thread object on which this method is called. 
- When a thread is blocked because of a call to one of the Thread's sleep() or join() methods, the thread's interrupted status is cleared and `InterruptedException` is thrown. Otherwise, the interrupted status is set and some other action is taken depending on what the thread is doing.
####### static boolean interrupted() 
- Test whether the current thread has been interrupted, returning `true` in this case.
- The `interrupted status of the thread is cleared` by this method.
####### boolean isInterrupted()
- Test whether this thread has been interrupted, returning true in this case.
- The `interrupted status of the thread is unaffected` by this method.
```
public class ThreadDemoInterruption {
    public static void main(String[] args) {
        Runnable r = new Runnable()
        {
            @Override
            public void run()
            {
                String name = Thread.currentThread().getName();
                int count = 0;
                while (!Thread.interrupted())
                    System.out.println(name + ": " + count++);
            }
        };
        Thread thdA = new Thread(r);
        Thread thdB = new Thread(r);
        thdA.start();
        thdB.start();
        while (true){
            double n = Math.random();
            if (n >= 0.49999999 && n <= 0.50000001)
                break;
        }
        thdA.interrupt();
        thdB.interrupt();
    }
}
```
###### Joining Threads
- A thread (such as the default main thread) will occasionally start another thread to perform a lengthy calculation, download a large file, or perform some other time-consuming activity. After finishing its other tasks, the thread that started the `worker thread` is ready to process the results of the worker thread and waits for the worker thread to finish and die.
- The `Thread` class provides three `join()` methods that allow the invoking thread to wait for the thread on whose `Thread` object `join()` is called to die.
### What did I learn today?
- Thread API - Basics -> start(), run(), Runnable, Thread, Thread states
### Resources
- Java Threads and the Concurrency Utilities
by Jeff Friesen
