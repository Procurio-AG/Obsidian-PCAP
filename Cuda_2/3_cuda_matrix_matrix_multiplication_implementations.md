# Study Guide: CUDA Matrix-Matrix Multiplication Implementations

## Module Introduction
Matrix-matrix multiplication ($C = A \times B$) is a fundamental operation in scientific computing and machine learning. Because each output element in the resultant matrix $C$ can be calculated independently, it is a highly parallelizable task. This module explores how to implement this operation in CUDA by mapping data structures to the GPU’s thread hierarchy, linearizing multidimensional arrays, and optimizing for performance using shared memory.

---

## 1. Matrix Multiplication Variants
The PDF outlines three common ways to distribute the workload of matrix multiplication across threads. For two matrices, $A$ (dimension $ha \times wa$) and $B$ (dimension $hb \times wb$), where $wa = hb$, the resultant matrix $C$ has dimensions $ha \times wb$.

1.  **Row-wise (One thread per row of $C$):** 
    Each thread is responsible for calculating all elements in a single row of the result matrix.
2.  **Column-wise (One thread per column of $C$):** 
    Each thread is responsible for calculating all elements in a single column of the result matrix.
3.  **Element-wise (One thread per element of $C$):** 
    Each thread is responsible for computing exactly one specific element $(i, j)$ of the result matrix. This is the most granular form of parallelism and typically offers the best performance for massive matrices.

---

## 2. Matrix Element Access Patterns
Since CUDA (based on C/C++) represents memory as a 1D space, 2D matrices must be "linearized" or "flattened."

### Linearization Concept
Matrices are typically stored in **row-major layout**, meaning all elements of the first row are placed first, followed by all elements of the second row, and so on.

*   **Row-wise Access ($k$-th element in a row):**
    To access the $k$-th element of the `Row`-th row:
    $$Index = Row \times Width + k$$
    *Logic: Skip all preceding rows ($Row \times Width$) and add the current column offset ($k$).*

*   **Column-wise Access ($k$-th element in a column):**
    To access the $k$-th element of the `Col`-th column:
    $$Index = k \times Width + Col$$
    *Logic: Skip entire rows sequentially, jumping by the width of the matrix to reach the specific column index.*

---

## 3. Basic Matrix Multiplication Kernels
These kernels map the specific threading model to the multiplication logic.

*   **`multiplyKernel_rowwise`:** Uses `threadIdx.x` to assign a row to a thread. The kernel uses an outer loop for columns of $B$ and an inner loop to compute the dot product.
*   **`multiplyKernel_colwise`:** Uses `threadIdx.x` to assign a column of $B$ to a thread. The kernel iterates through rows of $A$ and computes the dot product for that column.
*   **`multiplyKernel_elementwise`:** Uses `threadIdx.y` for the output row and `threadIdx.x` for the output column. This allows every thread to compute a distinct element of the result matrix independently.

---

## 4. Advanced Matrix Multiplication Kernel: Tiling
The "basic" kernels suffer from high global memory traffic. Since global memory (DRAM) is slow, frequent access leads to a performance bottleneck.

### The Tiling Strategy
Instead of threads reading directly from slow global memory for every calculation, we use **Tiling (Blocking)**:
1.  **Divide and Conquer:** The matrices are broken into smaller blocks (tiles) that fit into the high-speed **Shared Memory**.
2.  **Collaboration:** All threads in a block work together to load a tile of $M$ and a tile of $N$ into shared memory.
3.  **Computation:** Threads compute the dot product using the shared data. Once finished, they load the next tile.

### The Role of `__syncthreads()`
Synchronization is critical in tiled kernels:
*   **Load Phase:** `__syncthreads()` ensures all threads in the block have finished loading their respective elements into `Mds` and `Nds` before anyone starts calculating.
*   **Compute Phase:** `__syncthreads()` ensures that no thread begins the next phase (loading the next set of tiles) until all threads have finished using the current data, preventing data race conditions.

### Autotuning with `BLOCK_WIDTH`
Using a `#define BLOCK_WIDTH 16` allows for easy code optimization (autotuning). If a developer wants to test performance with different tile sizes, they change one constant rather than manually updating every loop or index calculation.

---

## Key Takeaways
*   **Memory Hierarchy Matters:** Global memory is high-capacity but slow. Shared memory is low-capacity but fast. The goal of advanced kernels is to maximize **data locality** (reuse of data in shared memory).
*   **CGMA Ratio:** The *Compute to Global Memory Access* ratio is the key metric for performance. Tiling increases this ratio by ensuring data is read from global memory fewer times.
*   **Thread Mapping:** Correctly identifying the global ID of a thread relative to the Grid and Block structure is essential for mapping into linearized 1D arrays.
*   **Synchronization:** `__syncthreads()` is the "barrier" that enables thread cooperation. Using it incorrectly (e.g., inside branches where not all threads enter) leads to deadlocks.