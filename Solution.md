Great—this is a full 50-mark paper, so I’ll solve it **answer-wise with mark-oriented depth** (not too long, not too short). You can directly use this for practice/writing.

---

# **1A) CPU vs GPU Design Philosophy (5 marks)**

### **CPU (Central Processing Unit)**

- Optimized for **low latency**
    
- Few powerful cores (4–16 typically)
    
- Large cache memory
    
- Complex control logic
    
- Good for **sequential tasks**
    

### **GPU (Graphics Processing Unit)**

- Optimized for **high throughput**
    
- Thousands of lightweight cores
    
- Smaller cache, more ALUs
    
- SIMT (Single Instruction Multiple Threads)
    
- Best for **parallel tasks**
    

### **Diagram (conceptual)**

CPU:

```
[Core][Core][Core]
   |   Cache   |
```

GPU:

```
[ALU ALU ALU ALU ...] (Thousands)
Minimal Control + Small Cache
```

### **Key Difference**

|Feature|CPU|GPU|
|---|---|---|
|Cores|Few|Many|
|Execution|Sequential|Parallel|
|Latency|Low|High|
|Throughput|Low|Very High|

---

# **1B) CUDA Vector Addition (3 marks)**

```cpp
#include <stdio.h>

__global__ void vecAdd(float *A, float *B, float *C, int n) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    if (i < n)
        C[i] = A[i] + B[i];
}

int main() {
    int n = 5;
    float A[] = {1,2,3,4,5}, B[] = {5,4,3,2,1}, C[5];

    float *d_A, *d_B, *d_C;

    cudaMalloc(&d_A, n*sizeof(float));
    cudaMalloc(&d_B, n*sizeof(float));
    cudaMalloc(&d_C, n*sizeof(float));

    cudaMemcpy(d_A, A, n*sizeof(float), cudaMemcpyHostToDevice);
    cudaMemcpy(d_B, B, n*sizeof(float), cudaMemcpyHostToDevice);

    vecAdd<<<1, n>>>(d_A, d_B, d_C, n);

    cudaMemcpy(C, d_C, n*sizeof(float), cudaMemcpyDeviceToHost);

    for(int i=0;i<n;i++)
        printf("%f ", C[i]);

    cudaFree(d_A); cudaFree(d_B); cudaFree(d_C);
}
```

---

# **1C) Data Inconsistency in CUDA (2 marks)**

### Problem:

Occurs when multiple threads update same memory location.

### Solution:

Use **atomic operations**

```cpp
__global__ void increment(int *a) {
    atomicAdd(a, 1);
}
```

### Explanation:

- Prevents race condition
    
- Ensures correct updates
    

---

# **2A) MPI Communication Differences (4 marks)**

### (i) MPI_Ssend vs MPI_Bsend

|Feature|MPI_Ssend|MPI_Bsend|
|---|---|---|
|Type|Synchronous|Buffered|
|Blocking|Yes|Yes|
|Requires buffer|No|Yes|

Example:

```c
MPI_Ssend(&data, 1, MPI_INT, dest, 0, MPI_COMM_WORLD);
MPI_Bsend(&data, 1, MPI_INT, dest, 0, MPI_COMM_WORLD);
```

---

### (ii) MPI_Bcast vs MPI_Reduce

|Feature|MPI_Bcast|MPI_Reduce|
|---|---|---|
|Purpose|Send data to all|Combine results|
|Direction|One → All|All → One|

Example:

```c
MPI_Bcast(&data, 1, MPI_INT, 0, MPI_COMM_WORLD);
MPI_Reduce(&val, &sum, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);
```

---

# **2B) MPI Matrix Search (3 marks)**

```c
// Pseudocode (marks-oriented)

Root:
- Read 3x3 matrix
- Read element to search
- Scatter rows to processes

Each process:
- Count occurrences

Use MPI_Reduce to sum counts

Error Handling:
if(err != MPI_SUCCESS)
    MPI_Abort(MPI_COMM_WORLD, err);
```

---

# **2C) 1D Sequential Convolution (3 marks)**

Given:

```
N = [3,5,2,4,6,1]
M = [2,1,-1]
```

### Formula:

```
P[i] = Σ N[i+j] * M[j]
```

### Calculation:

```
P[0] = 3*2 + 5*1 + 2*(-1) = 6 + 5 -2 = 9
P[1] = 5*2 + 2*1 + 4*(-1) = 10 + 2 -4 = 8
P[2] = 2*2 + 4*1 + 6*(-1) = 4 + 4 -6 = 2
P[3] = 4*2 + 6*1 + 1*(-1) = 8 + 6 -1 = 13
```

### Output:

```
P = [9, 8, 2, 13]
```

---

# **3A) OpenCL String Reverse (5 marks)**

### Kernel:

```c
__kernel void reverse(__global char* input, __global char* output, int n) {
    int i = get_global_id(0);
    output[i] = input[n - i - 1];
}
```

### Timing:

```c
start = clock();
// enqueue kernel
end = clock();
printf("Time = %f", (double)(end-start)/CLOCKS_PER_SEC);
```

---

# **3B) CGMA Ratio (3 marks)**

### Given:

- Bandwidth = 160 GB/s
    
- 4 bytes per float
    
- CGMA = 1
    

### FLOPS:

```
FLOPS = Bandwidth / Bytes per op
= 160 GB/s / 4
= 40 GFLOPS
```

### CGMA:

```
CGMA = FLOPS / Bandwidth
= 1500 / 160 ≈ 9.375
```

### Increase CGMA:

- Use **shared memory**
    
- Reduce global memory access
    

---

# **3C) CSR Format (2 marks)**

CSR has 3 arrays:

- Values[]
    
- Column Index[]
    
- Row Pointer[]
    

Example:

```
Matrix:
1 0 0
0 2 3
0 0 4
```

```
Values = [1,2,3,4]
ColIdx = [0,1,2,2]
RowPtr = [0,1,3,4]
```

---

# **4A) Shared Memory vs Registers (4 marks)**

|Feature|Registers|Shared Memory|
|---|---|---|
|Scope|Thread|Block|
|Speed|Fastest|Fast|
|Size|Small|Larger|

### fadd execution:

1. Load operands → registers
    
2. Perform addition
    
3. Store result
    

---

# **4B) Tiled 1D Convolution (4 marks)**

```cpp
__global__ void conv1D(float *N, float *M, float *P, int width, int maskWidth) {
    __shared__ float Ns[256];

    int i = threadIdx.x;
    Ns[i] = N[i];
    __syncthreads();

    float sum = 0;
    for(int j=0;j<maskWidth;j++)
        sum += Ns[i+j]*M[j];

    P[i] = sum;
}
```

---

# **4C) Constant Memory Candidate (2 marks)**

### Best Candidate:

- **Mask M**
    

### Why:

- Small
    
- Read-only
    
- Frequently accessed
    

### Host Code:

```cpp
__constant__ float M_const[10];

cudaMemcpyToSymbol(M_const, M, size);
```

---

# **5A) Constant Memory Convolution (4 marks)**

```cpp
__constant__ float M[10];

__global__ void conv(float *N, float *P, int width, int maskWidth) {
    int i = threadIdx.x;
    float sum = 0;

    for(int j=0;j<maskWidth;j++)
        sum += N[i+j]*M[j];

    P[i] = sum;
}
```

### Benefit:

- Cached access → faster
    

---

# **5B) Tiled Matrix Multiplication (4 marks)**

```cpp
__global__ void matMul(float *A, float *B, float *C, int N) {
    __shared__ float As[16][16], Bs[16][16];

    int tx = threadIdx.x, ty = threadIdx.y;

    As[ty][tx] = A[ty*N + tx];
    Bs[ty][tx] = B[ty*N + tx];

    __syncthreads();

    float sum = 0;
    for(int k=0;k<16;k++)
        sum += As[ty][k]*Bs[k][tx];

    C[ty*N + tx] = sum;
}
```

### Importance:

- Reduces global memory access
    
- __syncthreads ensures correctness
    

---

# **5C) Thread/Block Calculation (2 marks)**

Given:

- Blocks = 128 (2D)
    
- Grid.x = 32 → Grid.y = 4
    
- Threads/block = 30
    
- Block.y = 5 → Block.x = 6
    

### Formulas:

```
blockIdx.y = blockId / gridDim.x
blockIdx.x = blockId % gridDim.x

threadIdx:
x = threadId % blockDim.x
y = threadId / blockDim.x
```

---

# ✅ Final Tip for Writing

- 2 marks → short definition
    
- 3 marks → small code + explanation
    
- 4–5 marks → diagram + code + explanation
    

---

If you want, I can:

- Turn this into **perfect exam answers (formatted)**
    
- Or take each question and **viva-style drill you** 🔥