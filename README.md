# Rapid Motor Adaptation for Legged Robots
Trying to implement the following paper: https://arxiv.org/pdf/2107.04034

## Installation

Install IsaacLab per: https://github.com/ut-amrl/social-isaacsim/blob/master/docs/SETUP_ISAACLAB.md

Setup Headless Video: https://github.com/ut-amrl/social-isaacsim/blob/master/docs/SETUP_HEADLESS_VIDEO.md

### Clone this repo

```bash
cd ~
git clone https://github.com/robertschnurr/rma.git
```

### Copy the rma mounting config into your Isaac installation

```bash
cp rma/docker/rma.yaml IsaacLab/docker/
```

### Build the container

```bash
cd ~/IsaacLab/
./docker/container.py start ros2 --files "rma.yaml"
```

### Enter the container

```bash
cd ~/IsaacLab/
./docker/container.py enter ros2
```

### Install the RMA packages


```bash
cd /rma
pip install -e source/rma_utils/
pip install -e source/rma_assets/
pip install -e source/rma_tasks/
pip install -e source/rma_mdp/
```
