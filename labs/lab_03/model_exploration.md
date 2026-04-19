# Model Exploration

## Initial Problem Idea

At the beginning of the project, we considered different ways to frame the prediction task using Formula 1 race data. The main idea was to predict race outcomes in a way that could be useful for strategic decisions.

We evaluated options such as:

* Predicting whether a driver finishes in the Top 10 (classification)
* Predicting finishing position
* Predicting championship points (regression)

We decided to start exploring the regression approach, as it preserves more information about race outcomes.

---

## Early Feature Selection

We focused on variables that are available before or at the start of the race:

* `grid` (starting position)
* `driver` (driver identity)
* `constructor` (team identity)

We expected **grid position** to be a strong predictor, since starting position often influences final results in Formula 1.

Driver and constructor were included to capture differences in skill and car performance.

---

## Initial Model Plan

We planned to compare a mix of simple and more complex models:

* Baseline 1: Predict the average number of points (global mean)
* Baseline 2: Use average points by grid position (domain heuristic)
* Linear Regression
* Random Forest

We expected Random Forest to perform best because it can capture nonlinear relationships between variables.

---

## Assumptions and Expectations

Before running the models, we had the following expectations:

* Grid position would be the most important feature
* More complex models would outperform simpler ones
* Including driver and constructor would improve predictions beyond simple baselines

---

## Anticipated Challenges

We identified some potential difficulties early on:

* Many drivers score zero points in a race, which could make prediction harder
* Race outcomes depend on unpredictable factors (incidents, weather, strategy)
* The dataset might not include enough detailed information to fully explain results

---

## Summary

This initial exploration guided the choice of features and models used in the final analysis. While the expectations were that more complex models would perform better, the final results showed that the problem is more constrained by the available data than by model complexity.
