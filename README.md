# micro-ROS and pingPong application


In this documentation we will implementing micro-ROS and testing it with Ping Pong application, which will help start learning micro-ROS and understand its architecture.


It is really recommended to visit [micro.ros.org](https://micro.ros.org/) to get more information.


*Note: These instructions have been done on `Ubuntu 20.04 LTS`.*


## Outline

- Install ROS 2 and the micro-ROS framework and tools.
- Testing the micro-ROS Ping Pong app.
- Multiple Ping Pong nodes.
- Problems faced while implementing this project.
- Resources.


## Install ROS 2 and The micro-ROS Framework and Tools

### install ROS 2 Foxy via Debian Packages


start by making sure that you have a locale which supports UTF-8, if not please visit [Installing ROS 2 via Debian Packages](https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html)
```bash
locale  # check for UTF-8
```


setup sources and add the ROS 2 apt repositories to your system
```bash
sudo apt update && sudo apt install curl gnupg2 lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```


now Iinstall ROS 2 packages
```bash
sudo apt update   # Update your apt repository caches
sudo apt install ros-foxy-desktop
```

set up the environment by sourcing to the following file
```bash
source /opt/ros/foxy/setup.bash
```


since we installed ros-foxy-desktop we can try some examples, we will try running the talker and listener apps which will verifies that both the C++ and Python APIs are working properly.


In new terminal, source the setup file and then run talker 
```bash
source /opt/ros/foxy/setup.bash
ros2 run demo_nodes_cpp talker
```

In another terminal source the setup file and then run listener 
```bash
source /opt/ros/foxy/setup.bash
ros2 run demo_nodes_py listener
```

you will see that the talker is Publishing messages [Hello world], and the listener is hearing those messages 


as showing in below video:


https://user-images.githubusercontent.com/85321139/128885632-df8b5a27-df69-4c2e-9851-be3d1b1781cd.mp4



### install the micro-ROS build system


```bash
# Source the ROS 2 installation
source /opt/ros/foxy/setup.bash

# Create a workspace and download the micro-ROS tools
mkdir microros_ws
cd microros_ws
git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup

# Update dependencies using rosdep
sudo apt update && rosdep update
rosdep install --from-path src --ignore-src -y

# Install pip
sudo apt-get install python3-pip

# Build micro-ROS tools and source them
colcon build
source install/local_setup.bash
```


### Creating a new firmware workspace


now that the build system is installed, we will create a firmware workspace to get the required codes and tools
```bash
ros2 run micro_ros_setup create_firmware_ws.sh host
```


### Building the firmware


```bash
ros2 run micro_ros_setup build_firmware.sh
source install/local_setup.bash
```


### Creating the micro-ROS agent


The app now is ready to be connecte to a micro-ROS agent to start talking with the rest of the ROS2.


so to create and build a micro-ROS agent run the following commands
```bash
ros2 run micro_ros_setup create_agent_ws.sh

ros2 run micro_ros_setup build_agent.sh
source install/local_setup.bash
```


## Testing The micro-ROS Ping Pong App

### Running the micro-ROS app

now that you have both the client and the agent correctly installed in your machine, you must give micro-ROS access to the ROS 2 dataspace and run the agent
```bash
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
```


in new terminal run the micro-ROS node and by sourcing the ROS 2 and micro-ROS , and setting the RMW Micro XRCE-DDS implementation
```bash
source /opt/ros/foxt/setup.bash
source install/local_setup.bash

export RMW_IMPLEMENTATION=rmw_microxrcedds

ros2 run micro_ros_demos_rclc ping_pong
```


### Demo of micro-ROS Ping Pong


We are going to listen to the ping topic with ROS 2, to check whether the micro-ROS Ping Pong node is publishing the expected pings.


so, in a new terminal source the setup file and then subscribe to micro-ROS ping topic, every few seconds you will see the topic messages published by the Ping Pong node.
```bash
source /opt/ros/foxy/setup.bash
ros2 topic echo /microROS/ping
```


now our app is publishing pings, we will check if it will answers to other's pings. if it's true then it will publish a pong.


in new terminal, subscribe with ROS 2 to the pong topic. initially we don't expect to receive pong yet.
```bash
source /opt/ros/foxy/setup.bash

ros2 topic echo /microROS/pong
```


in new terminal, publish a fake_ping with ROS 2 
```bash
source /opt/ros/foxy/setup.bash

ros2 topic pub --once /microROS/ping std_msgs/msg/Header '{frame_id: "fake_ping"}'
```


you will see the fake_ping in the ping subscriber terminal with the micro-ROS pings, since we have received the fake_ping, the micro-ROS node will answer with a pong. Then in the pong subscriber terminal, you will see the micro-ROS app answer to the fake_ping as showing in below video:


https://user-images.githubusercontent.com/85321139/128895799-d8c9d346-ff4e-4f9e-a693-c32998891d36.mp4



## Multiple Ping Pong Nodes


run the same micro-ROS agent as we did above, and open a number of different terminals and run the following commands on each one under microros_ws directory
```bash
source /opt/ros/foxy/setup.bash
source install/local_setup.bash

export RMW_IMPLEMENTATION=rmw_microxrcedds

ros2 run micro_ros_demos_rclc ping_pong
```


https://user-images.githubusercontent.com/85321139/128896088-321b59ce-bde2-412b-8ab2-5079c47a6f70.mp4



## Problems Faced While Implementing This Project


1- I had to install git separately before install the micro-ROS, to do so run this command 
```bash
sudo apt install git
```
![gitInstallation]()


2- when trying to build micro-ROS tools and source them using the command `colcon build`. I needed to install colcon tool separately using the following command
```bash
sudo apt install python3-colcon-common-extensions
```
![colconinstallation]()

## Resources

- https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html#id3
- https://micro.ros.org/docs/tutorials/core/first_application_linux/







