---
author: jsx
create_time: 2023-08-29_17:12
category: paper_review
---
## Rammer_Enabling Holistic Deep Learning Compiler Optimizations with rTasks 
- conference: OSDI
- time:2020
- noteworthy author:
---
## Contribution
### 1.Problem/Observation
###### computation pattern of DL tasks
- **data flow graph** (DFG)
	- node --> operator(e.g. MM)
	- edge --> dependencies
- two-level parallelism
	- inter-operator/ DFG-level
	- intra-operator/ operator-level

current method: **2-layer scheduling approach**
- DFG-level scheduler takes the DFG and plan the execution of operators, i.e. does kernel fusion
- intra-operator scheduler focus on arranging computations within an operator
###### limitations of current methods
- intra-op scheduling --> low GPU utilization
	- ![](attachments/Pasted%20image%2020230829173016.png)
- inter-op scheduling --> overheads
	- ![](attachments/Pasted%20image%2020230829173059.png)
- inter-op + intra-op --> sub-optimal
- low computation variance --> time can be estimated in compiling --> possible to merge inter-op and intra-op

### 2.Solution
###### programming model
![](attachments/Pasted%20image%2020230829173332.png)
- rOperator
	- correspond to operator
	- a group of rTasks
	- ==rely on external tools to partition==
- rKernel
	- different implementation of a rOperator
	- e.g. different tiling strategy in MM
	- different time and resource consumption
- rProgram
	- DFG subgraph --compiler--> rProgram
	- acts like an operator in GPU
	- is an execution plan of rTasks
	- will be mapped on vEU
- vEU
	- virtual Devices
	- using **PTB method** to 'pin' one special SM
	- one vEU is dispatched to a SM, while SM can run multiple vEUs concurrently

###### DFG compiler
- input: DFG/DFG subgraph
- output rProgram

- scheduling policy: ==comes from Wave==
- kind of like a dynamic programming
- one step
	- the compiler takes in all the operators in a *wave*
		- a *wave* is the "fringe nodes of a breadth-first-search on the DFG".
	- using heuristics to minimize the time 
	- Sync methods to ensure dependency

**maybe we can simplify this contribution of Rammer as:**
 - **use a unified pattern for operators**
 - **Thus can merge operators in a more arbitrary way**

###### workflow
![](attachments/Pasted%20image%2020230829175715.png)

### 3.Experiment
###### latency
![](attachments/Pasted%20image%2020230829204125.png)
- E2E time/latency when batch size = 1
- Rammer highly outperforms among the methods, reach average ~2x speedup compared with tensorRT
- As claimed, due to the simplicity of AlexNet, the performances are similar
![](attachments/Pasted%20image%2020230829204449.png)
- E2E time when batch size differs
- Rammer can't beat LSTM since it "used close-sourced kernel"
![](attachments/Pasted%20image%2020230829204643.png)
- E2E time when input size = 224x224
- it's quite interesting that Rammer's computing time doesn't changes linearly
- "model structure for larger dataset usually have more inter-operator parallelism that can be better leveraged by RAMMER’s optimization", the author claims

---
## Questions
- The task partitioning and deployment of Rammer is like: Operators --> tasks --> vEU. An Interesting point is, one task is assigned to an vEU, and its size is limited to one SM. Will this feature limit the overall performance?
- The vEU remain the same size of threads used by the largest rtask running on it, will this become an overhead?
---
## Reflection
- interesting fact: this paper is only cited once, but corresponding repo, [NNFusion](https://github.com/microsoft/nnfusion), get 800+ stars
- Currently I believe the great performance gain achieved is by the novel operator partitioning method, which gives more freedom to fully utilize the GPU resource
- from the view of optimization, by modifying the constrains, they get a obviously better search space
- 
