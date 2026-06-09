# Collaborative Manipulation in Cluttered Scenes: Dual-Branch Grasping and Stackelberg Pushing

---

## Executive Summary

This document summarizes a novel reinforcement learning framework designed to improve robotic manipulation in densely cluttered environments.

The research addresses the inherent challenges of occlusions and ambiguous geometries by integrating precise grasping with proactive scene rearrangement (pushing).

The system utilizes a dual-branch architecture that decouples grasp position and orientation, allowing for more expressive, continuous action representations.

Coordination between pushing and grasping is modeled as a Stackelberg game, where a pushing agent acts as a strategic leader, intervening only when a grasp is likely to fail.

Experimental results in simulation demonstrate that this method significantly improves grasp success rates and action efficiency, outperforming existing baselines like Visual Pushing for Grasping (VPG) while minimizing redundant movements.

---

# 1. Core Challenges in Cluttered Manipulation

Robotic manipulation in cluttered tabletop settings is hindered by several factors:

* **Physical Constraints:** High-density piles restrict access to feasible grasp configurations.
* **Perceptual Ambiguity:** Occlusions and complex object geometries make direct grasping unreliable.
* **Inefficient Coordination:** Existing frameworks often rely on heuristic triggers or loosely coupled modules, leading to excessive pushing actions that do not necessarily improve the environment for grasping.
* **Discretization Limits:** Many systems discretize grasp orientations into a fixed set of angles, reducing the robot's flexibility.

---

# 2. The Dual-Branch Grasping Framework

The proposed architecture focuses on a 2D top-down grasping formulation where grasps are defined by a planar position (x, y) and an in-plane rotation angle (yaw).

---

## 2.1 Decoupled Architecture

The grasping agent is divided into two specialized branches to enhance spatial and angular reasoning:

* **Position Branch:**  
  Processes a full heightmap (projected from RGB-D data) to predict pixel-wise grasp quality scores.

* **Angle Branch:**  
  Takes a 24×24 local heightmap patch centered at the selected position to regress a continuous grasp orientation.

  * **Representation:** Uses a 2D vector [\sin(2\theta), \cos(2\theta)] to handle the inherent symmetry of parallel-jaw grasping.

---

## 2.2 Pushing Angle Prediction

The pushing agent shares a similar structure with the angle branch but predicts the direction as [\sin(\theta), \cos(\theta)].

Unlike grasping, pushing lacks symmetry, requiring full directional resolution.

---

# 3. Strategic Coordination: The Stackelberg Game

To resolve the conflict between "when to push" and "when to grasp," the interaction is modeled as a Stackelberg game.

* **The Leader (Push Agent):**  
  This agent is strategic. It evaluates the scene and selects actions that rearrange objects to improve future graspability.

* **The Follower (Grasp Agent):**  
  This agent is reactive. It estimates its success probability based on the current (or modified) environment.

* **The Mechanism:**  
  A push is only triggered if the Angle Critic network predicts a Q-value (grasp success probability) below a specific threshold \tau.

  If the predicted grasp quality is high, the system bypasses the push and executes the grasp directly.

  This prevents "redundant or unnecessary pushing actions."

---

# 4. Multi-Stage Training Procedure

The framework is trained in three sequential stages to ensure stability and sample efficiency:

| Stage | Method | Focus |
|---|---|---|
| 1. Pretraining | Self-supervised learning | Uses successful grasp-only data from simulation to initialize position and angle networks. |
| 2. Grasp Training | Reinforcement Learning (TD3) | Refines the dual-branch policy using shaped rewards for position (clutter avoidance) and angle (occlusion awareness). |
| 3. Push Training | Delayed RL | The push agent is activated and trained using a "look-ahead" reward based on grasp success in the subsequent T=3 steps. |

---

## Reward Formulation Key Points

* **Position Reward:**  
  A hybrid signal combining exponential decay (sensitivity near object centers) and linear functions (measuring spatial clearance/openness).

* **Angle Reward:**  
  An occlusion-aware signal that penalizes orientations where pixels along the grasp line are higher than the grasp center.

* **Push Reward:**  
  Strictly tied to downstream efficiency, measuring the average grasp success rate in the steps following a push.

---

# 5. Experimental Results and Analysis

Experiments were conducted in the CoppeliaSim environment using scenes of 25 blocks with randomized dimensions.

---

## 5.1 Quantitative Performance

The system was evaluated against the VPG baseline and various ablated versions.

### Performance Comparison Table

| Method | Completion (%) | Grasp Success (%) | Action Efficiency (%) |
| :--- | :--- | :--- | :--- |
| VPG (Grasp Only) | 90.5 | 55.8 | 55.8 |
| VPG (With Pushing) | 100.0 | 67.7 | 60.9 |
| Grasp Agent Only (Ours) | 93.2 | 63.8 | 63.8 |
| Full System (\tau=0.7) | 100.0 | 81.5 | 76.4 |
| Full System (\tau=0.8) | 100.0 | 86.2 | 73.9 |

---

## 5.2 Key Findings

* **Efficiency vs. Reliability:**  
  Raising the Q-threshold to 0.8 increases the grasp success rate to 86.2% but decreases action efficiency (73.9%) because the robot pushes more frequently.

  The 0.7 threshold was found to be a practical balance.

* **Initialization Impact:**  
  Pretraining provided a significant performance advantage, with the grasp success rate starting higher and stabilizing at 65% before the push agent was even introduced.

* **Coordination Gains:**  
  The introduction of the push agent (Stage 3) raised the final grasp success rate from ~60% to over 80%, proving that strategic pushing effectively "enhances graspability by improving scene conditions."

---

# 6. Conclusion

The dual-branch reinforcement learning framework offers a robust solution for manipulation in clutter.

By decoupling position and angle prediction and employing a Stackelberg game for agent coordination, the system achieves:

1. **High Success Rates:** Over 80% in dense clutter.
2. **Operational Economy:** Significant reduction in redundant actions compared to traditional push-grasp synergies.
3. **Stability:** Efficient training through targeted pretraining and branch-specific reward shaping.

## 📖 Research Paper Reference

> **Collaborative Manipulation in Cluttered Scenes: Dual-Branch Grasping and Stackelberg Pushing**  
> Proceedings of IEEE International Conference on Control, Automation and Systems (ICCAS 2025)  
> http://dx.doi.org/10.23919/ICCAS66577.2025.11301175

---

## Disclaimer

This repository contains a personal study, technical review, and research-learning analysis based on the original research paper.

All original methodologies, experiments, and contributions belong to the respective authors.
