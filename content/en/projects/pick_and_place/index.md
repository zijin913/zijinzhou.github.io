---
title: Sim-to-Real Robotic Manipulation
summary: A high-precision manipulation system featuring a custom Numerical IK Solver and dynamic trajectory prediction.
tags:
  - Manipulation
  - Robotics
  - Python
  - Motion Planning
  - Kinematics
date: 2025-12-01

# Optional: Link to your code (if public)
links:
  - icon: github
    icon_pack: fab
    name: View Code
    url: https://github.com/your-repo-url
---

## Project Overview

This project tackles the challenge of **dynamic object grasping** in a cluttered environment. The system was developed for the MEAM520 challenge at UPenn, achieving a **98% hardware recovery rate** through robust failure detection logic.

### 🚀 Key Technical Contributions

#### 1. Custom Numerical IK Solver
Instead of using off-the-shelf solvers (like KDL or Trac-IK), I implemented a **Jacobian Pseudo-Inverse ($J^{\dagger}$)** solver from scratch in C++.
* **Singularity Avoidance:** Integrated **Null-Space Projection** to optimize joint centering, ensuring the arm stays away from singular configurations during complex maneuvers.
* **Optimization:** Used gradient descent to minimize pose error effectively.

#### 2. Dynamic Grasping Strategy
* **Predictive Modeling:** Utilizing kinematic modeling to estimate **future object poses** on a moving turntable.
* **Orientation Control:** Applied **Gram-Schmidt Orthogonalization** to compute the optimal end-effector orientation under 6-DOF constraints, ensuring stable grasps.

#### 3. Sim-to-Real Deployment
* Implemented a **Finite State Machine (FSM)** in Python to orchestrate the entire pick-and-place cycle.
* The system features automated **failure detection and retry logic**, bridging the gap between simulation stability and real-world noise.

---

### 📺 Demo & Results

*(建议这里放一张 GIF 动图，文件名建议叫 featured.gif 放在同目录下)*
![Robot Demo](featured.gif)