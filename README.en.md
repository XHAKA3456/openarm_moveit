# openarm_moveit

A standalone package for controlling the OpenArm bimanual robot via the MoveIt RViz UI.
Drag the interactive markers in RViz to set a goal pose, then click **Plan & Execute**.

---

## Environment

| Item | Version |
|---|---|
| OS | Ubuntu 24.04 |
| ROS | ROS 2 Jazzy |

> If ROS 2 Jazzy is not installed, please follow the official installation guide first.
> https://docs.ros.org/en/jazzy/Installation.html

---

## Directory Structure

```
openarm_moveit/
└── src/
    ├── setup_can.sh                    # CAN interface bring-up / teardown script
    ├── openarm_can/                    # CAN communication library (Damiao motor control)
    ├── openarm_description/            # Robot URDF/xacro models and mesh files
    └── openarm_ros2/
        ├── openarm/                    # Meta-package
        ├── openarm_bimanual_moveit_config/  # MoveIt configuration (core)
        │   ├── config/
        │   │   ├── openarm_bimanual.srdf     # Planning groups and end-effector definitions
        │   │   ├── openarm_bimanual.urdf.xacro  # Robot description for MoveIt
        │   │   ├── kinematics.yaml           # IK solver configuration (trac_ik)
        │   │   ├── joint_limits.yaml         # Joint velocity and acceleration limits
        │   │   ├── moveit_controllers.yaml   # MoveIt ↔ ros2_control bridge
        │   │   ├── moveit.rviz               # Default RViz layout
        │   │   └── sensors_3d.yaml           # Sensor configuration
        │   └── launch/
        │       └── demo.launch.py            # Main launch file
        ├── openarm_bringup/            # ros2_control controller configurations
        │   └── config/v10_controllers/
        │       └── openarm_v10_bimanual_controllers.yaml
        └── openarm_hardware/           # Hardware interface for the real robot (CAN-based)
```

---

## Installation & Usage

### Step 1: Verify ROS 2 Jazzy environment

Make sure your ROS 2 environment is sourced.

```bash
source /opt/ros/jazzy/setup.bash
```

To avoid running this every time, add it to `~/.bashrc`:

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Step 2: Install dependencies

```bash
sudo apt update && sudo apt install -y \
  ros-jazzy-moveit \
  ros-jazzy-moveit-configs-utils \
  ros-jazzy-moveit-kinematics \
  ros-jazzy-moveit-planners \
  ros-jazzy-moveit-planners-ompl \
  ros-jazzy-moveit-planners-chomp \
  ros-jazzy-moveit-planners-stomp \
  ros-jazzy-moveit-ros-move-group \
  ros-jazzy-moveit-ros-visualization \
  ros-jazzy-moveit-ros-planning-interface \
  ros-jazzy-moveit-simple-controller-manager \
  ros-jazzy-trac-ik-kinematics-plugin \
  ros-jazzy-ros2-control \
  ros-jazzy-ros2-controllers \
  ros-jazzy-controller-manager \
  ros-jazzy-joint-state-broadcaster \
  ros-jazzy-joint-trajectory-controller \
  ros-jazzy-forward-command-controller \
  ros-jazzy-gripper-controllers \
  ros-jazzy-robot-state-publisher \
  ros-jazzy-joint-state-publisher \
  ros-jazzy-rviz2 \
  ros-jazzy-xacro \
  ros-jazzy-tf2-ros
```

### Step 3: Create workspace and clone

```bash
mkdir -p ~/openarm_moveit/src
cd ~/openarm_moveit/src
git clone https://github.com/XHAKA3456/openarm_moveit.git .
```

> The trailing `.` clones directly into the current `src/` directory.

### Step 4: Build

```bash
cd ~/openarm_moveit
colcon build
source install/setup.bash
```

For every new terminal session, run:

```bash
source ~/openarm_moveit/install/setup.bash
```

### Step 5: Bring up CAN interfaces (required for real robot)

> Skip this step if you are using simulation mode (`use_fake_hardware:=true`).

```bash
cd ~/openarm_moveit/src
./setup_can.sh up
```

- `can0` → right arm
- `can1` → left arm
- CAN FD mode, bitrate: 1 Mbps / data bitrate: 5 Mbps

To bring down the interfaces:
```bash
./setup_can.sh down
```

### Step 6: Launch

**Simulation (fake hardware):**
```bash
ros2 launch openarm_bimanual_moveit_config demo.launch.py use_fake_hardware:=true
```

**Real robot:**
```bash
ros2 launch openarm_bimanual_moveit_config demo.launch.py use_fake_hardware:=false
```

---

## RViz Usage

1. Open the **MotionPlanning** panel on the left side of RViz
2. Select a planning group from the **Planning Group** dropdown:
   - `both_arms` — control both arms simultaneously (default)
   - `left_arm` — left arm only
   - `right_arm` — right arm only
3. Drag the **interactive marker** (sphere) at the end of the arm to set the goal pose
4. Click **Plan & Execute**

---

## Launch Parameters

| Parameter | Default | Description |
|---|---|---|
| `use_fake_hardware` | `false` | `true`: simulation, `false`: real robot |
| `right_can_interface` | `can0` | CAN interface for the right arm |
| `left_can_interface` | `can1` | CAN interface for the left arm |
| `arm_type` | `v10` | Arm version |

---

## Credits

Based on the following open-source repositories by [Enactic, Inc.](https://github.com/enactic), licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0):

- [openarm_ros2](https://github.com/enactic/openarm_ros2)
- [openarm_description](https://github.com/enactic/openarm_description)
- [openarm_can](https://github.com/enactic/openarm_can)

### Modifications from upstream

- `openarm_bimanual.urdf.xacro`: Fixed default xacro args so MoveItConfigsBuilder loads the correct bimanual robot model (`bimanual=true`)
- `openarm_bimanual.srdf`: Aligned robot name with URDF, added `both_arms` planning group, added `parent_group` to end-effector definitions
- `joint_limits.yaml`: Added acceleration limits required for trajectory planning
- `moveit.rviz`: Changed default planning group to `both_arms`

---

## License

Apache License 2.0 — see `LICENSE` files in each package.
