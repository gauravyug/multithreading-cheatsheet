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
