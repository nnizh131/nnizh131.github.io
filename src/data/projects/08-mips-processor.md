---
title: MIPS Processor Design
description: Single-cycle, multi-cycle, and pipelined MIPS microarchitectures implemented in Verilog and validated on a Zynq FPGA.
draft: false
tags: [Verilog, FPGA, Computer Architecture, MIPS]
---


![Xilinx Zynq-7000 SoC development board (ZedBoard) — the target platform for all three processor designs](/zynq-7000.webp)

## Overview

Three complete MIPS processor microarchitectures designed from scratch in Verilog HDL, progressively adding complexity and performance. Each design was tested by running real MIPS programs on a Zynq FPGA platform.

## The Three Architectures

### 1. Single-Cycle

The simplest design — every instruction completes in exactly one clock cycle. The critical path is determined by the slowest instruction (typically a load), so the clock period is fixed to that worst case. Clean and easy to reason about, but inefficient.

### 2. Multi-Cycle

Instructions are broken into multiple shorter steps, each taking one clock cycle. The clock runs faster because each cycle only does one piece of work. A finite state machine (FSM) sequences the steps for each instruction type, allowing simpler instructions to finish sooner than complex ones.

### 3. Pipelined

The full five-stage pipeline — **Fetch → Decode → Execute → Memory → Writeback** — with multiple instructions in-flight simultaneously. This is where architecture gets interesting: independent instructions overlap, but dependent instructions create hazards.

Two classes of hazards had to be resolved:

- **Data hazards** — an instruction reads a register before a prior instruction has written it. Solved with forwarding (bypassing the writeback stage directly to execute) and stalling when forwarding isn't sufficient (e.g., after a load).
- **Control hazards** — a branch changes the PC after instructions have already entered the pipeline. Handled by flushing instructions fetched from the wrong path.

## Findings

Pipelining is the point where the hardware stops being straightforward. The hazard unit alone requires carefully tracking every instruction in every stage simultaneously and deciding — every cycle — whether to forward, stall, or flush. Getting that right in Verilog, and then validating it against a suite of MIPS programs on real hardware, is where the intuition for processor design actually develops.

## Stack

- Verilog HDL
- Xilinx Zynq FPGA
- MIPS instruction set architecture
