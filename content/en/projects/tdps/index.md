---
title: "An Intelligent Robotic Car for Simulated Urban Driving"
summary: A six-person embedded-systems capstone — a 4-wheel autonomous robot car built from scratch with a custom STM32 PCB, dual OpenMV cameras, ultrasonic sensors, and NRF24L01 wireless — completing six urban-driving tasks within a 1000 RMB budget.
featured_video: demo.mp4
math: true
tags:
  - Embedded Systems
  - STM32
  - OpenMV
  - Control
  - Perception
date: 2025-10-02

links:
- name: Report
  url: /uploads/tdps_report.pdf
---

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap; margin-bottom: 1.5rem;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: 360px; object-fit: cover; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
      <source src="demo.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    A 1/10-scale autonomous robot car built for the UESTC × Glasgow joint program's <strong>Team Design Project</strong> (Summer 2024) — six people, three months, one live demo. Six urban-driving tasks in a single 10-minute run, no manual intervention, ≤ 1000 RMB budget, no Raspberry Pi / Arduino. Final cost: 995.67 RMB. Demo at <em>10× speed</em>, silent.
    <br><br>
    <strong>My role:</strong> motion-control and perception lead — custom motor-driver PCB, PID lane-tracking, and the OpenMV arrow/traffic-light pipeline.
  </div>
</div>

## The six tasks

| # | Task | Approach | Result |
| --- | --- | --- | --- |
| 1 | Lane tracking | OpenMV3 line detection + PID | **> 98 %** |
| 2 | Arrow recognition | NCC parking stop + Maximum Color Blob Analysis | 80 – 100 % |
| 3 | Traffic-light recognition | Brightness-threshold color blocks | **85.7 %** |
| 4 | Crosswalk + pedestrian | Dual-color patch centroid + ultrasonic | **90 %** |
| 5 | Obstacle avoidance | Servo-scanning ultrasonic + lateral maneuver | reliable |
| 6 | Wireless gate communication | NRF24L01 @ 2.4 GHz over SPI | **90 %** |

## System architecture

![System architecture: tasks mapped to four cooperating subsystems — Visual (dual OpenMV), Motion (STM32 + L298N + gear motors), Ultrasonic Sensing (HC-SR04 × 2 + servo), and Wireless Communication (NRF24L01 + gate)](system_architecture.png)

Four subsystems sharing an **STM32F103RCT6** brain. The OpenMV boards run vision on-device and report decisions over UART; the STM32 owns motion, ultrasonics, and the radio link. Motor speed is PWM-modulated through a custom L298N H-bridge PCB; lane-tracking lives on its own OpenMV3 M7 so the pattern-recognition loop never blocks driving. The wireless link uses NRF24L01 over SPI between the car's STM32 and an identical pair on the gate, which drives an MG90S servo and an I²C OLED to acknowledge the open command.

![Robot build: two-layer chassis with custom L298N PCB, dual OpenMV cameras, NRF24L01 radio, and Stm32F103RCT6 control board](robot_build.png)

## In the field

<div style="display: flex; gap: 1rem; align-items: stretch; flex-wrap: nowrap; margin: 1.5rem 0;">
  <img src="action_zebra.jpeg" alt="Stopping at a zebra crossing for a pedestrian (toy dog)" style="flex: 1 1 0; min-width: 0; width: 100%; height: 280px; object-fit: cover; border-radius: 8px; margin: 0;" />
  <img src="action_obstacle.jpeg" alt="Avoiding a cone obstacle and returning to the lane" style="flex: 1 1 0; min-width: 0; width: 100%; height: 280px; object-fit: cover; border-radius: 8px; margin: 0;" />
  <img src="action_gate.jpeg" alt="Stopped in the green finish zone, signaling the gate to open via NRF24L01" style="flex: 1 1 0; min-width: 0; width: 100%; height: 280px; object-fit: cover; border-radius: 8px; margin: 0;" />
</div>

*Left to right: pedestrian-aware stop at the zebra crossing, obstacle avoidance around a cone, and wireless gate communication at the finish line.*

![Lane-tracking parameter tuning (before / after) and robustness at a three-way intersection](lane_tracking.png)

Lane tracking ended up at a **25 cm camera height with a 30° pitch**, PID gains of (distance 0.25, angle 0.4) — chosen after sweeping mount geometries and gains in the lab. The resulting controller holds the lane through complex three-way intersections and recovers cleanly when the car drifts onto the line.

## Team

Team No. 57 — UESTC × Glasgow joint program, Summer 2024: Zhang Yifei, Xu Xinyao, Zijin Zhou, Ni Binqin, Wang Zeping, Li Jiajun.
