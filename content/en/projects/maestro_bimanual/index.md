---
title: "A Tool-Composing VLM Agent for Bimanual Manipulation"
summary: Extending Maestro — a VLM coding agent that composes perception, grasp, motion-planning, and learned-policy tool modules into programmatic policies — from single-arm to bimanual operation on a two-HexArm setup. Demonstrated on cloth folding and blackboard erasing.
featured_video: towel.mp4
tags:
  - Robot Learning
  - Bimanual Manipulation
  - VLM Agent
  - Tool Use
  - Imitation Learning
date: 2026-05-17

links:
- name: Maestro
  url: https://maestro-robot.github.io/
---

[**Maestro**](https://maestro-robot.github.io/) (Shi et al., UPenn) is a VLM coding agent that composes specialized perception, planning, control, and learned-policy *tool modules* into programmatic policies for generalist robots. The paper demonstrates a competitive modular policy on tabletop and mobile *single-arm* tasks. **I'm extending Maestro to bimanual operation** on a two-HexArm setup — many of the manipulation tasks that matter (cloth folding, surface cleaning, large-object handling, etc.) require two arms cooperating, and Maestro's modular tool-composition design makes this primarily a tool-orchestration problem rather than a from-scratch policy problem.

## Why bimanual is a coding-agent problem

The original Maestro paper makes the argument that *programmatic* policies — composed by an LLM/VLM agent from a small library of well-typed robotics tools — can match or exceed monolithic VLA models on generalist manipulation. The argument transfers cleanly to bimanual: the *toolbox* doesn't change much (you still need perception, grasps, plans, learned skills), but the *plan* now has to assign roles to two arms, coordinate their motion, and avoid them colliding. That's exactly the kind of structured reasoning a coding agent is good at and a monolithic policy is bad at.

Concretely, what changes is the *program* the agent writes, not the modules it calls. So the bimanual extension is mostly about (a) exposing per-arm tool variants the agent can target, (b) widening the planner to multi-arm awareness, and (c) updating the prompt and examples so the agent reliably reasons about two-arm role assignment.

## Toolbox composed by the coding agent

Each module below is exposed as a named tool that the VLM agent can call from the generated plan. All of them existed in single-arm Maestro; the bimanual work is mostly about per-arm variants, multi-arm planning, and the prompt-level reasoning over them.

- **Open-vocabulary perception** — [GroundedSAM](https://github.com/IDEA-Research/Grounded-Segment-Anything) for segmenting task-relevant objects from natural-language prompts; VLM-selected keypoints layered on top of the masks for finer spatial reasoning (e.g., picking *corner* vs. *centroid* of a cloth).
- **Grasp synthesis** — [GraspGen](https://research.nvidia.com/labs/gear/graspgen/) generates candidate gripper poses on the segmented point cloud, called *per-arm*: the agent chooses which arm grasps which object/keypoint.
- **GPU-accelerated motion planning** — [CuRoBo](https://curobo.org/) solves trajectory optimization with native multi-arm collision-avoidance, so the two arms plan around each other rather than around each other's stale plans.
- **High-level reasoning** — *Gemini 2.5 Pro* as the coding agent: it decomposes the task, assigns roles to each arm, and emits Python-like code that composes the modules above into a plan.
- **Local VLM monitor** — *Qwen2.5-VL* runs as a fast closed-loop progress checker (following the Maestro paper), triggering replans when a subtask doesn't complete cleanly.
- **Learned short-horizon skills** — VLA / diffusion-policy tools trained on demonstrations from a [bimanual teleop and data-collection pipeline](../teleop_pipeline/), invoked by the agent for the parts of a task that don't decompose into clean geometric primitives.

## What changes for bimanual

**Per-arm tool calls + shared perception.** Perception runs once per scene; grasp synthesis and motion planning are scoped per arm. The agent reasons over which arm is best positioned for each subtask and writes the calls accordingly.

**Role assignment and synchronization.** The agent decides whether the two arms move *sequentially* (one grasps and holds, then the other acts), *synchronously* (both arms move toward a meeting pose together), or *asymmetrically* (one arm stabilizes a fixture while the other manipulates). For cloth folding, the program looks like *grasp corner A with arm L → grasp corner B with arm R → synchronously lift-and-meet → release B → re-grasp to compress the fold*. The agent writes this control flow as code rather than memorizing a fixed skeleton.

**Inter-arm collision avoidance at the planning layer.** Two arms working close together collide in obvious and non-obvious ways; CuRoBo's multi-arm support handles this during trajectory optimization rather than via after-the-fact retries. The agent treats the planner as a black-box safety net and focuses on the higher-level role assignment.

**Closed-loop monitoring scales naturally.** The Qwen2.5-VL monitor checks subtask progress at low frequency; if the cloth slips out of one arm mid-fold, the agent replans (e.g., re-grasp) without restarting the whole task. Same loop as in single-arm Maestro, just with an extra arm in the action space.

## Demo tasks

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap; margin: 1rem 0 2rem;">
  <div style="flex: 0 0 360px; max-width: 360px;">
    <video style="width: 100%; height: auto; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
      <source src="towel.mp4" type="video/mp4">
    </video>
    <p style="text-align: center; font-size: 0.8em; color: #888; margin: 0.4rem 0 0;">2× speed.</p>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    <strong>Towel folding.</strong> Two arms cooperatively grasp opposite corners and bring them together. The agent picks which corner each arm grasps from the perception mask, sequences the lift-and-meet motion, and re-grasps to compress the fold. The program the agent writes is short and readable — <em>grasp corner A with arm L → grasp corner B with arm R → synchronously lift-and-meet → release B → re-grasp to compress</em> — which is the kind of control flow it would have to memorize as a fixed skeleton in a monolithic policy.
  </div>
</div>

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap; margin: 1rem 0 2rem;">
  <div style="flex: 0 0 360px; max-width: 360px;">
    <video style="width: 100%; height: auto; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
      <source src="handover.mp4" type="video/mp4">
    </video>
    <p style="text-align: center; font-size: 0.8em; color: #888; margin: 0.4rem 0 0;">30× speed.</p>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    <strong>Cross-workspace handover.</strong> When a target placement is outside one arm's reachable workspace, the agent doesn't fail or replan from scratch — it inserts a handover into the program. Here, a block on the left side has to be placed on a plate well outside the left arm's reach. The agent reasons about per-arm reachability, has the <strong>left arm pick up the block and hand it off to the right arm</strong>, and the <strong>right arm finishes the placement</strong>. The reach check is done against CuRoBo's workspace model; the handover itself is a short scripted primitive that both arms call.
  </div>
</div>

<div style="display: flex; gap: 1.5rem; align-items: center; flex-wrap: nowrap; margin: 1rem 0 2rem;">
  <div style="flex: 0 0 360px; max-width: 360px;">
    <video style="width: 100%; height: auto; border-radius: 10px; display: block;" autoplay muted loop playsinline controls preload="metadata">
      <source src="blackboard.mp4" type="video/mp4">
    </video>
  </div>
  <div style="flex: 1 1 auto; min-width: 0;">
    <strong>Selective blackboard erasing.</strong> Two demos side-by-side. The <em>top</em> half shows coordinated wiping across the whole board — one arm wipes while the other stabilizes or repositions the eraser, with the agent choosing the sweep pattern from the perception mask. The <em>bottom</em> half is the more interesting case: the prompt asks the system to <strong>erase only the green dots without touching the red ones</strong>. The perception module returns separate masks for green vs. red dots; the coding agent generates a wipe trajectory that visits each green-dot region while routing around the red ones — exactly the kind of structured spatial reasoning that benefits from a code-writing agent over a monolithic policy.
  </div>
</div>

## Acknowledgments

This work extends [Maestro](https://maestro-robot.github.io/) — thanks to Junyao Shi, Rujia Yang, Kaitian Chao, Selina Wan, Yifei Shao, Jiahui Lei, Jianing Qian, Long Le, Pratik Chaudhari, Kostas Daniilidis, Chuan Wen, and Dinesh Jayaraman for the modular VLM-agent framework and for sharing the tabletop infrastructure that this extension builds on.
