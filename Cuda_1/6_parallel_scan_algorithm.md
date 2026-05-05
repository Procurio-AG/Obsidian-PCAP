# Study Guide: Parallel Scan Algorithm

## Module Introduction
The **Parallel Scan Algorithm** (commonly known as **Prefix Sum**) is a fundamental parallel programming pattern. It allows us to take problems that appear inherently sequential or recursive—such as resource allocation, work assignment, or polynomial evaluation—and transform them into parallel operations. By computing prefix sums in parallel, we can significantly reduce the execution time for a variety of computational tasks.

---

## Deep Dives

### 1. Introduction to Parallel Scan
*   **What is it?** A pattern that takes an input array and returns an output array where each element contains the result of a reduction of all previous elements.
*   **Why use it?** If a computation can be expressed as a mathematical recursion, it is often a prime candidate for parallelization using a scan operation.
*   **Utility:** It is a core building block for many complex parallel algorithms, including parallel quicksort and stream compaction.

### 2. Inclusive Scan Operation
*   **Mathematical Definition:** Given an input array $[x_0, x_1, \dots, x_{n-1}]$ and a binary associative operator ($\oplus$), an inclusive scan returns an output array where each element is the sum of all elements up to that position, *including* the current element.
    *   **Output:** $[x_0, (x_0 \oplus x_1), (x_0 \oplus x_1 \oplus x_2), \dots, (x_0 \oplus x_1 \oplus \dots \oplus x_{n-1})]$
*   **Example:** If the operator is addition (+), an input $[3, 1, 7, 0, 4, 1, 6, 3]$ returns $[3, 4, 11, 11, 15, 16, 22, 25]$.

### 3. Exclusive Scan Operation
*   **Definition:** Similar to an inclusive scan, but it excludes the current element from the calculation of the result at that position.
*   **Key Difference:** The first element is always $0$, and the result at index $i$ reflects the sum of all elements up to $i-1$.
    *   **Output:** $[0, x_0, (x_0 \oplus x_1), \dots, (x_0 \oplus x_1 \oplus \dots \oplus x_{n-2})]$
*   **Relationship:** An exclusive scan is essentially an inclusive scan shifted one position to the right with a leading zero.

### 4. Work-Inefficient Parallel Inclusive Scan
*   **Concept:** The algorithm is performed "in-place" on an array $XY$. It uses an iterative reduction tree approach.
*   **Iteration Logic:**
    *   **Iteration 1:** Every position (except $XY[0]$) adds its value to its left neighbor ($XY[i] = x_i + x_{i-1}$).
    *   **Iteration 2:** Every position (except the first two) adds the value from the position two spots away.
    *   **Scaling:** As the iterations progress, the stride of the data being accessed doubles ($2^n$), allowing the algorithm to quickly calculate prefix sums over larger ranges.
*   **Terminology:** It is called "work-inefficient" because many threads perform redundant additions compared to the optimal sequential approach, though it is highly parallel.

### 5. Kernel Implementation for Inclusive Scan
The CUDA implementation relies on shared memory to minimize high-latency DRAM accesses:
*   **Shared Memory:** `__shared__ float XY[SECTION_SIZE]` is used to load data into on-chip memory for fast access by all threads in the block.
*   **The Iterative Loop:**
    ```cpp
    for (unsigned int stride = 1; stride <= threadIdx.x; stride *= 2) {
        __syncthreads(); // Ensures all threads finish previous iteration
        XY[threadIdx.x] += XY[threadIdx.x - stride];
    }
    ```
*   **`__syncthreads()`:** This barrier is critical. Because threads are updating shared memory, we must ensure all threads have completed the current addition iteration before any thread begins the next one.

### 6. Converting to Exclusive Scan Kernel
To convert the inclusive scan kernel into an exclusive scan, we simply manipulate the initial data load:
*   **Shift and Zero:** Instead of loading the full input array into $XY$ directly, we shift the values.
*   **Implementation logic:**
    *   Load $0$ into $XY[0]$.
    *   For any index $i > 0$, load $X[i-1]$ into $XY[threadIdx.x]$.
*   **Result:** This shift automatically aligns the data so that every position $i$ contains the prefix sum up to $i-1$. This avoids complex boundary condition checks inside the main loop.

---

## Key Takeaways
*   **Associativity is Key:** The scan operation requires a binary associative operator (like addition or multiplication) to ensure the parallel reduction order produces correct results.
*   **Shared Memory Performance:** By using `__shared__` memory, we avoid the bottleneck of global device memory during the intensive iterative calculation steps.
*   **Synchronization:** `__syncthreads()` is mandatory in scan kernels. Without it, race conditions occur as threads try to read/write the shared `XY` array while other threads are still processing.
*   **Inclusive vs. Exclusive:** Remember that inclusive scan *includes* the current element in the total, while exclusive scan *excludes* it, starting with a $0$.
*   **Efficiency:** While the provided approach is "work-inefficient," it serves as an excellent starting point for understanding how to break sequential dependencies and achieve massive parallelism on GPU architectures.