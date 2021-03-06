# Autonomous System Integration Handbook
This page describes how to integrate your autonomous system (AS) to the Formula Student Driverless Simulator (FSDS).
The rules and procedures set out in this document will be used during the FSOnline competition.

## High-level overview
Your AS is expected to continuously run a ROS master.
The simulator will launch a node ([fsds_ros_bridge](ros-bridge.md)) that connects to your ROS system.
This node will publish sensor data and listen for vehicle setpoints.
When the simulation is done, the node is closed and your AS will no longer be able to interact with the simulator.
[A more in-depth explanation of the FSDS system can be found here.](system-overview.md)

Integrating your autonomous system with a simulator is a matter of subscribing and publishing to topics and acting accordingly.

*During testing, your AS and the ROS bridge/simulator will probably run on the same machine (your local computer most likely). However, during the competition, the AS and the ROS bridge/simulator will be on different machines (cloud servers) and communicate over a local network.*

## System flow and Signals

Initially, when your AS launches, no simulation is connected.
The AS must wait for a GO signal before acting.

### Staging
At some point, your AS will be staged: The vehicle is placed at a staging line prior to the starting line, the `fsds_ros_bridge` node will connect to the ROS system.
From this point onwards, the AS will receive sensor data and can control the vehicle by publishing vehicle setpoints.
However, it shouldn't start driving the vehicle just yet!

### Starting 
Just like with a physical FS event, the vehicle must only start driving after a GO signal is received.
The GO signal indicates that the AS must start the mission and that the vehicle should start driving.
This signal is also known as the 'green flag' signal or 'starting' signal. Within this repo, we will reference it as 'GO'.
Within the GO signal, the mission and track are added. 
Currently, `autocross` and `trackdrive` are the supported missions.

In autocross, the vehicle has to complete a single lap on an unknown track.
On the trackdrive, the vehicle has to finish 10 laps on a track it has previously seen.
During the competition, on each track, every AS will do an autocross mission before a trackdrive mission.
It can occur that multiple autocross runs on different tracks take place before going to trackdrive.
It can also happen that multiple autocross runs take place on the same track.
For example, the AS might be requested to do:
1. autocross on track A
2. autocross on track B
3. trackdrive on track A
4. autocross on track C
5. autocross on track C (re-run)
6. trackdrive on track C

The AS must implement the following behaviour:
* When the AS is requested to do autocross on a track that it has seen before, it must delete any and all data it gathered during all previous runs on this track.
* When the AS is requested to do trackdrive on a track where it has done a trackdrive previously, it must delete any and all data it gathered during all previous trackdrive runs on this track. However, the data gathered during the last autocross on this track shouldn't be deleted.

An exception to this rule is data recorded with the exclusive intent to analyze the AS's behaviour after the event.
This includes all files that the AS only writes to but does not read from.

To make the AS aware of which track it is driving, the GO signal includes a unique identifier of the upcoming track.

After the initial GO signal, the signal is continuously re-sent at 1 Hz to ensure it arrives at the team's AS.
The timestamp of all consecutive GO signals is equal to the first one.

### Finishing
There are two ways to conclude a run: finishing or stopping.

When the autonomous system feels it concluded it's run, it should send a FINISHED signal.
The FINISHED signal tells the simulator that the AS no longer wants to control the vehicle.
As such, the simulator will stop the `fsds_ros_bridge` and the AS will no longer receive sensor data or be able to control the vehicle.

When the official decides that the run is over it will stop the simulation.
See the rulebook for a description of when the official does so.
When the simulation is stopped the `fsds_ros_bridge` is stopped immediately and the AS will no longer receive sensor data or be able to control the vehicle.
The AS will not receive a signal that this happened.
To detect a stop, the AS should keep an eye on the GO signal.

The general rule is: If the AS did not receive a GO signal for 4 seconds the AS can assume the `fsds_ros_bridge` is stopped.
When this state is detected, the AS can reset itself and prepare for the next mission.

## Sensor Suite
Every team can configure the sensors on their vehicle.
This configuration is called the sensor suite.
To ensure the simulation will perform as expected, the sensor suite has some restrictions.
Here you can read the requirements and restrictions that apply to every sensor.

### Camera
**A this moment camera's are a bit broken. You can use the camera's but the framerate and topic names might change. See #43 and #85.**

Every vehicle can have a maximum of 2 camera sensors. 
These camera(s) can be placed anywhere on the vehicle that would be allowed by FSG 2020 rules. 
The camera body dimensions are a 4x4x4 cm cube with mounting points at any side except the front-facing side.

All camera sensors output uncompressed RGBA8 images at 30 FPS. 
You can choose the resolution of the camera(s). 
In total, the camera’s can have 1232450 pixels. 
Every dimension (width or height) must be at least 240px and no greater than 1600px. 
The horizontal field of view (FoV) is configurable for each camera and must be at least 30 degrees and not be greater than 90 degrees. 
The vertical FoV will be automatically calculated using the following formula: `vertical FoV = image height / image width * horizontal FoV`.

The camera's auto exposure, motion blur and gamma settings will be equal for all teams.


### Lidar
A vehicle can have between 0 and 5 lidars.
The lidar(s) can be placed anywhere on the vehicle that would be allowed by FSG 2020 rules.
The body dimension of every lidar is a vertical cylinder, 8 cm height and 8 cm diameter with mounting points at the top and bottom.

A single lidar can have between 1 and 500 lasers. 
The lasers are stacked vertically and rotate on the horizontal plane. 
The lasers are distributed equally to cover the specified vertical field of view.

The vertical field of view is specified by choosing the upper and lower limit in degrees. 
The lower limit specifies the vertical angle between the horizontal plane of the lidar and the most bottom laser. 
The upper limit specifies the vertical angle between the horizontal plane of the lidar and most upper laser. 

The horizontal field of view of the lidar is specified with an upper and lower limit in degree as well. Only points within this FOV will be returned. 
The lower limit specifies the counterclockwise angle on a top view from the direction the lidar is pointing towards. 
The upper limit specifies the clockwise angle on a top view from the direction the lidar is pointing towards. 

For every lidar, the rotation speed (Hz) and capture frequency (Hz) must be chosen. 
The rotation speed specifies how fast the lasers spin and the capture frequency specifies how often a point cloud is created.

While rotating, only lasers within the horizontal field of view are captured. 

> For example, a lidar with 190 degrees horizontal field of view, rotating at 10hz with a capture frequency of 20 Hz will receive point clouds covering anything from 10 to 180 degrees of the field of view.

There is no guarantee that the rotation speed and capture frequency stay in sync. 
You won’t be able to rely on synchronization of rotation speed and capture frequency. 
A lidar rotating at 5 Hz would theoretically have rotated 50 times after 10 seconds but in reality, this will be somewhere between 45 and 55 times. 

For every lidar, you can specify the resolution: the total number of points collected if the lasers would do a 360 field of view sweep scan. 
This value is used to calculate the number of points in each laser and the spacing between the points. 

Every lidar capture is limited to collecting 10000 points. 
The maximum number of points collected during a capture is calculated by dividing the lidar’s resolution by the horizontal field of view fraction.

> For example, a lidar with a 30 degrees horizontal field (horizontal FOV fraction of 30/360 = 1/12) can have a maximum resolution of 120000.

The total number of points per second is limited to 100000 points.

> For example, a first lidar collects 10000 points per capture at 5 hz, a second lidar collects 8000 points per capture at 5 hz. 
  This is valid because in total they collect 90000 points per second.

### GPS
Every vehicle has 1 GPS, it is located at the centre of gravity of the vehicle.
This sensor cannot be removed or moved.

The GPS captures the position of the vehicle in the geodetic reference system, namely longitude [deg], latitude [deg], altitude [m].
More detailed technical information about the accuracy of the GPS can be found [here](gps.md).

### IMU
Every vehicle has 1 IMU, it is located at the centre of gravity of the vehicle.
This sensor cannot be removed or moved.

The IMU captures the acceleration, orientation and angular rate of the vehicle.
More detailed technical information about the IMU implementation of the IMU can be found [here](imu.md).

### Sensor specification
Teams are expected to provide their sensor suite as a single AirSim settings.json file.
Most of the parameters in the settings.json file will be set by the officials to ensure fairness during competition.
You are allowed to configure the following subset of parameters within the boundaries of the above rules.

* Cameras   
   * camera name
   * Width, Height
   * FOV_Degrees
   * X, Y, Z
   * Pitch, Roll, Yaw
* Lidars
   * NumberOfChannels
   * PointsPerSecond
   * RotationsPerSecond
   * HorizontalFOVStart
   * HorizontalFOVEnd
   * VerticalFOVUpper
   * VerticalFOVLower
   * X, Y, Z
   * Pitch, Roll, Yaw

The GPS and IMU are configured equally for all teams according to the rules in the previous chapter.

We recommend to copy the settings.json in this repository as a base and configure the cameras and lidar from thereon.

## Launching the simulator
To run the simulation, read the [simulation guide](how-to-simulate.md).

## ROS integration
Communication between the autonomous system and simulator will take place using ROS topics.
Sensor data will be published by the [ros bridge](ros-bridge.md) and received by the autonomous system.
The autonomous system will publish vehicle setpoints and the ROS bridge will listen for those messages.
Static transforms between sensors also are being published for usage by the autonomous system.

### Topics

The AS can subscribe to the following sensor topics:

- `/fsds/gps`
- `/fsds/imu`
- `/fsds/camera/CAMERA_NAME`
- `/fsds/camera/CAMERA_NAME/camera_info`
- `/fsds/lidar/LIDAR_NAME`

The AS will receive the GO signal on the following topic:

- `/fsds/signal/go`

And it AS can publish the FINISHED on this topic:

- `/fsds/signal/finished`

The AS must publish vehicle control commands on this topic:

- `/fsds/control_command`

Read more about the techincal detalis of these topics in the [ros-bridge documentation](ros-bridge.md)

## Vehicle dynamic model
The vehicle dynamic model is a third-party high-fodelity model and will be the same for all teams. 
More details and information on this choice can be found [here](vehicle_model.md).

## 3D vehicle model
//todo
This chapter will describe how to change the 3d model of the vehicle and how to provide the 3d model for usage during the competition. 
At this moment we have no idea how this works sooooo when we figure it out this will be filled in.

## Competition deployment
A few weeks before the competition, each team will receive the ssh credentials to an Ubuntu google cloud instance.
This instance will have 8 vCPU cores, 30 GB memory (configuration n1-standard-8), 1 Nvidia Tesla T4 video card and 100GB SSD disk.
The teams must install their autonomous system on this computer.

During the competition, a separate google cloud instance will run the simulation software and the ROS bridge. 
One by one the ROS bridge will connect to the different teams' computers and they will do their mission.

During the weeks leading up to the competition, FSOnline will host multiple testing moments where the autonomous computers will be connected to the simulator and drive a few test laps.

More information about the procedure will be added later.
