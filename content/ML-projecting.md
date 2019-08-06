---
title: "Project Work flow for Machine Learning"
date: 2018-10-11T08:23:19+10:00
draft: true
tags:
  - GCP
  - ML
  - TensorFlow
  - Jupyter
  - Datalab
---

# Intro
- Why ML?
- All depends on data. Junk in junk out.
- Ultimate goal?

# Respond to Business Requirements

Here our example Business requirement is to: *Provide accurate fair estimates for users of a taxi service*.  Ultimately this will improve the user experience and lead to higher usages rates and higher revenue.

# Explore Available Data
Data size: ??
Data cleanliness: ??
- Do the values look the way we expect? are their definitions ambiguous?
- What values look suspicious?

BigQuery/DataPrep:
- Explore data
    - Correlations?
- Guess a benchmark

# Establish a Performance Benchmark
- Perhaps run a naive model in BQ.

Lets assume that only time in the taxi is relevant.

$$$
Price = C * (Arrival_time - Departure_time)
$$$

Using only BigQuery (of maybe DataPrep!!):
1. Repeatably split data into training and test (80/20)
2. Use training set to calculate `C` and its std-dev
3. Apply `C` to the training set and produce the RMSE of the naive model
**Make sure cleaning process is used**

# Iterative develop

## Define Model architecture
- First run linear regression (if only to ember that it exists)

## Determine Features

## Package
1st: run line by line in Notebook
2nd: use `TrainSpec`/`EvalSpec`/`LatestExporter` to run in notebook. And change input from pandas to `Dataset` API for efficient loading
3rd: Put this model into a python package and run locally (`gcloud ml-engine local train` ??)
4th: Submit job to CMLE

## Evaluate
Measures of accuracy

# Finally - Productionize

# Summary
