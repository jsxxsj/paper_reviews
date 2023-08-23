---
author: jsx
create_time: 2023-08-22_13:47
category: paper_review
---
## CHARM Composing Heterogeneous AccelerRators for Matrix Multiply on Versal ACAP Architecture 
- conference: FPGA
- time: 2023
- noteworthy author: Zhuang, Jinming
---
## Contribution
### 1.problem/observation
Low utilization in (1)big monolithic accelerator + (2)small matrix multiplication size/ batch dot
![](attachments/Pasted%20image%2020230823100737.png)

### 2.solution
1. multiple acc. --> how to flexible to different size?
2. --> 2-level, large scale integer optimization

#### optimization for single acc
Target: max throughput --> Matrix size & time
Resources/ constrains:
- AIE unit
- PL unit
- performance modeling(TIME)

#### optimization for multi acc
Target: max overall throughput
- Task partition: sort-based
	- tasks of similar sizes are clustered
	- 

### 3.experiment

---
## Questions

---
## Reflection

