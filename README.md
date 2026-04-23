# Rapid Motor Adaptation for Legged Robots
Trying to implement the following paper: https://arxiv.org/pdf/2107.04034 

## Installation Options
* [Method 1: Build from Scratch](#method-1-build-from-scratch)
* [Method 2: Build from Docker Hub](#method-2-build-from-docker-hub)

## Method 1: Build from Scratch

Steps taken from https://github.com/ut-amrl/social-isaacsim and compiled here 

### Clone these repos

```bash
cd ~
git clone https://github.com/robertschnurr/rma.git
git clone git@github.com:ut-amrl/IsaacLab.git
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

### Install the RMA packages and IsaacLab

```bash
cd /rma
/workspace/isaaclab/isaaclab.sh --install
pip install -e source/rma_utils/
pip install -e source/rma_assets/
pip install -e source/rma_tasks/
pip install -e source/rma_mdp/
```
Only needs to be completed in initial container build.

### Install the headless video recording packages

```bash
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y   xvfb x11-xserver-utils x11-apps   libgl1 libglvnd0 libegl1 libxext6 libsm6   mesa-utils ffmpeg
```
Only needs to be completed once per container build.

### Export necessary variables
CUDA_VISIBLE_DEVICES can be changed to whatever gpu you are trying to use. For all gpus, change the 0 to all.

```bash
export PYOPENGL_PLATFORM=egl
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export CUDA_VISIBLE_DEVICES=0
```

Also, export your WANDB username and key:
```bash
export WANDB_USERNAME=""
export WANDB_API_KEY=""
```
Required every time container is entered. 

### Start Headless videos

```bash
export DISPLAY=:99
Xvfb :99 -screen 0 1280x720x24 &
```
You can change DISPLAY to a different value if needed. 
Required for every time container is built. 

## Method 2: Build from Docker Hub
```bash
cd ~
git clone https://github.com/robertschnurr/rma.git
docker pull rschnurr/vision_rma:latest
podman run -it \
  --security-opt=label=disable \
  --device nvidia.com/gpu=all \
  --network host \
  --ipc=host \
  -e CUDA_VISIBLE_DEVICES=7\
  -e WANDB_USERNAME="" \
  -e WANDB_API_KEY="" \
  -v ~/rma:/rma \
  -w /rma \
  --entrypoint /bin/bash \
  rschnurr/vision_rma:latest \
  -lc '
    nvidia-smi &&
    /isaac-sim/python.sh -m pip install -e source/rma_mdp &&
    /isaac-sim/python.sh -m pip install -e source/rma_tasks &&
    /isaac-sim/python.sh -m pip install -e source/rma_assets &&
    /isaac-sim/python.sh -m pip install -e source/rma_utils &&
    echo "Container ready." &&
    exec /bin/bash
  '
```
Fill in the required WANDB username and API keys. You can also change the CUDA_VISIBLE_DEVICES if needed. 

Export the required variables for video recording: 
```bash
export PYOPENGL_PLATFORM=egl
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export DISPLAY=:99
Xvfb :99 -screen 0 1280x720x24 &
```
Can change DISPLAY as needed. 

## Training Commands
Ideally, run the container in a tmux session so you can detach as needed for longer training sessions. All commands run from the /rma directory. 
Inside the container run:
For a quick test of WANDB, video recording, and IsaacLab enviornment.
```bash
python scripts/train.py --task RMA1-Spot-v0 --headless --video --video_interval 1 --max_iterations 5 --wandb
```

For Training of the base policy.
```bash
python train.py --task RMA1-Spot-v0 --headless --video --video_interval 500 --max_iterations 5000 --wandb
```

For training of the adaptation policy.
```bash
python scripts/train_adaptation.py --task RMA2-Spot-v0 --actor_model wandb --wandb_run "" --wandb_model "model_4999.pt" --headless --max_iterations 1500 --video --video_interval 100
```
Fill in the wandb_run and wandb_model with the wandb run path and the correct .pt model. You can supply a path to a .pt file in the --actor_model flag instead of using wandb if desired.

For recording videos of the adaptation policy. 
```bash
python scripts/play.py   --task RMA2-Spot-v0   --checkpoint /rma/logs/rsl_rl/spot_rma/2026-03-29_12-21-51/model_1500.pt   --teacher_model /rma/logs/rsl_rl/spot_rma/2026-03-29_02-27-22/model_4999.pt   --video   --video_length 500   --headless   --num_envs 200   --x 0.0 --y 0.0 --z 1.0   --origin_type world
```
Replace the checkpoint and teacher model with the paths to your adaptation and base policies. You can also change the position and origin of the camera.

### References

Instructions for building from scratch compiled from the following sources in the ut-amrl/social-isaacsim repo. There are more details about installation there as well as instructions for streaming. [Shuopu Wang](https://github.com/Shuopu-Wang) created the commands for pulling from docker hub. 

Install IsaacLab per: https://github.com/ut-amrl/social-isaacsim/blob/master/docs/SETUP_ISAACLAB.md

Setup Headless Video: https://github.com/ut-amrl/social-isaacsim/blob/master/docs/SETUP_HEADLESS_VIDEO.md
