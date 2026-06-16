<div align="center">

# Learning Gait-Aware Quadruped Locomotion with Temporal Logic Specifications

**Anonymous Author(s)**

*Affiliation | Address | Email*

Submitted to the 10th Conference on Robot Learning (CoRL 2026)

**[📄 Read Paper](#)** |  **[💻 View Code](#)** |  **[🎥 Watch Video](#)** <img src="assets/quadruped%20model.png" width="500" alt="Barkour Quadruped Model">

</div>

---

## Abstract

Reinforcement learning (RL) for quadruped locomotion commonly depends on fixed, hand-crafted, and Markovian reward functions that limit both interpretability of learned policies and lack explicit control over gait behaviors. We introduce a framework where distinct gaits are specified using parameterized constraints expressed in Signal Temporal Logic (STL). These include safety bounds, gait synchronization constraints, command tracking, and actuation bounds. 

From these specifications, we develop a reward shaping mechanism that provides learning agents a dense, continuous reward landscape that encodes desired behavior. We define parametric STL templates for three speed regimes (walking-trot, trot, bound), calibrate their parameters from reference rollouts, and compute rewards using smooth approximations of STL robustness over the rollouts. The generated rewards can be used to provide shaped gradients compatible with Proximal Policy Optimization (PPO). We instantiate the approach on Google’s Barkour quadruped robot in MuJoCo XLA (MJX). We show that compared to a baseline of hand-crafted rewards, the STL-shaped rewards yield tighter velocity tracking and more stable training.

---

## 📌 Introduction & Motivation

Classical model-based control approaches, such as differential dynamic programming (DDP) and model predictive control (MPC), enable impressive behaviors but rely on accurate system models and complex cost functions. Deep RL pipelines address command tracking and stability through engineered rewards, curriculum learning, and domain randomization, but these rewards are often difficult to interpret and provide indirect control over specific contact-sequence structures. 

Our framework addresses multi-gait locomotion (walking-trot $\rightarrow$ trot $\rightarrow$ bound) by encoding desired behaviors as logical specifications. Rather than loosely coupled rewards, we utilize mode-conditioned Signal Temporal Logic (STL) templates to smoothly handle speed-dependent gait transitions and provide specification-level feedback for debugging multi-gait policies.

---

## ⚙️ Methodology

Our framework combines interpretable specification-based design with the scalability of deep RL. The reward component corresponds directly to human-readable requirements.

### 1. Feature Extraction
We compile trajectory datasets from specialized models corresponding to low-speed, mid-speed, and high-speed regimes. Extracted features include:
* **Tracking features:** Linear and angular velocities.
* **Safety/stability features:** Center of Mass (CoM), Base roll/pitch, and slip proxy.
* **Contact-pattern features:** Stride period, duty factor, and diagonal phase error.

### 2. Parametric STL (PSTL) Templates
We define fixed PSTL templates for three locomotion modes, fitting parameters using empirical quantiles from the expert datasets. 
* **Walk-Trot:** Characterized by support-rich diagonal locomotion with no flight.
* **Trot:** Characterized by dominant diagonal 2-contact support.
* **Bound:** High-speed pair-synchronized running where forelegs and hind legs move in phase. For example, the bound mode actively suppresses trot-like diagonal support patterns using the formula: $G_{\mathcal{W}_{B}}(p_{diag2} \le p_{diag2,max})$.

### 3. Hierarchical Reward Machine
The final reward is derived from the quantitative robustness of the active specification over a finite horizon. The active locomotion mode $g(t) \in \{W,T,B\}$ is selected dynamically based on the commanded forward velocity $v_{x}^{cmd}$. The scalar reward aggregates safety, tracking, and gait structure robustness alongside a torque-effort penalty.

---

## 📊 Experimental Results

The locomotion controller is trained using PPO within MuJoCo XLA (MJX), utilizing domain randomization over friction and actuator parameters to robustify the learned policies. 

### Velocity Tracking & Stability

**Table 3:** Benchmark comparison across commanded forward velocities. Each entry reports mean $\pm$ standard deviation over 20 rollouts. Lower CoT is better; higher survival and success are better. Success means the average post-warmup forward speed stays within $\pm$15% of the commanded speed.

| $v_x$ | STL CoT $\downarrow$ | STL Survival $\uparrow$ | STL Success $\uparrow$ | Default CoT $\downarrow$ | Default Survival $\uparrow$ | Default Success $\uparrow$ | Best CoT $\downarrow$ | Best Survival $\uparrow$ | Best Success $\uparrow$ |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **0.3** | 2.1 ± 0.1 | **1.0 ± 0.0** | **1.0 ± 0.0** | 2.1 ± 0.1 | **1.0 ± 0.0** | 0.0 ± 0.0 | **1.2 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** |
| **0.5** | 1.5 ± 0.0 | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.3 ± 0.0 | **1.0 ± 0.0** | 0.0 ± 0.0 | **1.0 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** |
| **0.7** | 1.2 ± 0.0 | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.1 ± 0.0 | **1.0 ± 0.0** | 0.1 ± 0.2 | **1.0 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** |
| **1.0** | 1.2 ± 0.0 | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.5 ± 0.0 | **1.0 ± 0.0** | 0.8 ± 0.4 | **1.2 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** |
| **1.3** | **1.1 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.3 ± 0.1 | **1.0 ± 0.0** | 0.3 ± 0.5 | 1.3 ± 0.0 | **1.0 ± 0.0** | **1.0 ± 0.0** |
| **1.6** | **1.0 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.2 ± 0.0 | 1.0 ± 0.2 | 0.3 ± 0.4 | 1.4 ± 0.0 | **1.0 ± 0.0** | **1.0 ± 0.0** |
| **1.9** | **1.1 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.2 ± 0.1 | 0.5 ± 0.5 | 0.0 ± 0.0 | 1.4 ± 0.0 | **1.0 ± 0.0** | **1.0 ± 0.0** |
| **2.0** | **1.1 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.3 ± 0.1 | 0.5 ± 0.5 | 0.0 ± 0.0 | 1.4 ± 0.0 | **1.0 ± 0.0** | 0.1 ± 0.2 |
| **2.1** | **1.1 ± 0.0** | **1.0 ± 0.0** | **1.0 ± 0.0** | 1.3 ± 0.1 | 0.3 ± 0.4 | 0.0 ± 0.0 | 1.4 ± 0.0 | **1.0 ± 0.0** | 0.0 ± 0.0 |

* **100% Survival & Success:** The STL-based reward maintains perfect survival and command-tracking success at every tested velocity (up to 2.1 m/s).
* **High-Speed Efficiency:** At speeds of 1.3 m/s and above, the STL policy achieves the lowest Cost of Transportation (CoT) by successfully transitioning into a mechanically suitable bound gait, whereas static heuristic baselines continue to force a less efficient trot.

---

## 🎥 Locomotion Regimes

> *[Developer Note: Replace the src paths below with the actual paths to your video files]*

<div align="center">

**Trot Gait ($v_{x} = 1.3$ m/s)**
<video src="assets/trot.mp4" controls autoplay loop muted width="80%"></video>

**Bound Gait ($v_{x} = 1.8$ m/s)**
<video src="assets/bound.mp4" controls autoplay loop muted width="80%"></video>

</div>

---

## 📝 Citation

```bibtex
@inproceedings{anonymous2026learning,
  title={Learning Gait-Aware Quadruped Locomotion with Temporal Logic Specifications},
  author={Anonymous Author(s)},
  booktitle={Submitted to the 10th Conference on Robot Learning (CoRL)},
  year={2026}
}
