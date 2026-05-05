# Study Guide: GPU as Parallel Computers

## Module Introduction
Since 2003, the semiconductor industry has diverged into two primary design trajectories for microprocessors: the **multi-core trajectory** (CPU) and the **many-core trajectory** (GPU). Understanding these two paradigms is essential for modern high-performance computing, where workload-specific hardware selection determines the efficiency and speed of an application.

---

## 1. Multi-core vs. Many-core Trajectories

The performance gap between CPUs and GPUs arises from fundamentally different design philosophies.

### Multi-core Trajectory (CPU)
*   **Design Philosophy:** **Latency-oriented**. It is optimized for the speed of sequential code performance.
*   **Architecture:** Uses sophisticated control logic to allow instructions from a single thread of execution to run in parallel. It utilizes large cache memories to reduce instruction and data access latencies.
*   **Focus:** Minimizing the time taken to execute a single task. Memory bandwidth is often a bottleneck, limiting the rate at which data can be delivered to the processor.

### Many-core Trajectory (GPU)
*   **Design Philosophy:** **Throughput-oriented**. It focuses on maximizing the total amount of work (throughput) completed per unit of time.
*   **Architecture:** Built around a large number of much smaller, highly multithreaded cores.
*   **Focus:** Executing massive numbers of threads simultaneously. GPUs hide memory latency by switching to another thread while waiting for a long-latency memory access. 
*   **Performance:** GPUs dominate Floating-Point (FP) performance, often achieving a 10-to-1 throughput ratio over multicore CPUs for computationally intensive tasks.

| Feature | CPU (Multi-core) | GPU (Many-core) |
| :--- | :--- | :--- |
| **Optimization** | Latency-oriented | Throughput-oriented |
| **Control Logic** | Sophisticated (complex) | Simple (shared) |
| **Cache Size** | Large | Small (multi-thread support) |
| **Ideal Workload** | Sequential tasks | Massive parallel threads |

---

## 2. Modern GPU Architecture

Modern GPUs are designed to handle massive parallelism, specifically for graphics and numerically intensive applications.

*   **Streaming Multiprocessors (SMs):** The core building blocks of the GPU. An array of these SMs allows the GPU to be highly threaded.
*   **Streaming Processors (SPs):** Each SM contains multiple SPs that share control logic and an instruction cache.
*   **Global Memory (GDDR DRAM):** 
    *   GPUs use Graphics Double Data Rate (GDDR) DRAM, which acts as a "frame buffer" memory.
    *   Unlike system DRAM, it is optimized for high bandwidth to feed massive numbers of parallel threads.
    *   While GDDR has higher latency than standard system memory, the massive bandwidth compensates for this in parallel applications by allowing multiple threads to access common data efficiently.

---

## 3. Design Considerations for Developers

When developers select a processor for a specific application, performance is only one of many factors:

1.  **Installation Base:** Processors with a large market presence reduce the cost and risk of software development by ensuring a wide customer reach.
2.  **Practical Form Factors/Accessibility:** Software must be easily accessible to customers and provide solutions for common needs.
3.  **Support for IEEE Floating-Point Standards:** This ensures predictable results across different hardware vendors.
4.  **Ease of Programming:**
    *   Early **GPGPU** (General-Purpose programming using GPU) relied on complex graphics APIs, which were tedious and limited application scope.
    *   **Introduction of CUDA (Compute Unified Device Architecture):** NVIDIA developed CUDA to allow developers to use familiar C/C++ tools, providing a dedicated general-purpose programming interface that bypassed graphics-specific restrictions.

---

## Key Takeaways

*   **Complementary Strengths:** Most applications today use both CPUs and GPUs; sequential logic is handled by the CPU, while numerically intensive sections are offloaded to the GPU.
*   **Parallelism is Necessary:** Future applications—including "Superapps," big data analytics, and advanced human-machine interfaces—will demand significantly more computing power than sequential processors can provide.
*   **The CUDA Revolution:** The shift from graphics-specific APIs to specialized parallel frameworks like CUDA has significantly lowered the barrier to entry, enabling massive speedups (often >100x) for parallel-suitable applications.
*   **Architecture Matters:** The choice between CPU and GPU depends on whether the workload is latency-critical (requires complex control logic) or throughput-critical (requires massive thread-level parallelism).