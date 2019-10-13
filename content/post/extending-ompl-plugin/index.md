---
title: Extending OMPL Plugin for VREP
subtitle:  Making modifications to the motion-planning algorithms implemented in V-REP and testing custom versions.
summary:  Making modifications to the motion-planning algorithms implemented in V-REP and testing custom versions.
date: 2018-03-31
math: true
diagram: true
markup: mmark
tags: 
- Motion-Planning
- Set-up
categories:
- Robotics
image:
  placement: 1
  caption: 'Image credit: [**Google Images**](https://www.google.com/images)'
---

<!-- <img src="/assets/images/school/AT/ompl-home.jpg" width="700"> -->


[Motion planning](https://en.wikipedia.org/wiki/Motion_planning) is integral to the working of any autonomous agent. [OMPL](https://ompl.kavrakilab.org/) is an open-sourced library that provides implementations for various [sampling-based motion planning](https://en.wikipedia.org/wiki/Motion_planning#Sampling-based_algorithms) algorithms. It is used by various robotics platforms and simulators such as [MoveIt](https://moveit.ros.org/), [V-REP](http://www.coppeliarobotics.com/) etc and can be used in a straight-out-of-box manner.

But what if we want to make modifications to any of the motion-planning algorithms and use our version of the algorithm on these platforms? In this post, we will take a look at how to tackle this problem statement for V-REP. 

## Aim

Make code changes in any of the motion-planning algorithms in the OMPL library and use that updated version of OMPL in V-REP.

*Note: This tutorial has been tested with V-REP version 3.5 64 bit, OMPL [downloaded from source](https://github.com/ompl/ompl), Python 2.7, and Ubuntu 16.04*

## Installing the dependencies

First things first, let's install the dependencies, [Boost](http://www.boost.org) (version 1.54 or higher), [CMake](http://www.cmake.org) (version 2.8.7 or higher), and [Eigen](http://eigen.tuxfamily.org) (version 3.3 or higher) for OMPL. 

Run the following commands to install these dependencies.

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get -y install cmake pkg-config libboost-all-dev libeigen3-dev libode-dev 
export CXX=g++
export MAKEFLAGS="-j `nproc`"
```

These instruction have been taken from the [official OMPL installation script](https://github.com/ompl/ompl/blob/master/install-ompl-ubuntu.sh.in).


There are some additional dependencies that need to be installed for the plugin to work such as [Flann library](https://launchpad.net/ubuntu/+source/flann), [xsltproc](http://xmlsoft.org/XSLT/xsltproc.html) and python 2.7 or greater. 
You can run the following commands to install the first two dependencies.

```
sudo apt install xsltproc
sudo apt-get install libflann-dev
```

## Downloading VREP and OMPL

### V-REP

1. Download VREP (version 3.5) from [here](http://coppeliarobotics.com/files/V-REP_PRO_EDU_V3_5_0_Linux.tar.gz).
2. Extract the tar file to a folder.

This is all that is needed to install VREP. To run VREP, execute the script ```vrep.sh``` from inside the extracted folder. 

*Note: From here on, we will refer to this folder as the **V-REP Directory*** 

### OMPL

1. Download the OMPL source code from [its GitHub repository](https://github.com/ompl/ompl).

*Note: From here on, we will refer to this folder as the **OMPL Directory*** 


## The VREP-OMPL Integration

VREP-OMPL integration can be divided into following steps:
1. Building the OMPL library.
2. Building the OMPL Plugin for V-REP.
3. Updating the V-REP folder libraries


## Building the OMPL library.

You would need to run the following commands to build the OMPL library. 

*Note: In my case, ompl is the name of the OMPL Directory.*

```
cd ompl
mkdir -p build/Release
cd build/Release
cmake ../..
make
sudo make install
```

This will build the OMPL library (**named libompl.so**) from the source code which you just downloaded from github.
It will also install this directory in the location ```/usr/local/lib/```. You can also find the library created in the location ```OMPL Directory/build/Release/lib/```


## Building the OMPL Plugin for V-REP

To build the [OMPL plugin for V-REP](https://github.com/CoppeliaRobotics/v_repExtOMPL) plugin:
1. Navigate to the programming folder inside the VREP Directory. 
2. Run the following commands:

```
git clone --recursive https://github.com/CoppeliaRobotics/v_repExtOMPL.git
cd v_repExtOMPL
```

*We will call this the Plugin Directory*

Since this plugin depends on the OMPL library to integrate with VREP, we have to first make sure it picks up the updated OMPL library from the correct location. To do that, open the file ```CMakeCache.txt``` and check for keys ```OMPL_LIBRARY``` and ```OMPL_LIBRARIES``` to make sure that they point to the correct location. It should either be ```/usr/local/lib/libompl.so``` or  ```OMPL Directory/build/Release/lib/libompl.so```

Now that we know that the integrator plugin is configured correctly, let's compile the plugin code to build the plugin by running the following commands from inside the plugin folder.

```
cmake .
cmake --build .
```

This will build the plugin library in the Plugin Directory by the name **libv_repExtOMPL.so**


## Updating the VREP Libraries

Now that we have built the OMPL library and the plugin, there is one last thing left to be done. If you look inside the V-REP Directory, you will notice that all of these libraries (libompl.so, libompl.1.4.so, libompl.1.4.so and libv_repExtOMPL.so) are already present. Which means V-REP looks up these libraries from here. So we need to replace these libraries with our updated libraries.

Copy the following files from the location OMPL folder/build/Release/lib/

  * libompl.so
  * lib.ompl.so.1.3.2
  * lib.ompl.so.1.4

And libv_repExtOMPL.so from inside the plugin folder, <br>
and paste them inside the VREP folder, replacing the already existing ones. 

## Let's Test

1. Open VREP.
2. Open scene -> VREP Directory/scenes ->MotionPlanningDemo1.ttt
3. Play the scene.

Here's what the console output looks like

```
Info:    RRTConnect: Space information setup was not yet called. Calling now.
Debug:   RRTConnect: Planner range detected to be 7.260570
Info:    RRTConnect: Starting planning with 1 states already in datastructure
Info:    RRTConnect: Created 11 states (9 start + 2 goal)
Info:    RRTConnect: Space information setup was not yet called. Calling now.
Debug:   RRTConnect: Planner range detected to be 7.260570
Info:    RRTConnect: Starting planning with 1 states already in datastructure
Info:    RRTConnect: Created 4 states (2 start + 2 goal)
Info:    RRTConnect: Space information setup was not yet called. Calling now.
...
...
...
```

Now let's test if our extension works or now.

1. Let's modify two files:
```OMPL Directory/src/ompl/geometric/planners/rrt/src/RRTConnect.cpp``` and ```OMPL Directory/src/ompl/base/src/Planner.cpp```
2. Let's play safe for now and modify the print statements in these files. 
3. Now build the OMPL library, VREP-OMPL plugin and replace the VREP libraries with these updated ones as descried above.
4. Restart VREP if already running.
5. Now execute the same MotionPLanningDemo1.ttt script again.
5. Here's the expected output:

```
Info:    RRTConnect: **Peps Modified it**Space information setup was not yet called. Calling now.
Debug:   RRTConnect: Planner range detected to be 7.260570
Info:    RRTConnect: ***Peps Modified It***Starting planning with 1 states already in datastructure
Info:    RRTConnect: ***Peps Modified It***Created 15 states (13 start + 2 goal)
Info:    RRTConnect: **Peps Modified it**Space information setup was not yet called. Calling now.
Debug:   RRTConnect: Planner range detected to be 7.260570
Info:    RRTConnect: ***Peps Modified It***Starting planning with 1 states already in datastructure
Info:    RRTConnect: ***Peps Modified It***Created 78 states (76 start + 2 goal)
Info:    RRTConnect: **Peps Modified it**Space information setup was not yet called. Calling now.
```

Voila! It worked! 
Using this procedure, you can make any changes to any algorithm in the OMPL library and test those changes on V-REP.
That's all for now. Adios!