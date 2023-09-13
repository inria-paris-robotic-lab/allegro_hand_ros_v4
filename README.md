# allegro_hand_ros

Allegro Hand ROS
================================

This is a fork of the official release to control Allegro Hand with ROS Kinetic : [https://github.com/simlabrobotics/allegro_hand_ros_v4](https://github.com/simlabrobotics/allegro_hand_ros_v4). This repo has been heavily modified in order to :
* Patch some tremendous errors (e.g the velocity computation)
* Simplify it
* Follow ROS standard and making it compatible with ros_control
* Provide a simulator that replicate the real robot behavior, using Gazebo (which was easy once the above point was done).

Launch file instructions:
------------------------

There is now a single file, [allegro_hand.launch](src/allegro_hand/launch/allegro_hand.launch) that starts the hand. It takes many arguments, but at a minimum you must specify the handedness:

    roslaunch allegro_hand_controllers allegro_hand.launch HAND:=right

Optional (recommended) arguments:

          NUM:=0|1|...
          RESPAWN:=true|false   Respawn controller if it dies.
          AUTO_CAN:=true|false  (default is true)
          CAN_DEVICE:=/dev/pcanusb1 | /dev/pcanusbNNN  (ls -l /dev/pcan* to see open CAN devices)
          VISUALIZE:=true|false  (Launch rviz)
          GAZEBO:= true|false  To use the simulation or the real robot

Note on `AUTO_CAN`: The script `detect_pcan.py` will automatically finds an open `/dev/pcanusb` file. If instead you specify the can device manually (`CAN_DEVICE:=/dev/pcanusbN`), make sure you *also* specify `AUTO_CAN:=false`. Obviously, automatic detection cannot work with two hands.

The second launch file is for visualization, it is included in `allegro_hand.launch` if `VISUALIZE:=true`. Otherwise, it can be useful to run it separately (with `VISUALIZE:=false`), for example if you want to start rviz separately (and keep it running):

    roslaunch allegro_hand_controllers allegro_viz.launch HAND:=right

Note that you should also specify the hand `NUM` parameter in the viz launch if the hand number is not zero.

Packages
--------

 * **allegro_hand** Contains launchfiles to start everything easily.
 * **allegro_hand_driver** Driver for talking with the allegro hand.
 * **allegro_hand_controllers** Expose the allegro hand to be compatible with ros_control.
 * **allegro_hand_description** xacro descriptions for the kinematics of the
     hand, rviz configuration and meshes.

Note on polling (from Wonik Robotics): The preferred sampling method is utilizing the Hand's own real time clock running @ 333Hz by polling the CAN communication. In fact, ROS's interrupt/sleep combination might cause instability in CAN communication resulting unstable hand motions.


Useful Links
------------

 * [Allegro Hand wiki](http://wiki.wonikrobotics/AllegroHand/wiki).


Controlling More Than One Hand
------------------------------

When running more than one hand using ROS, you must specify the number of the hand when launching.

    roslaunch allegro_hand.launch HAND:=right NUM:=0 CAN_DEVICE:=/dev/pcan0 AUTO_CAN:=false

    roslaunch allegro_hand.launch HAND:=left NUM:=1 CAN_DEVICE:=/dev/pcan1 AUTO_CAN:=false


Known Issues:
-------------

While all parameters defining the hand's motor/encoder directions and offsets fall under the enumerated "allegroHand_#" namespaces, the parameter "robot_description" defining the kinematic structure and joint limits remains global. When launching a second hand, this parameter is overwritten. A fix must be found when the problem will occur.


Installing the PCAN driver
--------------------------

Before using the hand, you must install the pcan drivers. This assumes you have a peak-systems pcan to usb (or pcan to PCI) adapter.

1. Install these packages

    sudo apt-get install libpopt-dev ros-kinetic-libpcan

2. Download latest drivers: http://www.peak-system.com/fileadmin/media/linux/index.htm#download

Install the drivers:

    make clean; make NET=NO_NETDEV_SUPPORT
    sudo make install
    sudo /sbin/modprobe pcan

Test that the interface is installed properly with:

     cat /proc/pcan

You should see some stuff streaming.

When the hand is connected, you should see pcanusb0 or pcanusb1 in the list of
available interfaces:

    ls -l /dev/pcan*

If you do not see any available files, you may need to run:

    sudo ./driver/pcan_make_devices 2

from the downloaded pcan folder: this theoretically creates the devices files if the system has not done it automatically.

3. Build the sources

    catkin_make
    source devel/setup.bash

4. quick start
    cd src/allegro_hand_controllers/launch
    roslaunch allegro_hand.launch HAND:=right

