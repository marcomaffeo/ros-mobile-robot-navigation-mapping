# Autonomous 2D Navigation and Mapping for Mobile Robots via ROS and TEB Local Planner

This project implements an autonomous navigation software framework to control a mobile robot in a simulated Gazebo environment. The objective is to enable a mobile platform to perform 2D navigation and map the environment by following a predefined sequence of goals, avoiding static and dynamic obstacles, and integrating visual feedback via ArUco marker detection. The core navigation stack relies on `move_base` and the **TEB (Timed Elastic Band) Local Planner**.

## Features

*   **Environment Setup:** Configuration of a Gazebo world (`rl_racefield`) and spawning of a mobile robot (FRA2MO) with specific initial poses and obstacle layouts.
*   **Goal-Directed Navigation:** Implementation of TF listeners and `move_base` action clients to sequentially guide the robot through predefined waypoints (static TFs).
*   **Autonomous Mapping:** Expanding the waypoint sequence to ensure full exploration and mapping of the simulated environment.
*   **Navigation Stack Tuning:** Extensive parameter tuning of `costmap_2d` and `teb_local_planner` to analyze and optimize trajectory generation and obstacle avoidance behaviors.
*   **Vision-Based Navigation:** Integration of an RGB-D camera and `aruco_ros` to detect an ArUco marker, estimate its 3D pose, and dynamically adjust the robot's navigation goal based on visual feedback.

## Project Structure

*   **`rl_fra2mo_description/`**: Contains the URDF/Xacro models of the robot, including sensors (Lidar, RGB-D camera), and launch files for RViz and Gazebo spawning.
*   **`fra2mo_2dnav/`**: The core navigation package containing:
    *   Costmap configurations (`global_costmap_params.yaml`, `local_costmap_params.yaml`, etc.).
    *   TEB local planner configurations (`teb_local_planner_params.yaml`).
    *   C++ source code (`tf_nav.cpp`) for TF listening and Action Client goal management.
*   **`rl_racefield/`**: Contains the Gazebo world file and custom 3D models (obstacles, ArUco markers).
*   **`aruco_ros/`**: Package for ArUco marker detection and pose estimation.
*   **`bagfile/` & `Video/`**: Datasets containing recorded ROS bags of trajectories and demonstration videos.

## Core Implementations

### 1. Goal Sequencing via TF and ActionLib
The robot's mission is defined by a series of static TFs broadcasted in the `map` frame[cite: 1]. A custom C++ node (`tf_nav`) listens to these transforms using `tf::TransformListener`[cite: 1]. Upon receiving a transform, it extracts the position and quaternion, formulates a `move_base_msgs::MoveBaseGoal`, and dispatches it using a `SimpleActionClient`[cite: 1]. The system employs a state machine logic to ensure sequential progression only upon the `SUCCEEDED` state of the current goal[cite: 1].

### 2. TEB Local Planner and Costmap Tuning
To optimize navigation, several configurations were tested:
*   **Tight Spaces:** Reduced `min_obstacle_dist` to `0.05` to allow the robot to navigate through narrow tunnels instead of finding alternative, longer routes[cite: 1].
*   **Kinematic Limits:** Modified `max_vel_x` (from 0.6 to 1.2) and `acc_lim_x` (from 0.1 to 0.4) to evaluate the impact on navigation time, achieving a reduction from ~32s to ~27s between specific waypoints[cite: 1].
*   **Footprint Adjustments:** Increased the robot's footprint dimensions in `costmap_common_params.yaml` (from 0.15 to 0.5), demonstrating the expected failure in narrow passages due to conservative obstacle avoidance[cite: 1].

### 3. Visual Servoing with ArUco Markers
The robot navigates towards a predefined obstacle and activates its onboard camera to locate ArUco marker #115[cite: 1]. Using `aruco_ros`, the marker's 2D image coordinates are translated into a 3D pose within the `map` frame[cite: 1].
The node dynamically calculates a new navigation goal relative to the detected marker ($x_{new} = x_{marker} + 2.0$, $y_{new} = y_{marker}$), allowing the robot to autonomously position itself at a specific offset from the target[cite: 1]. The detected pose is subsequently broadcasted as a static TF (`aruco_pose_tf`)[cite: 1].

## Media

*   **Autonomous Navigation and Mapping**

https://github.com/user-attachments/assets/df783f29-3179-4ccc-b584-ba78a49441a6


*   **Vision-Based Target Approach**

https://github.com/user-attachments/assets/0085cd11-27c6-47ec-adb8-43f3db5c3773


## Dependencies

*   [ROS Noetic](http://wiki.ros.org/noetic)
*   [Gazebo 11](https://gazebosim.org/home)
*   [teb_local_planner](http://wiki.ros.org/teb_local_planner)
*   [aruco_ros](http://wiki.ros.org/aruco_ros)