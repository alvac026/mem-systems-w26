+++
title = "Extra Resources"
[extra]
hide = false
+++

# Extra Reading

## Other Memory Papers
- Onur Mutlu: [Research Problems and Opportunities in Memory Systems](https://pdl.cmu.edu/PDL-FTP/NVM/memory-systems-research_superfri14.pdf)
- [TMO](https://www.cs.cmu.edu/~dskarlat/publications/tmo_asplos22.pdf), Meta's memory offloading infrastructure. 
- [EMT](https://www.usenix.org/conference/osdi25/presentation/chai-siyuan) - OSDI '25

### Disaggregated Memory
- [LegoOS: A Disseminated, Distributed OS for Hardware Resource Disaggregation](https://www.usenix.org/conference/osdi18/presentation/shan) - OSDI '18
- [TrackFM: Far-out Compiler Support for a Far Memory World](https://dl.acm.org/doi/10.1145/3617232.3624856) - ASPLOS '24
- [AIFM: High-Performance, Application-Integrated Far Memory](https://www.usenix.org/system/files/osdi20-ruan.pdf) - OSDI '20
- [GiantVM: A Novel Distributed Hypervisor for Resource Aggregation with DSM-aware Optimizations
](https://dl.acm.org/doi/10.1145/3505251) - TACO 19(2) 2024
- [Managing Memory Tiers with CXL in Virtualized Environments](https://www.usenix.org/system/files/osdi24-zhong-yuhong.pdf) - OSDI '24
- [Serverless End Game: Disaggregation enabling Transparency](https://arxiv.org/abs/2006.01251) - SESAME '24
- [vtism: Efficient Tiered Memory Management for Virtual Machines with CXL](https://dl.acm.org/doi/10.1145/3757347.3759131) - SYSTOR '25
- [Tiered Memory Management Beyond Hotness](https://www.usenix.org/conference/osdi25/presentation/liu) - OSDI '25

### PMEM
- [Mnemosyne: lightweight persistent memory](https://dl.acm.org/doi/10.1145/1950365.1950379) - ASPLOS '11
- [NVHeaps](https://dl.acm.org/doi/10.1145/1961296.1950380) - ASPLOS '11
- [SplitFS](https://dl.acm.org/doi/10.1145/3341301.3359631) - SOSP '19
- [Rethinking File Mapping for Persistent Memory](https://www.usenix.org/conference/fast21/presentation/neal) - FAST '21
- [PACTree: A High Performance Persistent Range Index Using PAC Guidelines](https://multics69.github.io/pages/pubs/pactree-kim-sosp21.pdf) - SOSP '21
- [Persistent State Machines for Recoverable In-memory Storage Systems with NVRam](https://www.usenix.org/conference/osdi20/presentation/zhang-wen) - OSDI '20

### Archival Storage
- Microsoft Research (MSR): [Project Silica](https://www.microsoft.com/en-us/research/wp-content/uploads/2023/09/ProjectSilica-SOSP23.pdf), long-term storage on glass!
- [OceanStore: an architecture for global-scale persistent storage](https://dl.acm.org/doi/10.1145/356989.357007) - ASPLOS 2000
- [A DNA-Based Archival Storage System](https://homes.cs.washington.edu/~luisceze/publications/dnastorage-asplos16.pdf) - ASPLOS '16

### Bio Storage
- [Managing Reliability Skew in DNA Storage](https://arxiv.org/abs/2204.12261) - ISCA '22

### Quantum
- [Systems Architecture for Quantum Random Access Memory](https://assets.amazon.science/70/be/027e7a9c47239991fb982d8dae7e/systems-architecture-for-quantum-random-access-memory.pdf) - MICRO '23
- [Fat-Tree QRAM: A High-Bandwidth Shared Quantum Random Access Memory for Parallel Queries](https://arxiv.org/html/2502.06767) - ASPLOS '25

### AI
- [Alpa](https://www.usenix.org/conference/osdi22/presentation/zheng-lianmin) - OSDI '22
- [Orca](https://www.usenix.org/conference/osdi22/presentation/yu) - OSDI '22
- [FlashAttention](https://arxiv.org/abs/2205.14135) - NeurIPS '22
- [RingAttention](https://arxiv.org/pdf/2310.01889) - 2023
- [SGLang](https://arxiv.org/abs/2312.07104) - NeurIPS '24
- [vAttention](https://arxiv.org/abs/2405.04437) - ASPLOS '25
- [WaferLLM](https://www.usenix.org/conference/osdi25/presentation/he) - OSDI '25
- [BlitzScale](https://www.usenix.org/conference/osdi25/presentation/zhang-dingyan) - OSDI '25
- [GPU Multitasking](https://arxiv.org/pdf/2508.08448) - 2025
- [Prism](https://arxiv.org/abs/2505.04021) - 2025

## Miscellaneous Research Advice
Here are some extra readings containing priceless advice on conducting systems research (and research more broadly)
from people that are much smarter than me:
- John Ousterhout: [Always Measure One Level Deeper](https://cacm.acm.org/research/always-measure-one-level-deeper/)
- Torsten Hoefler and Roberto Belli's excellent [paper](https://dl.acm.org/doi/10.1145/2807591.2807644) on applying sound benchmarking, data reporting techniques, and statistical methods in systems work. 
- David Patterson on the [importance of benchmarks](https://dl.acm.org/doi/pdf/10.1145/2209249.2209271)
- Richard Hamming's [guide on doing research](https://www.cs.virginia.edu/~robins/YouAndYourResearch.html). I was once given the advice to "keep this on your desk and read it every year." My thoughts on it have changed over the years. 
- Remzi Arpaci-Dusseau on [a proven problem-finding method](https://www.usenix.org/conference/atc19/presentation/keynote)
- Lamport on [how to present a paper](https://lamport.azurewebsites.net/pubs/howto.txt)
- John Regehr on [picking research problems](https://blog.regehr.org/archives/46)
- Saurabh Bagchi on [doing systems research](https://cacm.acm.org/blogcacm/computer-systems-research-the-joys-the-perils-and-how-to-count-beans-well/)
- Remzi again on [finding research problems](https://dl.acm.org/doi/10.1145/3568161.3568316)
- Lamport on [writing proofs](https://lamport.azurewebsites.net/pubs/lamport-how-to-write.pdf). We don't use proofs _often_ in systems work (much to Lamport's chagrin), but there are certainly cases where it is necessary. 

As you may have surmised from the above list, it turns out one of the most challenging problems in doing research, is finding the right problem to work on. 

## Fun Stuff
- James Mickens: [My Love Letter To Computer Science Is Very Short And I Also Forgot To Mail It](https://www.youtube.com/watch?v=4vd2rCBjHp8)
- Mickens again: [The Night Watch](https://www.usenix.org/system/files/1311_05-08_mickens.pdf)
