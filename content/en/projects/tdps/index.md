---
title: "An Intelligent Robotic Car for Simulated Urban Driving"
summary: A six-person embedded systems capstone — a 4-wheel autonomous robot car built from scratch with a custom STM32 PCB, dual OpenMV cameras, ultrasonic sensors, and NRF24L01 wireless — completing six urban-driving tasks within a 1000 RMB budget.
featured_video: demo.mp4
math: true
tags:
  - Embedded Systems
  - STM32
  - OpenMV
  - Control
  - Perception
date: 2024-07-01

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
    A 1/10-scale autonomous robot car built for the UESTC × Glasgow joint program's <strong>Team Design Project</strong> (Summer 2024) — a six-person, three-month capstone in which we designed, built, and demonstrated a vehicle that completes <strong>six tasks in a simulated urban environment</strong>: lane tracking, arrow recognition, traffic-light response, crosswalk and pedestrian detection, obstacle avoidance, and wireless gate communication.
    <br><br>
    Hard constraints: total budget ≤ 1000 RMB, no Raspberry Pi / Arduino, no external compute, and all six tasks completed in a single 10-minute live demonstration. Final cost: 995.67 RMB. The demo on the left runs at <em>10× speed</em> (silent).
    <br><br>
    <strong>My role:</strong> motion-control and perception subsystem lead — custom motor-driver PCB, PID lane-tracking, and the OpenMV arrow/traffic-light pipeline.
  </div>
</div>

## The six tasks

| # | Task | Approach | Result |
| --- | --- | --- | --- |
| 1 | Lane tracking | OpenMV3 line detection + PID | **> 98 %** success |
| 2 | Arrow direction recognition | NCC stop + Maximum Color Blob Analysis | **80–100 %** across rooms |
| 3 | Traffic-light recognition | Brightness-threshold color-block on orange trigger line | **85.7 %** (30 / 35) |
| 4 | Crosswalk + pedestrian detection | Dual-color patch centroid + ultrasonic check | **90 %** (27 / 30) |
| 5 | Obstacle collision avoidance | Servo-scanning ultrasonic + lateral maneuver | reliable end-to-end |
| 6 | Wireless gate communication | NRF24L01 @ 2.4 GHz over SPI | **90 %** gate open, ~5 s latency |

The whole circuit had to come together inside a 10-minute live demo with no human intervention — debug-once, run-clean.

## Four subsystems

The car decomposes into four cooperating subsystems sharing one STM32F103RCT6 brain. Two OpenMV cameras run on-device computer vision and report decisions back to the STM32 over UART, while the STM32 owns motion, ultrasonics, and the wireless link.

### Motion — custom PCB + L298N + PID

The motion subsystem runs on an **STM32F103RCT6** with a **custom-designed L298N motor-driver PCB** (schematic capture, layout, fabrication, and hand-assembly — Top/Bottom of the two-sided board both populated). Dual H-bridges drive four DC gear motors with PWM speed modulation; Schottky diodes catch inductive kickback from the motor windings.

After hardware bring-up we characterized the drivetrain at multiple duty cycles and settled on **70 % PWM as the cruise speed** — fast enough to finish the 10-minute demo, slow enough for the vision loop to keep up.

**Lane tracking** runs in a separate OpenMV3 M7 module mounted **25 cm above the deck at a 30° pitch** (chosen experimentally — higher mount + smaller pitch angle = wider field of view + cleaner lane contour). OpenMV3 captures the blue lane lines, computes positional and angular offsets, and feeds them into a **PID controller** (distance gain 0.25, angle gain 0.4); the resulting steering correction is sent over UART to the STM32, which translates it into per-wheel PWM. Final success rate: **> 98 %** across repeated laps, including complex three-way intersections.

### Vision — two OpenMV cameras, three algorithms

Vision splits across two OpenMV boards so the lane loop never blocks on pattern recognition.

**OpenMV H7 Plus** (the front camera, STM32H743 + OV5640) runs the *pattern* pipeline:

- **Arrow recognition.** A two-stage state machine. The *parking stage* uses **NCC template matching** to detect the arrow placeholder and stop the car within ~30 cm of the sign. We tried CNN, NCC, and Maximum Color Blob Analysis for the *direction stage*; the blob analysis won on accuracy + speed: set ROI → grayscale → binary filter → find the largest connected component → use its orientation and the number of pixels on each side to classify left / right / forward. A 5-second averaging window suppresses single-frame anomalies.
- **Traffic-light recognition.** The OpenMV continuously scans for an orange cross-line on the ground; on detection, the car stops and the camera looks for the brightest region in the upper-right of the frame and classifies its color. Red / yellow → stay; green → go. Brightness-threshold color blocking beat both template-matching and an ML classifier on robustness and latency.
- **Crosswalk + pedestrian detection.** A **dual-color (black/white) patch centroid algorithm**: find the closest black and white patches, draw a line between their centroids, count pixels along the line. Above threshold → stop. The trick is that the threshold itself implicitly sets the stopping distance from the zebra crossing — exactly the distance our ultrasonic sensors want to start scanning for pedestrians.

**OpenMV3 M7** (the down-facing camera, dedicated to lane tracking) runs only the PID line-following loop and forwards corrections + decisions to the STM32 over UART.

### Ultrasonic ranging — two HC-SR04 + median filter

Obstacle avoidance and pedestrian detection both need accurate, low-latency distance — beyond what OpenMV can give at oblique angles. Two **HC-SR04** ultrasonic sensors are mounted **side-by-side on an MG90S servo**; the servo rotates to scan for the obstacle envelope.

Distance is the classic time-of-flight:

$$
\text{Distance} = \frac{v_\text{sound} \cdot t}{2}
$$

with $v_\text{sound} = 340$ m/s. Raw HC-SR04 readings jitter under fluorescent lighting, so each beam is passed through a **median filter (window = 11)**, then the two filtered beams are themselves median-merged. Measured standard deviation across 20–50 cm: **0.3 mm – 1.0 cm** — well within what the avoidance maneuver needs.

### Wireless — NRF24L01 between car and gate

The finish-line gate is a separate STM32 + servo + OLED system. The car and gate talk over **NRF24L01 @ 2.4 GHz**, with each MCU driving the transceiver over **SPI**. When the car crosses the green finish-zone, it loads a payload into the on-car NRF buffer; the gate's NRF receives it and triggers an MG90S servo to swing open while the OLED cycles `Wait / BUF[1] / BUF[2] / Success`. Round-trip latency ~5 s, with 90 % open-rate across 20 trials.

## What was hard

*The motor driver had to be **our** PCB.* No off-the-shelf modules allowed. The first board had a layout error that caused brownout under load; the second revision fixed power-trace width and decoupling and held up through the entire demo.

*Two cameras on UART means message framing matters.* OpenMV's USART writes can interleave between the two MV → STM32 streams, so we ended up with a tiny framed protocol (start byte + payload + checksum) instead of trusting newline-terminated strings. After that, drift was zero.

*Lighting tanks color-threshold vision.* Traffic-light recognition worked perfectly in Room 462 and 85 % in Room 463 because of overhead fluorescents reflecting off the lane paint. Switching from RGB thresholds to a brightness-first detector recovered most of the gap.

*The 10-minute timer.* All six tasks have to be completed in a single 10-minute run, so we tuned the cruise speed (70 % duty), the arrow stop distance (~30 cm), and the ultrasonic scan rate together — slow enough for the vision loop, fast enough for the deadline.

## Team

Team No. 57 (UESTC × Glasgow joint program, Summer 2024): Zhang Yifei, Xu Xinyao, Zijin Zhou, Ni Binqin, Wang Zeping, Li Jiajun.
