---
title: Caleb C. Reagor, Ph.D.
layout: default
---


# Research Skills
## High-dimensional & multi-modal data analysis
- In my Ph.D., I analyzed high-dimensional datasets of single cells' unique profiles from sparse `cell x gene` matrices with thousands of individual features and up to millions of unique observations:

|          | Gene_1 | Gene_2 | ⋯   | Gene_m |
|----------|--------|--------|-----|--------|
| Cell_1   |   ⋱    |   ⋯    | ⋯   |   ⋰    |
| Cell_2   |   ⋯    |   ⋰    | ⋱   |   ⋯    |
| ⋮        |   ⋮    |   ⋮    | ⋮   |   ⋮    |
| Cell_n   |   ⋰    |   ⋯    | ⋰   |   ⋱    |

<br>

- These datasets often comprise multiple `cell x feature` matrices of cells' distinct biological properties

- To analyze these datasets, I used unsupervised machine-learning methods such as *t*-SNE and Louvain clustering:

<img src="images/tsne-neuromast-annotated.svg" alt="t-SNE" width="400">

> Reagor & Hudspeth, 2024, [*bioRxiv*](https://doi.org/10.1101/2024.10.15.618534)

## Deep learning for causal time-series analysis

- I developed DELAY, a deep-learning method for reconstructing causal networks from single-cell datasets
- These datasets can reveal cells' relative maturity during processes like tissue regeneration:

<img src="images/slingshot-pseudotime.png" alt="Pseudotime" width="600">

- Inspired by Granger Causality, I designed DELAY to encode noisy gene-expression data as images for computer vision

![DELAY](images/DELAY.png)

- DELAY uses a supervised convolutional neural network to classify images as either interacting or non-interacting gene pairs

<img src="images/DELAY-fig1b.jpeg" alt="DELAY CNN" width="800">

> Reagor, Velez-Angel & Hudspeth, 2023, [*PNAS Nexus*](https://doi.org/10.1093/pnasnexus/pgad113)
