# Submission Metadata (PARMA-DITAM 2027)

## Title
Fused CPU-GPU Computing on RISC-V: A Unified RVV Architecture for Edge and Embedded Platforms

## Author
George Karani
FrontForum Focus
georgekarani44@gmail.com

## Abstract
Discrete GPUs provide high throughput for graphics and machine-learning workloads, but they impose separate instruction sets, memory spaces, and host-device data movement that are increasingly costly for energy-constrained edge and embedded platforms. This paper presents a fused CPU-GPU architecture built on RISC-V and its Vector (RVV) extension, in which general-purpose RISC-V cores equipped with wide vector units replace a discrete graphics processor. By unifying the instruction set, register-file conventions, and memory space, the design removes host-device transfers, simplifies the software stack, and enables fine-grained, energy-aware scheduling across heterogeneous workloads. We describe the microarchitecture, including a Tomasulo-style reorder buffer, per-thread L1 caches, sparsity-aware execution, and 128-bit packed INT8/INT4 vector units, and report on a Verilog prototype (tiny-gpu) targeting an AMD Xilinx Artix-7 FPGA carrier board. Preliminary simulation results demonstrate correct execution of matrix addition, matrix multiplication, and vector DP4A kernels with configurable thread counts. We discuss trade-offs, current limitations, and a roadmap toward cache coherence, branch divergence, and IEEE-754 FP32 support.

## Keywords
RISC-V; RVV; unified CPU-GPU; heterogeneous computing; vector SIMD; open-source hardware; edge AI; energy efficiency.
