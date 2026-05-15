---
title: Autonomous Robot Car
summary: An embedded mobile robot capable of lane tracking, traffic light recognition, and obstacle avoidance.
featured_video: demo.mp4
tags:
  - Embedded Systems
  - STM32
  - Control
  - Perception
date: 2024-07-01
---

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap; margin-bottom: 1.5rem;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: 360px; object-fit: cover; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
      <source src="demo.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    A 1/10-scale embedded mobile robot built for the UESTC × Glasgow joint program's <strong>Team Design Project</strong> (Summer 2024) — designed to simulate urban driving tasks on a closed-circuit track with lane markings, traffic lights, and obstacles.
    <br><br>
    I led the development of the motion-control and perception subsystems. The demo shows a full lap at <em>10× speed</em> (silent): lane-following at full speed, traffic-light response, and obstacle avoidance, all running on an STM32F103 with an OpenMV camera.
  </div>
</div>

## System Architecture

* **Embedded Control:** Designed a custom PCB with **STM32F103** and L298N drivers. Implemented **PID controllers** to achieve smooth lane tracking (>98% success rate).
* **Perception:** Developed a vision pipeline on OpenMV using **NCC template matching** for arrow recognition and brightness-based blob detection for traffic lights.
* **Sensor Fusion:** Integrated ultrasonic sensors (HC-SR04) with median filtering for reliable obstacle avoidance.
