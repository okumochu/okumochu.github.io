---
layout: page
title: "Ch 21 Trustworthy Graph AI"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 21
---

Chapter 21 focuses on Trustworthy Graph AI, which tackles the "black box" nature of deep learning and Graph Neural Networks (GNNs) by emphasizing Explainable AI (XAI). The primary goal is to make GNN predictions transparent so humans can trust them, understand causal relationships, safely transfer models to unseen data, and ensure ethical decision-making

**The Mechanics of Graph Explainability** In traditional machine learning, explainability might mean looking at the slope weights of a linear regression model or the specific logic splits in a decision tree. Because graphs are highly complex, an explanation in graph learning involves identifying a critical *subgraph structure* (a specific set of connected nodes and edges) and a small subset of *node features* that heavily influenced the model's output. Explanations generally fall into two categories

1. **Instance-level:** Explaining a single, specific prediction (e.g., determining exactly why the GNN denied a loan to Person X based on their local network).
2. **Model-level:** Explaining general characteristics across an entire class (e.g., identifying the common structural motif that makes the model classify certain molecules as toxic).

## **GNNExplainer**

GNNExplainer is highlighted as a primary method for extracting insights. It is "post-hoc" (applied after the GNN is already trained) and "model-agnostic" (it works with various architectures like GraphSAGE, GAT, etc.)

- **The Objective:** It aims to extract an explanation (a subgraph and feature mask) that has the highest possible "Mutual Information" with the GNN's actual prediction
- **The Optimization Trick:** Finding the exact perfect subgraph is computationally expensive because there are exponentially many possible edge combinations to test. GNNExplainer cleverly bypasses this discrete math problem by treating the explanation as a continuous distribution. It applies a continuous "mask" (squashed between 0 and 1 using a sigmoid function) to the graph's adjacency matrix and features, allowing the model to use standard gradient descent to learn which edges and features are most important

## **Evaluating Explanations**

Evaluating an explanation is difficult because you rarely have a "ground truth" explanation to compare it against. To measure the quality of an explanation, the chapter introduces two core "Fidelity" concepts (預測準不准)

1. **Sufficiency:**  An explanation is sufficient if you can strip away the rest of the graph, feed *only* the highlighted subgraph into the model, and still get the exact same prediction
2. **Necessity:** An explanation is necessary if *removing* that highlighted subgraph from the original graph causes the model to change its prediction. 

These two metrics are mathematically combined into a "characterization score" to provide a single, robust summary of how good and concise the explanation
