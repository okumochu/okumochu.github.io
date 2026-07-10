---
layout: page
title: "Ch 20 Scaling Up GNNs"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 20
---

現在的 graph data 都是千萬個 node 起跳，把所有 node 及其相鄰矩陣塞進來一次更新所有 node 基本不可能。所以需要一些 trick 來更新

## **Neighbor Sampling**

Instead of aggregating information from every single neighbor in a $$*K$$-*hop radius,which grows exponentially and crashes memory when hitting highly-connected "hub" nodes,this method samples a fixed maximum number of neighbors ($$*H_k*$$) at each layer

<img src="{{ '/assets/notes/mlg-ch20-scaling-up-gnns/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

## **Cluster-GCN**

This method partitions the large graph into smaller, densely connected communities (subgraphs) using algorithms like Louvain or METIS. During training, a mini-batch processes an entire subgraph at once, allowing the model to efficiently re-use layer embeddings

## **Simplified GCN**

This approach completely removes the non-linear activation functions (like ReLU) between GNN layers.  By removing non-linearity, the multi-layer neighborhood aggregation collapses into a single linear mathematical operation.
