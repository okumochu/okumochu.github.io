---
layout: page
title: "Ch 17 Geometric Graph Learning"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 17
---

Geometric Graph Learning extends standard graph neural networks by embedding nodes in a physical space, such as 3D Euclidean coordinates. A classic example is a molecule: standard graphs capture atom types and bonds, but geometric graphs also capture the exact 3D positions of those atoms

**Why does this matter?** You might wonder why we can't just use standard GNNs for 3D data. You technically *could* use a standard GNN by artificially rotating your training data millions of times (data augmentation) to force the model to learn 3D symmetries. However, Geometric GNNs bake the laws of physics directly into the math. This drastically shrinks the training time and guarantees physical accuracy.

> **Note**
> 有點像是 GNN physical inform version

**The Core Challenge: Physical Symmetries** When you rotate a molecule in real life, its physical properties don't fundamentally change. Geometric GNNs must respect these 3D special Euclidean symmetries (SE(3),rotations and translations). They handle this in two ways:

- **Invariant GNNs:** The model's output remains completely unchanged when the input is rotated. For example, a molecule's total energy is invariant. Models like SchNet use continuous-filter convolutions that look purely at the distances between atoms to maintain this invariance.
- **Equivariant GNNs:** The model's output transforms in the exact same way the input does. For example, if you rotate a molecule, the physical force vectors acting on its atoms should rotate identically. Models like PaiNN achieve this by passing both scalar features and vector directions through every layer.

**Geometric Generative Models** Beyond predicting properties, we can generate entirely new 3D structures (conformations) from a flat molecular graph. A model called GeoDiff does this using an equivariant diffusion process,it learns to gradually remove noise from random 3D coordinates until a stable, physically accurate molecule is formed.

Furthermore, invariant models that only use distances and angles (like SchNet and DimeNet) are computationally cheaper but can fail to distinguish between certain chemical isomers (like cis/trans isomers) that have identical distances but different 3D orientations. Equivariant models solve this blind spot.

<img src="{{ '/assets/notes/mlg-ch17-geometric-graph-learning/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
