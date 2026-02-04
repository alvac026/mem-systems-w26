+++
title = "nuKSM: NUMA-aware Memory De-duplication on Multi-socket Servers"
[extra]
bio = """ """
[[extra.authors]]
name = " Allen Lee(Leader / Presentor)"
[[extra.authors]]
name = "Jared Ho(Scribe)"
[[extra.authors]]
name = "Deptmer Ashely(Scribe)"
[[extra.authors]]
name = "John Aebi(Blogger)"
[[extra.authors]]
name = "Brian Castellon Rosales(Blogger)"
+++

# Introduction
Memory de-duplication is an important aspect of Linux’s Kernel memory management system. Where pages are being scanned in main memory for pages with duplicate content. When two pages are found with the same contents, one file will remain unchanged where the other file will be mapped to the same physical address. This releases or frees the extra physical pages to be allocated for other needs. When two virtual addresses share a physical address both pages are marked as “copy-on-write” where the kernel will remap the virtual address to have its own copy once the process has decided to write to the virtual address. This was implemented to run more virtual machines on a host by sharing memory between users and processes. 

This paper nuKSM: NUMA-aware Memory De-duplication on Multi-socket Servers” proposes a new way of memory deduplication by making “nuKSM” which is a NUMA “Non-uniform-memory-access” aware. The goal of this paper is to create a KSM implementation that equally spreads the “NUMA-tax” across all NUMA nodes. Instead of making an arbitrary decision of where to consolidate memory, the nuKSM makes a decision of which process to consolidate the data based on the priority of that node. 

# Background and Motivation
KSM is very good at what it is meant to do. It effectively can save memory through de-duplication in a cost-effective way. Where it struggles is on multi-core processes such as servers. This often will not work to the same extent on multi-socket processes because it is unaware of what CPU’s are close to what memory and the priority of each CPU comparative to others in a system. This leads to:

* Cores have a higher workload to fetch memory and process it which causes high performance variation between CPU nodes.
* Subverting priority from high priority processes that may have high remote memory access.
* KSM provides no way of tuning the system to balance and appropriate where data goes and the hierarchy of multi-core processes.

Overall this paper goes into detail about how these problems can be minimized by using a NUMA aware model. This can spread out where memory is being accessed. Memory accessed more frequently by a certain core will be placed in closer proximity to said core.

# Concepts and Definitions
* NUMA-Tax: Overhead cost or latency of a certain node trying to access remote memory.
* De-Duplication: Where two or more pages in virtual memory have the same data. They will be merged all sharing the same physical address.
* Snice Value: Positive integer between 1 and 41 where the lower the number is the higher priority of a certain process.
* nuShare(p): A number between 0 and 1 that captures the preference of the current process whose page is scanned. This is relative to all other pages being scanned for the same content that will be de-duplicated. The higher the value the more likely it is that the content of the page will be local to that process.
* Stable Tree: Already de-duplicated pages are stored here.
* Unstable Tree: All potential candidates (pages) that have not been de-duplicated between the two most recent scans are considered for de-duplication.

# Implementation
The nuKSM has 3 main goals to achieve when implementing this on top of the already existing KSM in Linux kernel version 5.4.0:
*  Addressing Performance Variability and Unfairness.

**  Keeps a de-duplicated page on a NUMA node that is expected to access that page often.
**  NUMA-Tax is paid when accessing a page on a remote node. This Tax is much smaller the more infrequently a node may access remote memory.
**  Acts to evenly distribute this Tax among all nodes by checking the amount of times data is accessed by nodes and tries to distribute how much “Tax” each of them will pay in order to access said data.
**  Requires knowledge of access frequency of pages to be de-duplicated. Uses accessed and referenced bits already available in the page reclamation algorithm in Linux.
*  Priority Based Memory De-Duplication
	
**  Creates a way to program priority for a virtual machine with a higher priority allowing for more access to local memory.
**  nuShare equation used to calculate if a process should have more priority than all others that share the soon to be de-duplicated page. 
**  This value means the higher it is, the more likely it will be local to that process.
**  Compared to a random number between 0 and 1 if nuShare is larger than this number, then the scanned page will be de-duplicated and local to said process.
**  Makes it so the ratio of priority between processes converges (meaning that the ratios are reflected in the amount of pages being de-duplicated).
*  Enhancing Responsiveness

**  Utilization of forests (many unstable and stable trees) instead of a single stable and unstable tree.
**  The index of a page is found by a function of the checksum of that page (index = page_checksum(page) % number of trees).
**  If pages index into different trees they will never be compared so this will reduce the amount of unnecessary page comparisons.
**  This is scalable because the amount of trees reflects the amount of physical memory. If memory is doubled the amount of trees will be proportionally doubled to fit that amount of physical memory.
**  Makes the average height of a tree stay the same which would limit tree traversal and then makes the amount of comparisons similar. This makes it scalable for many different systems with different memory sizes.
**  One stable and one unstable tree will have around 100MiB each which balances cost and benefit of using a de-centralized forest.

# Evaluation and Results
The authors conducted a study on a dual-socket Intel Xeon Gold 6140 server with 18 cores and 192 GiB memory per socket. Base frequency for the processor is  3.2 GHz. Using Linux v5.4.0 with the kernel running Ubuntu18.04 guest OS. They extended the same kernel to test KSM vs nuKSM. Both of these operate at the same scan rate for pages (1K pages every 100ms). They executed VM on specific nodes VM-0 running on node-0 and executed instance-0 of the applications, VM1 would run on node-1 and executes instance-1. They then would test specific workloads and logged memory intensive micro-benchmarks that are specifically sensitive to NUMA.

*  Two VMs running identical applications were placed on different nodes. KSM vs nuKSM performance difference ranged from 15% up to 46% for MySQL RandomAccess. Whereas the difference was negligible for nuKSM with a 4% variability with the same RandomAccess.
*  To calculate fairness they took the minimum and maximum slowdown and divided it by the maximum slowdown between the two identical application instances.
*  This produced a fairness value between 0 & 1, where values closer to 1 indicate more equal performance across VMs.
*  Under KSM, fairness values dropped significantly for memory-intensive workloads, showing a large imbalance between identical applications.
*  With nuKSM enabled, fairness values were consistently close to 1 across all tested benchmarks, indicating much more balanced execution.
*  nuKSM achieved nearly the same amount of memory de-duplication as KSM.
*  This demonstrates that nuKSM improves fairness without reducing memory savings or overall system throughput.
*  Priority-based evaluations showed that nuKSM correctly placed a larger fraction of de-duplicated pages on the NUMA node of higher priority VMs.
*  As a result, higher-priority workloads experienced fewer remote memory accesses and improved runtimes, avoiding priority subversion seen in KSM.

# Strengths and Weaknesses:
Strengths:
*  nuKSM provides a practical improvement over KSM by directly addressing NUMA-related performance variability rather than disabling de-duplication entirely.
*  It reuses existing Linux kernel mechanisms, such as active and inactive page lists, which avoids adding expensive new tracking overhead.
*  The priority-based de-duplication mechanism effectively prevents priority subversion and gives users control over where de-duplicated pages are placed.
*  nuKSM maintains nearly identical memory savings and overall throughput compared to KSM while significantly improving fairness.
*  The decentralized forest based design scales well to large-memory systems and improves responsiveness under heavy memory pressure.

Weaknesses:
*  nuKSM does not account for available memory capacity on individual NUMA nodes when making placement decisions.
*  Modifying KSM at this level is complex and may be difficult to deploy or maintain in production systems.
*  Many real-world servers disable KSM entirely due to unpredictability or because memory constraints are less critical today.
*  The approach assumes traditional NUMA architectures and may not extend cleanly to emerging heterogeneous memory systems such as NVLink or RDMA-based memory.

# Class Discussion
*  Does nuKSM have checks for how much memory is available on different NUMA nodes?
**  Not addressed, KSM will merge by default (last node accessed)
**  nuKSM often not implemented due to low-level issues, often solved before needing nuKSM implementation
*  Servers often just disable KSM due to availability
**  Good to look into how much memory KSM saves over large periods of time
**  Memory may not have been as cheap when paper was proposed
*  Modifying KSM may not be the best solution to solving the NUMA problem
**  Hard to implement KSM methods especially with increasing non-unified memory (NVLink, RDMA, etc.)
**  Works under the assumption that memory capacity is needed

# Conclusion
Overall, nuKSM demonstrates that memory de-duplication and NUMA management cannot be treated as independent systems on modern multi-socket servers. While KSM is effective at reducing memory usage, its NUMA-unaware design leads to unfair performance variability and priority subversion. nuKSM addresses these issues by making de-duplication decisions based on access frequency, priority, and scalability, resulting in more balanced performance without sacrificing memory savings. Although real-world adoption may be limited due to implementation complexity and changing hardware trends, the paper highlights an important systems lesson that optimizing one kernel subsystem in isolation can create significant side effects elsewhere, and that NUMA-awareness is critical for predictable performance on modern architectures.

# References
[1] A. Panda, A. Panwar, and A. Basu, “nuKSM: NUMA-aware Memory De-duplication on Multi-socket Servers,” *Proceedings of the 30th International Conference on Parallel Architectures and Compilation Techniques (PACT)*, Oct. 2021, pp. 258–269, doi: 10.1109/PACT52795.2021.00026.

# AI Disclosure
*  ChatGPT was used to aid in research and fixing grammatical errors.
*  Generative AI can be useful tools for tasks such as summarizing or drafting, however, they may give inaccurate information confidently and should always have generated information validated

