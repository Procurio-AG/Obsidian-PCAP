# Study Guide: Collective Communication in MPI

This guide provides an overview of collective communication in the Message Passing Interface (MPI), designed to help you understand how groups of processes work together to share and process data efficiently.

---

## 1. Introduction to Collective Communication

Collective communication involves a group of processes in a communicator working together. Unlike point-to-point communication (involving only two processes), collective operations engage the entire scope of the communicator.

### Key Concepts:
*   **Scope:** All processes within the communicator (default: `MPI_COMM_WORLD`) must participate. If a process does not reach the collective call, the program may hang or experience unexpected failures.
*   **Programmer Responsibility:** It is your duty to ensure every process executes the collective routine to prevent synchronization issues.

### Types of Collective Operations:
1.  **Synchronization:** Processes wait until all members of the group reach a specific point (e.g., `MPI_Barrier`).
2.  **Data Movement:** Processes exchange/distribute/gather data among themselves (e.g., `MPI_Bcast`, `MPI_Scatter`).
3.  **Collective Computation:** One or more processes collect data from the group and perform an operation (min, max, sum, etc.) on it (e.g., `MPI_Reduce`).

---

## 2. Predefined MPI Reduction Operators

When performing collective computations, MPI uses predefined operators to combine data values. These are passed to routines like `MPI_Reduce` and `MPI_Scan`.

*   **Arithmetic/Logic:**
    *   `MPI_SUM`: Sum of all elements.
    *   `MPI_PROD`: Product of all elements.
    *   `MPI_MAX`: Maximum value.
    *   `MPI_MIN`: Minimum value.
*   **Bitwise Operations:**
    *   `MPI_BAND`: Bitwise AND.
    *   `MPI_BOR`: Bitwise OR.
    *   `MPI_BXOR`: Bitwise Exclusive OR.
*   **Logical Operations:**
    *   `MPI_LAND`: Logical AND.
    *   `MPI_LOR`: Logical OR.
    *   `MPI_LXOR`: Logical Exclusive OR.
*   **Location Operators:**
    *   `MPI_MAXLOC`: Maximum and its location.
    *   `MPI_MINLOC`: Minimum and its location.

---

## 3. MPI_Bcast (One-to-All Broadcast)
**Analogy:** A teacher (root) hands out the same handout to every student (all other processes) in the class.

*   **Purpose:** Send data from one process (the root) to all other processes in the communicator.
*   **Function Signature:**
    ```c
    MPI_Bcast(void *buffer, int count, MPI_Datatype datatype, int root, MPI_Comm comm)
    ```
*   **Note:** All processes must call this function with the same `root`, `datatype`, and `count` arguments.

---

## 4. MPI_Reduce (Reduction Operation)
**Analogy:** All employees (processes) send their individual salary data to the accountant (root), who sums them up to calculate the total payroll.

*   **Purpose:** Collect data from all processes, apply an operator (e.g., `MPI_SUM`), and store the result on the `root` process.
*   **Function Signature:**
    ```c
    MPI_Reduce(void *sendbuf, void *recvbuf, int count, MPI_Datatype datatype, MPI_Op op, int root, MPI_Comm comm)
    ```
*   **Usage:** Often used for global calculations. Processes can perform local calculations (partial sums) before calling `Reduce` to combine them.

---

## 5. MPI_Scatter (One-to-All Scatter)
**Analogy:** A deck of cards is held by the dealer (root). The dealer distributes a portion of the deck to every player (all processes).

*   **Purpose:** Break a buffer at the root into chunks and distribute one chunk to each process. It is the inverse of `MPI_Gather`.
*   **Key Detail:** The `sendbuf` is only significant at the root process.
*   **Function Signature:**
    ```c
    MPI_Scatter(void *sendbuf, int sendcount, MPI_Datatype sendtype, void *recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm)
    ```

---

## 6. MPI_Gather (All-to-One Gather)
**Analogy:** Each student (process) hands a page of their report to the professor (root), who stapler them together in rank order.

*   **Purpose:** Collect distinct pieces of data from all processes and aggregate them into a single buffer at the root.
*   **Ordering:** The data is gathered into the root's `recvbuf` based on the rank of the sending process.
*   **Function Signature:**
    ```c
    MPI_Gather(void *sendbuf, int sendcount, MPI_Datatype sendtype, void *recvbuf, int recvcnt, MPI_Datatype recvtype, int root, MPI_Comm comm)
    ```

---

## 7. MPI_Allgather (All-to-All Gather)
*   **Purpose:** Similar to `MPI_Gather`, but instead of just the root receiving the aggregated data, **every process** receives the full gathered result.
*   **Comparison:** `MPI_Allgather` is effectively an `MPI_Gather` followed by an `MPI_Bcast`.
*   **Difference:** There is no `root` argument in the function signature, as all processes participate equally in the result.

---

## 8. MPI_Alltoall (All-to-All Personalized Communication)
*   **Purpose:** Each process sends a unique chunk of data to every other process. It is a full permutation of data among processes.
*   **Logic:** The $j^{th}$ block sent from process $i$ is received by process $j$ and placed in the $i^{th}$ block of its receive buffer.
*   **Relation:** An extension of `MPI_Allgather`. It is effectively `MPI_Scatter` performed by every process.

---

## 9. MPI_Scan (Prefix Reduction)
*   **Purpose:** Computes a prefix reduction (e.g., partial sums). Unlike `MPI_Reduce`, which gives one final scalar at the root, `MPI_Scan` returns the running operation result to *each* process.
*   **Example:**
    *   If you have 4 processes with values {1, 2, 3, 4}:
    *   `MPI_Reduce` (Sum) at root gives: 10.
    *   `MPI_Scan` (Sum) gives: Process 0 gets 1; Process 1 gets 3 (1+2); Process 2 gets 6 (1+2+3); Process 3 gets 10 (1+2+3+4).
*   **Function Signature:**
    ```c
    MPI_Scan(void *sendbuf, void *recvbuf, int count, MPI_Datatype datatype, MPI_Op op, MPI_Comm comm)
    ```

---

## Key Takeaways
1.  **Collective operations are holistic:** You must consider the entire group of processes.
2.  **Roots are critical:** Functions like `Scatter`, `Gather`, and `Reduce` define a specific process as the 'root' to coordinate data collection or distribution.
3.  **Avoid Deadlock:** Ensure all processes call the collective functions in the same order and with matching communication parameters.
4.  **Know your operators:** Use `MPI_Reduce` for a final global result, but use `MPI_Scan` when you need intermediate, cumulative steps across the process ranks.