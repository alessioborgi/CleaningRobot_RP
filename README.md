# Autonoous Mobile Robot: Cleaning Robot
#### Copyright © 2024 Alessio Borgi




## Introduction
Welcome to the Cleaning Robot project designed by Alessio Borgi! This project is designed to simulate a **Cleaning Robot** using the Robot Operating System (ROS), Webots, and Rviz on Ubuntu 20.04. 

### Key Capabilities
- **SLAM**: The robot meticulously builds a map of its environment, capturing every detail using sensors like Lidar and odometry. This comprehensive map paves the way for efficient and obstacle-aware navigation.
- **Planning Trajectories**: The robot employs sophisticated algorithms like NavfnROS and TrajectoryPlannerROS to calculate optimal paths to reach designated goals, taking into account both static obstacles in the map and dynamic ones encountered in real-time.
- **Dynamic Obstacle Avoidance**: Equipped with sensors and intelligent algorithms, it can detect and gracefully steer around obstacles, ensuring smooth and uninterrupted navigation.



## Instructions
1. **Installations**
   - Ensure you have Ubuntu 20.04 installed on your system.
   - Install ROS1 Noetic (Robot Operating System) following the instructions on the [official ROS website](https://www.ros.org/install/).
   - Install Webots R2021a from the [official website](https://cyberbotics.com/#download).

2. **Workspace Setup:**
   - Create a project folder using a command like `mkdir nameFolder`.
   - Go inside the project folder typing `cd nameFolder`.
   - Create inside the project folder a Workspace and a `src` folder inside it, using the command `mkdir -p CleaningRobot_ws/src`.
   - Go inside the `src` folder of the Workspace using `cd CleaningRobot_ws/src`.
   - Clone this repository inside the `src` folder using the command `git clone https://github.com/alessioborgi/CleaningRobot_RP.git`.

3. **Dependencies and Packages Installation**
   - Go inside the Workspace folder and install RosDepth using `sudo apt-get install python3-rosdep`.
   - Initialise RosDepth using `sudo rosdep init`.
   - Update rosdep in such a way to look for the required versions using `rosdep update`.
   - Go in the `robot_settings` package using `cd robot_settings`. 
   - Install all the dependencies required by the package using `rosdep install --from-paths src --ignore-src-r-y`.
   - Go back to the Workspace fodler using `cd ..`. 
   - Go in the `nav_plan_objavoid` package using `cd nav_plan_objavoid`. 
   - Install all the dependencies required by the package using `rosdep install --from-paths src --ignore-src-r-y`.

4. **Launching the Project:**
   - Go inside the Workspace folder using the command `cd nameFolder/CleaningRobot_ws`.
   - Build the whole Cleaning Robot Project using `catkin build`.
   - Configure ROS environment variables using the `source devel/setup.bash` command.
   - Launch the whole project using `roslaunch robot_settings master.launch`.
   - At this point two windows should be opened: one being Webots and the other being Rviz, through which you can monitor robot behavior and perform all the operations.


## Home Environment
The Cleaning Robot will work and perform its tasks in a WeBots' Home, with the following graphical representation, allowing you to both see the top-view of the robot in the house, but also the robot's point of view, thanks to the top-left Camera-viewpoint.  

<div style="text-align: center">
  <img src="images/Home.png" alt="Screenshot" width="1200"/>
</div>


## Robot Structure & URDF
The robot's body is a rectangular box capable of moving in four directions: front, back, left, and right. It is equipped with a Linear Actuator and a Rotary Actuator, both connected to a camera. This combination enables the robot to adjust the camera viewpoint vertically (up and down) and horizontally (left and right). Additionally, the robot is outfitted with two Distance Sensors, two GPS units, Lidar, and an IMU (Inertial Measurement Unit). It's important to note that the wheels, distance sensors, and other components are fixed relative to the center of the body box, essentially acting as fixed joints. Conversely, the camera mechanism, which moves relative to the body, is classified as a continuous joint.

<div style="text-align: center">
  <img src="images/Robot_Image.jpg" alt="Screenshot" width="1200"/>
</div>

The Robot Structure description is physically described using the **URDF (“Unified Robot Description Format”)**, being the way to physically describe the Robot to ROS. In summary, fixed parts are connected using fixed joints, while the linear_actuator (which can move up and down) and the camera_link (which can rotate over itself), are linked using a continuous joint. Also, the Navigation is classified as a dynamic/continuous joint. This is equivalent to passing local parameters to methods. Indeed, the robot_description package converts the “CleaningRobot.xacro” file in the URDF,  and places it into the Parameter Server. The “robot_state_publisher” robot node, will then extract the URDF file from the  Parameter Server and broadcast the 3D pose of the robot link to the transform library in ROS. 
 
<div style="text-align: center">
  <img src="images/Robot_URDF_Scheme.png" alt="Screenshot" width="1200"/>
</div>
We can verify that the whole Robot Setting opening a new terminal in the Workspace folder and to ask for the `rqt_gui`, using the following command: `rosrun rqt_gui rqt_gui`. If the project is correctly working, you will have the following: 

<div style="text-align: center">
  <img src="images/rqt_gui.png" alt="Screenshot" width="1200"/>
</div>



## TeleOp (Keyboard)
The first functionality provided in this project is the **TeleOperation through the Keyboard**, allowing you to control your robot movements in the Home and also the Camera Orientation and Position using keyboard inputs. This feature enables you to manually drive or manipulate your robot's movements and actions in real-time by sending commands via the keyboard.

<div style="text-align: center">
  <img src="images/TeleOp.png" alt="Screenshot" width="1200"/>
</div>

Notice that you can also monitor the TeleOperation Topic by, while the Project is running, do:
- Open another Terminal and type the command `rostopic list`.
- From this list, one of these has this form `/Cam_robot_xxxxxx_NameOfYourMachine` with `xxxxxx` changing at each master.launch.
- Save this result and now type the command `rostopic echo /Cam_robot_xxxxxx_NameOfYourMachine/keyboard/key`.
At this point, the Terminal should echo the keyboard keys it receives as input. 


## SLAM (Simultaneous Localization and Mapping) with GMapping

Another project functionality is the ability to perform **SLAM ("Simultaneous Localization and Mapping")** task, consisting in having the Robot that creates a map of the environment it needs to operate into and, in parallel, estimate the position and orientation in that built map (i.e., Localization). This is achieved by merging the sensed results obtained from the diverse sensors mounted on the robot. 

### GMapping
The package used for this task implementation is **GMapping**, implemented using the SLAM Algorithm called **Rao-Blackwellized Particle Filter**. This algorithm combines the advantages of particle filtering with those of Kalman filtering to provide an accurate and efficient estimation of a robot's pose and the map of its environment. By maintaining a set of particles representing different hypotheses about the robot's pose, and using a Kalman filter for each particle to estimate the map, the algorithm achieves a high degree of accuracy while remaining computationally tractable. Gmapping proposes an approach to compute an accurate proposal distribution taking into account not only the movement of the robot but also the most recent observation. This drastically decreases the uncertainty about the robot’s pose in the prediction step of the filter. The input that GMapping uses is **Raw Laser Data + Odometry**, used by the algorithm to estimate the Robot Pose w.r.t. the odometry, providing a map to Odom_link as an output. The other crucial output is the map or a 2D occupancy grid, being a representation of the environment, displaying the obstacles and the free spaces that have been scanned. 

### SLAM Building Instructions
In order to perform another **Simultaneous Localization and Mapping** of your house or whatever place you decide to employ the robot, you will need to checkout to the other branch present in this repo called `SLAM_Map_Building` by using the command `git checkout SLAM_Map_Building` in the workspace folder. In this way, the project will be redirected to the version of the project where SLAM is available, since in the master branch, there is the already saved image Home Map. 

To run it, then, we will have to open two terminals in the workspace:
- **Terminal 1**: 
   - Build the Project using `catkin build`.
   - Re-source the project with `source devel/setup.bash`.
   - Launch the master by using `roslaunch bringup master.launch`.
- **Terminal 2**:
   - Initialize the GMapping Node with `rosrun gmapping slam_gmapping scan:=/Cam_robot_xxxx_Ubuntu_22_04/Lidar/laser_scan/layer0`, with the precise name of the Cam_robot_xxxxxx_... taken by doing `rostopic list`.

You can see the **SLAM BUILDING VIDEO** by clicking on the image:

<div style="text-align: center">
  <a href="https://www.youtube.com/watch?v=iSu1aiwxvLg">
    <p style="font-size: 16px; margin-top: 5px;">Click the Photo to See the Video!</p>
    <img src="images/SLAM_Building_img.png" alt="Screenshot" width="1200"/>
  </a>
</div>


### Saving the Map
This map can be saved using the **Map Server** Ros Package and can be used for Navigational purposes. The following, can be achieved by typing in a new Terminal the following 	`rosrun map_server map_saver -f  src/robot_settings/maps/map`. 

The result that we will obtain in the folder is like the following.
<div style="text-align: center">
  <img src="images/SLAM_result.png" alt="Screenshot" width="1200"/>
</div>


## NAVIGATION, PLANNING & OBJECT AVOIDANCE
Another functionality that his project offers is the ability to perform **Robot Navigation** in an environment/map, and in particular in the one acquired in the step before. The Cleaning Robot, can go from point A to point B, merging two Paths, being:
- **Global Path**: This is computed over a **Global CostMap**, where it knows where the static obstacles are and in which place has a lower cost to pass for the robot and represents the optimal route. In my case, the implemented **Global Planner** is **NavfnROS**, providing a fast interpolated navigation function that can be used to create plans for a mobile base. The navigation function is computed with **Dijkstra's Algorithm**. 
- **Local Path**: This gives to the robot the ability to perform **Object Avoidance**, trying to find out alternative paths to go over obstacles and rejoin the Global Path. In my case, the implemented **Local Planner** is **TrajectoryPlannerROS**, being the basic Local Planner, that provides an implementation of the **Trajectory Rollout and Dynamic-Window** approaches to local robot navigation on a plane. Given a plan to follow and a cost-map, the controller produces velocity commands to send to a mobile base.
  
These abilities: **Navigation, Planning** and **Object Avoidance**, makes the Robot an **AMR ("Autonomous Mobile Robots")**. 

This implementation makes use of the **MoveBase** package, that is a Framework where you can plug different Global and Local Planners and different paradigms for your cost map, for it to work which suits your robot. MoveBase takes in input a map from the `map_server`. It then updates the global_costmap which is fed to the global_planner that computes the Global Path of how to reach the goal. It then goes to the Local Planner, to take smaller decisions on how to avoid obstacles. For Local_Planner, we have also a local_costmap for its assistance, together with the sensor data that gives information about the cars/people/objects around the robot. 
Once we have the Local Plan, we get a command velocity from MoveBase which is the output and our trigger is the MoveBase. 

To run the **Planning** and **Object Avoidance**, you can click on the *2D Nav Goal* button on the upper part of Rviz and then click on the point you would like to have the robot. The robot will automatically plan the path to from its starting point to the goal point and will go over this trajectory, taking into account also eventual obstacles in the path, with the ability to go over them. (Ex. In the map below, there are two obstacles in the map, that were not present during the SLAM process and that will be therefore treated as newly dynamic obstacles.)

You can see the **NAVIGATION & PLANNING VIDEO** by clicking on the image:

<div style="text-align: center">
  <a href="https://www.youtube.com/watch?v=JZtBGJTJ42g">
    <p style="font-size: 16px; margin-top: 5px;">Click the Photo to See the Video!</p>
    <img src="images/Navigation_Planning_Object_Avoidance_img.png" alt="Screenshot" width="1200"/>
  </a>
</div>

You can see the **OBJECT AVOIDANCE DURING PLANNING VISUALIZATION** by clicking on the image:

<div style="text-align: center">
  <a href="https://www.youtube.com/watch?v=Vp21lLMRADQ">
    <p style="font-size: 16px; margin-top: 5px;">Click the Photo to See the Video!</p>
    <img src="images/Object_Avoidance.png" alt="Screenshot" width="1200"/>
  </a>
</div>

There are cases in which the robot is not able to reach the goal. MoveBase has a great recovery behaviour in this case: it rotates around its axis in such a way to seek for new paths. If it finds that no solution can be found, it will send a cancel in move_base. 

Another very important thing is that you can remove data due to noisy sensor data (resulting in points in the map in Rviz, by opening a new terminal and using the clear service by typing `rosservice call /move_base/clear_costmaps`.

<div style="text-align: center">
  <img src="images/Noisy_Map.jpg" alt="Screenshot" width="400"/>
</div>



