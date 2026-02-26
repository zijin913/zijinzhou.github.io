---
title: 3D Reconstruction & AR Pipeline
summary: A complete multi-view stereo pipeline implemented from scratch using COLMAP, LOFTR, and PyTorch.
tags:
  - Computer Vision
  - SLAM
  - PyTorch
date: 2025-11-01

links:
- type: code
  url: https://github.com/zijin913/CIS5800-Machine-Perception
---

## Project Highlights

* **Full Pipeline Implementation:** Built a 3D reconstruction system from multi-view images using **COLMAP** and **LOFTR** for robust feature matching.
* **Custom Bundle Adjustment:** Implemented a gradient-descent based **Bundle Adjustment (BA)** optimizer in PyTorch to refine camera poses and 3D structure.
* **AR Application:** Developed an Augmented Reality module using **PnP/P3P algorithms** and AprilTag tracking to render virtual objects into real-world coordinates with precise alignment.