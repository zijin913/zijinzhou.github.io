---
title: F1Tenth Autonomous Racing
summary: Programming a 1/10th-scale autonomous racecar in ROS 2 — from reactive collision avoidance to SLAM-based navigation — competing in live races at UPenn's Levine Lobby.
math: true
tags:
  - Robotics
  - ROS 2
  - Autonomous
  - C++
  - Python
  - SLAM
  - Control Systems
date: 2026-02-25
---

## 🏎️ What is F1Tenth?

**F1Tenth** is a competitive autonomous racing platform built around 1/10th-scale racecars. Each car is equipped with a **Hokuyo LiDAR**, an **IMU**, and a **Jetson Orin NX** compute module — no GPS, no remote control, no safety driver. Everything runs on ROS 2 and the team's own code.

The course at UPenn (Spring 2026) progresses from safety-critical reactive systems to full SLAM-based autonomous racing over 6+ labs and multiple live race events inside Levine Hall.

---

## 🏁 Race 1: Reactive Racing — Completed ✅

**Date:** February 25, 2026 &nbsp;|&nbsp; **Venue:** Levine Lobby, UPenn

**Result:** 3rd Place 🥉

<div style="display: flex; gap: 1.5rem; align-items: flex-start; flex-wrap: nowrap;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: auto; border-radius: 10px;" controls playsinline preload="metadata">
      <source src="race1.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    Race 1 is a pure reactive challenge: <strong>no map, no localization</strong> — just a live LiDAR scan and real-time control logic. The car must navigate the circuit at full speed using only what it can sense at each instant.
    <br><br>
    Our approach used the <strong>Disparity Extender</strong> variant of Follow the Gap, selecting the widest free corridor ahead and maximizing speed within safety limits. The key tuning challenge: at racing speeds the LiDAR scan latency becomes the dominant bottleneck, so we profiled our ROS 2 callback chain and tightened loop timing before race day.
  </div>
</div>

---

## 🔬 Labs & Algorithms

### Lab 1 — ROS 2 Fundamentals

Bootstrapped the full ROS 2 (Humble) development environment, worked through topics, services, and actions, and got the **F1Tenth Gym** simulator running with a ROS 2 bridge.

### Lab 2 — Automatic Emergency Braking (AEB)

Implemented a dedicated safety node that computes **Time-To-Collision (TTC)** from raw LiDAR beams and triggers full braking when a collision is imminent. This node sits at the top of the priority stack and overrides all other velocity commands.

$$
TTC = \frac{r}{(-\dot{r})^+}
$$

where $r$ is the measured range and $\dot{r}$ is the rate of change projected onto the beam direction. Only beams with a positive closing rate (i.e., approaching obstacles) contribute.

### Lab 3 — Wall Following (PID)

Used paired LiDAR beams to estimate lateral distance and angle relative to the wall, then closed a **PID loop** to hold a target offset. The derivative term effectively dampens oscillation by penalizing heading error before it compounds.

### Lab 4 — Follow the Gap (Reactive Obstacle Avoidance)

Full implementation of the **Disparity Extender** algorithm:

1. Compute range disparities between adjacent LiDAR beams
2. Extend obstacle footprints radially by the car's half-width to create a conservative safety margin
3. Find the deepest, widest free gap in the processed scan
4. Steer toward the center of the best gap; scale speed inversely with steering angle

### Lab 5 — SLAM & Pure Pursuit *(in progress)*

Building the mapping and localization stack: **slam_toolbox** for online map construction, a **Particle Filter** for real-time position estimates on a pre-built map, and **Pure Pursuit** for smooth, speed-aware path tracking along a pre-computed racing line.

---

## 🗺️ Up Next: Race 2 — Racing with Map

**Date:** March 23, 2026 &nbsp;|&nbsp; **Venue:** Levine Lobby, UPenn

Race 2 introduces a **known map** and full localization. Instead of reacting to immediate sensor data, the car plans ahead: it localizes on the stored map and tracks a globally optimal racing line rather than a purely reactive heading. The gap between reactive and map-based strategies shows up most at high-speed corners, where lookahead distance makes the difference between smooth racing and emergency braking.

---

## 🛠️ Tech Stack

| Layer | Details |
|---|---|
| OS & Middleware | Ubuntu 22.04, ROS 2 Humble |
| Sensors | Hokuyo 10LX LiDAR, IMU |
| Compute | NVIDIA Jetson Orin NX |
| Languages | C++, Python |
| Algorithms | AEB, PID Wall Follow, Follow the Gap, Particle Filter, Pure Pursuit, RRT |
| Simulation | F1Tenth Gym (ROS 2 bridge) |

