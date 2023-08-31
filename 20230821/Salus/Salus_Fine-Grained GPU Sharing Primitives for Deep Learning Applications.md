---
author: jsx
create_time: 2023-08-25_10:53
category: paper_review
---
## Salus_Fine-Grained GPU Sharing Primitives for Deep Learning Applications 
- conference: MLsys
- time: 2020
- noteworthy author:
---
## Contribution
### 1.problem/observation
poor granularity of GPU allocation; the entire GPU
 - an application can have multiple GPUs, but each GPU can only be allocated to exactly one application
 - hinders the scheduling ability 
 - not all DL jobs can fully utilize a GPU all the time --> lower GPU utilization

### 1.5 Backgrounds
###### The Workload of GPU
- Two high-level resources: computation resources + GPU memory
	- GPU memory $!=$ shared memory
	- memory is more important for models to run(regardless quickness)
		- GPU needs to load all model in to execute

###### GPU memory usage pattern
- Heterogeneous ==Peak== Memory Usage Across Jobs 
	- ![](attachments/Pasted%20image%2020230825110944.png)
	- High memory variations suggest that even during peak allocation periods, it may be possible to run multiple networks on the same GPU
- Temporal Memory Usage Variations Within a Job
	- highly predicable among iterations
- Low Persistent Memory Usage
	- most of (temp) data created and destroyed within an iteration 
### 2.solution
###### Overview
![](attachments/Pasted%20image%2020230825111839.png)
1. When job created, **Salus adaptor** in the DL framework creates a corresponding **session**, and transfer the **computation graph**
2. session request a **lane** from the **memory manager**
3. During runtime, **iterations** are generated and forwarded to the corresponding session...
4. … then scheduled by the iteration scheduler and sent to GPU
###### job switching and scheduling
- Characterizing DL Memory Allocations
	- Model: persistent --> small
	- Ephemeral: temp --> large
	- Framework: almost persistent --> small 
- scheduling timing
	- between iterations
	- lower granularity brings larger overhead
	- growth allocation brings deadlock

###### Lane memory allocation
![](attachments/Pasted%20image%2020230825113058.png)
```
memory |--> persistent
	   |--> ephemeral |--> lane
					  |--> lane
					  ... 
``` 
- lanes can be assigned to multi jobs
- lanes can auto defragmentation
###### Results
These new mechanics enable Salus to use more advanced scheduling policy like SRT instead of FIFO:
- **PACK**: pack multiple jobs together to reach higher efficiency
- **SRTF**: shortest remaining time first
- **FAIR**: equal time sharing
### 3.experiment
###### setup
- Intel Xeon E5-2670 machine
- 2 NVIDIA Tesla P100 GPUs (16GB on-chip memory).
- Tensor-Flow v1.5.0
- CUDA 8.0

###### baseline 
- FIFO

###### results
![](attachments/Pasted%20image%2020230825114201.png)

![](attachments/Pasted%20image%2020230825114235.png)
![](attachments/Pasted%20image%2020230825114313.png)

---
## Questions
- This work focus on the memory sharing pattern, how about PUs?

---
## Reflection
- This work mainly focus on the multi model-single device pattern, and improves memory allocation to make sure models can run and and interleave efficiently
- still, this work lies on different level of our plan, what we need to know is:
	- What's the exact process in pyTorch, onnxruntime and tensorRT?
	- How to instruct resource allocation within a task(H-->d-->h)?
	- If the instruction is related to cuDNN, what should we do since it's not open-sourced?

