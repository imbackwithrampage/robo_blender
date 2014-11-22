# Robo Blender

This provides a ROS node which may be used to control Blender rigs,
and to publish the resulting rig position information (for the neck,
face, eyes).  The position information can then be used to drive motors
(for example).  The https://github.com/hansonrobotics/pau2motors ROS
node will listen to the published messages to control the Einstein
and Dmitry robot heads.

The node is started within Blender, and has direct access to the
Blender context.  It uses different modes to control the rig, with
different modes used for different rigs.

Currently only one rig is supported:
  * Neck rig (included).

The last working version of the Einstein rig is in the branch named
"einstein-dev".

## Pre-requisites
The following packages need to be installed:

    apt-get install python3-yaml python3-rospkg

Caution: python3+ros has issues; see this bug: 
https://github.com/ros-infrastructure/rospkg/issues/71


## Running
To run, start blender, and load the robo.blend head.  Verify that the
ros node has started:  'rostopic list' should show /cmd_blendermode
among other nodes.

## Modes

Listens for /cmd_blender to switch between the different blender nodes.
The currently supported modes are:

### Animations
Plays animations defined within the rig itself (i.e. defined as object
actions).  The list of valid animations can be obtained with the
/animations_list topic.
No error is thrown if an invalid animation name is specified.

##### Topics subscribed:
  * cmd_animations(std_msgs/String) - colon separated string which
    sends command (play,stop, pause) follwed optionaly the animation name.

    For example:
    rostopic pub /cmd_blendermode std_msgs/String play:blah

##### Outputs:
  * full_head - publishes expression neck and eyes information.

### LookAround
Move the head and eyes. Enabled once robot seeks attention. Movements
are defined via a python script in the rig.  The movements are randomly
generated, with eyes moving faster than head.

For example:
   rostopic pub /cmd_blendermode std_msgs/String LookAround

##### Inputs:
  * (none)

##### Outputs
  * neck_euler - publishes neck angle
  * eyes - publishes eyes movements

### Manual Head
Used for animation development and debugging. Allows the designer
to control and move the rig, thus controlling the physical robot head.

##### Outputs:
  * full_head  - publishes expressions for neck and eyes positions.


### TrackDev
Current head tracking topic. Has primary and secondary target that
can be changed during runtime.

##### Topics subscribed:
  * /tracking_action (eva_behavior/tracking_action) - Listens for
    information on objects to track.


#### Inputs
  * pi_vision (RegionOfInterest) - topic publishing ROI  for tracking
  * glancetarget (RegionOfInterest) - topic publishing ROI for glancing

##### outputs
  * neck_euler - publishes neck angle
  * eyes - publishes eyes movements


### Dummy
Idle.

## Inputs
Mainly one input class is currently used:
  * RegionOfInterest - Listens to specified topic for
    sensor_msgs/RegionOfInterest message. Converts it to 3D space
    in blender and updates the blender object position in the blender.
  * Faceshift - allows input from faceshift, changes shapekeys
    of the face mesh.

## Outputs
Outputs are used to publish rig data back to ROS.  Ros topics are
defined in config.

The following outputs are currently used:
  * neck_euler - gets neck's Euler rotation, publishes as
    basic_head_api.msg/PointHead
  * face - gets mesh shapekeys and publishes them as pau2motors/pau.
  * eyes - gets eyes angles and publishes them as pau2motors/pau
  * eyes_beorn - gets Beorn's rig eyes angles and publishes in
    pau2motors/pau message on specified topic.
  * neck_euler_beorn  - gets Beorn's rig neck rotation in Euler angles
    and publishes in basic_head_api.msg/PointHead
  * full_head - combines face, neck_euler_beorn, eyes_beorn outputs to one.

