# Real-Time Autonomous Navigation for Hiwonder Mecanum Robot

An autonomous navigation framework optimized for the **Hiwonder Mecanum robot** running on a **Raspberry Pi 5** (Docker container / host OS).

This package adapts Hiwonder's original navigation software stack into a **mapless, dynamic navigation pipeline**: rather than relying on pre-built static maps with AMCL localization, the robot navigates dynamically in an odometry-referenced rolling window while actively avoiding static and dynamic obstacles.

---

## Architecture: Underlay vs. Overlay

To safely customize behavior without altering vendor packages, this setup uses an underlay/overlay pattern:

- **Underlay (Base Workspace - `~/ros2_ws`):** Contains vendor-provided hardware drivers (chassis controllers, LiDAR node, sensor publishers).
- **Overlay (Custom Workspace - `~/my_nav_ws`):** Houses this package (`my_nav_pkg`), containing custom parameters, modified launch sequences, and custom nodes.

---

## Repository Structure

```text
my_nav_pkg/
├── config/
│   └── my_nav_params.yaml    # Nav2 parameters (odom global frame, rolling windows)
├── launch/
│   └── my_nav.launch.py      # Mapless Nav2 stack (bypasses map_server & AMCL)
├── my_nav_pkg/               # Core Python nodes
├── resource/                 # ament index marker
├── test/                     # Unit and integration tests
├── package.xml               # Package metadata & dependencies
├── setup.cfg                 # Build configuration
├── setup.py                  # Entry points and data files installation
└── README.md
```

## Set-up and Daily Operational Workflow

### Step 1: Stop Conflicting Background Services
The robot launches vendor background apps and hardware nodes automatically at boot. Stop them before running any custom stack:
```text
~/.stop_sh
```
(Note: To restart the original background service later, run: sudo systemctl restart start_node.service)

### Step 2: Create the Overlay Workspace
Set up a clean custom workspace separate from the factory underlay:
```text
mkdir -p ~/my_nav_ws/src
cd ~/my_nav_ws/src
```

### Step 3: Clone This Repository
Clone your package directly into the overlay src/ directory:
```text
git clone [https://github.com/](https://github.com/)amandinerondot/my_nav_pkg.git
```
Verify that the folder structure exists under ~/my_nav_ws/src/my_nav_pkg.

### Step 4: Build the Package
Source the factory underlay first so the build tool can find all Hiwonder hardware messages and drivers, then compile the overlay:
```text
cd ~/my_nav_ws

# Source factory drivers and ROS 2
source /opt/ros/humble/setup.bash # or /opt/ros/foxy/setup.bash depending on image
source ~/ros2_ws/install/setup.bash

# Build the overlay
colcon build --symlink-install
```

### Step 5: Run the Custom Mapless Stack

Terminal 1: Launch Underlay Hardware Drivers
```text
source /opt/ros/foxy/setup.bash # or your ROS 2 distro
source ~/ros2_ws/install/setup.bash
ros2 launch hiwonder_bringup bringup.launch.py
```
Terminal 2: Launch Overlay Mapless Navigation
```text
source /opt/ros/foxy/setup.bash # or your ROS 2 distro
source ~/ros2_ws/install/setup.bash       # Hardware underlay
source ~/my_nav_ws/install/setup.bash     # Your overlay
ros2 launch my_nav_pkg my_nav.launch.py use_sim_time:=false
```
