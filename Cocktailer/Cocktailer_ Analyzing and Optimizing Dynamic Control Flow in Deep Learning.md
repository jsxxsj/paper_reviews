---
author: jsx
create_time: 2023-09-06_10:00
category: paper_review
---
## Cocktailer_ Analyzing and Optimizing Dynamic Control Flow in Deep Learning 
- conference: OSDI
- time: 2023
- noteworthy author:
---
## Contribution
### 0.Background
###### control flow and DNN architecture
The introducing of control flow to DNN can enable more flexibilities
- Dynamic computing: adapt structure at runtime
	- e.g. RNN read variable-length sequences
- Conditional computation: 
	- e.g. CV model executing different parts depending on different resolution
- Efficient computation:
	- e.g. early-exiting mechanism based on intermediate results

###### models with dynamic computation
These models take a large proportion in DNNs
- loops for temporal-wise dynamism
	- LSTM, NASRNN, Seq2seq
- models with branches
	- BlackDrop, SkipNet
- tree-based architecture via recursion
	- RAE, Tree-LSTM
![](attachments/Pasted%20image%2020230906103702.png)

###### mainstream for dynamic computations
1. TensorFlow v1.x:
	- introducing a set of control flow operators
	- operated in framework runtime (in CPU)
2. Pytorch and JAX:
	- using programming language to represent and execute
	- control flow statements are running in the runtime (CPU)

### 1.Problem/Observation
###### different implementations between data and control flow
**phenomenon**: 
- data flow operators --> accelerators
- control flow --> maintained in CPU side
**Reason**: ***parallelism mismatch***
- data flow operators --> internal data parallelism
- control flow --> single-thread computation
- accelerators --> hierarchical parallel PUs
**Results**
- accelerators are hard to schedule control flow
- performance issue
![](attachments/Pasted%20image%2020230906104955.png)
###### reasons of performance issue
- Boundary overheads
	- incur synchronizations between the CPU and the accelerator
	- e.g. Branch 
		- Branch = body(GPU)+ operation(CPU)
		1. CPU(stall)/ GPU(provide data for deciding target)
		2. CPU(check condition)/GPU(stall)
- Boundary limits optimization scope
	- different side of 2 flows divides DNN into sub-programs --> suboptimal
	- e.g. LSTM:
		- DNN operators in LSTM cells in different layers should be scheduled
- Boundary prevents parallelism in DNN programs
	- operators in different control flow statements have to be executed sequentially due to the synchronization between h&d
	- e.g. RAE
		- operators without dependencies can be executed concurrently, but due to the control flow...
###### observations
**Most Hardware accelerators do have the ability to execute control flow instructions**

### 2.Solution
###### overview
![](attachments/Pasted%20image%2020230906111718.png)
- input: DNN program(control + data)
	- operator --> **uOperator**(independent uTasks)
	- **uTask** can be scheduled to one compute unit
		- definition: **compute logic** able to be scheduled to **one of** the **multi-level PUs**
	- COCKTAILER schedules control flow and data flow **in a single space** and generate uProgram
	- **uProgram** contains multiple independent utasks and can be scheduled in parallel
###### uTask and uOperator
1. Data flow
	- uTask: takes a slice of data, execute the corresponding computation
	- uOperator: a collection of all uTasks
	- *the partition of operator: Rammer*
2. Control Flow
	- problem: cannot be divided into parallel tasks
	- observation: controlling DNN operators is equal to apply control flow computation on each uTask
	- e.g. loop with a body of MM
		- before: executing the loop over the operator
		- after: let each unit of the hardware accelerator process the loop control flow over the uTask of the operator
	- depends on **Nested uTasks** to solve data dependencies
		- input data in the body_utasks is related to control flow
	- uTask is actually a interface
![](attachments/Pasted%20image%2020230906115252.png)
###### control flow uTasks
![](attachments/Pasted%20image%2020230906115458.png)
- Loop uTask:
	- The body_utask is executed multiple times in a loop with different input data
	- control_flow is another interface
- Branch uTask
	- ![](attachments/Pasted%20image%2020230906115643.png)
- Function:
	- natively represented by Nest_uTask
- Recursion
	- recursion: a uTask can call itself in the body
	- introduce uTask reference
		- like a function call to a utask

###### uProgram scheduling
- uProgram
	- the generated execution plan of the whole program


### 3.Experiment

---
## Questions

---
## Reflection

