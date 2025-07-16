# 🧵 Multithreading Cheatsheet

Quick reference and guide for multithreading concepts and code.

---

## 🧠 Core Threading Concepts

### ✅ Difference Between Processes vs Threads

#### 🔹 Processes

- **Independent Memory Space**: Isolated virtual address space.
- **Own Resources**: Code, heap, stack, file handles, etc.
- **Heavyweight**: Costly creation and context switching.
- **Robust**: Failure doesn't affect others.
- **Communicates via IPC**: Pipes, queues, sockets, etc.

#### 🔹 Threads

- **Shared Memory Space**: Shares heap/code/handles.
- **Own Stack & PC**: Has independent control flow.
- **Lightweight**: Faster to create and switch.
- **Direct Communication**: Requires synchronization.
- **Crash in One = Crash in All**

#### 📊 Summary Table

| Feature                | Process                             | Thread                              |
|------------------------|--------------------------------------|--------------------------------------|
| Definition             | Independent program                  | Execution unit within a process     |
| Memory Space           | Isolated                             | Shared                              |
| Resources              | Has own resources                    | Shares parent's resources           |
| Overhead               | High                                 | Low                                 |
| Context Switching      | Slow                                 | Fast                                |
| Communication          | IPC (pipes, queues, etc.)            | Shared memory (with sync)           |
| Isolation              | Strong                               | Weak                                |
| Crash Effect           | Other processes unaffected           | May crash whole process             |

#### 🔍 Google Context

- **Processes**: Ideal for microservices, isolation, sandboxing.
- **Threads**: Better for GUI responsiveness, background I/O tasks, or intra-service parallelism.

---

## 🔄 Thread Lifecycle

```
New → Runnable ↔ Running → Waiting/Blocked → Terminated
```

## 🟡 New
Created but not yet started.
---
When a thread object is first created but has not yet started execution. It exists as an object in memory but hasn't been allocated any CPU time by the scheduler.
### Action: 
In C++ with std::thread, this state exists right after std::thread myThread(myFunction); but before myThread.join() or myThread.detach() potentially leading to its execution. In Java, it's new Thread().
### Transition: 
From this state, the thread can be moved to the "Runnable" state by invoking a start()-like method (e.g., the std::thread constructor automatically starts the thread; for other paradigms, there might be an explicit start() call).

In C++, after std::thread t(f);, the thread starts immediately.

In Java, Thread t = new Thread(); — not started until .start() is called.

## 🟢 Runnable (Ready)
The thread is ready to run and waiting for CPU time.

It's in the OS scheduler's queue.

### Description: 
The thread is ready to be executed and is waiting for the CPU. It's in the queue of the operating system's scheduler, competing for CPU time. It might be immediately executed if a CPU is available, or it might wait if other threads or processes are currently running.
### Action: 
The thread is waiting for its turn to run.
### Transition:
To Running: When the scheduler selects it for execution on a CPU core.
To Waiting/Blocked: If it requests an I/O operation or tries to acquire a lock that's currently held.
To Terminated: (Less common directly, but possible if it's the only runnable task and completes quickly).


## 🔵 Running
#### Description: The thread is actively executing instructions on a CPU core.
#### Action: 
The CPU is dedicated to executing the thread's code.
#### Transition:
- To Runnable:
  - **Time Slice Expiry:** Its allocated time quantum (timeslice) expires, and the scheduler preempts it.
  -** Higher Priority Preemption**: A higher-priority thread becomes runnable.
- To Waiting/Blocked:
  - **I/O Request**: It performs an I/O operation (e.g., reading from a file, network request) and must wait for the operation to complete.
  - **Synchronization Primitive**: It tries to acquire a mutex, semaphore, or condition variable that is currently unavailable, so it blocks until it can acquire it.
  - **sleep()/wait(**): The thread explicitly calls a function to pause its own execution for a specified duration or until a certain condition is met.
- To Terminated: The thread completes its execution successfully.


## 🔴 Blocked / Waiting
#### Description: 
The thread is temporarily inactive and cannot be executed because it's waiting for some event to occur. It's not consuming CPU cycles.
#### Action: 
Waiting for an I/O operation to complete, a lock to be released, a timer to expire, or a notify() call on a condition variable.
#### Transition:
-To Runnable: When the event it was waiting for occurs (e.g., I/O completes, lock is released, sleep time expires, notify is called). The thread then re-enters the runnable queue.


## ⚫ Terminated
#### Description: 
The thread has completed its execution (either normally or due to an unhandled exception/error) and is no longer active. Its resources can be reclaimed by the operating system.
#### Action: 
The thread's function has returned, or an exception propagated out of its entry point.
#### Transition: 
- This is the final state. A terminated thread cannot be restarted

## Visual Representation (Simplified State Diagram):


```
           New
            |
            V
         Runnable <---> Running
            ^           |    |
            |           V    V
            +------- Blocked/Waiting
                        |
                        V
                     Terminated
```

## 🔄 Context Switching

Context switching is the mechanism by which the operating system's scheduler saves the state of one running thread (or process) and restores the state of another, allowing the CPU to switch from executing one to executing the other.

It's how the illusion of parallelism is created on a single CPU core, and how true parallelism is managed on multi-core systems when the number of runnable entities exceeds the number of available cores.

**Why is Context Switching Necessary?**

- Multitasking/Concurrency: To allow multiple threads/processes to share the CPU, giving each a slice of time and making it appear as if they are running simultaneously.

- Resource Management: When a thread blocks (e.g., for I/O), the CPU can be given to another ready thread instead of sitting idle.

- Preemption: To ensure fairness and responsiveness by taking the CPU away from a long-running thread if a higher-priority thread becomes ready or its time slice expires.

**Steps Involved in a Thread Context Switch:**

- When the operating system decides to switch from Thread A to Thread B:

- **Save State of Current Thread (Thread A)**:

  - The CPU's current state (value of all general-purpose registers, program counter, stack pointer, CPU flags).

  - The thread's current stack (the local variables and function call frames).

  - Any thread-specific data structures (part of the Thread Control Block - TCB).

  - This state is saved in Thread A's TCB.

-** Choose Next Thread (Thread B)**:

  - The operating system's scheduler (a part of the kernel) selects the next thread to run from the "Runnable" queue based on its scheduling algorithm (e.g., Round Robin, Priority-based, Shortest Job Next).

- **Restore State of Next Thread (Thread B)**:

  - The saved CPU state (registers, program counter, stack pointer, flags) from Thread B's TCB is loaded into the CPU.

  - The CPU's memory management unit (MMU) is often updated to reflect Thread B's stack context (if the switch is between threads in different processes, the entire virtual memory map might need to be switched, which is more expensive).

- **Resume Execution**:

  - Thread B then resumes execution from exactly where it left off, as if it had never been paused.

**Cost of Context Switching:**

Context switching is not free. It introduces overhead because:

- CPU Cycles: The CPU spends time saving and restoring states rather than doing "useful" work for the applications.

- Cache Invalidation: When switching between threads (especially if they are from different processes, or even within the same process if they access vastly different memory regions), the CPU's caches (L1, L2, L3) and Translation Lookaside Buffer (TLB) might become less effective. The new thread will likely incur cache misses as it fetches its own data and instructions into the cache, leading to performance degradation. This is a significant performance bottleneck in highly concurrent systems.

- System Call Overhead: The switch itself typically involves a system call into the kernel.

**Thread vs. Process Context Switching:**

- Thread Context Switch (within the same process): Relatively lightweight. The memory address space remains the same, so the MMU doesn't need a full reload. Only registers, stack pointer, program counter, and thread-specific TCB data are switched. Less cache/TLB invalidation.

- Process Context Switch: Heavyweight. Requires switching the entire virtual memory address space (including updating page tables and likely flushing the TLB), in addition to the register set and other process-specific data. Leads to more significant cache invalidation.

Understanding both the thread lifecycle (how a thread progresses) and context switching (how the OS manages multiple threads on limited CPU resources) is crucial for designing efficient and responsive concurrent applications. It helps in identifying potential performance bottlenecks and applying appropriate synchronization mechanisms to ensure correctness.

### Thread-Safe Data Structures
A thread-safe data structure is one that can be safely accessed and modified by multiple threads concurrently without leading to undefined behavior, data corruption, or crashes. This safety is achieved through internal synchronization mechanisms.

Conversely, a data structure that is not thread-safe (like most standard library containers by default) requires external synchronization when accessed concurrently by multiple threads.

#### 🚫 Why std::vector is NOT Thread-Safe**
std::vector in C++ is a prime example of a non-thread-safe data structure. It does not incorporate any internal locking or synchronization mechanisms. This design choice is deliberate to maximize performance in single-threaded scenarios, which are very common. Adding locks would introduce overhead (performance cost) even when not needed.

Here's why concurrent access to std::vector without external synchronization leads to Undefined Behavior:

#### Data Races (Most Common Issue):

- **Concurrent Write-Write:**

  - Imagine two threads, T1 and T2, both calling myVector.push_back(value); simultaneously.

  - push_back might involve:

    - Checking if capacity() is sufficient.

    - If not, allocating new, larger memory (reallocation).

    - Copying existing elements to the new memory.

    - Destroying old elements and deallocating old memory.

    - Constructing the new element at size().

    - Incrementing size().

  - If T1 is in the middle of a reallocation when T2 tries to add an element, T2 might try to write to the old, deallocated memory, or write to a location that T1 is also trying to write to, leading to memory corruption.

  - Even just incrementing size() is not atomic. size() is a simple integer, but size++ is typically a read-modify-write operation (load size, increment, store size). If two threads do this concurrently, one increment might be lost (known as a "lost update").

- **Concurrent Read-Write:**

  - If T1 is iterating through the vector (for (int x : myVector)) while T2 calls push_back(), erase(), or insert().

  - If T2 causes a reallocation, T1's iterators (and potentially pointers/references) become invalidated. Dereferencing an invalidated iterator leads to undefined behavior, often a crash or reading garbage data.

  - Even without reallocation, if T2 removes an element, T1 might access an element that no longer exists or misinterpret the vector's structure.

- **Iterator Invalidation:**

  - Many std::vector operations (e.g., push_back when capacity is exceeded, insert, erase, resize) can change the underlying memory block or shift elements. This invalidates existing iterators, pointers, and references to elements. If another thread holds onto such invalidated references, using them results in undefined behavior.

- **Lack of Atomicity:**

  - Individual operations on std::vector are not guaranteed to be atomic. An "atomic" operation is one that appears to happen instantaneously and completely, without any intermediate states being visible to other threads. As seen with push_back or even size++, they are composed of multiple steps.
 
**How to Make std::vector Thread-Safe (External Synchronization):**

The standard approach is to use a std::mutex to protect all accesses (reads and writes) to the shared std::vector instance.

```
#include <iostream>
#include <vector>
#include <thread>
#include <mutex> // For std::mutex and std::lock_guard

std::vector<int> shared_vector;
std::mutex vector_mutex; // Mutex to protect shared_vector

void add_elements() {
    for (int i = 0; i < 1000; ++i) {
        // Acquire lock before modifying
        std::lock_guard<std::mutex> lock(vector_mutex);
        shared_vector.push_back(i);
    }
}

void read_elements() {
    // Acquire lock before reading
    std::lock_guard<std::mutex> lock(vector_mutex);
    std::cout << "Vector size: " << shared_vector.size() << std::endl;
    // Iterating while locked ensures consistency
    for (int x : shared_vector) {
        // std::cout << x << " "; // Uncomment for full print, but might be slow
    }
    // std::cout << std::endl;
}

int main() {
    std::thread t1(add_elements);
    std::thread t2(add_elements); // Two threads adding elements
    std::thread t3(read_elements); // One thread reading elements

    t1.join();
    t2.join();
    t3.join(); // Joining the read thread here might print a partial result

    // For a final, consistent print after all writes are done:
    std::lock_guard<std::mutex> lock(vector_mutex);
    std::cout << "Final vector size: " << shared_vector.size() << std::endl;

    return 0;
}
```
**Trade-off:** While this makes std::vector thread-safe, it introduces contention. If many threads frequently try to access the vector, they will spend a lot of time waiting for the mutex, severely limiting parallelism and performance.

### Other Thread-Safe Data Structures:

- **Containers with std::mutex wrappers**: You can wrap any standard container with a mutex to make it thread-safe for basic operations.

- **Concurrent Queues**: concurrent_queue (TBB), moodycamel::ConcurrentQueue (third-party) are designed for safe multi-producer/multi-consumer scenarios.

- **Atomic Types:** std::atomic<T> for individual variables that need atomic operations.

- **Specialized Libraries:** Libraries like Intel TBB (Threading Building Blocks) offer genuinely concurrent containers (e.g., concurrent_vector, concurrent_hash_map) that use fine-grained locking or lock-free algorithms to allow more parallelism than a single mutex would.


## Thread Pools: Usage, Design, Trade-offs
A thread pool is a collection of pre-initialized, reusable worker threads that are available to execute tasks. Instead of creating a new thread for each task, tasks are submitted to the thread pool, which then assigns them to an available thread.

#### Usage
Thread pools are primarily used to:

- Reduce Thread Creation/Destruction Overhead: Creating and destroying threads is an expensive operation (involves OS system calls, memory allocation for stacks, etc.). A thread pool avoids this overhead by reusing existing threads.

- Manage Concurrency Levels: Limit the number of active threads to prevent resource exhaustion (too many threads can lead to excessive context switching, memory consumption, and contention). This helps maintain system stability and responsiveness.

- Improve Responsiveness: For short-lived tasks, submitting to a pool is much faster than creating a new thread.

- Decouple Task Submission from Execution: The submitter doesn't need to know or manage the lifecycle of the threads executing the task.

### Typical Use Cases:

- Web Servers: Handling incoming client requests. Instead of spawning a new thread for every request, requests are submitted to a thread pool.

- Asynchronous I/O: Performing file reads/writes, network operations in the background without blocking the main thread.

- Parallel Computations: Breaking down a large computation into smaller, independent tasks that can be executed concurrently by pool threads.

- Game Development: Background asset loading, physics calculations.

**Design of a Basic Thread Pool**
A common design for a thread pool involves these components:

- Task Queue (Thread-Safe):

  - A std::queue (or std::deque) protected by a std::mutex and a std::condition_variable.

  - Tasks (often std::function<void()>) are pushed onto this queue by "producer" threads.

  - Worker threads "consumer" tasks from this queue.

  - The condition_variable is used to:

    - Signal worker threads when new tasks are available.

    - Allow worker threads to wait when the queue is empty.

- Worker Threads:

  - A fixed (or sometimes dynamic) number of std::thread objects.

  - Each worker thread runs a loop:

      - Acquire Lock: Lock the mutex protecting the task queue.

      - Wait for Task: If the queue is empty, wait on the condition_variable (atomically releases the lock and waits).

      - Dequeue Task: Once a task is available (and the lock is reacquired), pop the task from the queue.

      - Release Lock: Release the mutex as soon as the task is dequeued (important to allow other workers to get tasks).

      - Execute Task: Run the task (the std::function<void()> object).

      - Repeat: Go back to acquire the lock and look for the next task.

- Submission Method:

   - A public method (e.g., enqueue_task(std::function<void()>)) that allows external code to submit tasks.

   - This method locks the task queue, pushes the new task, and then notifies the condition_variable (e.g., notify_one() or notify_all()) to wake up waiting worker threads.

- Shutdown Mechanism:

  - A way to gracefully stop all worker threads when the pool is no longer needed. This usually involves:

    - Setting a flag to indicate shutdown.

    - Notifying all waiting worker threads (even if the queue is empty) so they can check the shutdown flag.

    - Worker threads, upon waking, check the flag. If set, they exit their loop and terminate.

    - Joining all worker threads to ensure they have completed their work.

```
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional> // For std::function
#include <future>     // For std::packaged_task and std::future

class ThreadPool {
public:
    ThreadPool(size_t num_threads) : stop(false) {
        for (size_t i = 0; i < num_threads; ++i) {
            workers.emplace_back([this] { // Lambda for worker thread function
                while (true) {
                    std::function<void()> task;
                    { // Scope for lock_guard
                        std::unique_lock<std::mutex> lock(queue_mutex);
                        // Wait until there's a task or shutdown is requested
                        condition.wait(lock, [this] {
                            return stop || !tasks.empty();
                        });

                        if (stop && tasks.empty()) {
                            return; // Exit thread if stopping and no more tasks
                        }
                        task = tasks.front();
                        tasks.pop();
                    } // Lock released here
                    task(); // Execute task outside the lock to allow other tasks to be dequeued
                }
            });
        }
    }

    // Add new work item to the pool
    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args)
        -> std::future<typename std::result_of<F(Args...)>::type>
    {
        using return_type = typename std::result_of<F(Args...)>::type;

        auto task = std::make_shared< std::packaged_task<return_type()> >(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );

        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);

            // Don't allow enqueueing after stopping the pool
            if(stop)
                throw std::runtime_error("enqueue on stopped ThreadPool");

            tasks.emplace([task](){ (*task)(); });
        }
        condition.notify_one(); // Wake up one waiting worker
        return res;
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all(); // Wake all workers so they can exit
        for(std::thread &worker: workers) {
            worker.join(); // Wait for all workers to finish
        }
    }

private:
    // Need to keep track of threads so we can join them
    std::vector<std::thread> workers;
    // The task queue
    std::queue<std::function<void()>> tasks;

    // Synchronization
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

// Example usage:
// int main() {
//     ThreadPool pool(4); // Create a pool with 4 threads
//     for (int i = 0; i < 8; ++i) {
//         pool.enqueue([i] {
//             std::cout << "Task " << i << " running on thread " << std::this_thread::get_id() << std::endl;
//             std::this_thread::sleep_for(std::chrono::milliseconds(100));
//         });
//     }
//     // Pool destructor will join threads when main exits
//     return 0;
// }
```


### Trade-offs of Thread Pools
- **Advantages**:

  - Performance: Significantly reduces overhead for short-lived tasks by reusing threads.

  - Resource Management: Limits the number of concurrent threads, preventing system overload.

  - Responsiveness: Keeps a pool of ready threads, allowing tasks to start executing almost immediately.

  - Simplified Design: Decouples task submission from thread management, making application logic cleaner.

  - Load Balancing: Tasks can be distributed among available threads.

- **Disadvantages:**

  - Complexity: Implementing a robust thread pool (especially with error handling, dynamic sizing, task prioritization) can be complex.

  - Potential for Deadlocks: If tasks themselves require acquiring external resources or mutexes, and these are not properly managed, deadlocks can occur within the pool.

  - Overhead for Long-Running Tasks: For very long-running tasks, the benefits of thread reuse are minimal, and the overhead of managing the queue and synchronization might even be a slight disadvantage compared to just creating a dedicated thread.

  - Contention: The shared task queue can become a point of contention if tasks are submitted very frequently, leading to mutex overhead.  

  - Starvation: Lower-priority tasks might get starved if high-priority tasks continuously fill the queue.

  - Debugging: Debugging concurrent issues within a thread pool can be challenging due to non-deterministic execution order.

  - Optimal Pool Size: Determining the ideal number of threads for a pool is often tricky and depends on the nature of the tasks (CPU-bound vs. I/O-bound), the number of CPU cores, and system resources. Too few threads can underutilize resources, too many can lead to excessive context switching.

Thread pools are a powerful tool for managing concurrency, particularly in server applications and systems where a large number of independent, relatively short-lived tasks need to be executed efficiently. However, like any powerful tool, they require careful design and consideration of their inherent trade-offs.
