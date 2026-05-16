---
title: ML-Based Localization on Embedded Devices
description: Deep learning and gradient-boosted models for Angle-of-Arrival estimation in real-time embedded environments using Bluetooth Low Energy signals.
paper: https://ieeexplore.ieee.org/abstract/document/10332487
draft: false
tags: [Python, Embedded ML, BLE, Signal Processing]
---

## Overview

Direction finding systems based on RF signals suffer from multipath propagation, especially indoors — existing algorithms like MUSIC degrade badly when signals reflect off walls or operate in low-SNR conditions. This project explores using machine learning to estimate Angle-of-Arrival (AoA) from multichannel BLE data and run those inferences in real-time on embedded hardware.

## Approach

Both a deep neural network and a gradient-boosted decision tree (GBT) are trained to estimate up to two AoAs from a single snapshot of multichannel BLE IQ data. Both classification (discrete angle bins) and regression (continuous angle) formulations are evaluated.

The trained models are compiled and deployed to embedded targets running a real-time operating system (RTOS), where inference latency is measured against hard timing constraints.

## Results

- **Best model:** Gradient-boosted decision tree — outperformed the deep learning model on AoA estimation accuracy
- GBT's fast inference time made it well-suited for RTOS deployment where DNNs exceeded latency budgets
- Both classification and regression variants were evaluated; check the paper for the specific angle error metrics
- Models generalised to unseen indoor environments, a known failure mode for multipath-sensitive RF algorithms

## Stack

- Python, scikit-learn, XGBoost
- Bluetooth Low Energy (BLE), software-defined radio
- Real-time operating system (RTOS) deployment
