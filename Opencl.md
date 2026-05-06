Great — you already have a solid module, so I’ll do exactly what you asked:

1. **Give you a complete working OpenCL program (host + kernel)**
    
2. **Then teach OpenCL step-by-step using that code as the reference**, so you can _recreate it from scratch in an exam or interview_
    

---

# ✅ COMPLETE OPENCL PROGRAM (Vector Addition)

This is a **minimal but complete** OpenCL example.

### 🔹 File: `main.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <CL/cl.h>

#define N 1024

// OpenCL Kernel as string
const char* kernelSource =
"__kernel void vecadd(__global int* A, __global int* B, __global int* C) { \n"
"   int id = get_global_id(0);                                            \n"
"   C[id] = A[id] + B[id];                                                \n"
"}                                                                        \n";

int main() {

    // Host arrays
    int *A = (int*)malloc(sizeof(int) * N);
    int *B = (int*)malloc(sizeof(int) * N);
    int *C = (int*)malloc(sizeof(int) * N);

    for(int i = 0; i < N; i++) {
        A[i] = i;
        B[i] = i;
    }

    // OpenCL variables
    cl_platform_id platform;
    cl_device_id device;
    cl_context context;
    cl_command_queue queue;
    cl_program program;
    cl_kernel kernel;

    cl_mem bufferA, bufferB, bufferC;

    cl_int err;

    // STEP 1: Get platform
    clGetPlatformIDs(1, &platform, NULL);

    // STEP 2: Get device
    clGetDeviceIDs(platform, CL_DEVICE_TYPE_GPU, 1, &device, NULL);

    // STEP 3: Create context
    context = clCreateContext(NULL, 1, &device, NULL, NULL, &err);

    // STEP 4: Create command queue
    queue = clCreateCommandQueue(context, device, 0, &err);

    // STEP 5: Create buffers
    bufferA = clCreateBuffer(context, CL_MEM_READ_ONLY, sizeof(int)*N, NULL, NULL);
    bufferB = clCreateBuffer(context, CL_MEM_READ_ONLY, sizeof(int)*N, NULL, NULL);
    bufferC = clCreateBuffer(context, CL_MEM_WRITE_ONLY, sizeof(int)*N, NULL, NULL);

    // STEP 6: Copy data to device
    clEnqueueWriteBuffer(queue, bufferA, CL_TRUE, 0, sizeof(int)*N, A, 0, NULL, NULL);
    clEnqueueWriteBuffer(queue, bufferB, CL_TRUE, 0, sizeof(int)*N, B, 0, NULL, NULL);

    // STEP 7: Create and build program
    program = clCreateProgramWithSource(context, 1, &kernelSource, NULL, &err);
    clBuildProgram(program, 1, &device, NULL, NULL, NULL);

    // STEP 8: Create kernel
    kernel = clCreateKernel(program, "vecadd", &err);

    // STEP 9: Set arguments
    clSetKernelArg(kernel, 0, sizeof(cl_mem), &bufferA);
    clSetKernelArg(kernel, 1, sizeof(cl_mem), &bufferB);
    clSetKernelArg(kernel, 2, sizeof(cl_mem), &bufferC);

    // STEP 10: Define work size
    size_t globalSize = N;

    // STEP 11: Execute kernel
    clEnqueueNDRangeKernel(queue, kernel, 1, NULL, &globalSize, NULL, 0, NULL, NULL);

    // STEP 12: Read result
    clEnqueueReadBuffer(queue, bufferC, CL_TRUE, 0, sizeof(int)*N, C, 0, NULL, NULL);

    // Verify
    for(int i = 0; i < 10; i++)
        printf("%d + %d = %d\n", A[i], B[i], C[i]);

    // STEP 13: Cleanup
    clReleaseKernel(kernel);
    clReleaseProgram(program);
    clReleaseCommandQueue(queue);
    clReleaseContext(context);
    clReleaseMemObject(bufferA);
    clReleaseMemObject(bufferB);
    clReleaseMemObject(bufferC);

    free(A); free(B); free(C);

    return 0;
}
```

---

# 🧠 NOW: LEARN OPENCL FROM THIS CODE (CORE IDEA)

Forget theory overload. I’ll map everything to this program.

---

# 🔥 BIG PICTURE (MOST IMPORTANT)

From your module:

> OpenCL = **Host + Device model**

### ✔️ Host (CPU)

- Controls everything
    
- Runs `main()`
    

### ✔️ Device (GPU/CPU accelerator)

- Runs **kernel**
    

---

# 🧩 PART 1: KERNEL (Device Code)

```c
__kernel void vecadd(__global int* A, __global int* B, __global int* C) {
    int id = get_global_id(0);
    C[id] = A[id] + B[id];
}
```

### 🔹 Concepts

### 1. `__kernel`

- Means: “this runs on GPU”
    

### 2. `__global`

- Data is in **global memory (device memory)**
    

### 3. `get_global_id(0)`

- Gives **thread ID**
    
- Like:
    

```c
int id = threadIdx.x + blockIdx.x * blockDim.x; // CUDA equivalent
```

---

### 🔥 KEY IDEA

Instead of:

```c
for(i=0;i<N;i++)
```

We do:

👉 **1 thread = 1 element**

---

# 🧩 PART 2: HOST CODE FLOW (THE 13 STEPS)

Your PDF gives 13 steps — this code follows EXACTLY that.

---

## ✅ STEP 1–2: Platform & Device

```c
clGetPlatformIDs(...)
clGetDeviceIDs(...)
```

👉 You are asking:

- “Which GPU/CPU can run OpenCL?”
    

---

## ✅ STEP 3: Context

```c
context = clCreateContext(...)
```

👉 Think:

> Context = “environment where everything runs”

---

## ✅ STEP 4: Command Queue

```c
queue = clCreateCommandQueue(...)
```

👉 Think:

> Queue = “instructions sent to GPU”

---

## ✅ STEP 5: Buffers (IMPORTANT)

```c
bufferA = clCreateBuffer(...)
```

👉 GPU cannot access normal arrays

So we create:

- `bufferA` → GPU copy of A
    
- `bufferB`
    
- `bufferC`
    

---

## ✅ STEP 6: Copy Data

```c
clEnqueueWriteBuffer(...)
```

👉 CPU → GPU transfer

---

## ✅ STEP 7: Program Build

```c
clCreateProgramWithSource(...)
clBuildProgram(...)
```

👉 Compile kernel **at runtime**

---

## ✅ STEP 8: Kernel Object

```c
kernel = clCreateKernel(program, "vecadd", ...)
```

👉 Extract function from program

---

## ✅ STEP 9: Set Arguments

```c
clSetKernelArg(...)
```

👉 Pass buffers into kernel

---

## ✅ STEP 10–11: Execute

```c
clEnqueueNDRangeKernel(...)
```

👉 THIS launches parallel execution

### Important:

```c
size_t globalSize = N;
```

👉 Means:

- Create **N threads (work-items)**
    

From your module:

> Each work-item executes kernel independently

---

## ✅ STEP 12: Read Output

```c
clEnqueueReadBuffer(...)
```

👉 GPU → CPU transfer

---

## ✅ STEP 13: Cleanup

```c
clRelease...
```

👉 Free GPU resources

---

# 🧠 MOST IMPORTANT CONCEPTS (EXAM GOLD)

---

## ⭐ 1. Work-Item

👉 Single thread  
Executes kernel once

---

## ⭐ 2. NDRange

```c
globalSize = N;
```

👉 Total threads

---

## ⭐ 3. Work-Group

- Threads grouped together
    
- Share **local memory**
    

---

## ⭐ 4. Memory Types

From your module

|Type|Meaning|
|---|---|
|Global|GPU RAM|
|Local|Shared within group|
|Private|Per thread|
|Constant|Read-only|

---

# 🔥 HOW TO RECREATE THIS FROM SCRATCH (VERY IMPORTANT)

If I ask you in exam:

👉 “Write OpenCL program”

### You follow this template:

---

### 1. Kernel

```c
__kernel void func(...) {
   int id = get_global_id(0);
}
```

---

### 2. Host Steps

Memorize this order:

```
Platform
Device
Context
Queue
Buffers
Write Buffer
Program Build
Kernel
Set Args
NDRange
Read Buffer
Free
```

---

# 🚀 FINAL INTUITION

Think like this:

👉 CPU = Manager  
👉 GPU = Workers

Workflow:

1. Manager prepares data
    
2. Sends to workers
    
3. Workers run same function in parallel
    
4. Results come back
    

---

# If you want next:

I can:

- 🔥 Teach **memory model deeply (with diagrams)**
    
- 🔥 Convert this into **CUDA vs OpenCL comparison**
    
- 🔥 Give **exam-ready answers (2, 5, 10 marks)**
    
- 🔥 Or help you **write OpenCL from scratch without memorizing**
    

Just tell 👍