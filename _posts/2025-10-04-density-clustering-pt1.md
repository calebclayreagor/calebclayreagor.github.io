---
title: Caleb C. Reagor, Ph.D.
layout: default
date: 2025-10-04 00:00:00 -0000
categories: geospatial-analysis density-clustering
---

<script>
window.MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']],
    displayMath: [['$$','$$'], ['\\[','\\]']]
  }
};
</script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

# Ridesharing Is Caring — Geospatial Density-Based Clustering (Part 1)

October 6th, 2025

![NYC retro taxis](\images\posts\2025-10-04-density-clustering-pt1\Dodge_Polara_and_other_Yellow_Cabs_in_1973_NYC.jpg)

This is Part 1 of two blog posts that I'm writing to highlight my recent projects using density-based clustering to explore geospatial datasets and urban dynamics. In this post, I define and analyze taxi ridesharing efficiency using a public dataset of NYC yellow cab rides, and in Part 2 I adapt this approach to analyze urban density patterns across towns and cities in the US and abroad.

---

## What is density-based clustering?

Density-based clustering is a nonparametric method that can detect continuous regions with similar densities of observations across a dataset. Many popular implementations such as [DBSCAN](https://en.wikipedia.org/wiki/DBSCAN) (<ins>D</ins>ensity-<ins>B</ins>ased <ins>S</ins>patial <ins>C</ins>lustering of <ins>A</ins>pplications with <ins>N</ins>oise) allow outliers to remain unclustered. These algorithms generally rely on two key parameters: the neighborhood radius $\epsilon$, and the minimum number of neighbors required for "core" observations. In the example below, the darker regions show the core observations while the lighter regions depict the border between the clusters and the noise.

<div align="center" markdown="1">

![DBSCAN example](\images\posts\2025-10-04-density-clustering-pt1\DBSCAN-density-data.svg)

Wikimedia Commons, CC BY-SA 3.0

</div>

These methods are well suited for exploratory analysis of transportation or geodemographic datasets because humans are heterogeneously organized within and between populations. For example, natural features such as rivers and mountains can separate cities and their populations, whereas transit hubs like train stations (see below) can promote local clustering of passengers. Neither situation is well-approximated by a parametric prior like a Gaussian distribution, and we typically don't know the number of clusters *a priori*. However, density-based clustering relies on the contrasting local densities to identify clusters regardless of their shapes or sizes.

<div align="center" markdown="1">

![DBSCAN train station](\images\posts\2025-10-04-density-clustering-pt1\epsilon_parameter_hdbscan_eps.webp)

HDBSCAN Read the Docs, BSD 3-Clause

</div>

---