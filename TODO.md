### 1. Gait Tuning & Dynamic Posture Control
* **Stance & Body Pose Control:**
  - CHAMP publishes to `/body_pose` (`geometry_msgs/msg/Pose`). You can publish roll, pitch, yaw, and body translation commands to adjust the robot's posture while standing or walking (e.g., crouching or tilting when climbing slopes).
* **Gait Parameter Optimization:**
  - In [`gait.yaml`](file:///home/robotics/navneeth/m2_ros2_champ/m2_ws/src/m2_ros2_champ/m2_sim/config/gait/gait.yaml), experiment with:
    - `swing_height` & `stance_depth`: To control how high the feet lift during each step.
    - `stance_duration`: Adjusting step frequency and walking cadences.
    - `com_x_translation`: Shifting the center of mass offset to eliminate tilting or drift during straight-line walking.
* **Actuator Effort PID Tuning:**
  - In [`ros_control.yaml`](file:///home/robotics/navneeth/m2_ros2_champ/m2_ws/src/m2_ros2_champ/m2_sim/config/ros_control/ros_control.yaml), tune individual joint trajectory PID gains (`p`, `i`, `d`) to balance stiffness vs compliant shock absorption.
* **Rough Terrain Testing:**
  - Modify [`default.sdf`](file:///home/robotics/navneeth/m2_ros2_champ/m2_ws/src/m2_ros2_champ/m2_description/worlds/default.sdf) or load staircases, ramps, and heightmaps in Gazebo Sim to test the robot's ability to cross uneven obstacles.

---

### 2. Sensor Integration (Perception & State Estimation)
* **Real Contact Sensor Logic:**
  - Refine [`foot_contacts_bridge.cpp`](file:///home/robotics/navneeth/m2_ros2_champ/m2_ws/src/m2_ros2_champ/m2_application/src/foot_contacts_bridge.cpp) by tuning `min_normal_force` and `contact_timeout` to filter out foot-chatter noise on impact.
* **3D LiDAR Integration:**
  - Add a LiDAR link (e.g. Livox Mid-360 or Velodyne VLP-16) to [`m2_metal.urdf.xacro`](file:///home/robotics/navneeth/m2_ros2_champ/m2_ws/src/m2_ros2_champ/m2_description/urdf/m2_metal.urdf.xacro).
  - Bridge Gazebo's PointCloud2 topic into ROS 2 for obstacle detection.
* **RGB-D Depth Camera:**
  - Mount an Intel RealSense D435/D455 or Luxonis OAK-D to the torso for forward perception, visual odometry, and terrain mapping.

---

### 3. SLAM & Autonomous Navigation (Nav2)
* **2D SLAM (`slam_toolbox`):**
  - Project 3D point clouds to a 2D laser scan using `pointcloud_to_laserscan`.
  - Run `slam_toolbox` to map rooms and indoor environments in real time.
* **3D LiDAR Odometry & Mapping:**
  - Integrate algorithms like [KISS-ICP](https://github.com/PRBonn/kiss-icp) or [FAST-LIO](https://github.com/hku-mars/FAST_LIO) using M2's IMU and point cloud.
* **Nav2 Integration:**
  - Set up standard ROS 2 Navigation (Nav2) costmaps and planners (e.g. Smac Planner / Regulated Pure Pursuit) feeding velocity commands directly to `/cmd_vel` for autonomous waypoint following and dynamic obstacle avoidance.

---

### 4. Sim-to-Real Hardware Bridge
* **`ros2_control` Hardware Driver:**
  - Write a custom C++ `hardware_interface::SystemInterface` plugin that talks to your actual motor controllers (e.g., CAN bus for CyberGear, Unitree A1/Go motors, ODrive, or custom BLDC drivers).
  - Because CHAMP already commands `joint_group_effort_controller/joint_trajectory`, switching from Gazebo to the real robot only requires changing the `<hardware>` plugin from `gz_ros2_control/GazeboSimSystem` to your custom hardware driver.
* **Inertia & Mass Real-World Calibration:**
  - Verify actual weights of 3D prints, CNC metal parts, and batteries against the link inertias in [`m2_metal.urdf.xacro`](file:///home/robotics/navneeth/m2_ros2_champ/m2_ws/src/m2_ros2_champ/m2_description/urdf/m2_metal.urdf.xacro).

---

### 5. Advanced Locomotion Control
* **Reinforcement Learning (RL) Locomotion:**
  - Export the M2 URDF to Isaac Lab / Isaac Gym or MuJoCo to train an end-to-end RL locomotion policy (parkour, rough terrain running), then deploy the policy in ROS 2 using ONNX Runtime.
