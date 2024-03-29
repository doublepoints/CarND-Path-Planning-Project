# CarND-Path-Planning-Project-P1
Udacity Self-Driving Car Nanodegree - Path Planning Project

# Abstraction

In this project, The path planning algorithms are implemented to drive a car on a highway on a simulator provided by Udacity([the simulator could be downloaded here](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2)). The car's information (position and velocity) and sensor fusion information (Ex. car id, velocity, position) are sended by the simulator. It expects a set of points spaced in time at 20 ms (50 frames per second) representing the car's trajectory. The communication between the simulator and the path planner is done by using [WebSocket](https://en.wikipedia.org/wiki/WebSocket). In this project, [uWebSockets](https://github.com/uNetworking/uWebSockets)  are introduced to handle this communication. Moreover, Udacity provides a seed project to start from on this project ([here](https://github.com/udacity/CarND-Path-Planning-Project)).

# Prerequisites

The project has the following dependencies (from Udacity's seed project):

- cmake >= 3.5
- make >= 4.1
- gcc/g++ >= 5.4
- libuv 1.12.0
- Udacity's simulator.

For instructions on how to install these components on different operating systems, please, visit [Udacity's seed project](https://github.com/udacity/CarND-Path-Planning-Project). As this particular implementation was done on Mac OS, the rest of this documentation will be focused on Mac OS. I am sorry to be that restrictive.

In order to install the necessary libraries, use the [install-mac.sh](./install-mac.sh).

# Compiling and executing the project

In order to build the project there is a `./build.sh` script on the repo root. It will create the `./build` directory and compile de code. This is an example of the output of this script:

```
> sh ./build.sh
-- The C compiler identification is AppleClang 8.0.0.8000042
-- The CXX compiler identification is AppleClang 8.0.0.8000042
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: REPO_ROOT/CarND-Path-Planning-Project-P1/build
Scanning dependencies of target path_planning
[ 50%] Building CXX object CMakeFiles/path_planning.dir/src/main.cpp.o
[100%] Linking CXX executable path_planning
[100%] Built target path_planning
```

The project could be executed directly using `./build/path_planning`

```
> cd build
> ./path_planning
Listening to port 4567
```

OK. It means the path planner is running and listening on port 4567 for messages from the simulator. 
Next step is to open Udacity's simulator:

Click the "Select" button of Project1: Path Planning.

# [Rubic points](https://review.udacity.com/#!/rubrics/1020/view) 

You can get all the detail by clicking the Subtitle. As a result. I have clear all the points listed below

## The code compiles correctly.

No changes were made in the cmake configuration. A new file was added [src/spline.h](./scr/spline.h). It is the [Cubic Spline interpolation implementation](http://kluge.in-chemnitz.de/opensource/spline/): a single .h file you can use splines instead of polynomials. 
I have gotten a lot of suggestion from QA video. It works great.

## The car is able to drive at least 4.32 miles without incident.
There is no accident happend during 5 miles.

![5 miles](images/img1.png)

## The car drives according to the speed limit.
No speed limit red message was seen.

## Max Acceleration and Jerk are not Exceeded.
Max jerk red message was not seen.

## Car does not have collisions.
No collisions.

## The car stays in its lane, except for the time between changing lanes.
The car stays in its lane most of the time but when it changes lane because of traffic or to return to the center lane.

## The car is able to change lanes
The car change lanes when the there is a slow car in front of it, and it is safe to change lanes (no other cars around) or when it is safe to return the center lane.

## There is a reflection on how to generate paths.

The path planning algorithms start at [src/main.cpp](./src/main.cpp#L246) line 246 to the line 416.

The code consist of three parts:

### Prediction [line 255 to line 290](./src/main.cpp#L255)
This part of the code deal with the telemetry and sensor fusion data. It intents to reason about the environment. In the case, we want to know three aspects of it:

- Is there a car in front of us blocking the traffic.
- Is there a car to the right of us making a lane change not safe.
- Is there a car to the left of us making a lane change not safe.

These questions are answered by calculating the lane and the position. A car is considered "dangerous" when its distance to our car is less than 90 meters in front or behind us.

### Behavior [line 292 to line 314](./src/main.cpp#L293)
This part decides what to do:
  - If there is a car in front of us, should the car change lanes?
  - Does car speed up or slow down?

Based on the prediction of the situation of the car , this code increases or decrease the speed, or makes a lane change when it is safe. Instead of increasing the speed at this part of the code, a `speed_diff` is created to be used for speed changes when generating the trajectory in the last part of the code. This approach makes the car more responsive acting faster to changing situations like a car in front of it trying to apply breaks to cause a collision.

### Trajectory [line 317 to line 416](./src/main.cpp#L313)
This code does the calculation of the trajectory based on the speed and lane output from the behavior, car coordinates and past path points.

First, the last two points of the previous trajectory (or the car position if there are no previous trajectory, lines 321 to 345) are used in conjunction three points at a far distance (lines 348 to 350) to initialize the spline calculation (line 370 and 371). To make the work less complicated to the spline calculation based on those points, the coordinates are transformed (shift and rotation) to local car coordinates (lines 361 to 367).

In order to ensure more continuity on the trajectory (in addition to adding the last two point of the pass trajectory to the spline adjustment), the pass trajectory points are copied to the new trajectory (lines 374 to 379). The rest of the points are calculated by evaluating the spline and transforming the output coordinates to no-local coordinates from lines 388 to 407. The speed change is decided on the behavior part of the code, but it is used in that part to increase/decrease speed on every trajectory points instead of doing it for the complete trajectory from line 393 to 398.
