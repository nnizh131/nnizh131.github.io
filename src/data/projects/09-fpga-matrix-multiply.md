---
title: FPGA Matrix Multiplier
description: Hardware-accelerated matrix multiplication on an FPGA using DSP slice MAC units for parallel dot product computation.
draft: false
tags: [Verilog, FPGA, DSP, Hardware]
---

## Overview

Matrix multiplication is the core operation behind neural networks, signal processing, and scientific computing. Running it on a CPU is straightforward but slow — each multiply-accumulate happens sequentially. An FPGA lets you unroll that computation into actual parallel hardware, where many MAC units operate simultaneously.

## The MAC Unit

The fundamental building block of an FPGA matrix multiplier is the multiply-accumulate (MAC) unit. Because a dot product requires taking the sum of products:

$$
C_{ij} = \sum_{k} A_{ik} \cdot B_{kj}
$$

the FPGA uses hardware primitives inside its DSP slices to perform these calculations concurrently. Each DSP slice contains a dedicated multiplier and accumulator — running at full clock speed, with no shared resources.

## Why DSP Slices

Modern FPGAs include hardened DSP48 blocks specifically for multiply-accumulate workloads. Using them instead of soft logic means:

- **Higher clock frequency** — the multiplier path is pre-placed in silicon
- **Lower resource usage** — one DSP slice vs. hundreds of LUTs for the same operation
- **Deterministic timing** — no routing variation across synthesis runs

## Parallelism

The key advantage over a CPU is structural parallelism. For an $N \times N$ matrix multiply, a fully unrolled design instantiates $N^2$ MAC units — one per output element — computing all dot products simultaneously. In practice the design is partially unrolled to fit within the available DSP slice budget, with the tradeoff between throughput and resource utilization tuned to the target device.

## Findings

Structural parallelism is the FPGA's fundamental advantage over a CPU. A CPU computes one multiply-accumulate at a time; an FPGA computes $N^2$ simultaneously. The bottleneck shifts from computation to resource budgeting — how many DSP slices are available and how to partition the problem to fit within them. Timing closure was the other constraint: each additional pipeline stage increases throughput but lengthens the critical path through the accumulator chain, and the synthesis tool's timing report becomes the primary feedback loop.

Floating-point on an FPGA is more expensive than fixed-point but unavoidable for general matrix workloads where dynamic range matters. The DSP48E1 slices handle integer multiply natively; floating-point required chaining multiple slices together, which made resource utilization and clock frequency directly trade off against each other.

## Stack

- Verilog HDL
- Xilinx Zynq-7000 SoC / DSP48E1 slices
- Floating-point arithmetic
