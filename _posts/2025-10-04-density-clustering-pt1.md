---
title: Caleb C. Reagor, Ph.D.
layout: default
date: 2025-10-04 00:00:00 -0000
categories: geospatial-analysis density-clustering
---

# Ridesharing Is Caring — Geospatial Density Clustering (Part 1)

October 4th, 2025

![NYC retro taxis](\images\posts\2025-10-04-density-clustering-pt1\Dodge_Polara_and_other_Yellow_Cabs_in_1973_NYC.jpg)

This is Part 1 of two blog posts that I'm writing to highlight my recent projects using density-based clustering to explore geospatial datasets and urban dynamics. In this post, I define and analyze taxi ridesharing efficiency using a public dataset of NYC yellow cab rides, and in the second post I adapt this approach to analyze urban density patterns across towns and cities in the US and abroad.

---

## What is density-based clustering?

Density-based clustering is a nonparametric method that can detect continuous regions with similar densities of observations across a dataset. Many popular implementations such as DBSCAN (<ins>D</ins>ensity-<ins>B</ins>ased <ins>S</ins>patial <ins>C</ins>lustering of <ins>A</ins>pplications with <ins>N</ins>oise) also allow for outliers to remain unclustered as noise. These algorithms generally rely on two key parameters: the neighborhood radius $\epsilon$, and the density required to define a "core" observation. In the example below, the darker shaded regions contain core observations while the lighter regions define the border between clusters and noise.

| ![DBSCAN example](\images\posts\2025-10-04-density-clustering-pt1\DBSCAN-density-data.svg) |
|:--:|
| “DBSCAN density data” — Chire, Wikimedia Commons, CC BY-SA 3.0 |

(Next, talk about why density-based clustering is good for urban dynamics)