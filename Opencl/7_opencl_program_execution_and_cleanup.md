# Module: OpenCL Program Execution and Cleanup

## Introduction
This module covers the final, critical stages of the OpenCL program lifecycle. After an OpenCL kernel has been compiled and its arguments set, it must be dispatched to the hardware, its results retrieved, and all allocated resources properly freed to ensure system stability and avoid memory leaks.

---

## 1. OpenCL Primitive Data Types for Host Applications
To ensure portability across different hardware architectures, OpenCL provides standard data types that have fixed bit widths. These types should be used in host-side applications when dealing with OpenCL buffers.

| Scalar Data Type | Bit Width | Purpose |
| :--- | :--- | :--- |
| `cl_char` | 8 | Signed two's complement integer |
| `cl_uchar` | 8 | Unsigned two's complement integer |
| `cl_short` | 16 | Signed two's complement integer |
| `cl_ushort` | 16 | Unsigned two's complement integer |
| `cl_int` | 32 | Signed two's complement integer |
| `cl_uint` | 32 | Unsigned two's complement integer |
| `cl_long` | 64 | Signed two's complement integer |
| `cl_ulong` | 64 | Unsigned two's complement integer |
| `cl_half` | 16 | Half-precision floating-point value |
| `cl_float` | 32 | Single-precision floating-point value |
| `cl_double` | 64 | Double-precision floating-point value |

---

## 2. STEP 11: Enqueue the Kernel for Execution
Once the memory objects (buffers) are populated and the kernel arguments are correctly mapped, the host must request that the device begin execution. This is performed using the `clEnqueueNDRangeKernel()` function.

### Key Concepts
*   **The Function:** `clEnqueueNDRangeKernel()` acts as the "trigger" to start parallel computation on the device.
*   **Dimensionality (`work_dim`):** Specifies the number of dimensions (1, 2, or 3) for the work-items.
*   **Work Sizes:**
    *   **Global Work Size:** The total number of work-items to be executed.
    *   **Local Work Size:** The size of the work-groups (sub-divisions of the global range).
*   **Asynchronous Nature:** This call is **asynchronous**. It returns control to the host immediately after the command is placed in the command queue, rather than waiting for the kernel to finish executing on the GPU.

---

## 3. STEP 12: Read the Output Buffer Back to the Host
After the kernel completes its execution on the device, the computed results reside in device memory (specifically, in the output buffer objects). The host must explicitly move this data back to host-accessible memory.

### The Process
*   **Retrieval:** Use `clEnqueueReadBuffer()`. This command copies data from the device buffer (e.g., `bufferC`) into a host-side pointer (e.g., array `C`).
*   **Blocking/Non-blocking:** The call can be configured to block the host until the transfer is complete, ensuring data integrity before the host attempts to process it.
*   **Verification:** Once data is back on the host, it is common practice to loop through the results and compare them against expected values to ensure the kernel executed correctly (e.g., checking if `C[i]` matches the sum of input indices).

---

## 4. STEP 13: Release OpenCL Resources
Resource management is mandatory. Because OpenCL objects and buffers are often allocated in special device memory, failing to release them will result in memory leaks that can degrade system performance or cause crashes.

### Cleanup Protocol
The final step of the program must release resources in the correct order:
1.  **OpenCL Objects:** Use the respective release functions:
    *   `clReleaseKernel(kernel)`
    *   `clReleaseProgram(program)`
    *   `clReleaseCommandQueue(cmdQueue)`
    *   `clReleaseMemObject(bufferA/B/C)`
    *   `clReleaseContext(context)`
2.  **Host Memory:** Use the standard C `free()` function to release the host-side arrays (`A`, `B`, `C`) and any dynamically allocated lists (e.g., `platforms`, `devices`) created during the discovery phase.

---

## Key Takeaways
*   **Portability:** Use OpenCL-specific types (e.g., `cl_int`, `cl_float`) to ensure your host application behaves consistently across different vendor platforms.
*   **Async Dispatch:** `clEnqueueNDRangeKernel` does not block; your program must manage synchronization (often via events or `clFinish`) if it needs to wait for the device to finish before proceeding.
*   **Data Flow:** Remember the lifecycle: *Allocate (Host) -> Create Buffer (Device) -> Write (Host to Device) -> Execute (Kernel) -> Read (Device to Host) -> Cleanup.*
*   **Discipline:** Always pair every "Create" call with a corresponding "Release" call to maintain system health.