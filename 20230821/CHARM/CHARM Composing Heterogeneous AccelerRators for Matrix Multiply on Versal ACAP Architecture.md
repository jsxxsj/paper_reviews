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
- Resource partition
	- AIE and PL: proportional
	- memory: same at first --> fine-tune

#### runtime scheduling
2 processes --> track each layer

### 3.experiment
- setup
	- VCK 190
- baseline
	- one monolithic, 8 duplicate acc on VCK 190
- results
	- ![](attachments/Pasted%20image%2020230823103725.png)
---
## Questions
- about single acc modeling
	- What's the exact shape changing in a MM/convolution/AIE MM
---
## Reflection
Using the view of optimization to plan the resources and tasks on GPU will be a future target of ours. However, the tasks and resources on GPU may not be all the same as on VCK190, and so does the performance, thus a new analytic model have to be explored and setup. 

