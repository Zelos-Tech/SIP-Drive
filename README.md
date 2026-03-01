# SIP-Drive: Self-Improving Policy for Driving

> **Human Expert Data is not Necessary for Autonomous Driving**

SIP-Drive is a novel autonomous driving planning framework that achieves high-performance trajectory planning **without requiring any human expert driving data**. By leveraging a dual-model architecture and iterative self-improvement, SIP-Drive enables robust planning for specialized vehicles (e.g., robovans) where expert demonstrations are scarce or impractical to collect.

---

## 🎬 Visual Results

### Framework Overview
![SIP-Drive Framework](assets/framework.pdf)
*Overview of the SIP-Drive framework showing dual-model architecture, self-improving pre-training, RL fine-tuning, and closed-loop iteration.*

### Trajectory Comparison
![Trajectory Comparison](assets/trajectory_comparison.pdf)
*Comparison of closed-loop trajectories between human expert (top) and SIP-Drive (bottom). Green denotes ego vehicle's planned trajectory, while blue and violet represent trajectories of different types of obstacles.*

### Real-World Deployment
![Real-World Deployment](assets/real_world.pdf)
*SIP-Drive's trajectory execution in real-world deployment across critical driving scenarios including cut-in, nudge maneuvers, intersection crossing, and merge operations.*

---

## 🌟 Key Features

- **Expert-Free Training**: Eliminates dependency on human expert demonstrations by using rule-based planners as initial supervision.
- **Self-Improving Supervision**: Dynamically disables imitation loss when model outputs surpass rule-based trajectories under a shared reward function.
- **Iterative Closed-Loop Optimization**: Enables continuous performance gains through a self-boosting cycle where the learned policy progressively replaces the rule-based planner.
- **Dual-Model Architecture**: Separates surrounding-agent behavior prediction (frozen world model) from ego-vehicle planning for stable multi-agent interaction modeling.
- **Real-World Deployment**: Validated on large-scale autonomous delivery fleets, supporting hundreds of kilometers of daily driverless operation.

---

## 📦 Installation

### 1. Create Conda Environment

```bash
conda create -n SipDrive python=3.9
conda activate SipDrive
```

### 2. Install nuPlan DevKit

```bash
git clone https://github.com/motional/nuplan-devkit.git
cd nuplan-devkit
pip install -e .
pip install -r ./requirements.txt
cd ..
```

### 3. Install SIP-Drive

```bash
git clone https://github.com/Zelos-Tech/SIP-Drive.git
cd SIP-Drive
pip install -r requirements.txt
```

### 4. Environment Configuration

Set up nuPlan data paths and environment variables:

```bash
# Example: Add to ~/.bashrc or set manually
export NUPLAN_DATA_ROOT=/path/to/nuplan/data
export NUPLAN_MAPS_ROOT=/path/to/nuplan/maps
export NUPLAN_DEVKIT_ROOT=/path/to/nuplan-devkit
```

---

## 🗂️ Data Preparation

### Download nuPlan Dataset

Follow the official [nuPlan dataset download instructions](https://www.nuplan.org/dataset) to obtain the dataset.

### Preprocess Dataset

```bash
python preprocess_dataset.py \
    --data_root /path/to/nuplan/data \
    --output_dir ./data/processed \
    --split train  # or val/test
```

---

## 🚀 Training Pipeline

SIP-Drive follows a three-stage training procedure:

### Stage 1: World Model Training

Train the surrounding-agent behavior prediction model:

```bash
python train.py --config config/train/World.yaml
```

### Stage 2: Ego Planner Pre-Training

Initialize ego planner with rule-based trajectories and apply self-improving supervision:

```bash
python train.py --config config/train/PreTraining.yaml
```

### Stage 3: Reinforcement Learning Fine-Tuning

Further optimize the policy via rule-based reward-guided RL:

```bash
python train.py --config config/train/Rl.yaml
```

### Iterative Closed-Loop Refinement (Optional)

To enable iterative self-improvement, update the configuration to use the latest policy as the new supervision source and repeat Stages 2–3. Typically 2–4 iterations yield convergence.

---

## 🧪 Simulation & Evaluation

### Configure Simulation

Edit `sim_sip_drive_runner.sh` to set:

- Evaluation benchmark (`Val14`, `Test14`, `Test14-hard`)
- Simulation mode (`non-reactive` / `reactive`)
- Checkpoint path and hyperparameters

### Run Closed-Loop Simulation

```bash
bash sim_sip_drive_runner.sh
```
---



## 📊 Results

SIP-Drive achieves competitive performance on the nuPlan benchmark. Below is the complete comparison with state-of-the-art methods:

> **NR**: Non-reactive simulation mode | **R**: Reactive simulation mode (IDM-based)  
> **\***: With standard refinement (rule-based initialization + iterative optimization)  
> **Bold**: Best score in each column

| Type | Planner | Val14-NR | Val14-R | Test14-hard-NR | Test14-hard-R | Test14-random-NR | Test14-random-R |
|------|---------|----------|---------|----------------|---------------|------------------|-----------------|
| **Expert** | Log-Replay | 93.53 | 80.32 | 85.96 | 68.80 | 94.03 | 75.86 |
| **Rule-based & Hybrid** | | | | | | | |
| | IDM | 75.60 | 77.33 | 56.15 | 62.26 | 70.39 | 72.42 |
| | PDM-Closed* | 92.84 | 92.12 | 65.08 | 75.19 | 90.05 | 91.64 |
| | PDM-Hybrid* | 92.77 | 92.11 | 65.99 | 76.07 | 90.10 | 91.28 |
| | GameFormer* | 79.94 | 79.78 | 68.70 | 67.05 | 83.88 | 82.05 |
| | PLUTO* | 92.88 | 89.84 | **80.08** | 76.88 | 92.23 | 90.29 |
| | PlanAgent* | 93.26 | 92.75 | 72.51 | 76.82 | – | – |
| | Diffusion Planner* | 94.26 | 92.90 | 78.87 | **82.00** | **94.80** | 91.75 |
| | CarPlanner* | – | – | – | – | 94.07 | 91.10 |
| | Plan-R1* | **94.72** | **93.54** | 78.46 | 81.70 | 94.64 | **93.71** |
| | **SIP-Drive* (ours)** | 93.81 | **93.49** | **78.54** | **80.71** | 92.93 | **92.85** |
| | *% of SOTA* | 99.04% | 99.94% | 97.20% | 98.41% | 98.03% | 99.08% |
| **Learning-based** | | | | | | | |
| | UrbanDriver | 68.57 | 64.11 | 50.40 | 49.95 | 51.83 | 67.15 |
| | PDM-Open | 53.53 | 54.24 | 33.51 | 35.83 | 52.81 | 57.23 |
| | PlanTF | 84.27 | 76.95 | 69.70 | 61.61 | 85.62 | 79.58 |
| | PLUTO | 88.89 | 78.11 | 70.03 | 59.74 | 89.90 | 78.62 |
| | Diffusion Planner | **89.87** | 82.80 | 75.99 | 69.22 | 89.19 | 82.93 |
| | Plan-R1 | 88.98 | **87.69** | **77.45** | **77.20** | **91.23** | **90.04** |
| | **SIP-Drive (ours)** | 85.20 | 83.62 | 74.11 | 73.53 | 87.88 | 86.44 |
| | *% of SOTA* | 95.12% | 95.35% | 95.68% | 95.24% | 96.33% | 96.00% |

---


## 🙏 Acknowledgements

We thank the following open-source projects for their contributions and inspiration to this work:

- [nuPlan](https://github.com/motional/nuplan-devkit) – Large-scale simulation benchmark and dataset for autonomous driving planning.
- [Diffusion Planner](https://github.com/ZhengYinan-AIR/Diffusion-Planner) – Transformer-based diffusion model for joint prediction and planning.
- [Plan-R1](https://github.com/XiaolongTang23/Plan-R1) – Two-stage trajectory planning framework with rule-based reward fine-tuning.
- [PlanTF](https://github.com/jchengai/planTF) – Pure imitation learning-based planning framework without hand-crafted rules.

We also thank the nuPlan team and the broader autonomous driving research community for fostering open collaboration and advancing the field.

---
