#######################
External ROS 2 Packages
#######################

ros2-vicon-receiver
===================
* Written by the `OPT4SMART <http://opt4smart.dei.unibo.it/index.html>`_ group at The University of Bologna, publishes objects tracked by VICON to ROS 2 topics
* Software repository: `ROS2_VICON <https://github.com/OPT4SMART/ros2-vicon-receiver>`_
    * Make sure to follow the installation instructions given in this repository
* This package makes use of the VICON `DataStream SDK <https://www.vicon.com/software/datastream-sdk/>`_, and therefore does not work on some edge-computing devices like the RPi. It must be launched and run on a companion laptop or other computer.


The specific configuration details used by the AREAL lab are given below. The 
configuration details for different motion-capture applications may all vary,
but these are provided to hopefully help out other users of the `GT_IFL <https://ifl.ae.gatech.edu>`_.




