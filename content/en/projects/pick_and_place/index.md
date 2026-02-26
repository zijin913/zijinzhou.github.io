---
title: Autonomous Robotic Manipulation
summary: A full-stack autonomous manipulation system capable of static stacking and dynamic grasping on a moving turntable.
math: true
tags:
  - Manipulation
  - Robotics
  - Python
  - Motion Planning
  - Kinematics
date: 2025-12-13

links:
- type: code
  url: https://github.com/zijin913/meam520_final
---

## 🎯 Project Scope
Developed for the UPenn MEAM 5200 Final Competition, this project required the **Franka Emika Panda** arm to autonomously detect, pick, and stack blocks to build the tallest tower. The system had to handle:
* **Static Environment:** Precision stacking of scattered blocks.
* **Dynamic Environment:** Grasping moving targets on a rotating turntable with unknown speed.

<br>**My Role:** Motion Planning (IK), State Machine Architecture, Sim-to-Real Deployment.

---

## 🧠 Core Algorithms

### 1. Dynamic Trajectory Prediction
To grasp objects on a moving turntable, I implemented a kinematic prediction model. The future pose $P_{pred}$ at time $t+\Delta t$ is estimated by transforming the current center-relative position using the rotation matrix $R_{turn}$:

$$
P_{pred} = P_c + R_{turn}(\omega \cdot \Delta t) \cdot (P_{curr} - P_c)
$$

Where $P_c$ is the turntable center and $\omega$ is the estimated angular velocity. This allowed the robot to **intercept** the object rather than chasing it.

### 2. Robust State Machine
We designed a hierarchical Finite State Machine (FSM) to ensure robust autonomy. Unlike linear pipelines, our FSM includes a dedicated **RECOVERY** state to handle grasp failures or IK singularities dynamically.

<a href="fsm_flow_chart.svg" target="_blank">
  <img src="fsm_flow_chart.svg" alt="FSM Flow Chart" style="cursor: zoom-in; width: 100%;">
</a>

---

## 📺 Demo & Results

<div style="position: relative; display: inline-block; width: 80%;">
  <video width="100%" controls muted autoplay loop>
    <source src="demo_5x.mp4" type="video/mp4">
  </video>
  <span style="position: absolute; bottom: 50px; left: 10px; background: rgba(0,0,0,0.7); color: #fff; padding: 4px 8px; border-radius: 4px; font-size: 14px; font-weight: bold;">5x</span>
</div>