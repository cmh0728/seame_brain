# System Architecture Overview

This document describes the overall system architecture, including software stack, hardware selection, perception & localization, and planning & control modules.

---

## Software

- **Python**
- **C++**
- **ROS 2 Humble**

---

## Hardware

### Implemented Sensors & Computing Unit
- **Intel RealSense D455f** (Depth Camera)
- **AMT103-V Wheel Encoder**
- **NVIDIA Jetson Orin Nano**

### Why These Components?

#### AMT103-V Wheel Encoder
The AMT103-V wheel encoder is used to measure wheel rotation, which enables accurate estimation of the vehicle’s longitudinal speed.  
Although an IMU is available, relying solely on IMU data leads to accumulated drift over time.

By fusing **wheel encoder** and **IMU** data, we can improve velocity estimation accuracy and achieve more stable vehicle pose estimation.

#### Intel RealSense D455f
Since the vehicle is not equipped with a LiDAR sensor, direct distance measurement to obstacles using laser-based sensing is not possible.  
Instead, a **depth camera** is used to obtain per-pixel distance information, enabling obstacle distance estimation and spatial perception.

The Intel RealSense D455f provides sufficient depth range and accuracy for indoor track environments.

#### NVIDIA Jetson Orin Nano
Object detection is performed using deep learning–based models such as **YOLO**.  
Compared to Raspberry Pi 5, the Jetson Orin Nano offers significantly higher GPU performance, making it suitable for real-time inference and parallel ROS 2 node execution.

---

## Perception & Localization

### Lane Detection
- Inverse Perspective Mapping (IPM)
- Canny edge detection

### Object Detection
- Traffic sign detection
- Obstacle detection

For object detection, we use the **YOLO** algorithm.  
A suitable YOLO model is selected and optimized for the competition environment.

In addition, **HSV color-space filtering** is applied to distinguish traffic light colors, improving robustness under varying lighting conditions.

### Localization
- Sensor fusion using **Extended Kalman Filter (EKF)**
  - Wheel encoder
  - IMU
- **Particle Filter–based localization** initialized from EKF estimation

In the final competition environment, **GPS data is not available**.  
Therefore, localization must rely entirely on onboard sensors.

First, the **EKF** estimates the vehicle’s pose by fusing wheel encoder and IMU data, providing an initial position estimate.  
Then, particles are generated around this EKF-estimated pose.

Using **lane-map matching**, the Particle Filter evaluates each particle by comparing detected lane features (in Bird’s-Eye View) with a pre-mapped lane map.  
Through iterative resampling, the particle distribution converges to the most likely vehicle position.

The combination of **EKF** for motion-based estimation and **Particle Filter** for map-based correction enables accurate localization without GPS.

---

## Planning & Control

- Global path planning
- Local path planning
- Model Predictive Control (MPC)
