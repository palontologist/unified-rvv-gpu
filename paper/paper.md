# Fused CPU-GPU Computing on RISC-V: A Unified RVV Architecture for Edge and Embedded Platforms

**Author:** George Karani
**Affiliation:** FrontForum Focus
**Email:** georgekarani44@gmail.com

**Workshop:** PARMA-DITAM 2027
**Co-located with HiPEAC, Glasgow, Scotland, 18/01/2027**

---

## Abstract

Discrete GPUs provide high throughput for graphics and machine-learning workloads, but they impose separate instruction sets, memory spaces, and host-device data movement that are increasingly costly for energy-constrained edge and embedded platforms. This paper presents a fused CPU-GPU architecture built on RISC-V and its Vector (RVV) extension, in which general-purpose RISC-V cores equipped with wide vector units replace a discrete graphics processor. By unifying the instruction set, register-file conventions, and memory space, the design removes host-device transfers, simplifies the software stack, and enables fine-grained, energy-aware scheduling across heterogeneous workloads. We describe the microarchitecture, including a Tomasulo-style reorder buffer, per-thread L1 caches, sparsity-aware execution, and 128-bit packed INT8/INT4 vector units, and report on a Verilog prototype (tiny-gpu) targeting an AMD Xilinx Artix-7 FPGA carrier board. Preliminary simulation results demonstrate correct execution of matrix addition, matrix multiplication, and vector DP4A kernels with configurable thread counts. We discuss trade-offs, current limitations, and a roadmap toward cache coherence, branch divergence, and IEEE-754 FP32 support.

**Keywords:** RISC-V; RVV; unified CPU-GPU; heterogeneous computing; vector SIMD; open-source hardware; edge AI; energy efficiency.

---

## 1. Introduction

The rapid growth of on-device artificial intelligence, real-time graphics, and sensor processing has pushed computation toward the edge. In these settings, the classical split between a general-purpose CPU and a discrete GPU is increasingly inefficient: data must be copied across buses, two distinct ISAs and programming models must be maintained, and the energy cost of moving data often exceeds the cost of computing on it. At the same time, RISC-V has emerged as an open, extensible, and royalty-free ISA, and its Vector (RVV) extension provides a standardized, variable-length SIMD programming model that is well suited to both CPU-style control flow and GPU-style data parallelism.

This paper proposes a **fused CPU-GPU architecture** in which RISC-V cores with RVV vector units subsume the role of a separate GPU. Rather than treating the accelerator as a distinct device, every core is a first-class RISC-V processor that can execute scalar control code and wide vector kernels interchangeably. The result is a single ISA, a unified memory space, and a single programming model, while retaining the parallelism and throughput expected of a GPU.

Our contributions are:

1. A microarchitectural blueprint for a unified CPU-GPU RISC-V many-core, including scheduling, memory, and execution policies.
2. A concrete, open-source Verilog prototype (tiny-gpu) that implements RV32IM, custom INT4/FP32 and 128-bit vector extensions, sparsity-aware execution, out-of-order execution via a Tomasulo-style reorder buffer (ROB), per-thread L1 caches, and a graphics pipeline.
3. A custom Artix-7 FPGA carrier board specification for physical bring-up.
4. Preliminary simulation results validating matrix addition, matrix multiplication, and vector DP4A kernels.

The remainder of this paper is organized as follows. Section 2 reviews background and related work. Section 3 presents the proposed architecture. Section 4 describes the implementation. Section 5 reports evaluation. Section 6 discusses limitations and future work. Section 7 concludes.

## 2. Background and Motivation

### 2.1 The Cost of Discrete Acceleration

In a conventional system, the CPU and GPU maintain separate virtual address spaces and communicate through explicit copies over PCIe or a shared interconnect. For edge devices, this separation manifests as three penalties:

- **Data-movement energy:** transferring operands between host and device can dominate energy consumption for small kernels.
- **Software complexity:** two ISAs, two compilers, and two runtimes must be maintained and synchronized.
- **Latency:** kernel launch and synchronization overheads penalize fine-grained, interactive workloads.

### 2.2 RISC-V and RVV as a Unifying Substrate

RISC-V's modularity allows the same base ISA to scale from microcontrollers to high-performance cores. The RVV extension adds vector registers and instructions that operate on variable-length vectors, enabling a single binary to adapt to different hardware widths. Crucially, RVV lives in the same register file and memory model as the scalar base ISA, so a core can interleave scalar control flow with vector data-parallel operations without device boundaries. This makes RISC-V + RVV a natural substrate for fusing CPU and GPU functionality.

### 2.3 Related Work

Open-source GPU implementations such as Miaow and VeriGPU demonstrate that GPU microarchitecture can be studied and extended in the open, but they typically retain a distinct accelerator model and a complex codebase. tiny-gpu takes a different approach: it starts from a minimal, fully documented RISC-V scalar core and incrementally adds GPU-relevant features (custom ML extensions, sparsity, out-of-order execution, caches, graphics, and vector SIMD), making the design transparent and educational while remaining extensible.

## 3. Proposed Unified Architecture

### 3.1 Design Principles

The fused architecture follows four principles:

1. **One ISA, one memory space.** All cores execute RISC-V; vector operations use RVV-style packed registers. There is no separate device address space.
2. **Composable cores.** Each core is a RISC-V CPU with a vector unit; cores are replicated to scale throughput like a GPU.
3. **Energy-aware execution.** Sparsity detection skips work for zero operands; caches reduce off-chip traffic.
4. **Open and sovereign.** The design is implemented in open Verilog and targets open toolchains, aligning with sovereign-hardware goals.

### 3.2 Core Microarchitecture

Each core contains:

- A 7-stage pipeline (IDLE, FETCH, DECODE, REQUEST, WAIT, EXECUTE, UPDATE).
- Scalar RV32IM ALU and register file (32 x 32-bit).
- Vector register file (16 x 128-bit) and vector ALU.
- Per-thread private L1 data cache (2-way set-associative, write-through).
- Tomasulo-style ROB for out-of-order writeback and hazard management.
- Program counter and load-store unit.

Threads are grouped into blocks and dispatched to cores by a central dispatcher. Each thread maintains private architectural state (PC, scalar and vector registers, L1 cache) while executing the same kernel in SIMD fashion.

### 3.3 Scheduling and Sparsity

A single scheduler advances a block through the pipeline. Before EXECUTE, it checks the ROB for unresolved writes to the source registers; if a hazard exists, the pipeline stalls until resolution or forwarding. For ML workloads, the scheduler detects when all threads in a block have zero-valued operands for an ALU operation and skips EXECUTE, writing zero directly to save power and cycles.

### 3.4 Memory System

Global memory is shared and unified. Each core's threads access it through private L1 caches (write-through, LRU replacement). A memory controller arbitrates among channels to throttle traffic and relay responses. The data bus is 128 bits wide when vector mode is enabled, allowing a single load/store to move a full vector register.

### 3.5 ISA Extensions

The prototype extends RV32IM with custom opcodes:

| Opcode | Name | Description |
|---|---|---|
| CUSTOM0 | RET | Thread retirement |
| CUSTOM1 | DP4A / FP32 | Scalar INT4 dot-product accumulate; FP32 stubs |
| CUSTOM2 | Vector SIMD | 128-bit packed vector operations |

Vector operations include 16-lane INT8 add/multiply/MAC, 4-lane INT4 DP4A (each 8 x INT4), and FP32 stubs, plus a prefetch hint. Special registers (%blockIdx, %blockDim, %threadIdx) are mirrored in both scalar and vector register files to support the SIMT programming model.

## 4. Implementation

The prototype is implemented in approximately 20 SystemVerilog files and evolved through seven milestones:

1. RV32IM scalar core.
2. INT4/FP32 custom extensions for ML inference.
3. Sparsity-aware execution.
4. Out-of-order execution with Tomasulo-style ROB.
5. Per-thread L1 data cache.
6. Graphics pipeline (rasterizer, texture unit, framebuffer).
7. 128-bit vector extensions (INT8/INT4 SIMD, FP32 stubs).

### 4.1 Hardware Platform

For physical bring-up, a custom carrier board targets:

- FPGA: AMD Xilinx Artix-7 (XC7A100T-2FGG484I).
- Memory: DDR3L x16 (MT41K256M16TW-107:P).
- Video output: HDMI 1.4 via SII9022A transmitter.
- Power: USB-C PD with multi-rail regulation (1.0 V, 1.8 V, 1.35 V, 3.3 V).

The board was developed with a code-first workflow (Solder CLI) and is KiCad-compatible.

### 4.2 Verification

Simulation uses Icarus Verilog and cocotb. Kernels are expressed as RISC-V instruction sequences loaded into program memory; data is loaded into unified data memory; thread count is set in the device control register; execution is launched via a start signal.

## 5. Evaluation

### 5.1 Functional Validation

The prototype has been validated in simulation on three kernels:

- **Matrix addition (RV32I):** 8 threads, each computing C[i] = A[i] + B[i], exercising load/store and the %blockIdx/%blockDim/%threadIdx SIMT pattern.
- **Matrix multiplication (RV32IM):** accumulated dot-products using mul/div and branching.
- **Vector DP4A (CUSTOM2):** 128-bit vector SIMD using VDP4A.I4 for high-throughput INT4 dot-products.

### 5.2 Design Parameters

| Parameter | Default | Description |
|---|---|---|
| NUM_CORES | 2 | Compute cores |
| THREADS_PER_BLOCK | 4 | SIMD width per core |
| ROB_DEPTH | 16 | ROB entries per core |
| L1_SETS | 16 | L1 sets per thread |
| VECTOR_ENABLE | 1 | 128-bit data bus (0 = 32-bit) |
| DATA_MEM rows | 256 | Word-addressable data rows |
| PROGRAM_MEM rows | 256 | Word-addressable instruction rows |

### 5.3 Discussion of Metrics

Cycle counts are kernel- and configuration-dependent; the matrix-addition test completes in approximately 1.33 x 10^9 simulation nanoseconds at the configured clock (see results.xml). Rather than reporting a single headline number, we emphasize correctness, configurability, and the ability to study microarchitectural trade-offs (cache size, ROB depth, vector width) in an open, modifiable design. Area and power estimates will be reported after synthesis on the target FPGA.

## 6. Limitations and Future Work

- **FP32 is stubbed.** Vector and scalar FP32 paths are placeholders pending IEEE-754-compliant IP.
- **No cache coherence.** Caches are per-thread and write-through; a shared L2 and coherence protocol are future work.
- **No branch divergence handling.** All threads in a block execute in lockstep.
- **No multi-warp scheduling.** One block runs to completion per core.
- **Memory coalescing** is not implemented.
- **Physical bring-up** of the carrier board is in progress.

## 7. Conclusion

This paper presented a fused CPU-GPU architecture on RISC-V that replaces a discrete accelerator with vector-capable RISC-V cores, unifying ISA, memory, and programming model. An open Verilog prototype demonstrates the approach across seven milestones, from a scalar RV32IM core to 128-bit vector SIMD, with sparsity-aware and out-of-order execution. Preliminary simulation validates core kernels. The design is intended as a transparent, extensible platform for research in heterogeneous, energy-efficient, and sovereign computing, and aligns with the PARMA-DITAM focus on RISC-V, accelerators, and embedded multicore platforms.

---

## References

1. A. Waterman and K. Asanovic, "The RISC-V Instruction Set Manual, Volume I: User-Level ISA," RISC-V Foundation, 2019.
2. RISC-V International, "RISC-V Vector Extension (RVV) Specification," 2021.
3. K. Asanovic et al., "The Rocket Chip Generator," UC Berkeley, 2016.
4. M. Owaida et al., "Miaow: An Open Source GPU," 2015.
5. H. Perkins, "VeriGPU: An Open Source Verilog GPU," 2020.
6. J. L. Hennessy and D. A. Patterson, "Computer Architecture: A Quantitative Approach," 6th ed., Morgan Kaufmann, 2019.
7. G. Karani, "tiny-gpu: A RISC-V-based GPU in Verilog," GitHub, 2026.
8. Dagstuhl Publishing, "OASIcs - OpenAccess Series in Informatics," https://www.dagstuhl.de/oasics.

---

*Draft prepared for PARMA-DITAM 2027. Author metadata and references to be finalized before submission.*
