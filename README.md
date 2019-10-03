## Software 

### Camera Setup 

Before running the Docker image, the camera drivers for the SR300 Intel Realsense camera need to be set up. Detailed instructions for setting up the drivers can be found here. 

###  Docker Image 
Pull our Docker image for operating a cell directly with:  
```
docker pull bhyang12/replab 
```

###  To run the image: 
```
docker run -it --rm --privileged bhyang12/replab 
```
If operating multiple rigs on one machine, you may need to manually specify which ports/devices the Docker container can access. As an example: 
```
docker run -it --rm --device=/dev/video0 --device=/dev/video1 --device=/dev/ttyUSB0 bhyang12/replab 
```
### To run with GPU and display port access (requires nvidia-docker 2), use: 
```
docker run --runtime=nvidia -e NVIDIA_DRIVER_CAPABILITIES=compute,utility -e NVIDIA_VISIBLE_DEVICES=all -it --net=host --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" --rm --privileged bhyang12/replab 
```

## Reinforcement Learning on REPLAB

This is the code for training and evaluating RL algorithms on REPLAB cells. This is heavily based off of RLkit, available here: https://github.com/vitchyr/rlkit, and a modified Viskit repo, available here: https://github.com/vitchyr/viskit.


### Directory Structure
Files for Reinforcement Learning on REPLAB are located in two places: ``/root/ros_ws/rl_scripts/`` and ``/root/ros_ws/src/replab_rl/``. 

``/root/ros_ws/src/replab_rl/`` contains two folders, ``gym-replab``, a pip package that has the OpenAI Gym Environment for REPLAB and ``src``, which contains a file that allows us to communicate with ROS through Python3. The actual RL scripts are located in ``/root/ros_ws/rl_scripts/``, which contains two folders: ``rlkit`` and ``viskit``. ``rlkit`` contains the base code from the RLkit repository, and modified example scripts for both fixed and randomized reaching tasks. ``viskit`` contains code from the Viskit repository.


### Prerequisites

To train or evaluate a model on a REPLAB cell, you must first run two scripts in the docker container.

In one window, run 

``` sh /root/ros_ws/start.sh ``` to start communicating with the MoveIt commander.

In another, run

```rosrun replab_rl replab_env_subscriber.py``` to enable the Python3 environment to communicate with the Python2 ROS build.

### Training the models

The task is to reach a point in 3d space through controlling the 6 joints of the arm.

There are 4 examples in ``/root/ros_ws/rl_scripts/rlkit/examples``, three is designed for a fixed goal (``{td3, sac, ddpg}.py``) and the other is designed for a randomized goal (``her_td3_gym_fetch_reach.py``).

By default, all of these use the GPU. If you aren't running this with a GPU, please change ``ptu.set_gpu_mode(True)`` to ``ptu.set_gpu_mode(False)`` near the bottom of the example files.

To get started, run


```
cd /root/ros_ws/rl_scripts/rlkit/

source activate rlkit

python examples/[EXAMPLE_FILE].py
```

For each of these example scripts, the parameters and hyperparameters are easily adjustable by modifying them directly in the file.

For the fixed goal environments, you can modify the fixed goal by directly modifying ``/root/ros_ws/src/replab_rl/gym_replab/gym_replab/envs/replab_env.py``


### Visualizing results

RLkit recommends Viskit to visualize the results. To view them, run:

```
source activate rlkit

python /root/ros_ws/rl_scripts/viskit/viskit/frontend.py [DATA_DIRECTORY]
```

Then, in your browser, navigate to the IP address of the docker container and the port listed by viskit.

Note: by default, the data directory containing parameters and stats are saved in ``/root/ros_ws/rl_scripts/rlkit/data/[NAME]/[DATE_TIME]``

### Evaluating a Policy

The policy is evaluated at every epoch during training (and this data is saved), however you can also manually evaluate a saved policy.

Example scripts for evaluating a policy are in ``/root/ros_ws/rl_scripts/rlkit/scripts``. For example, to visualize a policy on the real robot, run:

```
cd /root/ros_ws/rl_scripts/rlkit

source activate rlkit

python scripts/[POLICY_SCRIPT].py --[args specified in script] [path_to_params.pkl]
```
