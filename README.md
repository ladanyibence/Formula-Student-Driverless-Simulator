# Formula Student Driverless Simulation
![Front picture](docs/images/fsds_pretty.png)

[Read the docs here.](https://fs-online.github.io/Formula-Student-Driverless-Simulator/)

This is a Formula Student Driverless Simulation Competition System (FSDS).
It will provide a virtual environment where Autonomous Systems from different Formula Student teams can compete in time-trial challenges. 
The first competition will take place during the driverless event, FS-Online 2020.

FSDS is brought to you by your friends at Formula Student Team Delft, MIT Driverless and FSEast.

You can read all about how FSDS works in this [systems overview document](/docs/system-overview.md)

## Repo Overview

`/UE4Project` is the Unreal Engine 4 project.
In here you will find everything that runs inside Unreal Engine.
This includes the world, all assets and the AirSim server.
The AirSim server uses the AirLib shared code (see `/AirSim/AirLib`).

`/operator` is the simulation control system. 
This is a python project that offers a web gui for officials to control the simulation, stores lap times and chooses what car is currently connected to the world.
It launches the ASBridge to connect a autonomous system to the Unreal world and stops the bridge when the autonomous system is no longer allowed to control the car.

`/ros` is a ROS workspace that contains the `fsds_ros_bridge`. 
This node can connect 1 autonomous system with the simulated world.

`/AirSim` is a slimmed-down, hard-fork of the [AirSim](https://github.com/microsoft/AirSim) project.
There is only code located that is shared between operator, ros-bridge and UE4 plugin.
When AirSim is compiled, the AirLib binaries are placed within `/UE4Project/Plugins/AirSim/Source/AirLib`.

This repo uses LFS for some large files. All files bigger than 90MB are added to LFS.

## Running & Development

To **run** the simulation, read the [simulation guide](docs/how-to-simulate.md).

To **build from source and develop** this project, read the [development guide](docs/how-to-develop.md).

## Participants of FSOnline Driverless
We have written an [integration guide](docs/integration-handbook.md) for getting your autonomous systems connected to this simulator.


## Credits
This project is based on the work of some amazing other open-source projects. 

Primarily, [the AirSim project built by Microsoft and many open-source contributors](https://github.com/microsoft/AirSim). 
Without it, we could have never done this.

Many game assets like the surrounding world are based on assets from the [Formula Student Technion Driverless AirSim fork](https://github.com/FSTDriverless/AirSim). These assets made this project look as cool as it does now.


![Closing picture](docs/images/fsds_cam_view.png)


## License

Copyright (C) 2020 Formula Driverless Competition Simulator Contributors

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program; if not, write to the Free Software Foundation, Inc.,
51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
