---
title: Energy Market Forecaster
description: Predicts the gap between day-ahead and real-time electricity market prices using weather and grid data.
paper: https://www.mdpi.com/1999-4893/16/11/508
draft: false
tags: [Python, Time Series, ML]
---

## Overview

Electricity prices in the day-ahead market often diverge from real-time prices due to weather surprises, unexpected demand, or generation outages. Predicting this gap ahead of time is valuable for grid operators and energy traders.

## Approach

A Random Forest is used first to rank feature importance across weather variables (solar radiation, wind speed, temperature) and lagged market prices. The top features are passed to an LSTM — 100 hidden units, MSE loss, Adam optimizer — which captures the temporal autocorrelation in the price gap series.

The key modelling choice is predicting the gap directly rather than forecasting DAM and RTM prices separately and subtracting — direct prediction avoids compounding errors from two independent models.

$$
\hat{\delta}_t = f(\mathbf{x}_t^{\text{weather}}, p_t^{\text{DA}}, \hat{\delta}_{t-1}, \dots, \hat{\delta}_{t-k})
$$

Where $\hat{\delta}_t = p_t^{\text{RT}} - p_t^{\text{DA}}$ is the price gap being predicted.

## Results

- **Best model:** LSTM with exogenous weather features (100 cells, Adam, MSE loss)
- Without weather features, the LSTM achieved MAE = 31.8 and RMSE = 62.15
- Adding weather features (solar radiation, wind, temperature) substantially reduced both metrics
- Direct gap prediction outperformed the subtract-two-forecasts baseline across all models tested

## Stack

- Python, scikit-learn, PyTorch
- CAISO electricity market data (2-year period)
- Random Forest for feature selection, LSTM for sequence modelling
