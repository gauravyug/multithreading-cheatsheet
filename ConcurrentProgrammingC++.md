# Quick Revision Guide: Parallel and Concurrent Programming with C++

---

## Chapter 1: Parallel Computing Hardware

- **Sequential vs Parallel**:
  - Sequential: One task at a time.
  - Parallel: Multiple tasks simultaneously (multi-core, SIMD, GPU).

- **Architectures**:
  - **SISD**: Single instruction, single data.
  - **SIMD**: Single instruction, multiple data (e.g., AVX).
  - **MISD**: Rare, for fault tolerance.
  - **MIMD**: Multiple instruction, multiple data (multi-core CPUs).

- **Memory Models**:
  - **Shared Memory**: Threads access shared address space.
  - **Distributed Memory**: Processes communicate via messages (e.g., MPI).

---

## Chapter 2: Threads and Processes

- **Threads vs Processes**:
  - Processes: Independent, separate memory.
  - Threads: Lightweight, shared memory space.

- **Concurrent vs Parallel**:
  - Concurrent: Interleaved execution.
  - Parallel: Truly simultaneous execution.

- **Thread Lifecycle**:
  - Create → Run → Block → Terminate

- **Scheduling**:
  - Preemptive and cooperative

### Example

```cpp
std::thread t([]() { std::cout << "Hello from thread"; });
t.join();
```


## **Chapter 3: Mutual Exclusion
- **Data Races**:

  - Occur when multiple threads access shared memory unsynchronized.

  - Atomic Operations:

Avoid race conditions without locks.

## Example
```
#include <atomic>

std::atomic<int> counter{0};
counter.fetch_add(1);
```

## Chapter 4: Locks
- mutex: Basic lock.

recursive_mutex: Same thread can re-lock.

try_lock: Non-blocking lock attempt.

shared_mutex: Multiple readers, one writer.

```
#include <mutex>

std::mutex m;
m.lock();
// critical section
m.unlock();

```
## Chapter 5: Liveness
Deadlock: Circular waiting on resources.

Abandoned Lock: Lock left held by exited thread.

Starvation: Thread never gets CPU time.

Livelock: Threads keep running but make no progress.

## Chapter 6: Synchronization
Condition Variables:

Used for signaling between threads.

Producer-Consumer Problem:

Producer adds to buffer, consumer removes, both synchronized.

Semaphores (C++20):

Counting mechanisms for resource control.

'''
#include <condition_variable>
#include <mutex>

std::condition_variable cv;
std::mutex m;
bool ready = false;


cv.wait(lock, []{ return ready; });
```


```
//Example: Semaphore (C++20)

#include <semaphore>

std::counting_semaphore<1> sem(1);
sem.acquire();

// critical section
sem.release();
```

## Chapter 7: Barriers
Race Condition: Outcome depends on unpredictable timing.

Barrier: Synchronization point for threads.

## Example (C++20)
cpp
Copy
Edit
#include <barrier>

std::barrier sync_point(3); // 3 threads

sync_point.arrive_and_wait(); // waits until all arrive
📘 Chapter 8: Asynchronous Tasks
Futures & std::async:

Task starts in background and returns future.

Thread Pool:

Reusable threads for multiple tasks.

Divide and Conquer:

Break a task into subtasks, solve in parallel.

🧪 Example
cpp
Copy
Edit
#include <future>

auto fut = std::async(std::launch::async, []{ return 42; });
int result = fut.get();
📘 Chapter 9: Evaluating Parallel Performance
Speedup: T_sequential / T_parallel

Latency: Time to finish one task.

Throughput: Tasks per unit time.

📈 Amdahl’s Law
mathematica
Copy
Edit
Speedup ≤ 1 / (S + (1 - S) / N)
S = Serial fraction, N = Number of processors
📘 Chapter 10: Designing Parallel Programs
Partitioning: Break problem into independent parts.

Communication: How threads/processes exchange info.

Agglomeration: Combine fine-grained tasks to reduce overhead.

Mapping: Assign tasks efficiently to cores/threads.

💡 Final Tip
Use modern C++ tools:

std::thread

std::mutex, std::unique_lock

std::condition_variable

std::async, std::future

C++20: std::barrier, std::semaphore, std::latch, std::stop_token
