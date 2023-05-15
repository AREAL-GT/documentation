########################################
Motion-Capture Experimental Instructions
########################################

All of the flight-tests or experiments on this page are run using a 
motion-capture system. In specific, the descriptions are for a VICON system. It
is also assumed that the setup instructions from :ref:`AREAL Hardware Standup Guide`
have been followed.

VICON-ROS 2-PX4 Setup
=====================
The following sections detail how to get VICON motion-capture positon feedback 
onboard the Pixhawk PX4 FMU. This position feedback is routed into EKF2 in 
PX4, and could be used for either manual position-control flight or autonomous
offboard control.

The approach presented on this page and in the :ref:`AREAL Hardware Standup Guide`
is generally based on the `PX4 Motion Capture Documentation <https://docs.px4.io/main/en/ros/external_position_estimation.html>`_,
however some additional modifications of PX4 parameters were required, and ROS 2
was used.

VICON UDP Broadcast
-------------------
The particular VICON settings used are as follows (all in the UDP Object Stream 
section):

* Box next to "Enabled:" is checked
* Data Block Size: 256
* Box next to "Object per port:" is not checked
* IP Address: 192.168.10.2
* Port: 51001

ROS 2 VICON Pub-Sub Node Launch
-------------------------------
In order to start broadcasting the motion-capture over ROS 2, a launch file in 
the areal_landing_px4_communication package will be used. Make sure 
ros2-vicon-receiver, areal_landing_px4_communication, and px4_msgs are in a 
workspace together. Also make sure the subscriber name in px4_vicon_pubsub.py 
has been changed to whatever ros2-vicon-receiver is publishing. This can be 
checked by just running the ROS 2 reciever as a test. Finally, make sure the
computer-side workspace has been built.

Next, a launch file will start both the ROS 2 reciever node and the pub-sub 
node that formats the VICON data into the proper message for PX4. Use the 
command:

.. code-block:: console

    ros2 launch areal_landing_px4_communication areal_landing_vicon.launch.py

.. note::

    (5/8/2023) The members of our lab who have been flight testing more
    recently are out of town for a few weeks to start off the summer. The 
    method they used to launch the VICON nodes is not totally clear, so 
    a launch file was put together for this documentation guide. This launch
    file has not been thoroughly tested, but it should give a close indication
    of the proper syntax. This note will be updated as soon as the VICON
    launch file has been tested.

At this point, motion-capture position and orientation data should be 
broadcast on the ROS 2 topic vehicle_visual_odometry. 

ROS 2-PX4 Bridge Startup
------------------------
A good test to run at this juncture to make sure the system is working is to 
ssh onto the UAV-based RPi and list the ROS 2 topics. The same 
vehicle_visual_odometry topic should be visible, and show some data when echoed.

In order to start the ROS 2-PX4 bridge, first some commands will be run through
the PX4 terminal in QGroundControl (over telemetry radio link). The instance of
the microdds client that runs on PX4 on startup uses a serial connection by 
default, but we have a connection over ethernet. Therefore, the first command
to run in the PX4 terminal is:

.. code-block:: console

    microdds_client stop

Then run the following in the PX4 terminal:

.. code-block:: console

    microdds_client start -t udp -h <IP of RPi set in earlier step>

Finally, open up an ssh connection to the RPi and run:

.. code-block:: console

    ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888

A list of all of the publishers and subscribers for the ROS 2-PX4 bridge should
print out, meaning the bridge is active.

This is a good junture to run any final tests desired to check system operation.
For example, the UAV can be moved around the motion-capture arena while a 
topic giving the UAV pose estimation from EKF2 is printed. The UAV local
position should match that given by the motion-capture.

Waypoint Navigation
===================
Finally, the autonomous flight experiments can be run at this point. Run
whatever autonomous flight tests are desired, but the instructions for 
waypoint travesal with the ROS 2 behavior tree are given below.

First, open up an ssh terminal to the RPi and call the following launch file:

.. code-block:: console

    ros2 launch waypoint_action_server.launch.py

Next, run the behavior tree. This is done with the command below. **Warning**,
the UAV will move immediately upon running this command.

.. code-block:: console

    ros2 run areal_new_bt new_bt

Example videos and flight logs from autonomous indoor motion-capture flights
performed with this framework can be found in the flight-test demonstration
repository described in :ref:`Basic Indoor Motion-Capture Flights`

