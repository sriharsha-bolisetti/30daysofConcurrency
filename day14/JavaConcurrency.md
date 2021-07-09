#### Responding To Interruption
- When called an `interruptible blocking method such as Thread.sleep or BlockingQueue.put`, there are two practical strategies for handling `InterruptedException`
1. Propagate the exception, making your method an interruptible blocking method too;
2. Restore the interruption status so the code higher up on the call stack can deal with it.

- If you don't want to or cannot propagate `InterruptedException`, you need to find another way to preserve the interruption request.
- Standard way to do this is to restore the interrupted status by calling `interrupt` again.
- What you should not do is swallow the `InterruptedException` by catching it and doing nothing in the catch block, unless your code is actually implementing the interruption policy for a thread.
- Only code that implements a thread's interruption policy may swallow an interruption request.

#### Handling Abnormal Thread Termination
- It is obvious when a single-threaded console application terminates due to an uncaught exception——the program stops running and produces a stack trace that is very different from typical program output.
- Failure of a thread in a concurrent application is always not so obvious. The stack trace may be printed on the console, but no one may be watching the console. Also, when the thread fails, the application may appear to continue to work, so its failure could go unnoticed. 
- Leading cause of premature thread death is `RuntimeException`. Because these exceptions indicate a programming error or other unrecoverable problem, they are generally not caught. Instead, they will propogate the way up the stack, at which point the default behavior is to print a stack trace on the console and let the thread terminate.
- The consequences of abnormal thread death range from benight to disastrous, depending on the thread's role in the application. Losing a thread from a thread pool can have performance consequences, but an application that runs well with a 50-thread pool will probably run fine with a 49-thread pool too. But losing the event dispatch thread in a GUI application would be quite noticeable——the application would stop processing events and the GUI would freeze.
Eg. Typical Thread-pool worker thread structure
```
public void run(){
    Throwable thrown = null;
    try{
        while(!isInterrupted)
        runTask(getTaskFromWorkQueue());
    } catch (Throwable e){
        thrown = e;
    } finally {
        threadExited(this, thrown);
    }
}
```
- If a task throws an unchecked exception, it allows the thread to die, but not before notifying the framework that the thread has died.
- The framework is being shut down or there are already enough worker threads to meet current demand. 
- ThreadPoolExecutor and Swing use this technique to ensure that a poorly behaved task doesn't prevent subsequent tasks from executing. 
- If you are writing a worker thread class that executes submitted tasks, or calling untrusted external code, use one of these approaches to prevent a poorly written task or plugin from taking down the thread that happens to call it.
##### Uncaught Exception Handlers
- The Thread API also provides the `UncaughtExceptionHandler` facility, which lets you detect when a thread dies due to an uncaught exception. The two approaches are complementary: taken together, they provide defence-indepth against thread leakage.
- When a thread exits due to an uncaught exception, the JVM reports this event to an application-provided `UncaughtExceptionHandler`; if no handler exists the default behavior is to print the stack trace to `System.err`
