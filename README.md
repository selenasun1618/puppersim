# puppersim
Simulation and Reinforcement Learning for DJI Pupper v2 robot


## Conda setup link
Install [conda](https://docs.conda.io/en/latest/miniconda.html), then
```
conda create --name rl_pupper python=3.7
conda activate rl_pupper
pip install ray arspb
```

## Getting the code ready
```
git clone https://github.com/jietan/puppersim.git
cd puppersim
pip install -e .         (there is a dot at the end)
python3 puppersim/pupper_example.py       (this verifies the installation, you should see pybullet window show up with a pupper in it)
```

If pupper_example.py is running very slowly, try
```bash
python3 puppersim/pupper_minimal_server.py
```
then in a new terminal tab/window
```bash
python3 puppersim/pupper_example.py --render=False
```
This runs the visualizer GUI and simulator as two separate processes.

## Training
```
ray start --head
python3 puppersim/pupper_ars_train.py --rollout_length=200
ray stop (after training is completed)
```

### Troubleshooting
* **Pybullet hangs when starting training**. Possible issue: You have multiple suspended pybullet clients. Solution: Restart your computer. 
* **ConnectionRefusedError: [Errno 61] Connection refused**. Issue: Your ray server is running on a different IP address than the training script expects. Solution: Look at the output of `ray start --head` for the address of the Ray runtime. It'll say something like `To connect to this Ray runtime from another node, run ray start --address='x.x.x.x:xxxx'`. Use the address provided and add the argument `--redis_address=[x.x.x.x:xxxx` when running `pupper_ars_train.py`.

### Guidelines for saving policies
If you want to save a policy, create a folder within `puppersim/data` with the type of gait and date, eg `pretrained_trot_1_22_22`. From the `data` folder, copy the following files into the folder you just made.


* The `.npz` policy file you want, e.g. `lin_policy_plus_latest.npz`
* `log.txt`
* `params.json`

From `puppersim/config` also copy the `.gin` file you used to train the robot, e.g. `pupper_pmtg.gin` file into the folder you just made. When you run a policy on the robot, make sure your `pupper_robot_*_.gin` file matches the `pupper_pmtg.gin` file you saved.

Then add a `README.md` in the folder with a brief description of what you did, including your motivation for saving this policy. 


## Test an ARS policy during training (file location may be different)
```
python3 puppersim/pupper_ars_run_policy.py  --expert_policy_file  data/lin_policy_plus_latest.npz  --json_file data/params.json --render
```

## Prerequisites before deployment to Pupper

Set up Avahi (for linux only, one time only per computer)
```
sudo apt install avahi-* (one time)
```
Run the following, you should see ip address of pupper
```
avahi-resolve-host-name raspberrypi.local -4
```
Setup the zero password login for your pupper (Original password on raspberry pi: raspberry)

One time only per computer, run
```
ssh-keygen
```
One time only per pupper, run
* Linux
```
cat ~/.ssh/id_rsa.pub | ssh pi@`avahi-resolve-host-name raspberrypi.local -4 | awk '{print $2}'` 'mkdir .ssh/ && cat >> .ssh/authorized_keys'
```
* MacOs
```
cat ~/.ssh/id_rsa.pub | ssh pi@raspberrypi.local 'mkdir -p .ssh/ && cat >> .ssh/authorized_keys'
```

## Run a pretrained policy on the Pupper robot
* Turn on the Pupper robot, wait for it to complete the calibration motion.
* Connect your laptop with the Pupper using an USB-C cable
* Run the following command on your laptop:
```
./deploy_to_robot.sh python3 puppersim/puppersim/pupper_ars_run_policy.py --expert_policy_file=puppersim/data/lin_policy_plus_latest.npz --json_file=puppersim/data/params.json --run_on_robot
```

## Using the heuristic control
Navigate to the outer puppersim folder and run
```bash
python3 puppersim/pupper_server.py
```

Clone the the [heuristic controller](https://github.com/stanfordroboticsclub/StanfordQuadruped.git):
```bash
git clone https://github.com/stanfordroboticsclub/StanfordQuadruped.git
cd StanfordQuadruped
git checkout dji
```
The `dji` branch is checked out so you can use the version of code for Pupper V2 rather than the servo-based Pupper V1.

In a separate terminal, navigate to StanfordQuadruped and run 
```bash
python3 run_djipupper_sim.py
```

Keyboard controls:
* wasd: left joystick --> moves robot forward/back and left/right
* arrow keys: right joystick --> turns robot left/right
* q: L1 --> activates/deactivates robot
* e: R1 --> starts/stops trotting gait
* ijkl: d-pad
* x: X
* square: u
* triangle: t
* circle: c
