#### Cyclic Barrier
- A cyclic barrier lets a set of threads wait for each other to `reach a common barrier point`.
- The barrier is `cyclic` because it can be reused after the waiting threads are released.
- This synchronizer is useful in applications involving a fixed-size parity of threads that must occasionally wait for each other.
```
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        float[][] matrix = new float[3][3];
        int counter = 0;
        for (int row = 0; row < matrix.length; row++)
            for (int col = 0; col < matrix[0].length; col++)
                matrix[row][col] = counter++;
        dump(matrix);
        System.out.println();
        Solver solver = new Solver(matrix);
        System.out.println();
        dump(matrix);
    }
    static void dump(float[][] matrix){
        for (float[] floats : matrix) {
            for (int col = 0; col < matrix[0].length; col++)
                System.out.print(floats[col] + " ");
            System.out.println();
        }
    }
}
class Solver{
    final int N;
    final float[][] data;
    final CyclicBarrier barrier;
    class Worker implements Runnable {
        int myRow;
        boolean done = false;

        Worker(int row) {
            myRow = row;
        }

        boolean done() {
            return done;
        }

        void processRow(int myRow) {
            System.out.println("Processing row: " + myRow);
            for (int i = 0; i < N; i++)
                data[myRow][i] *= 10;
            done = true;
        }

        @Override
        public void run() {
            while (!done()) {
                processRow(myRow);
                try {
                    barrier.await();
                } catch (InterruptedException ie) {
                    return;
                } catch (BrokenBarrierException bbe) {
                    return;
                }
            }
        }
    }
    public Solver(float[][] matrix){
        data = matrix;
        N = matrix.length;
        barrier = new CyclicBarrier(N,
                new Runnable()
                {
                    @Override
                    public void run()
                    {
                        mergeRows();
                    }
                });
        for (int i = 0; i < N; ++i)
            new Thread(new Worker(i)).start();

        waitUntilDone();
    }
    void mergeRows()
    {
        System.out.println("merging");
        synchronized("abc")
        {
            "abc".notify();
        }
    }
    void waitUntilDone()
    {
        synchronized("abc")
        {
            try
            {
                System.out.println("main thread waiting");
                "abc".wait();
                System.out.println("main thread notified");
            }
            catch (InterruptedException ie)
            {
                System.out.println("main thread interrupted");
            }
        }
    }
}
```

#### Semaphores
- A `semaphore` maintains a set of permits for `restricting the number of threads that can access a limited resource`.
- A thread attempting to acquire a permit when no permits are available blocks until some other thread releases a permit.
- Semaphores whose current values can be incremented past 1 are known as `counting semaphores`, where semaphores whose current values can be only 0 or 1 are known as `binary semaphores` or `mutexes`. In either case, the current value cannot be negative.
- When the `fairness` setting is false, Semaphore makes no guarantees about the order in which threads acquire permits.
- In particular, `barging` is permitted; that is, a thread invoking `acquire()` can be allocated a permit ahead of a thread that has been waiting——logically the new thread places itself at the head of the queue of waiting threads.
- When `fair` is set to `true`, the semaphore guarantees that threads invoking any of the `acquire()` methods are selected to obtain permits in the order in which their invocation of those methods was processed
- Because FIFO ordering necessarily applies to specific internal points of execution within these methods, it's possible for one threads to invoke `acquire()` before another thread but reach the order point after the other thread, and similarly upon return from the method.
- Also, the untimed tryAcquire() method don't honor the fairness setting; they'll take any available permits.
- Generally, `semaphores used to control resource access should be initialized as fair`, to ensure that no thread is starved out from accessing a resource. 
- When using semaphores for other kinds of synchronization control, the throughput advantages of unfair ordering often outweigh fairness considerations.
```import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreDemo {
    public static void main(String[] args) {


        final Pool pool = new Pool();
        Runnable r = new Runnable() {
            @Override
            public void run() {
                String name = Thread.currentThread().getName();
                try {
                    while (true) {
                        String item;
                        System.out.println(name + " acquiring " +
                                (item = pool.getItem()));
                        Thread.sleep(200 +
                                (int) (Math.random() * 100));
                        System.out.println(name + " putting back " +
                                item);
                        pool.putItem(item);
                    }
                } catch (InterruptedException ie) {
                    System.out.println(name + "interrupted");
                }
            }
        };
        ExecutorService[] executors =
                new ExecutorService[Pool.MAX_AVAILABLE + 1];
        for (int i = 0; i < executors.length; i++)
        {
            executors[i] = Executors.newSingleThreadExecutor();
            executors[i].execute(r);
        }
    }
}

final class Pool {
    public static final int MAX_AVAILABLE = 10;

    private final Semaphore available = new Semaphore(MAX_AVAILABLE, true);

    private final String[] items;

    private final boolean[] used = new boolean[MAX_AVAILABLE];

    Pool(){
        items = new String[MAX_AVAILABLE];
        for (int i = 0; i < items.length; i++)
            items[i] = "I" + i;
    }

    String getItem() throws InterruptedException
    {
        available.acquire();
        return getNextAvailableItem();
    }
    void putItem(String item)
    {
        if (markAsUnused(item))
            available.release();
    }
    private synchronized String getNextAvailableItem()
    {
        for (int i = 0; i < MAX_AVAILABLE; ++i)
        {
            if (!used[i])
            {
                used[i] = true;
                return items[i];
            }
        }
        return null; // not reached
    }
    private synchronized boolean markAsUnused(String item)
    {
        for (int i = 0; i < MAX_AVAILABLE; ++i)
        {
            if (item.equals(items[i]))
            {
                if (used[i])
                {
                    used[i] = false;
                    return true;
                }
                else
                    return false;
            }
        }
        return false;
    }
}```