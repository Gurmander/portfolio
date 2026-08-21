---
title: "Building Route Optimization with Google OR-Tools and OSRM"
date: 2026-07-08
description: "How I combined route optimization and real road-network data for a field operations application."
categories:
  - Software Engineering
tags:
  - Python
  - OR-Tools
  - OSRM
  - Optimization
  - Geospatial
draft: false
---

While working on **Syngenta Field Force Intelligence**, one of the problems we needed to solve was route planning for agricultural field representatives.

<a href="https://www.youtube.com/watch?v=cGh5quoFujw&t=14s&pp=ygUSc3luZ2VudGEgaGFja2F0aG9u" target="_blank" rel="noopener noreferrer">Syngenta Demo Video</a>

A representative may need to visit several retailers during the day. Knowing which locations to visit is only part of the problem — the application also needs to determine a sensible order in which to visit them.

This led me to work with **Google OR-Tools** and **OSRM**.

## The routing problem

Suppose a field representative needs to visit several locations:

```text
Start

  ├── Retailer A
  ├── Retailer B
  ├── Retailer C
  ├── Retailer D
  └── Retailer E
```

Simply visiting them in the order they appear can result in unnecessary travel.

The goal is to find a better sequence:

```text
Start
  ↓
Retailer C
  ↓
Retailer A
  ↓
Retailer E
  ↓
Retailer B
  ↓
Retailer D
```

This is closely related to the **Travelling Salesman Problem (TSP)** and vehicle-routing problems.

## Why OR-Tools?

I used **Google OR-Tools** for the optimization part.

The optimizer receives a cost matrix representing the travel cost between every pair of locations.

Conceptually:

```text
             A     B     C     D

A            0    12     8    20
B           12     0     7    11
C            8     7     0    15
D           20    11    15     0
```

OR-Tools then searches for a route that minimizes the overall travel cost.

The interesting part is that creating a good cost matrix is just as important as running the optimizer.

## Why straight-line distance was not enough

Latitude and longitude coordinates can be used to calculate straight-line distance, but this does not accurately represent real driving.

Two locations may look close geographically while being separated by:

* highways
* rivers
* restricted roads
* one-way streets
* different road layouts

For this reason, I integrated **OSRM — Open Source Routing Machine**.

OSRM provides road-network-based distance and duration information.

The overall pipeline became:

```text
Location Coordinates
        │
        ▼
      OSRM
        │
        ▼
Road Distance Matrix
        │
        ▼
 Google OR-Tools
        │
        ▼
Optimized Visit Order
        │
        ▼
Route Geometry
        │
        ▼
       Map
```

## Geospatial processing

A significant part of the work involved handling coordinates correctly.

Each field location needed valid latitude and longitude values before it could participate in route generation.

The backend therefore had to prepare the locations, obtain the road matrix, pass the matrix to OR-Tools, and finally map the optimized sequence back to the original locations.

This made the routing system more than simply calling an optimization library.

It involved coordinating several parts of the application.

## Route analysis

The optimized route was also connected with the AI features of the application.

Once a route was generated, additional context could be used to explain or analyze the planned visits.

This allowed the route to become more than a sequence of coordinates — it could also be presented to the field representative in a more useful and understandable form.

## What I learned

The biggest lesson from this work was that optimization algorithms depend heavily on the quality of the data supplied to them.

OR-Tools can solve the optimization problem, but it still needs realistic travel costs.

Combining **OSRM for road-network information** with **OR-Tools for optimization** created a much more practical solution than using straight-line distance alone.

I also gained experience working with geospatial coordinates, external routing services, optimization models, and integrating the result into a full-stack application.
