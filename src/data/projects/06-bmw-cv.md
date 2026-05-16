---
title: Visual Anomaly Detection for BMW Production Lines
description: Self-supervised anomaly detection for BMW assembly line cameras using SimCLR contrastive learning and Mahalanobis outlier scoring — no anomaly labels required for training.
github: https://github.com/kliyer-ai/simclr
draft: false
tags: [Python, Computer Vision, Deep Learning, GANs]
---

## Overview

BMW's production lines use AI systems to inspect vehicles in real time — verifying engine type and model inscriptions, checking component states, and flagging anomalies before a car leaves the assembly line.

![BMW assembly line — automated detection of engine (blue) and model (red) inscriptions](/bmw-cv/assembly-line.jpeg)

Three problems arise when deploying and maintaining these systems at scale: detecting when the camera itself behaves anomalously, detecting when the input distribution has drifted enough to degrade the model, and generating high-quality synthetic training data to compensate for class imbalance. Each became a separate project.

---

## 1. Fast Anomaly Detection with SimCLR

During production at Plant Dingolfing, car bodies pause in front of a pre-calibrated camera for a clean rear shot. The dataset spans ~10 months of production with roughly 15% anomalous frames — a highly imbalanced setting where labels are available only for validation, not for training.

Anomalies take several forms: the camera fires too early or late, the trunk is open when it should be closed, or the camera has shifted from its calibrated position.

<div class="grid grid-cols-2 gap-4 not-prose my-6">
  <img src="/bmw-cv/normal-rear.jpg" alt="Normal production frame — rear panel closed, correctly framed" class="rounded-lg w-full object-cover" />
  <img src="/bmw-cv/anomaly-dark.jpg" alt="Anomalous frame — camera triggered too early, inscriptions not visible" class="rounded-lg w-full object-cover" />
</div>

<div class="grid grid-cols-2 gap-4 not-prose my-6">
  <img src="/bmw-cv/anomaly-top.jpg" alt="Anomaly — camera angle shifted, trunk visible from above" class="rounded-lg w-full object-cover" />
  <img src="/bmw-cv/anomaly-open-trunk.jpg" alt="Anomaly — trunk open during inspection" class="rounded-lg w-full object-cover" />
</div>

### The SimCLR Framework

Rather than training a supervised classifier (which would require labeled anomalies), we used **SimCLR** — a self-supervised contrastive learning framework that learns visual representations purely from data augmentation.

SimCLR takes each image and produces two augmented views, then trains a network to maximize agreement between views of the same image while pushing apart views from different images. For BMW data, the augmentation pipeline starts with a central crop (to standardize the framing), followed by random crop and resize with horizontal flip, color distortion, and Gaussian blur.

![Augmented crop of assembly line frame — central crop removes the pixelated border region](/bmw-cv/augmentation-crop.jpg)

The NT-Xent (normalized temperature-scaled cross entropy) loss formalizes this objective: for a pair of views $(\tilde{x}_i, \tilde{x}_j)$ from the same image, the loss is:

![NT-Xent contrastive loss — maximize similarity of same-image pairs against all other pairs in the batch](/bmw-cv/nt-xent-loss.png)

The encoder produces embedding $h$, which is projected to $z$ through a nonlinear head during training. The key insight is that $h$ retains richer information — color, orientation, spatial structure — while $z$ is invariant to augmentation. Downstream tasks use $h$.

### Mahalanobis Outlier Detector

Once SimCLR produces embeddings, anomaly detection uses a **Mahalanobis distance** outlier detector rather than a softmax classifier. The key observation is that hidden features follow class-conditional Gaussian distributions — so a test sample's distance to the nearest class centroid in embedding space is a principled anomaly score.

![Mahalanobis detector architecture — penultimate layer features are modeled as class-wise Gaussians](/bmw-cv/mahalanobis-arch.png)

Mahalanobis distance accounts for the covariance structure of the embedding space, making it more sensitive than Euclidean distance to samples that fall outside the learned distribution.

![Mahalanobis (blue ellipse) vs Euclidean (red circle) — point B is close in Euclidean distance but far in Mahalanobis distance](/bmw-cv/mahalanobis-distance.png)

### Benchmarking on MVTec-AD

Before testing on BMW production data, the pipeline was validated on **MVTec-AD** — an industrial inspection benchmark with 5,300 images across 15 defect categories (cables, capsules, textures, hazelnuts, etc.), each with clean training images and defective test images with annotated masks.

![MVTec-AD dataset — green-bordered: defect-free; red-bordered: defective samples with annotated regions](/bmw-cv/mvtec-dataset.png)

### Results on Plant Dingolfing

With 100% ground truth labels available for calibration, the unsupervised SimCLR embedding combined with the Mahalanobis detector achieved strong separation between normal and anomalous frames (~47% anomaly rate in one test split, ~46% in another).

The more practically relevant scenario: what if only 1% of frames are labeled? Across 10 folds with 1% labeled data each, the detector still produced useful signal, demonstrating that the self-supervised framework drastically reduces labeling labour — a critical property for production deployment.

Performance degraded when the data was simultaneously small and diverse, confirming that contrastive learning needs either quantity or low diversity to build a reliable embedding.

---

## 2. Data Drift Detection

Even a well-trained model degrades over time. Lighting conditions change across shifts, cameras drift slightly from calibration, and new car models introduce visual variation the model was never trained on.

![Data drift: static models degrade monotonically; periodic retraining keeps quality within bounds](/bmw-cv/data-drift.jpeg)

The goal is a monitoring system that detects when the input distribution has shifted enough to trigger retraining — before production metrics visibly drop. The image quality metrics developed for the synthetic evaluation project (below) are directly reusable here as distribution statistics.

---

## 3. Synthetic Image Quality Evaluation

Anomalies are rare, so labeled anomaly examples for training are scarce. One solution is synthetic data generation via GANs. BMW had trained StyleGAN and conditional StyleGAN on model inscription images — but determining whether generated images are actually useful as training data requires a quantitative framework.

<div class="grid grid-cols-2 gap-4 not-prose my-6">
  <img src="/bmw-cv/real-m340i.jpeg" alt="Real M340i model inscription badges" class="rounded-lg w-full object-cover" />
  <img src="/bmw-cv/stylegan-fakes.jpeg" alt="StyleGAN fakes — blurred, distorted characters" class="rounded-lg w-full object-cover" />
</div>

*Real inscriptions (left) vs unconditional StyleGAN fakes (right) — blurring and color artifacts make the fakes unsuitable as training data.*

The conditional StyleGAN assigns a class ID to each inscription type and conditions generation on a one-hot vector, giving more control over the output. Training progression shows the generator moving from heavy artifacts toward legible inscriptions — but only the later, cleaner outputs are usable.

![Conditional StyleGAN training progression — top rows show artifacts, bottom rows converge to legible inscriptions](/bmw-cv/conditional-stylegan-grid.jpeg)

The project's output is a scoring framework using discriminator-derived confidence and standard image quality metrics to filter generated images above a quality threshold before adding them to the training set.

---

## Findings

**SimCLR for anomaly detection** works best when data is large and diverse, or small and homogeneous. With 100% labeled calibration data it reliably detects production anomalies. With as little as 1% labels per fold it still produces signal — a practically significant result for a setting where labeling is expensive. The failure case is small, diverse data where the contrastive objective cannot build a stable embedding.

**Mahalanobis distance** consistently outperforms a softmax classifier for anomaly scoring because it is sensitive to the full covariance structure of the embedding space — a sample can appear close in Euclidean distance but be a clear outlier under Mahalanobis.

**Data drift** and **synthetic quality evaluation** are tightly linked: the same feature-space statistics that detect distribution shift also measure whether a generated image falls within the real data distribution. A unified monitoring and quality-filtering framework across both problems is a natural next step.

## Stack

- Python (PyTorch, scikit-learn, OpenCV)
- SimCLR (contrastive self-supervised learning)
- StyleGAN, conditional StyleGAN
- Google BigQuery (MVTec-AD benchmarking data)
