---
author: jsx
create_time: 2023-08-24_17:15
category: paper_review
---
## Pagoda_Fine-Grained GPU Resource P Virtualization for Narrow Tasks 
- conference: PPoPP
- time: 2017
- noteworthy author:
---
## Contribution
### 1.problem/observation
GPUs are suffering from underutilization when executing "narrow tasks", i.e. tasks with less than 500 GPU threads

### 2.backgrounds
###### the structure of GPU(Titan X)
GPU cores are organized into ==Streaming Multi processors (SMMs)==
- A SMM -- **128 CUDA cores**
- A SMM can schedule **64 Warps**
- 96KB shared memory/on-chip programmer managed cache
- 64K, 32-bit registers
==Warp== is the basic Single Instruction, Multiple Thread(SIMT) work unit

###### CUDA programming model
programmer organize parallel work in ==kernels==

Threads of a kernel are grouped into ==threadblocks==, the size is limited to **1024 threads, or 32 warps**

Multi threadblocks can reside on one SMM

CUDA scheduling is in the level of threadblock
###### Occupancy: measuring GPU utilization
ratio of the total number of resident GPU warps divided by the maximum number of warps that can co-exist in the GPU

typically a task takes one kernel in CUDA, so even works like hyper-Q enables 32-kernels execution, the occupancy is still low.
###### Related works
1. static fusion solution
	- static fuse multi smaller tasks to generate a large kernel
	- need to fuse manually
	- require static knowledge of kernels
	- in a fused task, the resources are distributed evenly
2. runtime solution
	- virtualize the resources
	- naturally overcome the hardware-imposed kernel limit.
	- low execution latencies

### 2.solution
- main challenges:
	- CPU-GPU communication overhead
	- overheads involved in task spawning and scheduling
	- support native CUDA functionality
![](attachments/Pasted%20image%2020230824192055.png)
- Resource Virtualization: MasterKernel
	- acquires all GPU resources, namely warps, shared memory, and registers
	- launching two, 32-warp threadblocks, called **MTBs** (Master Kernel Threadblocks)
		- The first warp of each MTB: scheduler warp
		- the rest: executor warps
- task spawning: TaskTable, mirrored on the CPU and GPU
	- 1 *cudamemcopy* per task table entry --> low communication overhead
	- use **Lazy Aggregate TaskTable Updates**: CPU/GPU works on different parts --> no need to update until no entry to write
- scheduling: multu-row TaskTable + MTB WarpTable
	- spawning, scheduling and execution are overlapped
- support CUDA function
	- shared memory management
		- ==CUDA lacks support for software-driven dynamic shared memory allocation once a kernel has been launched==
		- use a buddy system 
	- Sub-Thread Block Synchronization
### 3.experiment
###### setup:
- NVIDIA Maxwell Titan X GPU
- Intel core-i7 4.0GHz quad-core CPU
- nvcc from CUDA 7.5, with -O3

###### workloads
![](attachments/Pasted%20image%2020230824190031.png)

###### Results
![](attachments/Pasted%20image%2020230824190131.png)



---
## Questions
- The CUDA core seems can't be allocated explicitly(or say, with shape), will the shape affect?
- The shared memory seems only allows implicit control, a similar manage system seems to be necessary?
- The CUDA and GPU versions are too old for now, do things change recently?
- What is a task? Does a task stand for one htod-dtoh?
- What the relationship between task and a machine learning application?
- And what's the mechanic of GPU threads, are they generated automatically?
---
## Reflection
- The GPU utilization does suffers from so-called "narrow tasks"
- The CUDA core resource and shared memory can be managed in some extent, while require some heavy efforts
- This work may not directly guide our plan, for it's in the level between tasks, however, in our plan, we plan to optimize the behavior in a task
- Though, it seems that we have some space to further optimize:
	- The vEUs are not explicitly controlled, it's size is defined by max rtask
	- TRT shows that through highly-optimized kernels, we can still reach high performance, how can we get a multi-layer, fine-grained and automatic strategy?
