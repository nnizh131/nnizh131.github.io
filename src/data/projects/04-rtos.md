---
title: Embedded OS Internals
description: Implementations of core OS subsystems — multithreading, CPU scheduling (FCFS/SJF/RR/MLFQ), energy-aware real-time scheduling, virtual memory, and file systems.
draft: false
tags: [C, Python, Operating Systems, Real-Time Systems]
---

## Overview

A series of OS subsystem implementations covering the full stack of an embedded operating system: concurrency and multithreading, CPU scheduling, real-time task scheduling with power management, virtual memory, and file system internals. Each subsystem was built from scratch and evaluated against concrete metrics — response time, page faults, disk accesses, and energy consumption.

## Multithreading

A baseline single-threaded program computes the sum $\sum_{i=0}^{N} i$ sequentially. The multithreaded version divides the range across $k$ worker threads using POSIX pthreads, each computing a partial sum over its slice and writing the result into a shared array. The main thread then reduces:

$$
S = \sum_{t=0}^{k-1} \text{partial}[t]
$$

Speedup scales predictably with thread count up to the number of physical cores, then plateaus as scheduling and synchronization overhead erode the gains.

## CPU Scheduling

Four schedulers implemented in C, each managing a pool of child processes whose workload is prime factorization — CPU-bound with predictable variation in runtime depending on input size:

**FCFS** — processes run to completion in arrival order. Simple, but a single long job blocks everything behind it.

**SJF** — shortest job runs first. Optimal for average response time when job sizes are known in advance, which is the key assumption.

**Round Robin** — each process gets a fixed time quantum before being preempted. The quantum is the main tuning parameter: too small and context-switch overhead dominates; too large and it degrades to FCFS.

**MLFQ** (Multi-Level Feedback Queue) — a two-level queue where the top queue uses Round Robin and the bottom uses FCFS. New processes enter at the top; if they exhaust their quantum, they drop to the lower queue. This approximates SJF without knowing job lengths in advance.

Process control used UNIX signals: `SIGSTOP` to pause a process, `SIGCONT` to resume it. Response time across workloads with varied and uniform job sizes was the primary metric.

## Real-Time Scheduling with Energy Optimization

A set of periodic tasks — each with a period $T_i$ and worst-case execution time $\text{WCET}_i$ — must be scheduled on a CPU that supports four discrete frequencies: 1188, 918, 648, and 384 MHz. Running slower uses less power, but takes longer, and tasks have hard deadlines.

Two scheduling policies:

**Rate Monotonic (RM)** — fixed priority assigned by period: shorter period → higher priority. Optimal among fixed-priority algorithms for independent periodic tasks. Schedulability can be checked with the utilization bound:

$$
U = \sum_{i=1}^{n} \frac{\text{WCET}_i}{T_i} \leq n\left(2^{1/n} - 1\right)
$$

**Earliest Deadline First (EDF)** — dynamic priority: whichever task has the nearest absolute deadline runs next. Optimal for preemptive uniprocessor scheduling; can achieve full utilization ($U \leq 1$) where RM cannot.

Energy is computed as:

$$
E = \frac{P \cdot t}{1000} \quad \text{(mJ)}
$$

where $P$ is active power in mW at the chosen frequency and $t$ is execution time in seconds. The scheduler finds idle intervals after meeting all deadlines and uses them to run at reduced frequency — the DVFS (Dynamic Voltage and Frequency Scaling) strategy that saves energy without missing any deadline.

## Virtual Memory

A simulation of how an OS manages a 64 KB address space (16-bit) against 16 KB of physical memory — 32 page frames of 512 bytes each. Every memory reference decomposes into a page number and offset:

$$
\text{virtual address} = \underbrace{\text{page number}}_{7\text{ bits}} \;\|\; \underbrace{\text{page offset}}_{9\text{ bits}}
$$

A one-level page table per process maps virtual pages to physical frames. On a miss the OS pages in from disk, evicting a resident page if all 32 frames are occupied. If the evicted frame was dirty (written since it was loaded), it must be written back first — costing an extra disk access.

Four replacement policies were implemented and compared:

- **RAND** — evicts a random page; no metadata required
- **FIFO** — evicts the oldest resident page; subject to Bélády's anomaly
- **LRU** — evicts the least recently used page, tracked via access timestamps; on ties, prefers non-dirty pages, then the lower-numbered page
- **PER** (Periodic Reference Reset) — maintains a reference bit per page, reset to 0 every 200 references; eviction priority: unused → unreferenced+clean → unreferenced+dirty → referenced+clean → referenced+dirty

## File Systems

The file system layer imposes structure on flat block storage using a small set of core abstractions.

**Storage layout** — a disk is divided into fixed-size blocks. A superblock at a known offset describes the volume: block size, total block count, the location of the inode table, and the free-block bitmap.

**Inodes** — each file or directory is represented by an inode storing metadata (size, permissions, timestamps) and an array of block pointers. Small files fit entirely in direct pointers; larger files add an indirect block whose content is an array of further block pointers — extending addressable file size without enlarging the inode.

**Directories** — a directory is a file whose content is a list of `(name, inode number)` pairs. Resolving `/data/log.txt` walks the chain: root inode → directory block → `data` entry → its inode → its directory block → `log.txt` entry → its inode → file data.

**Free-space management** — a bitmap tracks which blocks are free. Allocation scans for the first 0-bit, flips it, and returns the block number.

**Journaling** — writes are recorded in a circular log before being applied to their final locations. On a crash, the OS replays the journal from the last checkpoint, preventing corruption from a mid-write power loss.

## Findings

**Scheduling** — MLFQ delivered the best average response time across mixed workloads by naturally prioritizing short jobs without requiring advance knowledge of job length. FCFS was worst under heterogeneous loads; SJF was optimal only under its unrealistic assumption. Round Robin's performance was highly sensitive to quantum size — the right value was workload-dependent.

**Real-time scheduling** — EDF consistently achieved higher CPU utilization than RM. RM becomes unschedulable as utilization approaches ~69% for large task sets, while EDF can schedule up to 100% utilization. DVFS on idle intervals reduced energy consumption measurably without any deadline misses.

**Virtual memory** — LRU performed best under workloads with temporal locality. PER outperformed LRU on bursty patterns because the periodic reference reset approximates a working-set boundary without tracking exact timestamps. FIFO showed the worst fault rates and exhibited Bélády's anomaly under certain reference strings.

**Multithreading** — linear speedup held until the thread count matched the physical core count. Beyond that, synchronization overhead and OS scheduling noise flattened the curve.

## Stack

- C (pthreads, UNIX signals, process management)
- Python (virtual memory simulation, real-time scheduler and energy optimizer)
