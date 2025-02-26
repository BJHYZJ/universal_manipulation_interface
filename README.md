# UMI-ARX Cup Task Deployment Tutorial

# Diffusion Policy Setup
Please follow the instructions in [README_original.md] to install the environment

Then you can setup arx5 for umi.

1. Download `detached-umi-policy`
```bash
curl -o detached_policy_inference.py https://raw.githubusercontent.com/real-stanford/detached-umi-policy/main/detached_policy_inference.py

mkdir data && cd data && mkdir models && mkdir experiments
```

2. Download checkpoint and put it into data/models
```bash
cd ../
wget https://real.stanford.edu/umi/data/pretrained_models/cup_wild_vit_l_1img.ckpt && mv cup_wild_vit_l_1img.ckpt data/models
```

3. Test whether diffusion policy is successfully installed
```bash
python detached_policy_inference.py -i data/models/cup_wild_vit_l_1img.ckpt
```
After PolicyInferenceNode is listening on 0.0.0.0:8766, the policy inference process is successfully set up. Keep it running in the background when running umi code.


# ARX5 Robot Arm Setup
1. GitHub - real-stanford/arx5-sdk at yihuai
```bash
git clone git@github.com:real-stanford/arx5-sdk.git
cd arx5-sdk
mkdir build && cd build
cmake .. && make -j
```

2. Setup robot arm connection and spacemouse service following instructions in [arx5-sdk/README.md](arx5-sdk/README.md) of arx5-sdk repository (⚠️ Please note that the `umi` environment already contains all the required configurations for arx, you do not need to reinstall the `arx-py310` environment). 

3. When turning on the ARX5 arm, make sure the gripper is fully closed

4. Test spacemouse cartesian space control. After modifying the ARX robot arm, YOUR_MODEL should be either `X5_umi` or `L5_umi`

```bash
# activate can interface: sudo slcand -o -f -s8 /dev/arxcan0 can0 && sudo ifconfig can0 up
cd python
python examples/spacemouse_teleop.py YOUR_MODEL YOUR_CAN_INTERFACE
# python examples/spacemouse_teleop.py L5 can0
```

5. Calibrate gripper (after installing the fin-ray gripper fingers)
    
```bash
python examples/calibrate.py YOUR_MODEL YOUR_CAN_INTERFACE
# python examples/calibrate.py L5 can0
```

6. Calibrate gripper (after installing the fin-ray gripper fingers)
    
```bash
python examples/calibrate.py YOUR_MODEL YOUR_CAN_INTERFACE
```
This server will keep running in the background by default on `0.0.0.0:8765` . Robot arm will set to home pose then passive if no commands are sent to the server in 60 seconds to avoid overheating.


# UMI Minimal Deployment for ARX Robot Arm
1. Clone the github repository https://github.com/real-stanford/umi-arx
```bash
cd ../../  # cd to */universal_manipulation_interface/
git clone https://github.com/real-stanford/umi-arx.git
cd umi-arx
```

2. Check camera connection: there should be a blue circle with an hdmi icon on the front screen of gopro


4. Run deployment code
- First make sure the policy inference node and arx5 server are already turned on
- After the elgato capture card is connected, run the following line:
    
```bash
sudo chmod -R 777 /dev/bus/usb
```
1. Check hardware connection:
```bash
sudo slcand -o -f -s8 /dev/arxcan0 can0 && sudo ifconfig can0 up
```

2. USB permission:
```bash
sudo chmod -R 777 /dev/bus/usb
```

3. Policy inference node (In script 1):
```bash
conda activate umi
# python detached_policy_inference.py -i data/models/cup_wild_vit_l_1img.ckpt
python detached_policy_inference.py -i data/models/cup_wild_vit_l_1img.ckpt
```

4. ARX5 communication server (In script 2):
    
```bash
cd arx5-sdk && conda activate umi
# python python/communication/zmq_server.py YOUR_MODEL YOUR_CAN_INTERFACE
python python/communication/zmq_server.py L5 can0
```

5. Policy deployment code  (In script 3):
    
```bash
cd umi-arx && conda umi
pkill -f eval_arx5
# python scripts/eval_arx5.py -i YOUR_PATH_TO/detached-umi-policy/data/models/cup_wild_vit_l_1img.ckpt -o data/experiments/DATE --no_mirror
python scripts/eval_arx5.py -i /home/yanzj/workspace/code/universal_manipulation_interface/data/models/cup_wild_vit_l_1img.ckpt -o /home/yanzj/workspace/code/universal_manipulation_interface/data/experiments/DATE --no_mirror
```
    
- Use spacemouse to set the gripper to an initial pose. The initial pose should be roughly $15^\circ$ over the table.
- Press `c` to start policy inference, `s` to stop. Stop the policy immediately after the cup is put on the dish, otherwise arx arm may easily go into singularity state and is not safe.
- When this program is running, it will monitor all your keyboard movements, even if the cursor is not in the terminal. Stop the program if you want to type anything else.
- After a few rounds of policy running, press `q` to stop


# Reference
- [UMI-ARX-Cup-Task-Deployment-Tutorial](https://yihuai.notion.site/UMI-ARX-Cup-Task-Deployment-Tutorial-112e4c9a5c4980f1ac20fe4da261d9a1)