---
author: jsx
create_time: 2023-09-06_10:02
category: paper_review
---
## Welder_ Scheduling Deep Learning Memory Access via Tile-graph 
- conference: OSDI'23
- time:
- noteworthy author:
---
## Contribution
### 0.Background

### 1.Problem/Observation
###### Modern DNNs are memory bounded
- Modern models "tend to process **high-fidelity data** and generate **large activations** across layers."
![](attachments/Pasted%20image%2020230907104715.png)
- highlight the need for optimizing memory efficiency

###### Conflicted intra- and inter-operator data reuse patterns
- An operator is often implemented as nested multi-level loops over all tensor dimensions
- Within the operator, the data reuse **across multiple memory layers** are often **implicitly optimized** using **loop tiling** 
- e.g. Matmul + SoftMax
	- (32x64)output tile for Matmul + (4x128) for SoftMax -->high performance each, poor reuse
	- (16x128)for each --> worse for each but better in total --> better performance
- an **aligned tile configuration** across operators can be deduced based on a **chain of shape inference** starting from an output tile shape
	- the memory traffic then can be computed
### 2.Solution
key insight: compromise a little in intra-layer reuse to achieve better inter-layer and overall reuse
###### overview
![](attachments/Pasted%20image%2020230907194816.png)
- input: full DNN model
- DNN will be converted into **operator-tiles**
	- tile-based computing tasks
- resolve data reuse through "first connect then schedule"
	1. assume 2 adjacent operators can reuse data in a certain memory layer
	2. modify tile shape to see if data traffic can be reduced
- 2-step scheduling algorithm(connecting + scheduling)

###### operator-tile and tile-graph
- operator can be implemented as multiple homogeneous operator-tiles
- Each operator-tile takes a data tile slice and computes a data tile
- two adjacent operator-tiles can be “connected” through a common **intermediate data tile**, also known as a **reuse-tile**
![](attachments/Pasted%20image%2020230907195915.png)

- Tile propagation
	-  Once connected, most tiles are correlated, thus we can get their configuration by propagating the output tile
	- for sparse and noncontinuous operations, a upper bound of input tile can be computed
	- after propagation, the memory traffic and footprint of a tile-graph can be determined.

###### tile-graph scheduling
![](attachments/Pasted%20image%2020230908100908.png)
- **Overview**
	- recursively partition each operator into operator-tiles to **fit in each layer**
	- connect operator-tiles at higher memory layers for inter-operator data reuse
	- DNN computation can then be modeled as a **data streaming** pipeline over a **2-d space**

- **Decoupling optimization space**
	- observation: DNN computation is mostly memory-bounded --> major goal: minimizing the memory traffic
	- Independence of optimization across layers
		- the **total data traffic from and to a lower memory layer** for a given tile-graph can be estimated by just its output tile shape
		- different tile-graphs can independently optimize their memory traffic by searching for the optimal tile shapes
		- tile size at the ==lower memory level(L2)== must be larger than the tile size at the ==upper memory level(L0)==
- **Scheduling Interface**
	- *SetConnect* is used to add a connection between two nodes and *Propagate* is used to set tile configuration for a node
	- other interfaces for computing cost
- **Scheduling Algorithm**
![](attachments/Pasted%20image%2020230908104552.png)
- line 1-11: graph-connecting scheduler
	- (2-3) go through every one of nodes and every one of their edges
	- (4) for everyone of the level can be assigned to this edge, (5-6)we extract its subgraph and try to generate an exec plan and its latency
		- a subgraph is one node and its output nodes with a connection larger than *level*
	- we use the level that can generate the fastest plan as the final decision
- line 12-25 sub-graph scheduler
	- (14-19)enumerate the tile size, (15-17)and store all the tiling configs not exceeding the capacity, sort them by traffic
	- (20-24) recursively get the configurations at higher level
- result: a hierarchical tile-graph, a full graph at the lowest memory level and recursively split into sub-graphs in the upper layers

###### implementation
- based on open-source DNN compilers
	- It leverages TVM for writing kernel schedule
	- Roller for enumerating efficient tile configurations 
	- Rammer for the end-to-end graph optimization(and to cut operator into tile-operator)
- exec flow
	- input: onnx graph
	- common graph optimization(constant folding, element-wise fusion)
	- converts to tile-graph

### 3.Experiment

---
## Questions

---
## Reflection

