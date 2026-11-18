# unified-rvv-gpu

**Fused CPU-GPU computing on RISC-V** — an independent research and prototyping project exploring a unified, vector-capable architecture and software stack.

This repository collects my design notes, paper draft, and early runtime/codegen scaffolding for a unified RISC-V CPU+GPU direction (RVV/custom-extension inspired), with a focus on a single programming model, capability-driven runtime, and bandwidth-aware execution on constrained hardware targets.

## Contents

- **`paper/`** — Draft paper and submission materials: *"Fused CPU-GPU Computing on RISC-V: A Unified RVV Architecture for Edge and Embedded Platforms"* (PARMA-DITAM 2027 draft), including source and PDF.
- **`docs/unified-cpu-gpu-rvv.md`** — Unified CPU-GPU RVV architecture/design notes.
- **`tinygpu/`** — A minimal, backend-pluggable runtime and codegen scaffold (Python) intended to target an RTL simulator, FPGA, and future unified silicon.
  - `device.py` — portable `Device` interface (`caps`, alloc/free, copy, compile, launch, sync) and `DeviceCaps`
  - `buffer.py`, `kernel.py` — core types
  - `backends/sim.py` — simulator backend stub (to integrate with RTL/cocotb or a functional ISA simulator)
  - `codegen.py` — lowering stubs (RV32IM / custom / vector-style instruction emission)
- **`examples/tinygpu/`** — Example flows using the runtime scaffold (e.g. `vector_add.py`).

## Status

Early-stage / work in progress:
- Runtime and codegen are scaffolds; simulator launch/copy paths need to be wired to a real RTL harness or functional ISA simulator.
- Architecture and software co-design are evolving (memory coalescing, caching, tiling, and bandwidth-aware scheduling are key priorities for FPGA targets such as Artix-7 + DDR3L).

## Quick start (scaffold)

```bash
PYTHONPATH=. python3 examples/tinygpu/vector_add.py
```

This runs the current scaffold flow (device capabilities, buffer allocation, kernel placeholder) and prints TODOs for simulator/FPGA integration.

## Related work

This project is developed independently and builds on lessons from prior open-source GPU experimentation. A companion Linux kernel/driver exploration for tiny-gpu bring-up lives in a separate repository.

## Author

George Karani (FrontForum Focus) — georgekarani44@gmail.com
