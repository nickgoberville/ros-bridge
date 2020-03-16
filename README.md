# The winning team's custom ROS-bridge for the CARLA simulator

This ROS package aims at providing a custom ROS bridge for the CARLA simulator used during 
the spring 2020 Intro to Self-Driving Cars course at Western Michigan University.

**Important Note:**
This documentation is for CARLA version 0.9.8

![rviz setup](./docs/images/rviz_carla_default.png "rviz")

## CARLA_0.9.8 Download and Install

    #Setup directory
    mkdir ~/CARLA_0.9.8

    #Download CARLA_0.9.8.tar.gz from https://github.com/carla-simulator/carla/releases 
    # and place in ~/CARLA_0.9.8
    cd ~/CARLA_0.9.8
    tar xvf CARLA_0.9.8.tar.gz

    #OPTIONAL STEP: Download additional maps (AdditionalMaps_0.9.8.tar.gz)
    # from https://github.com/carla-simulator/carla/releases
    #  Place AdditionalMaps_0.9.8.tar.gz in ~/CARLA_0.9.8/Import
    tar xvf AdditionalMaps_0.9.8.tar.gz
    cd ..
    ./ImportAssets.sh

## ROS-bridge Setup

### Create a catkin workspace and install carla_ros_bridge package

    #setup folder structure
    mkdir -p ~/CARLA_0.9.8/ros-bridge/src
    cd ~/CARLA_0.9.8/ros-bridge/
    git clone https://github.com/nickgoberville/carla-ros-bridge.git
    cd src
    ln -s ../carla-ros-bridge
    cd carla-ros-bridge
    cat PATH >> ~/.bashrc
    source ~/.bashrc

    #install required ros-dependencies
    cd ~/CARLA_0.9.8/ros-bridge
    rosdep update
    rosdep install --from-paths src --ignore-src -r

    #build
    catkin_make

For more information about configuring a ROS environment see
<http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment>

## Start Carla and the ROS bridge

First run the simulator (choose one option):

    # Option 1: Run carla like normal
    cd ~/CARLA_0.9.8
    ./CarlaUE4.sh

    # Option 2: Run carla in background
    cd ~/CARLA_0.9.8
    export SDL_VIDEODRIVER=offscreen 
    ./CarlaUE4.sh -opengl

Wait a few seconds, then make custom map/weather/etc changes using config.py:

    # Recommended: Change map to Town04 with Clear and Sunny weather 
    #(may need to change spawn_point arg in me5950/carla_ego_vehicle/launch/carla_example_ego_vehicle if NOT using Town04)
    **New terminal**
    cd ~/CARLA_0.9.8/PythonAPI/util
    ./config.py -m Town04 --weather ClearNoon

start the ros-bridge and ego vehicle:

    # Option 1: Start ros-bridge node WITHOUT rviz
    source ~/CARLA_0.9.8/ros-bridge/devel/setup.bash
    roslaunch carla_ros_bridge carla_ros_bridge.launch

    # Option 2: Start the ros-bridge node WITH rviz
    source ~/CARLA_0.9.8/ros-bridge/devel/setup.bash
    roslaunch carla_ros_bridge carla_ros_bridge_with_rviz.launch
    
    # Start ego_vehicle node
    **New termianl**
    source ~/CARLA_0.9.8/ros-bridge/devel/setup.bash
    roslaunch carla_ego_vehicle carla_example_ego_vehicle.launch


## Settings provided in original carla-ros-bridge github page

You can setup the ros bridge configuration [carla_ros_bridge/config/settings.yaml](carla_ros_bridge/config/settings.yaml).

If the rolename is within the list specified by ROS parameter `/carla/ego_vehicle/rolename`, the client is interpreted as an controllable ego vehicle and all relevant ROS topics are created.

### Mode

#### Default Mode

In default mode (`synchronous_mode: false`) data is published:

-   on every `world.on_tick()` callback
-   on every `sensor.listen()` callback

#### Synchronous Mode

CAUTION: In synchronous mode, only the ros-bridge is allowed to tick. Other CARLA clients must passively wait.

In synchronous mode (`synchronous_mode: true`), the bridge waits for all sensor data that is expected within the current frame. This might slow down the overall simulation but ensures reproducible results.

Additionally you might set `synchronous_mode_wait_for_vehicle_control_command` to `true` to wait for a vehicle control command before executing the next tick.

##### Control Synchronous Mode

It is possible to control the simulation execution:

-   Pause/Play
-   Execute single step

The following topic allows to control the stepping.

| Topic            | Type                                                       |
| ---------------- | ---------------------------------------------------------- |
| `/carla/control` | [carla_msgs.CarlaControl](carla_msgs/msg/CarlaControl.msg) |

A [CARLA Control rqt plugin](rqt_carla_control/README.md) is available to publish to the topic.

## Available ROS Topics

### Ego Vehicle

#### Sensors

The ego vehicle sensors are provided via topics with prefix /carla/ego_vehicle/&lt;sensor_topic>

Currently the following sensors are supported:

##### Camera

| Topic                                                          | Type                                                                                   |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `/carla/<ROLE NAME>/camera/rgb/<SENSOR ROLE NAME>/image_color` | [sensor_msgs.Image](http://docs.ros.org/api/sensor_msgs/html/msg/Image.html)           |
| `/carla/<ROLE NAME>/camera/rgb/<SENSOR ROLE NAME>/camera_info` | [sensor_msgs.CameraInfo](http://docs.ros.org/api/sensor_msgs/html/msg/CameraInfo.html) |

##### Lidar

| Topic                                                     | Type                                                                                     |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `/carla/<ROLE NAME>/lidar/<SENSOR ROLE NAME>/point_cloud` | [sensor_msgs.PointCloud2](http://docs.ros.org/api/sensor_msgs/html/msg/PointCloud2.html) |

##### Radar

| Topic                                               | Type                                                                                                                                          |
| --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `/carla/<ROLE NAME>/radar/<SENSOR ROLE NAME>/radar` | [ainstein_radar_msgs.RadarTargetArray](https://github.com/AinsteinAI/ainstein_radar/blob/master/ainstein_radar_msgs/msg/RadarTargetArray.msg) |

Radar data can be visualized on rviz using [ainstein_radar_rviz_plugins](https://wiki.ros.org/ainstein_radar_rviz_plugins).

##### IMU

| Topic                    | Type                                                                              |
| ------------------------ | --------------------------------------------------------------------------------- |
| `/carla/<ROLE NAME>/imu` | [sensor_msgs.Imu](https://docs.ros.org/melodic/api/sensor_msgs/html/msg/Imu.html) |

##### GNSS

| Topic                                            | Type                                                                                 | Description           |
| ------------------------------------------------ | ------------------------------------------------------------------------------------ | --------------------- |
| `/carla/<ROLE NAME>/gnss/<SENSOR ROLE NAME>/fix` | [sensor_msgs.NavSatFix](http://docs.ros.org/api/sensor_msgs/html/msg/NavSatFix.html) | publish gnss location |

##### Collision Sensor

| Topic                          | Type                                                                     | Description              |
| ------------------------------ | ------------------------------------------------------------------------ | ------------------------ |
| `/carla/<ROLE NAME>/collision` | [carla_msgs.CarlaCollisionEvent](carla_msgs/msg/CarlaCollisionEvent.msg) | publish collision events |

##### Lane Invasion Sensor

| Topic                              | Type                                                                           | Description                     |
| ---------------------------------- | ------------------------------------------------------------------------------ | ------------------------------- |
| `/carla/<ROLE NAME>/lane_invasion` | [carla_msgs.CarlaLaneInvasionEvent](carla_msgs/msg/CarlaLaneInvasionEvent.msg) | publish events on lane-invasion |

#### Object Sensor

| Topic                        | Type                                                                                                     | Description                                      |
| ---------------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `/carla/<ROLE NAME>/objects` | [derived_object_msgs.ObjectArray](http://docs.ros.org/api/derived_object_msgs/html/msg/ObjectArray.html) | all vehicles and walkers, except the ego vehicle |

#### Control

| Topic                                                             | Type                                                                           |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `/carla/<ROLE NAME>/vehicle_control_cmd` (subscriber)             | [carla_msgs.CarlaEgoVehicleControl](carla_msgs/msg/CarlaEgoVehicleControl.msg) |
| `/carla/<ROLE NAME>/vehicle_control_cmd_manual` (subscriber)      | [carla_msgs.CarlaEgoVehicleControl](carla_msgs/msg/CarlaEgoVehicleControl.msg) |
| `/carla/<ROLE NAME>/vehicle_control_manual_override` (subscriber) | [std_msgs.Bool](http://docs.ros.org/api/std_msgs/html/msg/Bool.html)           |
| `/carla/<ROLE NAME>/vehicle_status`                               | [carla_msgs.CarlaEgoVehicleStatus](carla_msgs/msg/CarlaEgoVehicleStatus.msg)   |
| `/carla/<ROLE NAME>/vehicle_info`                                 | [carla_msgs.CarlaEgoVehicleInfo](carla_msgs/msg/CarlaEgoVehicleInfo.msg)       |

There are two modes to control the vehicle.

1.  Normal Mode (reading commands from `/carla/<ROLE NAME>/vehicle_control_cmd`)
2.  Manual Mode (reading commands from `/carla/<ROLE NAME>/vehicle_control_cmd_manual`)

This allows to manually override a Vehicle Control Commands published by a software stack. You can toggle between the two modes by publishing to `/carla/<ROLE NAME>/vehicle_control_manual_override`.

[carla_manual_control](carla_manual_control/) makes use of this feature.

For testing purposes, you can stear the ego vehicle from the commandline by publishing to the topic `/carla/<ROLE NAME>/vehicle_control_cmd`.

Examples for a ego vehicle with role_name 'ego_vehicle':

Max forward throttle:

     rostopic pub /carla/ego_vehicle/vehicle_control_cmd carla_msgs/CarlaEgoVehicleControl "{throttle: 1.0, steer: 0.0}" -r 10

Max forward throttle with max steering to the right:

     rostopic pub /carla/ego_vehicle/vehicle_control_cmd carla_msgs/CarlaEgoVehicleControl "{throttle: 1.0, steer: 1.0}" -r 10

The current status of the vehicle can be received via topic `/carla/<ROLE NAME>/vehicle_status`.
Static information about the vehicle can be received via `/carla/<ROLE NAME>/vehicle_info`

##### Additional way of controlling

| Topic                                       | Type                                                                             |
| ------------------------------------------- | -------------------------------------------------------------------------------- |
| `/carla/<ROLE NAME>/twist_cmd` (subscriber) | [geometry_msgs.Twist](http://docs.ros.org/api/geometry_msgs/html/msg/Twist.html) |

CAUTION: This control method does not respect the vehicle constraints. It allows movements impossible in the real world, like flying or rotating.

You can also control the vehicle via publishing linear and angular velocity within a Twist datatype.

Currently this method applies the complete linear vector, but only the yaw from angular vector.

##### Carla Ackermann Control

In certain cases, the [Carla Control Command](carla_msgs/msg/CarlaEgoVehicleControl.msg) is not ideal to connect to an AD stack.
Therefore a ROS-based node `carla_ackermann_control` is provided which reads [AckermannDrive](http://docs.ros.org/api/ackermann_msgs/html/msg/AckermannDrive.html) messages.
You can find further documentation [here](carla_ackermann_control/README.md).

### Other Topics

#### Object information of other actors

| Topic               | Type                                                                                                     | Description                           |
| ------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| `/carla/objects`    | [derived_object_msgs.ObjectArray](http://docs.ros.org/api/derived_object_msgs/html/msg/ObjectArray.html) | all vehicles and walkers              |
| `/carla/marker`     | [visualization_msgs.Marker](http://docs.ros.org/api/visualization_msgs/html/msg/Marker.html)             | visualization of vehicles and walkers |
| `/carla/actor_list` | [carla_msgs.CarlaActorList](carla_msgs/msg/CarlaActorList.msg)                                           | list of all carla actors              |
| `/carla/traffic_lights` | [carla_msgs.CarlaTrafficLightStatusList](carla_msgs/msg/CarlaTrafficLightStatusList.msg)             | list of all traffic lights with their status |

#### Status of CARLA

| Topic               | Type                                                           | Description                                            |
| ------------------- | -------------------------------------------------------------- | ------------------------------------------------------ |
| `/carla/status`     | [carla_msgs.CarlaStatus](carla_msgs/msg/CarlaStatus.msg)       |                                                        |
| `/carla/world_info` | [carla_msgs.CarlaWorldInfo](carla_msgs/msg/CarlaWorldInfo.msg) | Info about the CARLA world/level (e.g. OPEN Drive map) |

### Walker

| Topic                                                | Type                                                                         | Description        |
| ---------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------ |
| `/carla/walker/<ID>/walker_control_cmd` (subscriber) | [carla_msgs.CarlaWalkerControl](carla_msgs/msg/CarlaWalkerControl.msg)       | Control a walker   |
| `/carla/walker/<ID>/odometry`                        | [nav_msgs.Odometry](http://docs.ros.org/api/nav_msgs/html/msg/Odometry.html) | odometry of walker |

### Other Vehicles

| Topic                          | Type                                                                         | Description         |
| ------------------------------ | ---------------------------------------------------------------------------- | ------------------- |
| `/carla/vehicle/<ID>/odometry` | [nav_msgs.Odometry](http://docs.ros.org/api/nav_msgs/html/msg/Odometry.html) | odometry of vehicle |

### Debug Marker

It is possible to draw markers in CARLA.

Caution: Markers might affect the data published by sensors.

The following markers are supported in 'map'-frame:

-   Arrow (specified by two points)
-   Points
-   Cube
-   Line Strip

| Topic                              | Type                                                                                                   | Description                 |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------- |
| `/carla/debug_marker` (subscriber) | [visualization_msgs.MarkerArray](http://docs.ros.org/api/visualization_msgs/html/msg/MarkerArray.html) | draw markers in CARLA world |


## Additional Functionality

| Name                              | Description                                                                                             |
| --------------------------------- | ------------------------------------------------------------------------------------------------------- |
| [Carla Ego Vehicle](carla_ego_vehicle/README.md) | Provides a generic way to spawn an ego vehicle and attach sensors to it. |
| [Carla Infrastructure](carla_infrastructure/README.md) | Provides a generic way to spawn a set of infrastructure sensors defined in a config file. |
| [Carla Waypoint Publisher](carla_waypoint_publisher/README.md) | Provide routes and access to the Carla waypoint API |
| [Carla ROS Scenario Runner](carla_ros_scenario_runner/README.md) | ROS node that wraps the functionality of the CARLA [scenario runner](https://github.com/carla-simulator/scenario_runner) to execute scenarios. |
| [Carla AD Agent](carla_ad_agent/README.md) | A basic AD agent, that can follow a route and avoid collisions with other vehicles and stop on red traffic lights. |
| [Carla AD Demo](carla_ad_demo/README.md) | A meta package that provides everything to launch a CARLA ROS environment with an AD vehicle. |
| [RVIZ Carla Plugin](rviz_carla_plugin/README.md) | A [RVIZ](http://wiki.ros.org/rviz) plugin to visualize/control CARLA. |
| [RQT Carla Plugin](rqt_carla_plugin/README.md) | A [RQT](http://wiki.ros.org/rqt) plugin to control CARLA. |

## Troubleshooting

### ImportError: No module named carla

You're missing Carla Python. Please execute:

    export PYTHONPATH=$PYTHONPATH:<path/to/carla/>/PythonAPI/carla/dist/<your_egg_file>

Please note that you have to put in the complete path to the egg-file including
the egg-file itself. Please use the one, that is supported by your Python version.
Depending on the type of CARLA (pre-build, or build from source), the egg files
are typically located either directly in the PythonAPI folder or in PythonAPI/dist.

Check the installation is successfull by trying to import carla from python:

    python -c 'import carla;print("Success")'

You should see the Success message without any errors.