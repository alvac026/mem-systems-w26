+++
title = "An Empirical Guide to the Behavior and Use of Scalable Persistent Memory"
[extra]
# bio = """ """
[[extra.authors]]
name = "Nolan Cutler (Leader/Presentor)"
[[extra.authors]]
name = "Soren Emmons (Leader/Presentor)"
[[extra.authors]]
name = "Jared Ho (Blogger)"
[[extra.authors]]
name = "Deptmer Ashley (Blogger)"
[[extra.authors]]
name = "Allen Lee (Scribe)"
+++

## Purpose

This paper studies **how real Intel Optane Persistent Memory behaves** and explains why earlier research based on emulation produced misleading conclusions.

Main goals:
- Characterize Optane on real hardware
- Compare real behavior vs emulation assumptions
- Extract practical software design guidelines
- Reevaluate prior persistent-memory systems

Key insight:

> Persistent memory is **not just slower DRAM** — performance depends heavily on access patterns, access size, and concurrency.

## What Scalable Persistent Memory Is

Optane DIMMs introduce a new tier between DRAM and storage.

### Core properties
- Non-volatile (data survives power loss)
- Byte-addressable
- Installed on the memory bus
- Slower than DRAM, faster than SSDs
- Higher density than DRAM

## Operating Modes

#### App-Direct Mode
Persistent memory visible to software\
Applications use load/store instructions\
Used for durable data structures

#### Memory Mode
DRAM acts as cache\
Appears as a volatile memory extension\
Persistence hidden from software

The paper focuses on **App-Direct mode**.

---

## Internal Architecture Overview

The internal architecture of scalable persistent memory comprises several components that collectively shape its performance characteristics. The Integrated Memory Controller (iMC) manages memory traffic through dedicated read and write pending queues, while persistence guarantees are provided by the Asynchronous DRAM Refresh (ADR). Specifically, stores become durable once they reach the write pending queue (WPQ) within the iMC, even before data is committed to the underlying media. Access to the persistent memory device is coordinated by an on-DIMM controller (XPController), which performs address translation and mediates operations on the storage medium. Internally, the device operates at a 256-byte access granularity (XPLine), causing small writes to incur additional internal operations. To reduce this overhead, the controller employs a small write-combining buffer (XPBuffer, approximately 16 KB) that merges adjacent writes prior to media updates. Consequently, write operations may exhibit low apparent latency because completion is acknowledged at the ADR boundary. However, sustained bandwidth remains limited by the rate at which the XPBuffer drains data to the underlying 3D-XPoint media.

## Experimental Methodology

To accurately characterize persistent memory behavior, the authors developed a custom microbenchmarking framework called LATTester. The methodology was designed to minimize software and operating-system interference in order to isolate hardware-level effects. Kernel threads were pinned to fixed CPU cores, interrupts and hardware prefetchers were disabled, and memory accesses were performed on pre-populated addresses to eliminate page-fault overhead. Measurements relied on precise cycle-level timing while systematically sweeping a large parameter space, including access patterns, access sizes, and concurrency levels. The primary objective of this approach was to capture intrinsic hardware behavior rather than performance artifacts introduced by software abstractions or system noise.

---

## Key Hardware Findings

### Latency

- Read latency: ~2-3× slower than DRAM
- Sequential reads are significantly faster than random reads
- Write latency appears similar due to DRAM because it is acknowledged at the iMC (ADR domain)


### Tail Latency

Rare outliers exist where some writes take up to 50 µs (100× slower than normal). This is likely caused by internal remapping for wear-leveling or thermal management.


### Bandwidth and Thread Scaling

**DRAM:** Scales predictably with thread count.\
> **vs.**

**Optane:**  Performance is non-monotonic. It peaks at low thread counts (e.g., 1-4 threads for non-interleaved writes) and then drops due to contention in the XPBuffer


### Access Size Effects

- Internal access granularity = **256 bytes**
- Writes smaller than 256B cause severe bandwidth loss
- Caused by internal read-modify-write operations


### 4KB Performance Dip

Observed performance drop around 4KB:

- Memory controller contention
- DIMM interleaving imbalance

Behavior not captured by emulation.


### Sequential vs Random Access

Optane strongly prefers:
- Sequential access
- Large contiguous writes

Random small writes cause:
- Bandwidth collapse
- Increased latency
- Poor scaling


## Emulation Was Inaccurate

Common emulation techniques:
- DRAM with added delays
- NUMA DRAM emulation
- Software simulators
- Hardware emulators

Problems:
- Failed to model read/write asymmetry
- Ignored sequential preference
- Missed small-write penalties
- Produced misleading conclusions


### Case Study: RocksDB

Emulation result:
Fine-grained persistence looked best.

Real Optane result:
Write-ahead logging performs better.

Reason:
Sequential logging matches hardware strengths.

---

## Software Design Guidelines

Based on their empirical characterization, the authors derive a set of practical design guidelines for software targeting persistent memory systems. First, applications should avoid random accesses smaller than 256 bytes, as the device’s internal write granularity introduces read–modify–write amplification for fine-grained updates; preserving spatial locality improves efficiency. Second, large transfers should preferentially use non-temporal stores, which bypass the cache hierarchy and reduce unnecessary cache-line traffic, thereby improving sustained write bandwidth. Third, the number of concurrent threads targeting a single DIMM should be limited, since excessive concurrency increases contention within controller queues and write buffers, leading to performance collapse beyond a small optimal operating point. Finally, software should avoid NUMA accesses whenever possible, as remote persistent memory accesses incur significant latency and bandwidth penalties, particularly under mixed read–write workloads.

## Case Study Insights

### NOVA Filesystem

The paper demonstrates the practical impact of its design guidelines through optimizations applied to the NOVA persistent-memory file system. In its original design, frequent small metadata updates resulted in inefficient fine-grained writes that degraded performance on Optane hardware. To mitigate this issue, the authors restructured updates by embedding write data within larger sequential log entries, thereby increasing write locality and reducing internal write amplification. This modification significantly improved performance, yielding up to a sevenfold reduction in latency for small writes.

### PMDK Micro-Buffering

The authors further evaluate micro-buffering techniques within PMDK-based transactional workloads. Their analysis shows that the choice of persistence mechanism should depend on object size: conventional cached stores combined with cache-line write-back instructions are more efficient for small objects, whereas non-temporal stores provide higher bandwidth for larger updates by bypassing the cache hierarchy. Experimental results identify a performance crossover point at approximately 1 KB, beyond which non-temporal stores become preferable.

### Multi-DIMM Awareness

Another case study highlights the importance of hardware-aware thread placement. By balancing and pinning threads across multiple DIMMs rather than allowing uncontrolled sharing, the system reduces contention within memory controllers and improves overall bandwidth utilization. This optimization resulted in performance improvements of up to 34%, demonstrating the importance of aligning software parallelism with the underlying memory topology.

---

## Class Discussion Notes Summary

### Historical context
- CPU clock scaling slowed around early 2000s.
- Industry shifted toward thread-level parallelism.
- Adding cores became more effective than increasing complexity.

### Architecture discussion
- Simpler cores scale more easily.
- Memory bottlenecks dominate modern systems.
- Directory coherence helps multi-core scalability.

### Memory bottleneck insight
- Reads can be shared between caches.
- Writes serialize through memory controllers.
- Memory bandwidth becomes the limiting factor.

---

## Conclusion

This paper highlights how persistent memory introduces a new set of design challenges that cannot be understood by simply thinking of it as slower DRAM. Through real hardware measurements, the authors show that performance is highly sensitive to access size, locality, and concurrency, and that many assumptions made in earlier emulation-based studies do not hold in practice. The key takeaway is that software must be designed with clear awareness of how the hardware actually behaves. In practice, this means favoring sequential access patterns, avoiding small random writes, and carefully managing thread placement and memory topology. More broadly, the paper serves as a reminder that emerging hardware technologies often require rethinking established design intuition, and that meaningful system optimization ultimately depends on evaluating real devices rather than relying solely on simulation or emulation.

### AI Disclosure 
ChatGPT was used to summarize the paper, notes, and class discussion. The generative AI created a template which was then reviewed, edited and revised by the group.
