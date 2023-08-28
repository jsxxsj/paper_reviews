---
autthor: jsx
---
# Related information
- transaction: TPDS
- time: 2022

# Contribution
## Problems
- two goals of pipeline scheduling:
	- carefully ==manage all stages' training memory== to avoid exceeding the physical memory capacity on any GPU
	-  enforce a ==“balanced” partition== (G2) such that all stages achieve roughly the same high throughput
- current systems can't manage the 2 goals nicely
- severe throughput degradation when a DNN model enables NAS

## Backgrounds
###### 2 categories in pipeline system
- keeps activation tensors produced during forward passes directly in GPU memory
	- have to keep many more copies of activations
	- have to keep a moderate batch size, even increasing batch size will lead to higher utilization
- discards all activation tensors in the forward passes and recomputes them in the backward passes
	- alleviates the imbalanced GPU memory utilization
	- alleviates the imbalanced GPU memory utilization

###### 2 categories in scheduling methods in pipline system
- barrier synchronous parallel (BSP)
	- enforce a barrier that stops the pipeline to apply the gradients
	- logically still incurs bubbles during each barrier synchronization
- asynchronous parallel (ASP)
	- remove the sync barrier and let each input batch directly update the model parameters
	- no bubbles
	- parameter staleness
		- parameter version differs between a pipeline forward pass and backward pass.
		- parameter version differs among stages within the training of an input batch

## solutions
- VPIPE, the first ==dynamic== DNN layer partitioning and memory management system
- a ==virtualized layer== between a typical pipeline parallel system
- computes a hybrid plan of both swap and recompute for all layers on each stage
###### architecture
![](attachments/Pasted%20image%2020230828091902.png)
- **Virtualized Tensor Manager (VTM)** provides fine-grained management to each parameter and activation tensor
- **Training monitor** monitors each stage’s runtime statistics
- **Global planner** collects the runtime statistics of all stages at the end of every forward pass.
- **Layer manager** receives a new partition strategy from the global planner
---
# Questions
- What's the detailed track of a pipeline training? and then why sync methods will influent the performance?

# Reflections
- This work focus on pipeline training, though, it's based on tasks instead of within tasks
- The modeling part contains a very interesting part of optimization in managing the hybrid plan, maybe we can back to it in some time properly.