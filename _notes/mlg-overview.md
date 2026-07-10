---
layout: page
title: "Overview"
description: "Course map plus my conclusions on how to design and train a GNN."
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: -1
---

# Machine Learning with Graphs (Stanford, 2024): lecture notes

## Lecture Note



## Basic Knowledge

[Ch.0 Introduction]({{ '/notes/mlg-ch0-introduction/' | relative_url }})

[Ch.1 Traditional Machine Learning on Graphs]({{ '/notes/mlg-ch1-traditional-ml-on-graphs/' | relative_url }})

[Ch.2 Node Embedding]({{ '/notes/mlg-ch2-node-embedding/' | relative_url }})

[Ch.3 Graph Neural Networks]({{ '/notes/mlg-ch3-graph-neural-networks/' | relative_url }})

[Ch.4 A General Perspective on Graph Neural Networks]({{ '/notes/mlg-ch4-general-perspective-on-gnn/' | relative_url }})

[Ch.5 GNN Augmentation and Training]({{ '/notes/mlg-ch5-gnn-augmentation-and-training/' | relative_url }})

[Ch.6 Theory of Graph Neural Networks]({{ '/notes/mlg-ch6-theory-of-gnns/' | relative_url }})

[Ch.7 Limits of Graph Neural Networks]({{ '/notes/mlg-ch7-limits-of-gnns/' | relative_url }})

[Ch.8 Graph Transformers]({{ '/notes/mlg-ch8-graph-transformers/' | relative_url }})

[Ch.9 Machine Learning with Heterogeneous Graphs]({{ '/notes/mlg-ch9-heterogeneous-graphs/' | relative_url }})

## Advance Applications

[Ch.10, 11, 15 Knowledge Graph]({{ '/notes/mlg-ch10-11-15-knowledge-graph/' | relative_url }})

[Ch.12 GNNs for Recommender Systems]({{ '/notes/mlg-ch12-gnns-for-recommender-systems/' | relative_url }})

Ch.13 Relational Deep Learning

[Ch.14 Advanced Topics in Graph Neural Networks]({{ '/notes/mlg-ch14-advanced-topics/' | relative_url }})

[Ch.16 Deep Generative Models for Graphs]({{ '/notes/mlg-ch16-deep-generative-models/' | relative_url }})

[Ch.17 Geometric Graph Learning]({{ '/notes/mlg-ch17-geometric-graph-learning/' | relative_url }})

Ch.18 Fast Neural Subgraph Matching and Counting

[Ch.19 Label Propagation]({{ '/notes/mlg-ch19-label-propagation/' | relative_url }})

[Ch.20 Scaling Up GNNs]({{ '/notes/mlg-ch20-scaling-up-gnns/' | relative_url }})

[Ch.21 Trustworthy Graph AI]({{ '/notes/mlg-ch21-trustworthy-graph-ai/' | relative_url }})

## Conclusion

### How to design GNN in general

**Design choice**

- Example
    
   <img src="{{ '/assets/notes/mlg-overview/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    

<img src="{{ '/assets/notes/mlg-overview/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/mlg-overview/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

### Train Trick

**Train different level prediction simultaneously**

<img src="{{ '/assets/notes/mlg-overview/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

**Pre-Training and Avoiding Negative Transfer**

1.  **Attribute Masking:** Hide certain node attributes (like an atom's type in a molecule) and force the GNN to predict what was hidden. This forces the network to learn the fundamental physical or chemical rules of the domain
2. **Context Prediction:** You sample a center node's local neighborhood and its broader surrounding graph (the context). The GNN is trained to maximize the similarity between the neighborhood and context vectors. This teaches the model that subgraphs found in similar environments usually have similar semantic meanings
