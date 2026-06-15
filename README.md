# Learning Gait-Aware Quadruped Locomotion with Temporal Logic Specifications

**Authors:** Anonymous Author(s)[cite: 1] | **Presentation By:** Merve Atasever[cite: 2]
**Affiliation:** [Your Institution/Lab Here][cite: 1]
**Conference:** Submitted to the 10th Conference on Robot Learning (CoRL 2026)[cite: 1]



---

## Abstract

Reinforcement learning (RL) for quadruped locomotion commonly depends on fixed, hand-crafted, and Markovian reward functions[cite: 1]. These static rewards limit the interpretability of learned policies and lack explicit control over multi-gait behaviors[cite: 1]. 

We introduce a framework where distinct gaits are specified using parameterized constraints expressed in Signal Temporal Logic (STL)[cite: 1]. By calibrating parametric STL templates across three speed regimes (walking-trot, trot, and bound) from reference rollouts, we develop a continuous reward landscape that encodes desired behavior[cite: 1]. Tested on Google's Barkour quadruped robot using MuJoCo XLA (MJX), the STL-shaped rewards yield tighter velocity tracking, superior training stability, and greater interpretability for failure analysis compared to heuristic baselines[cite: 1].

---

## 📌 The Problem: Limitations of Hand-Crafted Rewards

Classical RL reward design presents several critical bottlenecks for complex legged locomotion[cite: 2]:
* Standard Markovian rewards fail to provide explicit control over speed-dependent gait transitions[cite: 1].
* Hand-tuned objectives often result in emergent, uncontrolled behaviors rather than structured, biomechanically sound gaits[cite: 2].
* Static heuristic functions lack interpretability, making it difficult to diagnose why a policy fails[cite: 2].
* A single standard reward struggles to generalize across diverse speed regimes without extensive task-specific tuning[cite: 2].

---

## ⚙️ Methodology: A Specification-Driven Pipeline

Our framework bridges classical formal methods and modern deep RL by using quantitative STL robustness as a reward signal[cite: 1]. The training pipeline leverages JAX-accelerated physics via MuJoCo XLA (MJX) for highly parallelized Proximal Policy Optimization (PPO)[cite: 1].

### 1. Data Collection & Feature Extraction
* The system extracts safety and gait-structure features from a compact dataset of expert trajectories[cite: 1].
* Evaluated features include foot contacts, stride period, duty factor, diagonal phase error, and center of mass (CoM) boundaries[cite: 1].

### 2. STL Template Fitting
* We define fixed Parametric Signal Temporal Logic (PSTL) templates for three locomotion modes: walking-trot, trot, and bound[cite: 1].
* Parameters for these templates are fit using empirical quantiles from the expert datasets[cite: 1].
* For example, the bound gait explicitly suppresses trot-like diagonal support patterns using the formula: $G_{\mathcal{W}_{B}}(p_{diag2} \le p_{diag2,max})$[cite: 1].

### 3. Hierarchical Reward Shaping
* The active locomotion mode $g(t) \in \{W,T,B\}$ is selected dynamically based on the commanded forward velocity $v_{x}^{cmd}$[cite: 1].
* The framework utilizes hysteresis around transition boundaries (e.g., 0.7 m/s and 1.7 m/s) to prevent mode chattering[cite: 1].
* The final scalar reward combines normalized robustness scores for safety limits, command tracking, and gait structure, alongside a torque-effort penalty[cite: 1].

---

## 📊 Key Results & Performance

A single, unified policy successfully adapts its gait structure across all speed regimes[cite: 1]. 

### Tracking Success Across Velocities

The table below demonstrates the superiority of STL-based rewards in maintaining high success rates (defined as average post-warmup speed staying within 15% of the commanded speed) compared to manually tuned heuristic baselines[cite: 1].

| Commanded Velocity ($v_{x}$) | STL-Based Success | Heuristic-Default Success | Heuristic-Best Success |
| :--- | :--- | :--- | :--- |
| **0.3 m/s** | 100%[cite: 1] | 0%[cite: 1] | 100%[cite: 1] |
| **0.7 m/s** | 100%[cite: 1] | 10%[cite: 1] | 100%[cite: 1] |
| **1.3 m/s** | 100%[cite: 1] | 30%[cite: 1] | 100%[cite: 1] |
| **1.6 m/s** | 100%[cite: 1] | 30%[cite: 1] | 100%[cite: 1] |
| **1.9 m/s** | 100%[cite: 1] | 0%[cite: 1] | 0%[cite: 1] |

### Interpretability & Failure Analysis
* The STL robustness signals provide immediate diagnostic value after training[cite: 1].
* Policy errors can be mathematically attributed to specific violated specifications, such as inadequate support states, excessive pitch variations, or actuator-limit violations[cite: 1, 2].
* Predicting upcoming failures is achievable by monitoring whether robustness values for critical predicates drop below zero within a recent time window[cite: 2].

---

## 🎥 Visualizing Locomotion Regimes

> *[Developer Note: Replace the src paths below with the actual paths to your video files once uploaded to the repository]*

**Trot Gait ($v_{x} = 1.3$ m/s)**

<video src="assets/trot.mp4" controls autoplay loop muted width="100%"></video>

The robot maintains strict diagonal phase coordination as defined by the active STL template[cite: 1].

**Bound Gait ($v_{x} = 1.8$ m/s)**

<video src="assets/bound.mp4" controls autoplay loop muted width="100%"></video>

The high-speed policy transitions to an alternating fore/hind leg synchronization, satisfying the aerial-phase occupancy constraints[cite: 1].

---

## 📝 Citation

If you find this work useful in your research, please consider citing:

```bibtex
@inproceedings{anonymous2026learning,
  title={Learning Gait-Aware Quadruped Locomotion with Temporal Logic Specifications},
  author={Anonymous Author(s)},
  booktitle={Submitted to the 10th Conference on Robot Learning (CoRL)},
  year={2026}
}
