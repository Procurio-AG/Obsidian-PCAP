# AGENTS.md

This file provides context about this Obsidian vault for AI agents.

## Vault Overview

This vault is a structured technical knowledge base and curriculum dedicated to parallel computing and high-performance programming. It serves as a comprehensive repository for learning and documenting GPU architectures, CUDA development, OpenCL frameworks, and the Message Passing Interface (MPI).

## Organization

The vault is organized into distinct modules based on specific technologies and architectural concepts. Each folder represents a core pillar of parallel computing:

- **Cuda_1 & Cuda_2**: Split into fundamental and advanced CUDA concepts, covering memory management, 1D convolution, and complex kernels.
- **Opencl**: Detailed breakdown of the OpenCL execution model, environment, and program lifecycle.
- **MPI_new**: Focuses on distributed memory programming, point-to-point communication, and performance benchmarking.
- **Mod_1 new**: Covers foundational parallel computer structures, Flynn's taxonomy, and modern GPU architecture.

Notes follow a strict sequential numbering pattern (e.g., `1_...`, `2_...`) within each folder, suggesting a linear curriculum or a step-by-step learning path. The file naming convention is consistent, using lowercase descriptions with underscores for readability.

## Key Topics

- CUDA Programming (Memory management, thread organization, global ID calculation, and atomic operations)
- OpenCL Framework (Kernels, execution environment, program objects, and data transfer)
- Message Passing Interface (MPI) (Collective communication, point-to-point messaging, and basic datatypes)
- GPU Architecture (Memory hierarchy, thread scheduling, latency hiding, and Streaming Multiprocessors)
- Parallel Algorithms (Parallel scan, Sparse Matrix-Vector Multiplication (SpVM), Tiled Matrix Multiplication, and 1D Convolution)
- Parallel Computing Fundamentals (Flynn's taxonomy, architectural classification, and parallel programming models)

## User Preferences

The user prefers highly structured, technical, and sequential documentation. The systematic numbering of files indicates that the progression of topics is critical, moving from introductory concepts to advanced implementation details.

Writing should be technically precise and detailed, reflecting the depth of the existing notes on hardware architecture and low-level optimization. The user values clarity and a logical flow that mirrors a textbook or course structure.

## Custom Instructions

- When suggesting new content or files, strictly adhere to the existing `[Number]_[descriptive_title]` naming convention using underscores and lowercase.
- Maintain a high level of technical detail, especially regarding memory hierarchies, synchronization, and hardware-specific optimizations.
- Reference existing parallel computing concepts like 'tiling', 'latency hiding', or 'SM assignment' to maintain consistency with the vault's technical vocabulary.
- Ensure any new notes are placed within the appropriate module folder (e.g., CUDA-specific content in `Cuda_1` or `Cuda_2`).
