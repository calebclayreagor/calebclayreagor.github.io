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

.nowrap { white-space: pre !important; overflow-x: auto; }

# Ridesharing Is Caring — Geospatial Density-Based Clustering (Part 1)

October 6th, 2025

![NYC retro taxis](\images\posts\2025-10-04-density-clustering-pt1\Dodge_Polara_and_other_Yellow_Cabs_in_1973_NYC.jpg)

This is Part 1 of two blog posts that I'm writing to highlight my recent projects using density-based clustering to explore geospatial datasets and urban dynamics. In this post, I define and analyze taxi ridesharing efficiency using a public dataset of NYC yellow cab rides, and in Part 2 I adapt this approach to analyze urban density patterns across towns and cities in the US and abroad.

---

## What is density-based clustering?

Density-based clustering is a nonparametric method that can detect continuous regions with similar densities of observations across a dataset. Many popular implementations such as [DBSCAN](https://en.wikipedia.org/wiki/DBSCAN) (<ins>D</ins>ensity-<ins>B</ins>ased <ins>S</ins>patial <ins>C</ins>lustering of <ins>A</ins>pplications with <ins>N</ins>oise) allow outliers to remain unclustered. These algorithms generally rely on two key parameters: the neighborhood radius $\epsilon$, and the minimum number of neighbors required for "core" observations. In the example below, darker regions show the core observations while lighter regions depict the border between the clusters and noise.

<div align="center" markdown="1">

![DBSCAN example](\images\posts\2025-10-04-density-clustering-pt1\DBSCAN-density-data.svg)

Wikimedia Commons, CC BY-SA 3.0

</div>

These methods are well suited for exploratory analysis of transportation and geodemographic datasets because humans are heterogeneously organized within and between populations. For example, natural features such as rivers and mountains can separate cities and their populations, whereas transit hubs like train stations (see below) can promote local clustering of passengers. Neither situation is well-approximated by a parametric prior like a Gaussian distribution, and we typically don't know the number of clusters *a priori*. However, density-based clustering relies on the contrasting local densities to identify clusters regardless of their shapes or sizes.

<div align="center" markdown="1">

![DBSCAN train station](\images\posts\2025-10-04-density-clustering-pt1\epsilon_parameter_hdbscan_eps.webp)

HDBSCAN Read the Docs, BSD 3-Clause

</div>

---

## Density-based clustering of NYC taxi trips

I recently completed a [project](https://github.com/calebclayreagor/nyc-taxi-efficiency) exploring the ridesharing efficiency of NYC taxi trips using an iterative density-based clustering algorithm. The rest of this post and Part 2 will outline the three key ingredients to a successful exploratory analysis of geospatial datasets and highlight my most interesting (and surprising!) findings.

### Downloading and cleaning the data

Thanks to a [FOIA request]((http://www.andresmh.com/nyctaxitrips/)) by Chris Wong, the NYC taxi and limousine commission released trip and fare data from January through December 2013 containing medallion numbers, pickup and dropoff datetimes/locations, passenger counts, and payment breakdowns. For my analysis, I focused on the data from the first full week of June, Monday (6/3) to Sunday (6/9). After merging trip and fare data and selecting the entries for these dates, I used the following filters to keep high-quality rides:

```python {.nowrap}
trip = (trip
        .loc[(trip.passenger_count > 0) & (trip.passenger_count < 10)]                   # 0 < passengers < 10
        .loc[trip.trip_time > dt]                                                        # time > 1 min
        .loc[trip.time_delta > dt]                                                       # end minus start time > 1 min
        .loc[(trip.trip_time - trip.time_delta).abs() < dt]                              # time equals time delta
        .loc[(trip.trip_distance > dlim[0]) & (trip.trip_distance < dlim[1])]            # .1 mi < actual distance < 30 mi
        .loc[(trip.euclidean_distance > dlim[0]) & (trip.euclidean_distance < dlim[1])]  # .1 mi < linear distance < 30 mi
        .loc[trip.trip_distance < 2 * trip.euclidean_distance]                           # distance < 2x linear distance
        .loc[(trip.avg_speed > 2) & (trip.avg_speed < 50)]                               # 2 mph < average speed < 50 mph
        .loc[trip.fare_amount < 200]                                                     # fare < $200
        .loc[trip.tip_fare_ratio < 5]                                                    # tip-to-fare ratio < 5
        .loc[(trip.sum_charges - trip.total_amount).abs() < 1e-2]                        # price equals charges
        .loc[~trip.index_pickup.isna() | ~trip.index_dropoff.isna()])                    # pickup or dropoff in NYC
```

After final deduplication, my [data cleaning pipeline](https://github.com/calebclayreagor/nyc-taxi-efficiency/blob/main/notebooks/00_cleaning.ipynb) selected just over 1.5 million trips for downstream clustering and efficiency analysis.
