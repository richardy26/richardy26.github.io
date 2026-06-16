<div align="center">

<h1>Learning Gait-Aware Quadruped Locomotion with Temporal Logic Specifications</h1>

<p><b>Anonymous Author(s)</b></p>

<p>Submitted to the 10th Conference on Robot Learning (CoRL 2026)</p>

<img src="assets/quadruped%20model.png" width="500" alt="Barkour Quadruped Model">

</div>

---

## Abstract

Reinforcement learning (RL) for quadruped locomotion commonly relies on fixed, hand-crafted, and Markovian reward functions, which limit the interpretability of learned policies and provide no explicit control over gait behaviors. We introduce a framework in which distinct gaits are specified using parameterized constraints expressed in Signal Temporal Logic (STL), including safety bounds, gait synchronization constraints, command tracking objectives, and actuation limits. From these specifications, we develop a reward-shaping mechanism that provides learning agents with a dense and continuous reward landscape that encodes the desired behavior. We define parametric STL templates for three speed regimes—walking-trot, trot, and bound—calibrate their parameters from reference rollouts, and compute rewards using smooth approximations of STL robustness over these rollouts. The resulting rewards provide shaped gradients that are compatible with Proximal Policy Optimization (PPO). We instantiate the approach on Google’s Barkour quadruped robot in MuJoCo XLA (MJX), leveraging simulator parallelization to accelerate training and domain randomization to improve policy robustness. Experimental results show that, compared to conventional hand-crafted rewards, STL-shaped rewards achieve tighter velocity tracking and more stable training performance.

---

## Introduction

Classical model-based control approaches, such as differential dynamic programming (DDP) and model predictive control (MPC), enable impressive behaviors but rely on accurate system models and complex cost functions. Deep RL pipelines address command tracking and stability through engineered rewards, curriculum learning, and domain randomization, but these rewards are often difficult to interpret and provide indirect control over specific contact-sequence structures. 

Our framework addresses multi-gait locomotion (walking-trot → trot → bound) by encoding desired behaviors as logical specifications. Rather than loosely coupled rewards, we utilize mode-conditioned Signal Temporal Logic (STL) templates to smoothly handle speed-dependent gait transitions and provide specification-level feedback for debugging multi-gait policies.

---

## Methodology

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
* **Bound:** High-speed pair-synchronized running where forelegs and hind legs move in phase.

### 3. Hierarchical Reward Machine
The final reward is derived from the quantitative robustness of the active specification over a finite horizon. The active locomotion mode g(t) &isin; {W, T, B} is selected dynamically based on the commanded forward velocity v<sub>x</sub><sup>cmd</sup>. The scalar reward aggregates safety, tracking, and gait structure robustness alongside a torque-effort penalty.

---

## Experimental Results

The locomotion controller is trained using PPO within MuJoCo XLA (MJX), utilizing domain randomization over friction and actuator parameters to robustify the learned policies. 

### Velocity Tracking & Stability

Benchmark comparison across commanded forward velocities. Each entry reports mean &plusmn; standard deviation over 20 rollouts. Lower CoT is better; higher survival and success are better. Success means the average post-warmup forward speed stays within 15% of the commanded speed.

<table>
  <thead>
    <tr>
      <th rowspan="2">v<sub>x</sub></th>
      <th colspan="3" align="center">STL-based reward</th>
      <th colspan="3" align="center">Heuristic-default</th>
      <th colspan="3" align="center">Heuristic-best</th>
    </tr>
    <tr>
      <th align="center">CoT &darr;</th>
      <th align="center">Survival &uarr;</th>
      <th align="center">Success &uarr;</th>
      <th align="center">CoT &darr;</th>
      <th align="center">Survival &uarr;</th>
      <th align="center">Success &uarr;</th>
      <th align="center">CoT &darr;</th>
      <th align="center">Survival &uarr;</th>
      <th align="center">Success &uarr;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center">0.3</td>
      <td align="center">2.1 &plusmn; 0.1</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">2.1 &plusmn; 0.1</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">0.0 &plusmn; 0.0</td>
      <td align="center"><b>1.2 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
    </tr>
    <tr>
      <td align="center">0.5</td>
      <td align="center">1.5 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.3 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">0.0 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
    </tr>
    <tr>
      <td align="center">0.7</td>
      <td align="center">1.2 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.1 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">0.1 &plusmn; 0.2</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
    </tr>
    <tr>
      <td align="center">1.0</td>
      <td align="center">1.2 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.5 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">0.8 &plusmn; 0.4</td>
      <td align="center"><b>1.2 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
    </tr>
    <tr>
      <td align="center">1.3</td>
      <td align="center"><b>1.1 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.3 &plusmn; 0.1</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">0.3 &plusmn; 0.5</td>
      <td align="center">1.3 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
    </tr>
    <tr>
      <td align="center">1.6</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.2 &plusmn; 0.0</td>
      <td align="center">1.0 &plusmn; 0.2</td>
      <td align="center">0.3 &plusmn; 0.4</td>
      <td align="center">1.4 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
    </tr>
    <tr>
      <td align="center">1.9</td>
      <td align="center"><b>1.1 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.2 &plusmn; 0.1</td>
      <td align="center">0.5 &plusmn; 0.5</td>
      <td align="center">0.0 &plusmn; 0.0</td>
      <td align="center">1.4 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
    </tr>
    <tr>
      <td align="center">2.0</td>
      <td align="center"><b>1.1 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.3 &plusmn; 0.1</td>
      <td align="center">0.5 &plusmn; 0.5</td>
      <td align="center">0.0 &plusmn; 0.0</td>
      <td align="center">1.4 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">0.1 &plusmn; 0.2</td>
    </tr>
    <tr>
      <td align="center">2.1</td>
      <td align="center"><b>1.1 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">1.3 &plusmn; 0.1</td>
      <td align="center">0.3 &plusmn; 0.4</td>
      <td align="center">0.0 &plusmn; 0.0</td>
      <td align="center">1.4 &plusmn; 0.0</td>
      <td align="center"><b>1.0 &plusmn; 0.0</b></td>
      <td align="center">0.0 &plusmn; 0.0</td>
    </tr>
  </tbody>
</table>

<br>

* **100% Survival & Success:** The STL-based reward maintains perfect survival and command-tracking success at every tested velocity (up to 2.1 m/s).
* **High-Speed Efficiency:** At speeds of 1.3 m/s and above, the STL policy achieves the lowest Cost of Transportation (CoT) by successfully transitioning into a mechanically suitable bound gait, whereas static heuristic baselines continue to force a less efficient trot.

---

## Locomotion Regimes

<div align="center">

**Trot Gait (v<sub>x</sub> = 1.3 m/s)**
<video src="assets/trot.mp4" controls autoplay loop muted width="80%"></video>

**Bound Gait (v<sub>x</sub> = 1.8 m/s)**
<video src="assets/bound.mp4" controls autoplay loop muted width="80%"></video>

</div>

---

## Citation
