# Laboratory 2

Seongmin Jung  
Professor Lee  
ECSE 373  
26 September 2022  

---

This project is for learning and implementing basic functions of ROS, such as:  
- urdf
  - link
  - joint
- xacro
- rviz
- robot_state_publisher
- joint_state_publisher
- static_transform_publisher
- roslaunch

## How to run the project

### 1. Install ROS Noetic

[How to install ROS Noetic in Ubuntu Focal Fossa](http://wiki.ros.org/noetic/Installation/Ubuntu)

### 2. Create ROS package

    $ catkin_create_pkg navvis_description rviz urdf xacro sensor_msgs geometry_msgs

### 3. Create `urdf` file

`urdf` file describes how the robot is configured. It includes `link`, which are movable parts such as arms and wheels, and `joint` that defines the connection between `link`s. The overall format of the `urdf` file is as follows.  

    -robot
      -link
        -visual
          -origin
          -geometry
        -collision
          -origin
          -geometry
    -joint
      -parent
      -child
      -origin
      -limit (not necessary for fixed links)

### 4. Use `xacro`

`xacro` is a convenient feature that allows you to make as many copies as you need with a single definition of `link` or `joint`. To distinguish the `xacro` file from others, change the extension of the file created above from `.urdf` to `.xacro`.

Wrap the `link` or `joint` you want to create repeatedly with the following code.

    <xacro:macro name="xacro_name" params="param1 param2">
      <link name="...">
      ...
      </link>
      <joint name="..." type="...">
      ...
      </joint>
    </xacro:macro>

You can include as many `link`s and `joint`s as you want. In addition, parameters that can be used differently for each call may be defined as above.

To use parameters, use the format of `${param}` as shown in the example below.

    <xacro:macro name="xacro_name" params="side reflect">
      <link name="wheel_${side}”">
      ...
        <origin xyz="0.0 ${reflect * 0.4} 0.0" rpy="-1.5708 0.0 1.5708" />
        ...
    </xacro>

To load the generated `link` and `joint`, insert the following code at the desired location in the `xacro` file.

    <xacro:xacro_name param1="value1" param2="value2" />

### 5. Create `launch` file

Create `my.launch` file inside `launch` directory. This file contains instructions that allow multiple functions to be performed simultaneously, such as `node` generation and `rviz` execution, while executing `roscore`.

    <launch>
      <!-- Create an argument to determine whether to use a XACRO or URDF file. -->
      <arg name="use_xacro" default="false" />
      
      <!-- The filename can be passed into the launch file to override the default. -->
      <arg if="$(arg use_xacro)" name="filename" default="<filename>.xacro" />
      <arg unless="$(arg use_xacro)" name="filename" default="<filename>.urdf" />
      
      <!-- The full path of the URDF/XACRO file can be passed into the launch file instead. -->
      <arg name="file" default="$(find my_robot_description)/urdf/$(arg filename)" />
      
      <!-- Use the file argument as the name of the file used to set the /robot_description parameter on the parameter server -->
      <param if="$(arg use_xacro)" name="robot_description" command="$(find xacro)/xacro.py --inorder $(arg file)" />
      <param unless="$(arg use_xacro)" name="robot_description" textfile="$(arg file)" />
      
      <!-- Run the robot_state_publisher. -->
      <node pkg="robot_state_publisher" type="robot_state_publisher" name="robot_state_publisher" />
      
      <!-- Run RVIZ with a configuration file. If RVIZ is closed, close everything. -->
      <node pkg="rviz" type="rviz" name="rviz" args="-d $(find <package_name>)/config/config.rviz" required="true" />
    </launch>

### 6. Run `roslaunch` command

    $ roslaunch <package_name> <launch_file>.launch

## Nodes

- ### `robot_state_publisher`

    `robot_state_publisher` is a node that publishes information on the current state of the robot, such as links and joints. This message can be read from `rviz` and is displayed on the screen.

- ### `joint_state_publisher`

    The `joint_state_publisher` is a node that rotates the joint to allow the robot to move. This node delivers movement by publishing a message to `joint_states` topic.

- ### `static_transform_publisher`

    `static_transform_publisher` is a node for manually manipulating toe rotation of the robot joint.

---

I referred to the instruction of Laboratory #2 in ECSE 373 class.