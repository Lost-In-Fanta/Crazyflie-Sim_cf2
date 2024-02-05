# Configuration Guide of ROS2 Gazebo Simulation for the Crazyflie 

This is a fully operational step-by-step guide contains all the details to install and fulfill the function of *sim_cf2* from https://github.com/CrazyflieTHI/sim_cf2.

Sim_cf2 is based on the Gazebo Simulation for Crazyflie CRTP *sim_cf* from https://github.com/wuwushrek/sim_cf

[//]: # (It features)

[//]: # ()
[//]: # (* Full software-in-the-loop &#40;SITL&#41; Crazyflie firmware implementation utilizing the FreeRTOS Linux Port https://www.freertos.org/FreeRTOS-simulator-for-Linux.html#SimulatorApp)

[//]: # (* Compatibility with the Crazyflie python library &#40;cflib&#41;)

[//]: # (* Integration of the Crazyflie firmware for SITL build process into KBuild)

[//]: # (* Build for ROS2 and Gazebo 11)

[//]: # ()
[//]: # (In contrast to the original *sim_cf*, hardware-in-the-loop &#40;HITL&#41; is not supported.)

[//]: # ()
[//]: # (Beside this ROS2 package you will need)

[//]: # ()
[//]: # (* Crazyflie firmware with modifications for SITL https://github.com/CrazyflieTHI/crazyflie-firmware)

[//]: # (* Crazyflie python library with additional *simlink* driver https://github.com/CrazyflieTHI/crazyflie-lib-python)
## Installation - Ubuntu 22.04 with python 3.10
Tested on an Ubuntu 22.04 LTS machine

### Install ROS2 humble
https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html

Note: Other versions of ROS 2 should be available, but here I only test with ROS2-humble.
### ROS2 humble installation test
Use the following command to test whether ros2 can be used properly.
```sh
source /opt/ros/humble/setup.bash
ros2  --help
```
### Install gazebo11
https://classic.gazebosim.org/tutorials?tut=install_ubuntu

Install
```sh
curl -sSL http://get.gazebosim.org | sh
```
Run
```sh
gazebo
```
### Install crazyflie-lab
https://github.com/CrazyflieTHI/crazyflie-lib-python/blob/master/docs/installation/install.md

Then install dependencies following:

https://github.com/CrazyflieTHI/crazyflie-lib-python

Clone *crazyflie-lib-python* to local
```sh
git clone https://github.com/CrazyflieTHI/crazyflie-lib-python.git
```

### Install Crazyflie Firmware for sim_cf2 SITL 
Clone *Crazyflie Firmware* to local
```sh
git clone https://github.com/CrazyflieTHI/crazyflie-firmware.git
```
Install dependencies https://github.com/CrazyflieTHI/crazyflie-firmware

The firmware for SITL is built using the systems default C/C++ compiler. KBuild requires some additional packages
```sh
sudo apt install make build-essential libncurses5-dev
```
Make sure all submodules are up-to-date
```sh
cd ~/crazyflie-firmware
git submodule update --init --recursive
```
Install GSL
Download package from https://ftp.gnu.org/gnu/gsl/?C=M;O=D
```sh
tar -zxvf gsl-latest.tar.gz 
cd gsl-2.7.1
```
Follow https://github.com/CrazyflieTHI/crazyflie-firmware to finish building for SITL.

Notice:

>Encountered a very rare error：Your display is too small to run Menuconfig!

Solution: 

>Expand terminal window。

### Install Dependencies for *sim_cf2*

Basic dependencies

```sh
sudo apt-get install cmake build-essential genromfs ninja-build
```

Protobuf, eigen3 and google-glog dependencies

```sh
sudo apt-get install protobuf-compiler libgoogle-glog-dev libeigen3-dev libxml2-utils
```

Cyclone DDS
```sh
sudo apt install ros-humble-rmw-cyclonedds-cpp
```

Xacro library
```sh
sudo apt install ros-humble-xacro
```
### Create ros2 workspace
```sh
mkdir -p dev_ws/src
```

### Additional ROS2 Packages

Clone ROS2 version of *mav_comm* into your colcon overlay (assuming ~/dev_ws)

```sh
cd ~/dev_ws/src
```

```sh
git clone https://github.com/CrazyflieTHI/mav_comm.git
```

### Build the Packages

Move into your colcon overlay (assuming ~/dev_ws) and use colcon build

```sh
source /opt/ros/humble/setup.bash
cd ~/dev_ws
```
Clone *sim_cf2* to local workspace
```sh
git clone https://github.com/CrazyflieTHI/sim_cf2.git
```

```sh
colcon build
```
Notice:

>Encountered error:
>
>Could not find a package configuration file provided by "ament_cmake" with any of the following names:
>
>ament_cmakeConfig.cmake 
>
>ament_cmake-config.cmake

Solution:

Not sure which one fixed this error.
>source /opt/ros/humble/setup.bash
>
>sudo apt-get install ros-humble-ros-ign-bridge
>
>sudo apt install ros-humble-gazebo-ros-pkgs

Update: I believe following command can fix this error.
```sh
sudo apt install ros-humble-gazebo-ros-pkgs
```

## Setting Up the Simulation

Running the simulation requires

* *sim_cf2* package built
* Crazyflie firmware compiled for SITL
* Crazyflie python library installed containing the *simlink* driver

### Adjust the Launch Script

The number of simulated Crazyflies is determined by the main launch file *main.launch.xml* in the *sim_cf2/launch* folder. By default, two Crazyflies are simulated and code for the instantiation of four further Crazyflies is present but commented out. Adjust the launch file to your needs by adding or removing Crazyflies. Initial pose and rotor color is set in the *main.launch.xml* file.

## Run the Simulation

### 1. Start Gazebo

Open a terminal and source the ROS2 workspace if not done already (assuming the overlay or workspace in ~/dev_ws)

```sh
source /opt/ros/humble/setup.bash
```
Sometime this command also works.
```sh
source ~/dev_ws/install/local_setup.bash
```
Use the ROS2 launch command to start the *sim_cf2* simulation

```sh
ros2 launch sim_cf2 main.launch.xml
```

Gazebo will start in paused mode and will try to establish a connection to the software-in-the-loop Crazyflie firmware instances.

[//]: # (![gazebo_successfully_spawned_entities]&#40;images/gazebo_successfully_spawned_entities.png&#41;)

### 2. Run Crazyflie Firmware SITL Instances

Open a new terminal and move to the scripts/sim_cf2 folder in the Crazyflie firmware repository (assuming ~/repos/crazyflie-firmware)

```sh
cd ~/repos/crazyflie-firmware/scripts/sim_cf2
```

Run the *run_cfs.sh* script with as many instances of simulated Crazyflies as needed by providing the amount of Crazyflies as argument (e.g. two)

```sh
./run_cfs.sh 2
```

The SITL instances will establish a connection to Gazebo

[//]: # (![sitl_connection_established_with_gazebo]&#40;images/sitl_connection_established_with_gazebo.png&#41;)

**Press PLAY in Gazebo**

### 3. Run a Python Script

Start your *cflib*-based python script with activated *simlink* driver. Example scripts can be found in the Crazyflie python library https://github.com/CrazyflieTHI/crazyflie-lib-python/tree/master/examples/sim_cf2

Run for example the autonomousSequence.py script

```sh
cd ~/repos/crazyflie-lib-python/examples/sim_cf2
python3 autonomousSequence.py
```

[//]: # (![gazebo_successfully_spawned_entities]&#40;images/python_autonomousSequence.png&#41;)

### 4. Exit Gazebo and SITL

Press Ctrl+C in the terminals running Gazebo and the SITL instances

Processes should get killed and POSIX message queues should get deleted. If POSIX message queues are not deleted due to errors, they eventually may cause issues when starting the simulation again. The queues are located in */dev/mqueue*

If problems occur, delete the queues manually

Show queues
```sh
ls /dev/mqueue
```

Delete the queues
```sh
sudo rm /dev/mqueue/<sim_cf2_queues>
```
## Notice:
Need to restart gazebo every time running the script.

The latest ROS2 Iron should work, but some packages need to be updated for installation.

Example: Install *Cyclone DDS*
```shell
sudo apt install ros-iron-rmw-cyclonedds-cpp
```
## Test video
[Screencast from 02-03-2024 01:39:02 PM.webm](https://github.com/Lost-In-Fanta/Crazyflie-Sim_cf2/assets/98923769/d01a4596-795d-46bb-8bc6-25f16b7c1433)
