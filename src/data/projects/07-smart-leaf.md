---
title: Smart Leaf
description: Autonomous smart greenhouse using sensor arrays and machine learning to monitor and control temperature, humidity, light, and irrigation.
paper: https://electrical.sdsu.edu/design-day/files/2020/smartgreenhouse.pdf
draft: false
tags: [Python, TensorFlow, Raspberry Pi, IoT]
---

*Senior Design Project — San Diego State University, Spring 2020*

**Team:** Hanieh Moein, Siamak Doraghi, Emilio Nuno, Nika Nizharadze
**Advisor:** Dr. Saeed Manshadi

---

![Smart Leaf prototype — greenhouse enclosure connected to Raspberry Pi and monitor](/smart-leaf/prototype.jpeg)

## The Problem

Climate unpredictability makes greenhouse farming hard. Temperature swings, inconsistent light, and over- or under-watering all reduce yield. Traditional greenhouses rely on manual monitoring — which means energy waste, labor cost, and human error.

We set out to eliminate the human from the control loop entirely.

## System Overview

![Product overview diagram showing sensors, indoor/outdoor monitoring, plant & soil tracking, remote management, and ML-powered equipment control](/smart-leaf/system-overview.png)

The system closes the loop between sensing and actuation: sensors feed readings into ML models running on a Raspberry Pi, which then drives the actuators — AC, irrigation, and lighting — to keep conditions optimal for the crop.

## Architecture

![System architecture: Solar Panel → Battery → Raspberry Pi → Camera + Sensors → AC / Irrigation / Light / Water → Greenhouse](/smart-leaf/architecture.png)

The entire system runs on solar power. The Raspberry Pi 4 is the central controller — it ingests data from all sensors, runs inference, and commands the environment control systems.

## Machine Learning

Three distinct models handle the intelligence layer:

**1. Tomato detection** — SSD MobileNet fine-tuned via transfer learning on manually labeled images. The base model doesn't know what tomatoes look like, so we retrained it on our own dataset.

**2. Ripeness classification** — KNN clustering on visual features. Three clusters: *unripe*, *medium*, and *ripe*. The ripeness level determines the target environmental conditions.

**3. Disease detection** — CNN trained on the PlantVillage dataset. **Achieved 90% accuracy** on held-out test data.

![Object detection output — bounding boxes with tomato detection confidence scores](/smart-leaf/detection.jpeg)

The ripeness level isn't just a label — it feeds back into the control system. A ripe tomato needs different light and temperature conditions than an unripe one.

## Hardware

| Component | Role |
|---|---|
| Raspberry Pi 4 | Central processor, runs TensorFlow models |
| RPi Camera Module V2 | Image capture for all three ML models |
| TSL2591 Light Sensor | Measures lux for lighting control |
| Si7021 Temp & Humidity | Drives AC and ventilation decisions |
| YL-69 Soil Moisture | Triggers and limits irrigation |

<div class="grid grid-cols-2 gap-4 not-prose my-6">
  <img src="/smart-leaf/raspberry-pi.jpeg" alt="Raspberry Pi 4" class="rounded-lg w-full object-cover" />
  <img src="/smart-leaf/camera.jpeg" alt="Raspberry Pi Camera Module V2" class="rounded-lg w-full object-cover" />
  <img src="/smart-leaf/light-sensor.jpeg" alt="TSL2591 Light Sensor" class="rounded-lg w-full object-cover" />
  <img src="/smart-leaf/humidity-sensor.jpeg" alt="Si7021 Temperature & Humidity Sensor" class="rounded-lg w-full object-cover" />
  <img src="/smart-leaf/soil-sensor.jpeg" alt="YL-69 Soil Moisture Sensor" class="rounded-lg w-full object-cover" />
</div>

The Raspberry Pi 4 was chosen specifically for its TensorFlow support — running inference on-device, no cloud dependency.

## Budget Breakdown

| Component | Share |
|---|---|
| Raspberry Pi 4 | 54% |
| Camera | 16% |
| Other | 18% |
| Sensors (×3) | 12% |
