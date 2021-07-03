### Task Execution
- Most concurrent applications are organized around the execution of `tasks`: abstract, discrete units of work. 
- Dividing the work of application into tasks simplifies program organization, facilitates error recovery by providing natural transaction boundaries, and promotes concurrency by providing a natural structure for parallelizing work.

##### Executing Tasks in Threads

- The first step in organizing a program around task execution is identifying sensible `task boundaries`
- Independence facilitates concurrency, as independent tasks can be executed in parallel if there are adequate processing resources.
- For greater flexibility in scheduling and load balancing tasks, each task should also reprsent a small fraction of your application's processing capacity.
- Server applications should exhibit both `good throughput` and `good responsiveness` under normal load.
- Application providers want applications to support as many users as possible, so as to reduce provisioning costs per user; users want to get their response quickly.
- Further, applications should exhibit `graceful degradation` as they become overloaded, rather than simply falling over under heavy load.
- Choosing good task boundaries, coupled with a sensible `task execution policy` can help achieve these goals.
#### Executor Framework
- `Tasks` are logical units of work, and threads are a mechanism by which tasks can run asynchronously.
- Executing tasks sequentially in a single thread approach suffers from poor responsiveness and throughput.
- Executing tasks in its own thread suffers from poor resource management.
- `Thread pools` offer the same benefit for thread management as `bounded queues` to prevent an overloaded application from running out of memory.
-  `java.util.concurrent` provides a flexible thread pool implementation as part of the Executor framework.
- The `primary abstraction for task execution` in the Java class libraries is not Thread, but `Executor`
```
public interface Executor{
    void execute(Runnable command);
}
```
- `Executor` may be a simple interface, but it forms the basis for a flexible and powerful framework for asynchronous task execution that supports a wide variety of task execution policies.
- It provides a standard means of decoupling `task submission` from `task execution`, describing tasks with `Runnable`
- The `Executor` implementations also provide `lifecycle support and hooks` for adding statistics gathering, application management and monitoring.
- `Executor` is based on producer-consumer pattern.
###### Thread Pools
- A thread pool is tightly bound to a `work queue` holding tasks waiting to be executed.
- Worker threads have a simple life: request the next task from the work queue, execute it, and go back to waiting for another task.
- Executing tasks in pool threads has a number of advantages over the thread-per-task approach.
- Reusing an existing thread instead of creating a new one amortizes thread creation and teardown costs over multiple requests.
- Since the worker thread often already exists at the time the request arrives, `the latency associated with thread creation does not delay task execution`, thus improving responsiveness.
- By properly tuning the size of the thread pool, you can have enough threads to keep the processors busy while not having so many that your application runs out of memory or thrashes due to competition among threads for resources.
##### Finding Exploitable Parallelism
- The executor framework makes it easy to specify an execution policy, but in order to use an `Executor` you have to be able to describe your task as a `Runnable`
- In most server applications, there is an obvious task boundary: a single client request.
- Sometimes good task boundaries are not quite so obvious.
- There may also be exploitable parallelism within a single client request in server applications.
##### Result Bearing Tasks - Callable and Future
- The `Executor` framework uses `Runnable` as its basic task representation.
- `Runnable` is a fairly limiting abstraction.
- `run()` cannot return a value or throw checked exception, although it can have `side effects` such as writing to a log file or placing a result in a shared data structure.
- Many tasks are effectively `deferred computations`——executing a database query, fetching a resource over the network, or computing a complicated function. 
- For these types of tasks, `Callable` is a better abstraction: it expects that the main entry point, `call` will return a value and anticipates that it might throw an exception.
- Executors includues several utility methods for wrapping other types of tasks, including `Runnable`, `java.security.PrivilegedAction`, with a `Callable`
- To express a non-value-returining task with `Callable`, use `Callable<Void>`
- `Runnable` and `Callable` describe abstract computational tasks.
- Tasks are usually finite: they have a clear starting point and they eventually terminate.
- The lifecycle of a task executed by an `Executor` has four phases: created, submitted, started, and completed.
- Since tasks can take a long time to run, we also want to be able to `cancel a task`
- In the `Executor` framework, tasks that have been submitted but not yet started can always be cancelled, and tasks that have started can sometimes be cancelled if they are responsive to interruption.
- `Future` represents the lifecycle of a task and provides methods to test whether the task has completed or been cancelled, retrieve its result, and cancel the task.
- Implicit in the specification of `Future` is that task lifecycle `can only move forwards, not backwards`. Once a task is completed, it stays in that state forever.
- The behavior of `get` varies depending on the task state. It returns immediately or throws an Exception if the task has already completed, but if not it blocks until the task completes.
- If the task completes by throwing an exceptoin, `get rethrows it wrapped in an ExecutionException`, if it is cancelled, get throws `CancellationException`
- If get throws ExecutionException, the underlying exception can be retrieved with getCause.
```
public interface Callable<V>{
    V call() throws Exception;
}

public interface Future<V>{
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException, CancellationException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, CancellationException, TimeoutException;
}
```
- There are several ways to create a `Future` to describe a task.
- The submit methods in ExecutorService all return a `Future`, so that you can submit a `Runnable` or a `Callable` to get an executor and get back a `Future` that can be used to retrieve the result or cancel the task.
- You can explicitly instantiate a `FutureTask` for a given `Runnable` or `Callable`. Because `FutureTask` implements `Runnable`, it can be submitted to an `Executor` for execution or executed directly by calling its `run` method.
- Submitting a Runnable or Callable to an Executor constitutes a `safe publication` of the Runnable or Callable from the submitting thread to the thread that will eventually execute the task. 
- Similarly, setting the result value for a Future constitutes a `safe publication of the result from the thread` in which it was computed to any thread that retrieves it via `get`.
##### CompletionService: Executor Meets BlockingQueue
- If you have a batch of computations to submit to an `Executor` and you want to retrieve their results as they become available, you could retain the `Future` associated with each task and repeatedly poll for completion by calling `get` with a timeout of zero. This is possible, but tedious. Fortunately there is a better way: a `completion service`
- `CompletionService` combines the functionality of an `Executor` and a `BlockingQueue`
- You can submit `Callable` tasks to it for execution and use the queuelike methods `take` and `poll` to retrieve completed results, packaged as `Futures`, as they become available.
- `ExecutorCompletionService` implements CompletionService, delegating the computation to an Executor.
```

public abstract class Renderer {
    interface ImageData {
    }
    interface ImageInfo {
        ImageData downloadImage();
    }
    abstract void renderText(CharSequence s);
    abstract List<ImageInfo> scanForImageInfo(CharSequence s);
    abstract void renderImage(ImageData i);
    private final ExecutorService executor;
    Renderer(ExecutorService executor){
        this.executor = executor;
    }
    void renderPage(CharSequence source){
        List<ImageInfo> info = scanForImageInfo(source);
        CompletionService<ImageData> completionService = new ExecutorCompletionService<ImageData>(executor);
        for (final ImageInfo imageInfo: info){
            completionService.submit(new Callable<ImageData>() {
                @Override
                public ImageData call() throws Exception {
                    return imageInfo.downloadImage();
                }
            });
            renderText(source);
            try {
                for (int t = 0, n = info.size(); t < n; t++){
                    Future<ImageData> f = completionService.take();
                    ImageData imageData = f.get();
                    renderImage(imageData);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } catch (ExecutionException e){
                System.out.println(e.getCause());
            }
        }
    }
}
```
