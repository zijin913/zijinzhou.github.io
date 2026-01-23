---
title: Autonomous Urban Robot
summary: An embedded mobile robot capable of lane tracking, traffic light recognition, and obstacle avoidance.
tags:
  - Embedded Systems
  - STM32
  - Control
  - Perception
date: 2024-07-01
---

## System Architecture

This robot was designed to simulate urban driving tasks. I led the development of the motion control and perception subsystems.

* **Embedded Control:** Designed a custom PCB with **STM32F103** and L298N drivers. Implemented **PID controllers** to achieve smooth lane tracking (>98% success rate).
* **Perception:** Developed a vision pipeline on OpenMV using **NCC template matching** for arrow recognition and brightness-based blob detection for traffic lights.
* **Sensor Fusion:** Integrated ultrasonic sensors (HC-SR04) with median filtering for reliable obstacle avoidance.