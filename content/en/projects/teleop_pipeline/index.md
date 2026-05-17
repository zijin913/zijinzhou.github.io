---
title: "Multi-Modal Teleoperation and Data Collection for 6-DOF Manipulation"
summary: A three-mode teleoperation suite (phone-based VIO, SO-101 leader-follower, Meta Quest 3S VR) and the data-collection pipeline behind it — synchronized dual-arm HDF5 logging, Eva-adapted hand-eye calibration, and a DROID/Eva-compatible schema so the same demonstrations train policies on a different embodiment.
featured_video: iphone.mp4
tags:
  - Robot Learning
  - Teleoperation
  - Imitation Learning
  - Data Collection
  - VR
date: 2026-05-16
---

I built a **three-mode teleoperation suite and the data-collection pipeline behind it** for a 6-DOF HexArm manipulator. The three input modes — a phone-based ARKit VIO, an SO-101 leader-follower arm, and a Meta Quest 3S VR rig — all sit behind a single controller abstraction, so one collection script captures demonstrations from any of them in a uniform HDF5 schema. The pipeline is paired with an Eva-adapted hand-eye calibration toolkit and a DROID/Eva-compatible data format, so the trajectories drop straight into existing imitation-learning training stacks.

## A single controller abstraction

Every input device — and every learned policy — implements the same compact interface: each step yields a target action plus a small bag of state (clutch on/off, gripper aperture, home button). Data collection, replay, sim-to-real switching, and policy deployment all share one code path. Adding a new input device is one new component; nothing else in the stack changes.

That single decision is what made it tractable to ship three teleop modes on the same schedule — each one is a controller that returns the same shape of output, and the rest of the system never has to know which mode the human is using.

## Three teleop modes

*The clips below show single-arm collection so the demos read clearly; the same pipeline runs in **dual-arm** mode — two phones, two leader-follower pairs, or one headset driving both arms — without any changes to the recording or training stack downstream.*

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap; margin: 1rem 0 1.5rem;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: 360px; object-fit: cover; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
      <source src="iphone.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    <strong>iPhone (ARKit VIO).</strong> A handheld phone reports an ARKit-tracked pose in a world-locked frame; the clutch and an analog gripper come from on-screen UI. Most accessible hardware on the bench — anyone with a phone can collect data — but no haptic feedback, so the operator reads the arm visually. The video shows single-arm collection; dual-arm runs the same way with one phone per arm.
  </div>
</div>

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap; margin: 1rem 0 1.5rem;">
  <div style="flex: 0 0 320px; max-width: 320px;">
    <video style="width: 100%; height: 360px; object-fit: cover; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
      <source src="so101.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    <strong>SO-101 leader-follower (master-slave).</strong> A second SO-101 is used as a kinaesthetic <em>leader</em>; joint readings flow through the same controller interface to drive the <em>follower</em> arm. Best mode for contact-rich precision tasks (insertions, careful grasps), because the operator's hand on the leader feels exactly what the follower is doing. The video shows a single leader-follower pair; dual-arm doubles the rig.
  </div>
</div>

**Meta Quest 3S (VR).** An app on the headset captures controller pose and button state; a reader on the host (built on top of Berkeley RAIL's [`oculus_reader`](https://github.com/rail-berkeley/oculus_reader)) streams it in and presents it as the same standard input the rest of the system expects. One headset drives a dual-arm setup with the two controllers in a single operator's hands. A one-time frame alignment at the start of each session brings the headset's heading-locked reference into the robot base frame. Best 6-DOF immersion and the only mode that lets a single operator control both arms simultaneously; pays for it with the per-session calibration step. *(Demo video coming soon.)*

## Data collection pipeline

This is the part that mattered most for downstream imitation learning, and where most of the engineering went.

**Off-thread recorder.** Trajectory recording runs on a background thread; cameras are grabbed in a subprocess. Vision throughput never backpressures the control loop, so the arm holds a stable 100 Hz control rate while images and joint states are captured at the recording rate.

**Always-on recording with clutch passthrough.** Releasing the clutch does *not* pause recording — the recorded action becomes "hold the current state." This keeps the dataset coherent during operator pauses: a learned policy sees a continuous observation/action stream rather than a sequence with mysterious gaps it would have to learn around.

**Synchronized dual-arm logging.** One HDF5 file per episode storing per-arm actions, observations, controller state (clutch, gripper, success flags), and nanosecond-precision timestamps. Image frames are aligned 1:1 with HDF5 rows by index, so there's no separate synchronization pass before training.

**Success / failure auto-categorization.** A single keystroke at the end of each episode files the trajectory into a success or failure folder. No manual cleanup pass before training; the failure trajectories are already segregated and can be opted into negative-example training if a method wants them.

**One workflow across all teleop modes.** A single command drives every collection session — the operator chooses the teleop mode and single- vs. dual-arm setup as flags. This is the payoff of the controller abstraction: recording, calibration, success tracking, and HDF5 writing are identical regardless of which input device the operator is using.

## Hand-eye calibration (adapted from Eva)

The calibration toolkit is **adapted from [Eva](https://github.com/willjhliang/eva)** — Eva's ChArUco-board-based hand-eye routine was the starting point. I extended it to fit the HexArm setup (which doesn't expose a hardware-side Cartesian pose API, so the wrist solution runs on joint states plus forward kinematics) and to handle the multi-camera + dual-arm cases used in collection.

The pipeline is **collect → solve → verify**. ChArUco frames are captured from each camera, reprojection-error filtering trims outliers, a standard hand-eye solver produces the rigid transform, and a held-out validation split confirms the result before it's written out.

Two scenarios are supported:

- **Eye-in-hand (wrist camera).** Uses paired joint-state snapshots and forward kinematics to recover the camera-to-end-effector transform.
- **Eye-to-hand (side camera).** Solved via global reprojection optimization across shared wrist / side ChArUco observations, so the side camera ends up expressed in the same robot base frame as the wrist camera.

Output is a small set of per-camera, per-arm transform files, consumed directly by the data recorder so every image in the dataset comes with a known camera-to-base transform.

Why this matters for a learning audience: the trajectories aren't just "phone-poses + RGB" — they're geometrically grounded RGB observations with consistent camera extrinsics, which is what makes them usable for downstream visuo-motor policy training rather than only end-effector replay.

## DROID / Eva-compatible HDF5 schema

The HDF5 layout is structured to match the format used by [Eva](https://github.com/willjhliang/eva), a Franka Panda research stack built on DROID. Trajectories collected on the HexArm platform can feed Eva's downstream training and replay tools without a conversion pass.

The practical payoff: the same demonstrations can be used for cross-embodiment imitation-learning experiments (HexArm vs. Franka Panda) without rewriting the data pipeline. Out-of-the-box export to LeRobot and robomimic formats is supported alongside, for compatibility with their training scripts.

## What was hard

Designing the controller interface so VIO, leader-follower, and VR all looked like the same upstream component — including the cases where the host has to solve inverse kinematics (VIO and VR) versus directly mapping joints (leader-follower) versus mixed-source dual-arm. Collapsing those into a single shape of output was the project's hinge.

Keeping the recording loop and the control loop honestly decoupled. Camera throughput vs. control frequency was the main tension; pushing capture into a subprocess and pushing recording onto a background thread was what let the control loop stay at 100 Hz while still producing aligned multi-camera datasets.

Schema discipline so HexArm data is genuinely drop-in for the Eva pipeline — not "compatible after a script." Mostly unit alignment, time-stamping conventions, and image-frame index hygiene; tedious work, but the alternative is every downstream consumer writing their own adapter.

## Acknowledgments

Calibration toolkit and HDF5 schema both adapted from [`willjhliang/eva`](https://github.com/willjhliang/eva) — thanks to Will Liang for the open-source Franka research infrastructure. The VR teleop builds on Berkeley RAIL's [`oculus_reader`](https://github.com/rail-berkeley/oculus_reader) for the headset-to-host bridge.
