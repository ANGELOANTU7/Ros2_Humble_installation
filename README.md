# ROS2 Humble Installation Guide for Ubuntu 22.04

Welcome to the ROS2 Humble Installation Guide! This guide will walk you through the process of installing Ubuntu 22.04 and setting up ROS2 Humble on your system.

## Installing Ubuntu 22.04

1. **Download Ubuntu 22.04 ISO**: Visit the [official Ubuntu website](https://releases.ubuntu.com/jammy/ubuntu-22.04.3-desktop-amd64.iso) or a trusted mirror and download the ISO file for Ubuntu 22.04.

2. **Create a Bootable USB Drive**: Use a tool like [Rufus](https://rufus.ie/en/) on Windows or Etcher on macOS and Linux to create a bootable USB drive with the Ubuntu 22.04 ISO.

3. **Boot from USB Drive**: Insert the bootable USB drive into your computer and restart it. Access the boot menu and select the USB drive to boot from it.

4. **Install Ubuntu**: Follow the on-screen instructions to install Ubuntu 22.04 on your system. You can choose to install alongside an existing operating system or replace it entirely.

5. **Complete Installation**: Once the installation process is complete, remove the USB drive and restart your computer. You should now have Ubuntu 22.04 installed and ready to use.

## Installing ROS2 Humble

### System Setup

**Set Locale**

Make sure you have a locale which supports UTF-8. If you are in a minimal environment (such as a docker container), the locale may be something minimal like POSIX. We test with the following settings. However, it should be fine if you’re using a different UTF-8 supported locale.

```bash
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```

**Add the ROS 2 apt repository**

You will need to add the ROS 2 apt repository to your system.

First ensure that the Ubuntu Universe repository is enabled.

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe

# Now add the ROS 2 GPG key with apt.
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# Then add the repository to your sources list.
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

**Install development tools and ROS tools**

Install common packages.

```bash
sudo apt update && sudo apt install -y \
  python3-flake8-docstrings \
  python3-pip \
  python3-pytest-cov \
  ros-dev-tools
```

Install packages according to your Ubuntu version.

```bash
sudo apt install -y \
   python3-flake8-blind-except \
   python3-flake8-builtins \
   python3-flake8-class-newline \
   python3-flake8-comprehensions \
   python3-flake8-deprecated \
   python3-flake8-import-order \
   python3-flake8-quotes \
   python3-pytest-repeat \
   python3-pytest-rerunfailures
```

**Get ROS 2 code**

Create a workspace and clone all repos:

```bash
mkdir -p ~/ros2_humble/src
cd ~/ros2_humble
vcs import --input https://raw.githubusercontent.com/ros2/ros2/humble/ros2.repos src
```

**Install dependencies using rosdep**

ROS 2 packages are built on frequently updated Ubuntu systems. It is always recommended that you ensure your system is up to date before installing new packages.

```bash
sudo apt upgrade

sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"
```

Note: If you’re using a distribution that is based on Ubuntu (like Linux Mint) but does not identify itself as such, you’ll get an error message like `Unsupported OS [mint]`. In this case, append `--os=ubuntu:jammy` to the above command.

**Install additional DDS implementations (optional)**

If you would like to use another DDS or RTPS vendor besides the default, you can find instructions [here](https://index.ros.org/doc/ros2/Installation/DDS-Implementations/).

### Build the code in the workspace

If you have already installed ROS 2 another way (either via Debians or the binary distribution), make sure that you run the below commands in a fresh environment that does not have those other installations sourced. Also ensure that you do not have `source /opt/ros/${ROS_DISTRO}/setup.bash` in your .bashrc. You can make sure that ROS 2 is not sourced with the command `printenv | grep -i ROS`. The output should be empty.

```bash
cd ~/ros2_humble/
colcon build --symlink-install
```

Note: if you are having trouble compiling all examples and this is preventing you from completing a successful build, you can use `COLCON_IGNORE` in the same manner as `CATKIN_IGNORE` to ignore the subtree or remove the folder from the workspace. Take for instance: you would like to avoid installing the large OpenCV library. Well then simply run `touch COLCON_IGNORE` in the `cam2image` demo directory to leave it out of the build process.

### Environment setup

**Source the setup script**

Set up your environment by sourcing the following file.

```bash
# Replace ".bash" with your shell if you're not using bash
# Possible values are: setup.bash, setup.sh, setup.zsh
. ~/ros2_humble/install/local_setup.bash
```

**Try some examples**

In one terminal, source the setup file and then run a C++ talker:

```bash
. ~/ros2_humble/install/local_setup.bash
ros2 run demo_nodes_cpp talker
```

In another terminal source the setup file and then run a Python listener:

```bash
. ~/ros2_humble/install/local_setup.bash
ros2 run demo_nodes_py listener
```

You should see the talker saying that it’s Publishing messages and the listener saying I heard those messages. This verifies both the C++ and Python APIs are working properly. Hooray!
