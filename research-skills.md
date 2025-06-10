---
title: Caleb C. Reagor, Ph.D.
layout: default
---


# Research Skills
## High-dimensional & multi-modal data analysis
- In my Ph.D., I analyzed high-dimensional single-cell datasets using sparse `cell x gene` matrices with thousands of individual features and up to tens of millions of unique cells:

> |          | Gene_1 | Gene_2 | ⋯   | Gene_m |
> |----------|--------|--------|-----|--------|
> | Cell_1   |   ⋯    |   ⋯    | ⋯   |   ⋯    |
> | Cell_2   |   ⋯    |   ⋯    | ⋯   |   ⋯    |
> | ⋯        |   ⋯    |   ⋯    | ⋯   |   ⋯    |
> | Cell_n   |   ⋯    |   ⋯    |  ⋯   |   ⋯    |

- These datasets comprised multiple `cell x feature` matrices that described the multi-modal profile of each cell

- To analyze the datasets, I used unsupervised machine-learning methods such as *t*-SNE and Louvain clustering:

> <img src="images/tsne-neuromast-annotated.svg" alt="t-SNE" width="400">

> Reagor & Hudspeth, 2024, [*bioRxiv*](https://doi.org/10.1101/2024.10.15.618534)

<br>

---

## Deep learning for causal time-series analysis

- I developed the deep-learning method [DELAY](https://github.com/calebclayreagor/DELAY) to reconstruct causal gene-regulatory networks from single-cell gene-expression datasets

- Using concepts from Granger Causality, I designed DELAY to encode noisy gene-expression data as images for deep learning 

> ![DELAY](images/DELAY.png)

> Reagor, Velez-Angel & Hudspeth, 2023, [*PNAS Nexus*](https://doi.org/10.1093/pnasnexus/pgad113)

- DELAY uses a custom convolutional neural network to classify images as either interacting or non-interacting gene pairs

> <img src="images/DELAY-fig1b.jpeg" alt="DELAY CNN" width="800">

<br>

---

## Analysis of large-scale networks

- To reconstruct large-scale genetic networks, I trained DELAY to predict causal interactions between small clusters of genes

- I then used DELAY to identify the nodes (genes) that control temporal expression dynamics in the network

> <img src="images/grn-hubs-bubble-edited.svg" alt="Network hubs" width="600">

> To validate this reconstructed network, I performed experiments on regenerating zebrafish

- Beyond DELAY, I also developed [custom scripts](https://github.com/agnikdasgupta/Sema7A_regulates_neural_circuitry) to quantify and analyze neuronal networks
