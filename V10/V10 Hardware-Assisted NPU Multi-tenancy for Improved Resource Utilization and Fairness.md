---
author: jsx
create_time: 2023-09-01_10:04
category: paper_review
---
## V10 Hardware-Assisted NPU Multi-tenancy for Improved Resource Utilization and Fairness 
- conference: ISCA
- time:23
- noteworthy author:
---
## Contribution
### 0.Background
###### NPU architecture
![](attachments/Pasted%20image%2020230901101103.png)
- ==systolic array==/Mat-mul unit(MXU)
	- processing elements --> multiply-accumulate operation
- ==vector unit==/Vector processing unit(VPU)
	- SIMD units
- vector registers
- ==SRAM buffers==/high bandwidth memory(HBM)
###### Single MM workload
- MM, Conv, etc. relies mainly on MXU
- shuffle, reshape, element-wise can only execute on VPU
- dependency usually happens between 2 kinds
- based on model structure, one ML model can be either MXU-bound or VPU-bound
### 1.Problem/Observation
###### The underutilization of NPU
- underutilized due to the **temporal idleness** of MXU and VPU.
	- e.g. when a reshape runs on VPU, the MXU is idle
- The idleness comes from limited operator-level parallelism caused by imbalanced use of MXU and VPU(**inter-dependency**)
- HBM **bandwidth is underutilized** as a consequence of FLOPS underutilization in NPUs
- use multi-tenancy/ preemptive multitasking may help, however, collocating similar works on the same NPU core may cause **resource contention**

### 2.Solution
- main idea: **simultaneously** executing **independent** SA and VU operators from **different tasks**

###### proposed structure
- online part:
	- core: tensor operator scheduler
		- operator dispatch logic
		- lightweight operator preemption mechanism
- offline part
	- collocation mechanism

###### Tensor Operator Scheduler
- dispatch logic
	- use DMA to load instr on chip
- Scheduling Policies
	- Round-Robin
	- Priority-Based
- preemption mechanism
	- to avoid large operator block small ones
	- ![](attachments/Pasted%20image%2020230901105907.png)

###### hardware support for preemptive
- Preempting a VU Operator: no intermediate states 
	- pause exec, save PC and regs
- Preempting an SA Operator: contain intermediate states
	- consider inputs, weights and partial sums
	- ![](attachments/Pasted%20image%2020230901110203.png)
	1. preemption invoked in cycle 1
	2. further inputs are saved
	3. After all partial sums **depending on earlier inputs** are popped, we pause the execution
	4. *the new weights begin to enter at the same time*
###### workload collocation
principle: collocate tasks not conflict in resources
- cluster-based mechanism
	- time: offline training phase
	- features: SA/VU utilizations, HBM bandwidth consumption, operator length
	- means of clustering: principal component analysis to extract features and K-means to classify
- runtime inference
	- use representative clusters' features to predict the collocation pair's performance
	- if prediction is higher than threshold, we can proceed to dispatching

### 3.Experiment
- setup:
	- models: 11models from MLPerf v2.1 and TPU reference models
	- collocating 2 work loads
	- simulator
	- baseline
		- PMT
		- V10-base: FIFO operator scheduling
		- V10-Fair: non-preemptive priority
		- V10-Full: preemption enabled
###### resource utilization in given models
![](attachments/Pasted%20image%2020230901112236.png)
###### context switch
![](attachments/Pasted%20image%2020230901112827.png)
###### effectiveness of clustering algorithm
![](attachments/Pasted%20image%2020230901113005.png)

---
## Questions
1. if the SA acts like the picture, there must be a cycle that new and old operator co-exist in the SA, won't this affect the results?
2. It's a bit strange that they don't talk about the memory model in the NPU even they measured the HBM bandwidth. They seems so sure that the HBM capacity won't be a bottle neck
---
## Reflection
1. The idea is very nice: in a GPU we also have a tensor core and a CUDA core. And we do observe that there're some element-wise operations that idle the whole pipeline. Can we have some optimization?
