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
	- 

### 2.Solution

### 3.Experiment

---
## Questions

---
## Reflection

