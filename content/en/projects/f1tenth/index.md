---
title: F1Tenth Autonomous Racing
summary: Programming a 1/10th-scale autonomous racecar in ROS 2 — from reactive collision avoidance to SLAM-based navigation — competing in live races at UPenn's Levine Lobby. 1st place in Race 2.
featured_video: final_race.mp4
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

links:
- type: code
  url: https://github.com/zijin913/ese615-f1tenth
---

## The platform

F1Tenth is a competitive autonomous racing platform built around 1/10th-scale racecars. Each car carries a Hokuyo LiDAR, an IMU, and a Jetson Orin NX — no GPS, no remote control, no safety driver. Everything runs on ROS 2 and the team's own code.

UPenn's Spring 2026 course (ESE 615) progresses from safety-critical reactive systems to map-based racing to motion planning, vision, and model predictive control — across eight labs and three live races inside Levine and Houston Hall.

---

## Race 1 — Reactive Racing

February 25, 2026 · Levine Lobby, UPenn · **3rd place**

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: 360px; object-fit: cover; border-radius: 10px; display: block;" controls playsinline preload="metadata">
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

## Race 2 — Racing with Map

March 23, 2026 · Levine Lobby, UPenn · **1st place**

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: 360px; object-fit: cover; border-radius: 10px; display: block;" controls playsinline preload="metadata">
      <source src="race2.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    Race 2 introduced a <strong>known map</strong> and full localization. Instead of reacting to immediate sensor data, the car plans ahead: it localizes on the stored map using a <strong>Particle Filter</strong> and tracks a globally optimal racing line with <strong>Pure Pursuit</strong>. The gap between reactive and map-based strategies shows up most at high-speed corners, where lookahead distance makes the difference between smooth racing and emergency braking.
  </div>
</div>

---

## Final Race — Head-to-Head Racing

Houston Hall, UPenn · **3rd place**

<div style="display: flex; gap: 1.5rem; align-items: flex-start; flex-wrap: nowrap;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: auto; border-radius: 10px;" controls playsinline preload="metadata">
      <source src="final_race.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    The final race takes everything from the semester and puts two cars on the same track. The challenge is no longer just tracking a single optimal raceline — it's also <strong>overtaking</strong> a moving opponent without leaving the track or stalling on contact.
    <br><br>
    Our stack combined a <strong>multi-raceline pure-pursuit planner</strong> (precomputed inner / outer lines, with lane-switching logic triggered by opponent proximity) with a reactive overtaking layer that watches the LiDAR for opponent footprints, and an <strong>RRT\*</strong> fallback for obstacle-heavy laps where the precomputed lines were no longer feasible. The biggest tuning work was around the hand-off: when to commit to a passing line, when to abandon it, and how to recover to the nominal raceline once clear.
  </div>
</div>

---

## Labs and algorithms

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

### Lab 5 — SLAM & Pure Pursuit

Built the full mapping and localization stack: **slam_toolbox** for online map construction, a **Particle Filter** for real-time localization on a pre-built map, and **Pure Pursuit** for smooth, speed-aware path tracking along a pre-computed racing line.

#### Particle Filter (Monte Carlo Localization)

The particle filter maintains a set of pose hypotheses $(x, y, \theta)$ and converges on the true position by fusing odometry and LiDAR observations each update cycle.

**Motion model** — odometry deltas are rotated into each particle's local frame and applied with Gaussian noise on $x$, $y$, and $\theta$ to account for wheel slip and model error:

$$
\begin{bmatrix} \Delta x_i \\ \Delta y_i \end{bmatrix} = R(\theta_i) \begin{bmatrix} \Delta x_\text{odom} \\ \Delta y_\text{odom} \end{bmatrix} + \mathcal{N}(0, \sigma)
$$

**Sensor model** — for each particle, simulated LiDAR ranges are cast against the occupancy grid using RangeLibc (CDDT or ray marching). Each beam's likelihood is a mixture of four terms: Gaussian hit, short-reading exponential, max-range spike, and uniform random noise. The full table is precomputed once and looked up at runtime:

$$
p(z \mid d) = z_\text{hit} \cdot \mathcal{N}(z; d, \sigma_\text{hit}) + z_\text{short} \cdot \frac{2(d-z)}{d} \cdot \mathbf{1}_{z < d} + z_\text{max} \cdot \mathbf{1}_{z = z_\text{max}} + z_\text{rand} \cdot \frac{1}{z_\text{max}}
$$

Particle weights are raised to an inverse squash factor before normalization to prevent premature filter collapse.

**Pose estimate** — the inferred pose is the weighted mean of the particle distribution. Heading is averaged in the circular sense ($\arctan2$ of mean $\sin/\cos$) to avoid wrap-around errors.

**TF publishing** — the filter resolves the full `map → odom → base_link` chain: it looks up the current `odom → base_link` transform from the VESC driver, then computes `map → odom = T_\text{map,base} \cdot T_\text{odom,base}^{-1}` and broadcasts it, keeping the downstream TF tree consistent.

#### Pure Pursuit

- **Arc-length lookahead**: target waypoint is found by accumulating actual path distance along the waypoint sequence, not Euclidean distance — prevents cutting corners at high speed.
- **Forward-only closest-point search**: searches only 200 waypoints ahead of the last known position, preventing backward jumps when the car completes a lap and re-enters the start zone.
- **Dynamic speed control**: speed is mapped inversely to steering angle magnitude (max speed on straights, min speed through tight corners) with exponential smoothing to avoid abrupt transitions.

### Lab 6 — Motion Planning (RRT / RRT\*)

Built a local planner for overtaking on top of a live **occupancy grid** generated from LiDAR. Implemented **RRT** (sampling-based path planning) and then upgraded to **RRT\*** with rewiring, so the path it returns converges toward the asymptotically optimal solution as the sample count grows. The planner is biased toward the goal point on the global raceline, with a collision check against the inflated occupancy grid at every node. Returned paths are smoothed and handed to Pure Pursuit for tracking.

### Lab 7 — Vision

Built the perception pipeline on an **Intel RealSense** camera: V4L2 capture, **OpenCV intrinsic calibration**, and **pinhole-geometry** distance estimation for cones of known size on the ground plane. Layered **YOLO** object detection on top and deployed it through **TensorRT** on the Jetson for real-time inference (cone classification → 2D bounding box → 3D position via known cone height and camera intrinsics).

### Lab 8 — Model Predictive Control

Implemented an **MPC** controller around a **kinematic bicycle model**. The continuous dynamics are linearized about the current operating point and discretized into a state-space form $x_{k+1} = A_k x_k + B_k u_k + C_k$, then a finite-horizon optimal control problem is solved as a **convex QP** at each step using CVXPY/OSQP. The cost balances tracking error against control effort and rate, with hard input constraints (steering angle, acceleration) and a soft track-boundary corridor. MPC produced visibly smoother lines through corners than Pure Pursuit at the same target speed.

---


## Stack

| Layer | Details |
|---|---|
| OS & Middleware | Ubuntu 22.04, ROS 2 Humble |
| Compute | NVIDIA Jetson Orin NX |
| Languages | C++, Python |
| Sensors | Hokuyo 10LX LiDAR, IMU, Intel RealSense |
| Algorithms | AEB, PID Wall Follow, Follow the Gap, Particle Filter, Pure Pursuit, RRT / RRT\*, YOLO + TensorRT, MPC |
| Simulation | F1Tenth Gym (ROS 2 bridge) |

