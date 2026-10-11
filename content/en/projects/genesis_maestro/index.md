---
title: "Genesis Maestro: Real-to-Sim Manipulation for VLM Agents"
summary: A Genesis simulation platform for vision-language robot agents — reconstructed manipulation scenes, stereo-estimated depth, physical interaction checks, and closed-loop evaluation from tabletop skills to long-horizon tool use.
featured_video: preview.mp4
date: 2026-10-10
tags:
  - Robot Simulation
  - VLM Agents
  - Real-to-Sim
links:
  - name: Related — bimanual Maestro on hardware
    url: /projects/maestro_bimanual/
---

I built a **Genesis-based manipulation and evaluation platform** to study how vision-language agents turn visual observations into physical actions. The work brings together reconstructed tabletop scenes, a DROID-style robot and camera setup, a simulation port of Maestro's tool interfaces, and independent checks of what the robot actually accomplished.

The same environment supports both **Maestro agents with perception, grasping, and geometry tools** and **agents using a basic robot-control interface**. This makes it possible to inspect where a task breaks down: perception, grasp selection, motion, physical contact, or the agent's judgment that it is finished.

<style>
.gm-figure { margin: 1.5rem 0 2rem; }
.gm-figure video { display: block; width: 100%; height: auto; aspect-ratio: 16 / 9; border-radius: 10px; background: #111827; }
.gm-caption { margin-top: .65rem; font-size: .86em; line-height: 1.6; opacity: .78; }
.gm-flow { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: .65rem; margin: 1.5rem 0; }
.gm-flow div { padding: .85rem; border: 1px solid #80808040; border-radius: 10px; font-size: .85em; line-height: 1.5; }
.gm-flow strong { display: block; margin-bottom: .35rem; }
.gm-grid { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 1.4rem; }
.gm-grid .gm-figure { margin: 0 0 1rem; min-width: 0; }
@media (max-width: 600px) { .gm-flow { grid-template-columns: repeat(2, minmax(0, 1fr)); } .gm-grid { grid-template-columns: minmax(0, 1fr); } }
</style>

## Discovering how to open a puzzle box

The instruction is simply: **“Place the red block in the green tray, then release it and withdraw.”** The agent receives no mechanism description, unlocking sequence, demonstration, or task-specific strategy. It works from camera images, stereo-estimated depth, and the robot's own state and feedback.

<figure class="gm-figure">
  <video controls muted playsinline preload="metadata" poster="featured.jpg" aria-label="Four camera views of a robot unlocking a puzzle box and placing its red block in a green tray">
    <source src="puzzle.mp4" type="video/mp4">
    <a href="puzzle.mp4">Watch the puzzle-box demonstration.</a>
  </video>
  <figcaption class="gm-caption">A complete successful episode through the basic Franka interface, without Maestro perception or grasp tools. The four-view recording includes an overhead review camera that is not supplied to the agent. API waiting time is omitted.</figcaption>
</figure>

The agent removes the yellow locking pin at response 4, then repeatedly tries to lift or rotate the lid as if it were hinged. At response 22 it tries a small sideways translation and discovers that the lid slides. It opens the box, extracts the block, and places it in the tray, finishing in **27 responses**.

The outside evaluator verifies unlocking, extraction, tray support, release, and final stability. Maximum recorded penetration is **0.93 mm**. This is one complete successful trial under this condition, demonstrating recovery through exploration rather than establishing a success rate.

## Retrieving a tool to retrieve an object

A second long-horizon task requires the robot to extract a long hook, use it to retrieve a block, and place the block in a tray. The useful action depends on first acquiring and positioning another object.

<figure class="gm-figure">
  <video controls muted playsinline preload="metadata" poster="tool-retrieval.jpg" aria-label="Robot retrieving a long hook and using it to recover a block">
    <source src="tool-retrieval.mp4" type="video/mp4">
    <a href="tool-retrieval.mp4">Watch the tool-retrieval demonstration.</a>
  </video>
  <figcaption class="gm-caption">The successful episode finishes in 28 responses: the long hook is extracted at response 13 and the block at response 24. This run includes one strategy sentence advising greater clearance around overhead structures.</figcaption>
</figure>

This case also exposes the difficulty of tool manipulation: the three preceding attempts with the same hint stopped on physical penetration checks. The shown attempt passes the outside checks, with a maximum recorded penetration of **2.54 mm**. Both long-horizon demos use FoundationStereo depth with no exact-depth fallback; their successful trajectories remain below 3 mm penetration, although the configured robot and static-contact limits for these runs were 5 mm.

## From reconstructed scenes to visual feedback

The platform combines a **Franka Panda with a Robotiq 2F-85 gripper**, two external cameras, and a moving wrist camera. The real-to-sim asset workflow keeps visual meshes, collision geometry, articulated joints, and physical metadata separate. A mug needs a usable handle opening; a cabinet needs a working hinge; an eraser needs contact with the board. Those details determine whether an interaction works.

<div class="gm-flow" role="list" aria-label="The agent's perception and action loop">
  <div role="listitem"><strong>01 · Observe</strong>Rendered stereo pairs and robot proprioception</div>
  <div role="listitem"><strong>02 · Perceive</strong>FoundationStereo depth, visual pointing, and optional perception tools</div>
  <div role="listitem"><strong>03 · Act</strong>Generated code calls robot motion and manipulation tools</div>
  <div role="listitem"><strong>04 · Replan</strong>New images and execution feedback inform the next action</div>
</div>

In stereo mode, depth is estimated from rendered left/right images and feeds pointing, point clouds, grasp proposals, and geometry checks. **Hidden object poses, simulator contact measurements, and task scores stay outside the model loop.** The independent observer uses those quantities to evaluate the run.

This remains a simulation: the cameras have exact calibration and no sensor noise, and asset mass and friction include declared assumptions. Reconstructed appearance is provisional. The platform is intended for controlled experiments, without claiming validated sim-to-real transfer.

## Contact-rich tabletop manipulation

These selected Maestro S3 episodes use stereo-estimated depth. Each video shows one run; the captions describe that run's outcome and conditions.

<div class="gm-grid">
  <figure class="gm-figure">
    <video controls muted playsinline preload="none" poster="cabinet.jpg" aria-label="Robot grasping a cabinet handle and opening the door">
      <source src="cabinet.mp4" type="video/mp4">
      <a href="cabinet.mp4">Watch the cabinet demonstration.</a>
    </video>
    <figcaption class="gm-caption"><strong>Open a cabinet.</strong> Both pads grasp the handle; the door remains at 57.5° after release. Local task guidance enabled.</figcaption>
  </figure>
  <figure class="gm-figure">
    <video controls muted playsinline preload="none" poster="mug.jpg" aria-label="Robot extracting a mug and hanging it on a peg">
      <source src="mug.mp4" type="video/mp4">
      <a href="mug.mp4">Watch the mug demonstration.</a>
    </video>
    <figcaption class="gm-caption"><strong>Hang a mug.</strong> Extract the mug, align its handle with a peg, and release it onto the holder. Completed in 14 responses without the local task strategy.</figcaption>
  </figure>
  <figure class="gm-figure">
    <video controls muted playsinline preload="none" poster="whiteboard.jpg" aria-label="Robot sweeping an eraser across a marked whiteboard">
      <source src="whiteboard.mp4" type="video/mp4">
      <a href="whiteboard.mp4">Watch the whiteboard demonstration.</a>
    </video>
    <figcaption class="gm-caption"><strong>Erase a whiteboard.</strong> Physical eraser contact drives visual ink removal: 396/396 segments cleared, with all action checks passing. Local task guidance enabled.</figcaption>
  </figure>
  <figure class="gm-figure">
    <video controls muted playsinline preload="none" poster="cube.jpg" aria-label="Robot rotating a colored cube and placing it purple side up">
      <source src="cube.mp4" type="video/mp4">
      <a href="cube.mp4">Watch the cube demonstration.</a>
    </video>
    <figcaption class="gm-caption"><strong>Reorient a cube.</strong> Grasp, rotate while held, and place the cube with its purple face up. Local task guidance enabled.</figcaption>
  </figure>
</div>

The four tabletop recordings above play at **4× their source-video speed**, with API waiting time already omitted in the source. The homepage uses a short, accelerated excerpt of the puzzle-box episode.

## What the experiments reveal

The evaluation spans a basic control interface and cumulative Maestro stages: **S0 visual pointing → S1 perception → S2 grasp generation → S3 geometry tools**, together with their associated prompts and replanning guidance. Runs preserve observations, generated actions, tool outputs, physical checks, and declared outcomes.

**Tool failures can look like reasoning failures.** In an early Gemini S2 cabinet cohort, the grasp tool returned no usable handle candidates. After repairing the grasp chain, all five reruns reached a two-pad handle grasp. The remaining difficulty was pulling along the hinge arc. Inspecting the intermediate milestones revealed a different bottleneck from the final task score alone.

**A success claim needs an independent check.** An agent can announce a completed cut without blade contact, or a correctly oriented cube when the target face is still sideways. The evaluator separates those claims from measured outcomes. The knife task checks held blade contact and a downward stroke; it does not simulate the banana being severed.

**Comparisons depend on the full experimental condition.** Some cohorts differ in depth source, task wording, tool fixes, or local guidance, and most cells contain only three to five trials. Staged progress scores are useful diagnostics, but are not success percentages or a clean causal ranking of tool stages.

## Relationship to my other robotics work

This project complements my [bimanual Maestro work on real hardware](/projects/maestro_bimanual/). Here, my focus is the simulation environment, asset and sensor integration, control adapters, and evaluation infrastructure. Maestro provides the agent framework; Genesis provides the physics engine; GroundedSAM, GraspGen, and FoundationStereo provide specialized perception components. The staged comparisons described here use the simulation's IK motion path; cuRobo integration is a separate supported configuration.
