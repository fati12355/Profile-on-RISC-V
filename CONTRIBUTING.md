# Contributing Guide

## Project Goal

This project is an experimental **RISC-V Vector Extension (RVV) workload profiler for AI operators**.

The goal is to compare how foundational AI kernels behave when implemented in:

1. A scalar RISC-V version
2. A vectorized RVV version

We are not only trying to answer:

> “Which version is faster?”

We are trying to understand:

> “Why is it faster or slower?”

The project focuses on performance engineering questions such as:

* How many dynamic instructions does each implementation execute?
* How many load/store operations are required?
* How many vector instructions are generated?
* Does vectorization reduce instruction count?
* Does the kernel become memory-bound?
* How does performance change with input size?
* What information can we extract from simulator traces?

This repository should read like a small architecture/performance investigation, not just a collection of C files.

---

## Repository Structure

```text
rvv-ai-operator-profiler/
  kernels/
  include/
  scripts/
  results/
  docs/
  tests/
  README.md
  CONTRIBUTING.md
```

---

## `kernels/`

This folder contains the AI operator implementations.

Each operator should have at least two versions:

```text
operator_scalar.c
operator_rvv.c
```

Example:

```text
gemm_scalar.c
gemm_rvv.c
relu_scalar.c
relu_rvv.c
```

### Scalar files

Scalar files should contain plain C implementations without RVV intrinsics.

They are the baseline used for comparison.

A scalar kernel should prioritize:

* Correctness
* Readability
* Simple loop structure
* Clear memory access patterns

### RVV files

RVV files should contain implementations using RISC-V Vector intrinsics from:

```c
#include <riscv_vector.h>
```

These files should clearly show where vector operations are used.

Each RVV implementation should include comments explaining:

* Vector length setup
* Load operations
* Compute operations
* Store operations
* Tail handling when input size is not divisible by vector length

---

## `include/`

This folder contains shared headers.

Possible files:

```text
benchmark.h
kernels.h
matrix_utils.h
```

Headers should define:

* Shared function declarations
* Common data types
* Constants used across benchmarks
* Helper function prototypes

Avoid putting large implementations in header files unless necessary.

---

## `scripts/`

This folder contains automation scripts.

Possible files:

```text
build.sh
run_spike.sh
run_qemu.sh
parse_trace.py
plot_results.py
```

### `build.sh`

Builds scalar and RVV binaries.

It should clearly show compiler flags such as:

```bash
-march=rv64gcv
-mabi=lp64d
```

### `run_spike.sh`

Runs compiled binaries using Spike.

This is mainly for correctness checking and instruction tracing.

### `run_qemu.sh`

Runs compiled binaries using QEMU if supported.

This can be used as an alternate execution environment.

### `parse_trace.py`

Parses simulator traces and extracts useful statistics.

Example metrics:

* Total dynamic instructions
* Number of vector instructions
* Number of scalar arithmetic instructions
* Number of loads
* Number of stores
* Number of branches

### `plot_results.py`

Generates visual summaries from benchmark output.

Example plots:

* Scalar vs RVV instruction count
* Vector instruction ratio
* Load/store count by matrix size
* Estimated arithmetic intensity

---

## `results/`

This folder contains measured benchmark results.

Results should be organized by experiment.

Example:

```text
results/
  gemm_64x64/
    scalar_trace.txt
    rvv_trace.txt
    summary.csv
    plots/
  gemm_128x128/
    scalar_trace.txt
    rvv_trace.txt
    summary.csv
    plots/
```

Each result folder should explain:

* Kernel tested
* Input size
* Compiler flags
* Simulator used
* Date of experiment
* Main observations

Do not commit extremely large trace files unless they are small enough to be useful.

For large traces, commit summaries instead.

---

## `docs/`

This folder contains technical explanations.

Possible files:

```text
methodology.md
rvv_notes.md
profiling_notes.md
future_work.md
```

### `methodology.md`

Explains how experiments are run.

Should include:

* Benchmark setup
* Input sizes
* Compiler flags
* Simulator choice
* Metrics collected
* Limitations of the measurements

### `rvv_notes.md`

Explains RVV concepts used in the project.

Possible topics:

* Vector registers
* Vector length
* `vsetvl`
* Tail handling
* Vector loads and stores
* Why RVV is different from fixed-width SIMD

### `profiling_notes.md`

Explains what each metric means.

Example:

* Instruction count
* Vector utilization
* Load/store pressure
* Arithmetic intensity
* IPC
* Cache misses

Important note:

Spike and basic QEMU are useful for ISA-level analysis, but they do not provide full microarchitectural bottleneck analysis such as accurate cache misses or pipeline stalls. Those require a more detailed simulator such as gem5 or real hardware performance counters.

### `future_work.md`

Tracks possible extensions.

Example ideas:

* Add LayerNorm
* Add Softmax
* Add INT8 GEMM
* Add gem5 support
* Add real RISC-V board measurements
* Add comparison against auto-vectorized compiler output

---

## `tests/`

This folder contains correctness tests.

Every optimized kernel must be checked against the scalar reference.

Possible files:

```text
test_gemm.c
test_relu.c
```

Tests should verify:

* Output correctness
* Edge cases
* Small input sizes
* Non-multiple-of-vector-length sizes

Performance results are only meaningful if correctness is proven first.

---

## Good First Tasks

These are small tasks that new contributors can pick up.

### Beginner-friendly

* Add comments to an existing scalar kernel
* Add a small correctness test
* Add a new input size to benchmark
* Improve README explanations
* Document one RVV intrinsic used in the project
* Clean up script output formatting

### Intermediate

* Implement ReLU scalar and RVV versions
* Add CSV output to benchmark results
* Improve trace parsing logic
* Add plots for instruction mix
* Add support for multiple matrix sizes

### Advanced

* Implement RVV GEMM optimization
* Add LayerNorm
* Add Softmax
* Add gem5 simulation support
* Add real hardware performance counter support
* Analyze cache behavior using a detailed simulator

---

## Contribution Rules

Before contributing, make sure your change is:

* Correct
* Reproducible
* Documented
* Easy to review

A good contribution should answer:

1. What did you change?
2. Why did you change it?
3. How did you test it?
4. What result changed?

---

## Coding Style

C code should be simple and readable.

Prefer:

```c
for (int i = 0; i < n; i++) {
    ...
}
```

over clever code that is difficult to understand.

Use meaningful names such as:

```c
matrix_a
matrix_b
matrix_c
vector_length
```

Avoid unclear names such as:

```c
x
y
z
tmp
```

unless the meaning is obvious in context.

---

## Benchmarking Rules

Do not compare two kernels unless:

* They compute the same result
* They use the same input data
* They are compiled with documented flags
* They are tested for correctness
* The experiment can be reproduced

Every benchmark result should include:

* Kernel name
* Input size
* Scalar or RVV version
* Compiler
* Compiler flags
* Simulator or hardware target
* Metrics collected

---

## Project Philosophy

This project is not just about writing a faster kernel.

It is about developing performance intuition.

A strong contribution helps explain the relationship between:

* Algorithm
* Compiler
* ISA
* Vectorization
* Memory access pattern
* Hardware behavior

The final goal is to build a project that demonstrates the mindset of a performance architect:

> Measure carefully, explain clearly, and connect software behavior to hardware consequences.
