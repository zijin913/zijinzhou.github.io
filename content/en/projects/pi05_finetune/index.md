---
title: "LoRA Fine-Tuning π0.5 on Self-Collected Bimanual Demonstrations"
summary: Closing the loop from teleop data collection to a deployed language-conditioned policy — LoRA fine-tuning Physical Intelligence's π0.5 VLA on demonstrations collected with my own teleop pipeline, and running it on a dual-HexArm setup for pick-and-place, cube stacking, and hanging a mug on a hanger.
featured_video: pi05.mp4
tags:
  - Robot Learning
  - VLA
  - Imitation Learning
  - Fine-Tuning
  - Bimanual Manipulation
date: 2026-08-18

links:
- name: π0.5 (openpi)
  url: https://github.com/Physical-Intelligence/openpi
- name: Teleop & data collection pipeline
  url: /projects/teleop_pipeline/
---

The [teleoperation and data-collection pipeline](/projects/teleop_pipeline/) only matters if the data it produces actually trains a policy that runs on the robot. This project is that last step: **LoRA fine-tuning π0.5**, Physical Intelligence's vision-language-action model, on demonstrations I collected with the pipeline, and deploying it closed-loop on the same dual-HexArm hardware.

<div style="margin: 1.5rem 0;">
  <video style="width: 100%; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
    <source src="pi05.mp4" type="video/mp4">
  </video>
  <div style="margin-top: 0.75rem; font-size: 0.95em; color: #666;">
    The fine-tuned policy running on hardware: language-conditioned pick-and-place into a bowl, cube stacking, and hanging a mug on a mug hanger.
  </div>
</div>

## Setup

**Model.** π0.5 from the [openpi](https://github.com/Physical-Intelligence/openpi) release, fine-tuned with **LoRA** rather than full fine-tuning so that the run fits on a single workstation GPU and the pretrained VLM backbone stays largely intact. Only the low-rank adapters and the action expert are updated.

**Data.** All training demonstrations were collected with my [three-mode teleop suite](/projects/teleop_pipeline/) — mostly via the Meta Quest 3S rig, since it is the only mode that lets one operator drive both arms at once. Episodes are logged in the pipeline's DROID-compatible HDF5 schema with synchronized dual-arm proprioception, multi-view RGB, and per-episode success tags, so converting to openpi's training format is a thin adapter rather than a re-labeling pass.

**Tasks.** Three language-conditioned tabletop tasks on the dual-arm setup:

- **Pick-and-place** — move a block into a bowl.
- **Cube stacking** — stack one cube on top of another.
- **Mug hanging** — pick up a mug by the handle and hang it on a mug hanger, which needs an orientation-aware grasp and a precise approach.

## What I learned

- **Data quality dominates.** The replay check in the teleop pipeline (165 insert-and-remove cycles, zero failures) was worth more than any training-side trick: once the recorded trajectories and hand-eye calibration were trustworthy, fine-tuning was mostly uneventful.
- **Embodiment mismatch is the real gap.** π0.5's pretraining mix does not include this arm, so the LoRA adapters are doing real work in mapping the VLM's visual understanding onto a new action space, not just polishing.
- **Clutch passthrough matters for VLAs.** Recording "hold current state" during operator pauses instead of cutting the episode gives the policy a continuous observation/action stream — the fine-tuned model never learned to freeze at arbitrary points the way it might have with gappy data.
