---
title: Ridesharing Is Caring
layout: default
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

<style>
.highlight, pre { overflow-x: auto; }
.highlight pre, pre, pre code, code {
  white-space: pre !important;
  word-break: normal !important;
  overflow-wrap: normal !important;
}
</style>

# Ridesharing Is Caring — Geospatial Density-Based Clustering (Part 1)

October 6th, 2025

![NYC retro taxis](\images\posts\2025-10-04-density-clustering-pt1\Dodge_Polara_and_other_Yellow_Cabs_in_1973_NYC.jpg)

This is Part 1 of two blog posts that I'm writing to highlight my recent projects using density-based clustering to explore geospatial datasets and urban dynamics. In this post, I define and analyze taxi ridesharing efficiency using a public dataset of NYC yellow cab rides, and in Part 2 I adapt this approach to analyze urban density patterns across towns and cities in the US and abroad.

---

## What is density-based clustering?

Density-based clustering is a nonparametric method that can detect continuous regions with similar densities of observations across a dataset. Many popular implementations such as [DBSCAN](https://en.wikipedia.org/wiki/DBSCAN) (<ins>D</ins>ensity-<ins>B</ins>ased <ins>S</ins>patial <ins>C</ins>lustering of <ins>A</ins>pplications with <ins>N</ins>oise) allow outliers to remain unclustered. These algorithms generally rely on two key parameters: the neighborhood radius $\epsilon$, and the minimum number of neighbors required for "core" observations. In the example below, darker regions show core observations while lighter regions depict the borders between clusters and noise.

<div align="center" markdown="1">

![DBSCAN example](\images\posts\2025-10-04-density-clustering-pt1\DBSCAN-density-data.svg){: width="50%" }

Wikimedia Commons, CC BY-SA 3.0

</div>

These methods are well suited for exploratory analysis of transportation and geodemographic data because humans are often heterogeneously organized across regions. For example, geographic features such as rivers and mountains can separate cities and populations, whereas transit hubs like train stations (see below) can promote local clustering of passengers. Neither situation is well-approximated by a parametric prior such as a Gaussian, and we typically don't know the number of clusters *a priori*. However, density-based clustering can instead rely on the contrast between local densities to identify clusters, regardless of their shapes or sizes.

<div align="center" markdown="1">

![DBSCAN train station](\images\posts\2025-10-04-density-clustering-pt1\epsilon_parameter_hdbscan_eps.webp){: width="50%" }

HDBSCAN Read the Docs, BSD 3-Clause

</div>

---

## Density-based clustering of NYC taxi trips

I recently completed a [project](https://github.com/calebclayreagor/nyc-taxi-efficiency) exploring the ridesharing efficiency of NYC taxi trips using an iterative density-based clustering algorithm to aggregate trips. The rest of this post will outline the main ingredients to successfully explore geospatial datasets using density-based methods, highlighting any interesting (and surprising!) results along the way.

### Downloading and cleaning the data

Thanks to a [FOIA request](http://www.andresmh.com/nyctaxitrips/) by Chris Wong, the NYC taxi and limousine commission released trip/fare data from January through December 2013, containing medallion numbers, pickup and dropoff datetimes/locations, passenger counts, and payment breakdowns. For my analysis, I focused on data from the first full week of June, Monday (6/3) to Sunday (6/9). After merging trip and fare data and selecting the rides on these dates, I used the following filters to keep high-quality trips only:

```python
trip = (trip
        .loc[(trip.passenger_count > 0) & (trip.passenger_count < 10)]                   # 0 < number of passengers < 10
        .loc[trip.trip_time > dt]                                                        # trip time > 1 min
        .loc[trip.time_delta > dt]                                                       # end minus start time > 1 min
        .loc[(trip.trip_time - trip.time_delta).abs() < dt]                              # trip time equals time delta
        .loc[(trip.trip_distance > dlim[0]) & (trip.trip_distance < dlim[1])]            # .1 mile < actual distance < 30 miles
        .loc[(trip.euclidean_distance > dlim[0]) & (trip.euclidean_distance < dlim[1])]  # .1 mile < linear distance < 30 miles
        .loc[trip.trip_distance < 2 * trip.euclidean_distance]                           # actual distance < 2x linear distance
        .loc[(trip.avg_speed > 2) & (trip.avg_speed < 50)]                               # 2 mph < average speed < 50 mph
        .loc[trip.fare_amount < 200]                                                     # total fare < $200
        .loc[trip.tip_fare_ratio < 5]                                                    # tip-to-fare ratio < 5
        .loc[(trip.sum_charges - trip.total_amount).abs() < 1e-2]                        # total price equals charges
        .loc[~trip.index_pickup.isna() | ~trip.index_dropoff.isna()])                    # pickup or dropoff in NYC
```

After deduplication, my [data cleaning pipeline](https://github.com/calebclayreagor/nyc-taxi-efficiency/blob/main/notebooks/00_cleaning.ipynb) selected just over 1.5 million trips for downstream analysis.

### Iterative density-based clustering

Because density-based clustering allows for outliers, these algorithms tend to leave many observations unclustered. To aggregate as many trips into clusters as possible, I implemented an iterative [HDBSCAN](https://github.com/scikit-learn-contrib/hdbscan) (<ins>H</ins>ierarchical DBSCAN) approach that sequentially clustered any remaining observations from the previous step while relaxing the minimum cluster size, from 6 to 2 riders (more on these values later). My input features consisted of pickup locations $x_0,y_0$ and times $t_0$ and dropoff locations $x_1,y_1$, and the outputs were cluster labels $k$ for each trip/passenger. Here's what my clustering results looked like after each iteration:

```
Iteration 0 (min_cluster_size = 6): % clustered = 10.89
Iteration 1 (min_cluster_size = 5): % clustered = 27.17
Iteration 2 (min_cluster_size = 4): % clustered = 44.57
Iteration 3 (min_cluster_size = 3): % clustered = 62.15
Iteration 4 (min_cluster_size = 2): % clustered = 84.11
```

Another important parameter for [my implementation](https://github.com/calebclayreagor/nyc-taxi-efficiency/blob/main/notebooks/01_clustering.ipynb) was the relative scaling of time *vs.* distance, which controlled the tradeoff between spatial and temporal coherence. To select the best value, I performed a parameter sweep from 10 to 60 minutes/mile for one HDBSCAN iteration only. The results showed that smaller values ($\leq$ 10 min/mile) favored tighter temporal clusters, while larger values ($\geq$ 60 min/mile) favored tighter spatial clusters:

<div align="center" markdown="1">

![Time scaling](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/2485ed79b1b9f15f304f2b1af9f2572a3efdfede/figures/scaling.svg){: width="100%" }

</div>

### Quality of the identified clusters

Based on the sweep, I performed my final clustering with a spatiotemporal scaling of 25 minutes/mile and identified clusters with ~5 passengers and pickup/dropoff locations ~0.2 miles and ~5 minutes apart:

<div align="center" markdown="1">

![Cluster size](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/06c282c566790cca3a67b4b4bb66335da1b4b393/figures/cluster_size.svg)![Location spread](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/9be412b09ddf24848c57f5a783562c9b434a4352/figures/cluster_rms_distance.svg)![Time spread](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/9be412b09ddf24848c57f5a783562c9b434a4352/figures/cluster_std_time.svg)

</div>

---

## Defining and exploring ridesharing efficiency

Identifying coherent clusters of taxi trips is already interesting and raises important questions for transit agencies and companies. For example, if passengers are departing/arriving at similar locations/times, is it possible to transport customers more efficiently by increasing their overall ridesharing? To answer this question, I proposed and implemented a complementary metric to assess ridesharing efficiency across the clustered taxi trips.

### Demand-responsive transport and packing efficiency

One common approach to increasing ridesharing is [demand-responsive transport](https://en.wikipedia.org/wiki/Demand-responsive_transport), where vans or small busses operate on flexible routes according to demand and passengers' pickup/dropoff locations. I used this microtransit model as a [benchmark for comparison](https://github.com/calebclayreagor/nyc-taxi-efficiency/blob/main/notebooks/02_efficiency.ipynb) with ridesharing in my taxi-trip clusters. I defined the efficiency $E$ of a rider/vehicle configuration for a given cluster $k$ as follows:

<div align="center" markdown="1" style="font-size:1.25rem; line-height:1.5;">

$E = \frac{c_v}{c} = \frac{\text{cost per capita microtransit}}{\text{cost per capita actual}}$.

</div>

Although this equation is inverse to some efficiency definitions, I chose it because $E$ is bounded on the interval $(0,1]$, with $E=1$ indicating a taxi rider/vehicle configuration was as efficient as microtransit and $E<1$ indicating the configuration was comparatively inefficient. If we assume that microtransit trips cost a scalar multiple $\alpha$ of the average taxi-trip cost per cluster, $E$ becomes a measure of rider packing:

<div align="center" markdown="1" style="font-size:1.25rem; line-height:1.5;">

$E = \alpha \cdot \frac{M_v}{M} \rightarrow \frac{E}{\alpha} = \frac{M_v}{M}$,

</div>

where $E/\alpha$ is packing efficiency (unitless), $M$ is the total number of taxi trips, and $M_v$ is the total number of van trips, which depends on the number of passengers in cluster $k$ and the total van capacity. Here I assumed a typical microtransit van seating capacity of six passengers, which is also the minimum cluster size that I used for `Iteration 0` of my density-based clustering algorithm. The following table outlines key advantages/disadvantages of using $E/\alpha$ to measure ridesharing efficiency:

<div align="center" markdown="1">

| Advantages | Disadvantages |
|:--:|:--:|
| $E/\alpha$ is scale-free and can meaningfully compare both long and short trips | $E/\alpha$ depends on urban density and may not accurately compare dense and sparse regions |
| $M_v$ is directly tunable to optimize van/bus capacity across different regions/times | $E/\alpha$ assesses aggregation/configuration and is agnostic of trip distance/duration |
| No need to introduce systematic errors due to biased/imprecise direct cost estimates |  |
| Does not require mid-trip pickups if well-calibrated to the minimum cluster size |  |

</div>

---

### Efficiency trends by time and location

Across both Manhattan and the outer boroughs, packing efficiency drops every day between ~6 AM-Noon. During weekdays, the outer boroughs have efficiency peaks before/after the trough (~Midnight-6 AM & ~Noon-6 PM), and in Manhattan both weekday/weekend packing efficiency have broad peaks elsewhere (~Noon-6 AM):

<div align="center" markdown="1">

![Efficiency trends](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/06c282c566790cca3a67b4b4bb66335da1b4b393/figures/efficiency_trends.svg){: width="100%" }

</div>

Focusing on Manhattan, we can see that weekday demand for taxis increases ~6 hours before efficiency (between ~6 AM-Noon):

<div align="center" markdown="1">

![Manhattan efficiency and demand](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/06c282c566790cca3a67b4b4bb66335da1b4b393/figures/efficiency_vs_demand_manhattan.svg){: width="100%" }

</div>

This suggests that passengers aren't sharing rides to their jobs in the morning but are instead sharing rides with their co-workers after leaving the office in the afternoon, constituting a significant opportunity to optimize efficiency (and profits!) during the morning rush hour. The AM efficiency dropoff is distributed unevenly across Manhattan's neighborhoods, with Upper Manhattan and the Lower East Side showing comparatively better packing efficiency and the Upper East Side showing the worst efficiency:

<div align="center" markdown="1">

![Manhattan efficiency map](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/06c282c566790cca3a67b4b4bb66335da1b4b393/figures/efficiency_manhattan_weekday_AM.svg){: width="66%" }

</div>

Returning our focus to the outer boroughs, we can see that demand peaks while efficiency drops during the period between ~6 PM-Midnight:

<div align="center" markdown="1">

![Outer boroughs efficiency and demand](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/06c282c566790cca3a67b4b4bb66335da1b4b393/figures/efficiency_vs_demand_outer_boroughs.svg){: width="100%" }

</div>

This means that the evening rush hour represents the best opportunity to optimize efficiency/profits in these boroughs. Interestingly, the PM efficiency dropoff is also distributed unevenly across Brooklyn/Queens neighborhoods, with the airports (LGA/JFK) showing the worst packing efficiency and Bedford-Stuyvesant, Bushwick, and Long Island City showing the best efficiency:

<div align="center" markdown="1">

![Outer boroughs efficiency map](https://raw.githubusercontent.com/calebclayreagor/nyc-taxi-efficiency/06c282c566790cca3a67b4b4bb66335da1b4b393/figures/efficiency_bk_qns_weekday_PM.svg){: width="80%" }

</div>

---

## Wrapping up

