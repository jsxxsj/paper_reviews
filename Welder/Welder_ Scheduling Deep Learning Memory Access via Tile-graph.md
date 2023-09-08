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
### 3.Experiment

---
## Questions

---
## Reflection

