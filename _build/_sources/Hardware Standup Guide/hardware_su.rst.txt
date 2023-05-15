############################
AREAL Hardware Standup Guide
############################

This guide was written using a Raspberry Pi (RPi) 4B with 8Gb RAM. The initial 
RPi setup steps will be easiest if a monitor and keyboard are avaliable for use.
A Holybro Pixhawk 6X with standard baseboard was used as the flight control 
unit (FCU). It is also assumed that the Pixhawk has a compatible telemetry
radio installed. This guide used the `Holybro SiK v3 <https://holybro.com/collections/telemetry-radios/products/sik-telemetry-radio-v3>`_.

Raspberry Pi Companion Computer Setup
=====================================

Ubuntu Image Installation
-------------------------
The first step is to burn a new image onto a fresh microSD using the official 
imager tool. A tutorial is found at the following `IMAGER <https://blog.electroica.com/raspberry-pi-official-imager-burn-raspbian-to-sd-card/>`_.
Use the image 'Ubuntu Server 20.04.5 64-bit', found in the general-purpose OS 
section of the choose operating system menu.

Next, connect the monitor and keyboard to the RPi, put in the microSD card, and 
power it up. The default username and password are both 'ubuntu'. Set a new 
password as prompted. 

Ubuntu Setup
------------

To start out, connect the RPi to an ethernet port for initial internet
connection. WiFi will be setup shortly, but the initial internet connection is 
most easily setup over ethernet.

In order to test the internet connection, use the following command to ping 
google.com:

.. code-block:: console

    ping google.com

Upon establishing an internet connection, the RPi will begin updating in the 
background. This might take a while, but it just has to be allowed to finish.
When the background update is complete, run the following two commands to 
update all of the system packages.

.. code-block:: console

    sudo apt update
    sudo apt upgrade

Adding Optionally-Active Ubuntu Desktop
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We are going to add an install of Ubuntu desktop to the RPi because 
configuration of networks and some other parts of the system is most easily 
done in the GUI interface. 

The guide to install Ubuntu desktop on a server installation is found at the 
following `DESKTOP-INSTALL <https://www.wikihow.com/Install-Gnome-on-Ubuntu>`_. 
Make sure to only install the "minimal" desktop option described in step 4.

Once the desktop is installed, it can be enabled and disabled as per the
instructions at the following `DESKTOP-CONTROL <https://linuxconfig.org/how-to-disable-enable-gui-on-boot-in-ubuntu-20-04-focal-fossa-linux-desktop>`_. 

The command to enable Ubuntu Desktop is:

.. code-block:: console

    sudo systemctl set-default graphical

The command to disable Ubuntu Desktop (change it back to server) is:

.. code-block:: console

    sudo systemctl set-default multi-user

After running either command, reboot the RPi to make the changes take effect.
The command to reboot the RPi is:

.. code-block:: console

    sudo reboot

Network Setup
^^^^^^^^^^^^^
First, enable Ubuntu desktop as described above. Allow any updates to occur and
restart the RPi if prompted. Make sure to decline the upgrade to Ubuntu 22.04
if offered.

Next, install the netplan package. This is done with the following command: 

.. code-block:: console

    sudo apt install net-tools

Next, follow the instructions given in the top answer at this post `WIRED-SETUP <https://askubuntu.com/questions/1189146/raspberry-pi-4b-ubuntu-19-10-wired-network-unmanaged>`_
to enable management of the wired network on the RPi. The changes were 
successful if the menu in the upper right-hand corner says "Wired Connected"
instead of "Wired Unmanaged".

We will cover configuring the wired network for PX4 communications in a later
section on the PX4 setup page. [ADD LINK TO PX4 WIRED SETUP SECTION HERE]

The last step in network setup is to connect to Wifi. Connect to your Wifi
network of choice using the settings managed in the Ubuntu GUI. A guide for 
connecting to eduroam in particular can be found at the following `EDUROAM-CONNECT <https://rose-hulman.microsoftcrmportals.com/knowledgebase/article/KA-01010/en-us>`_. 
A pdf of this webpage is avaliable in the "Raspberry Pi Setup" directory of 
the "AREAL Lab" DropBox folder.

Packages to Install
-------------------
Lists of the packages needed for future steps are shown below. It is recommended
that they all be installed at this point.


Software Development Tools
^^^^^^^^^^^^^^^^^^^^^^^^^^

**Git**

.. code-block:: console

    sudo apt-get install git

ROS 2 Installation
------------------
We will make use of the ros_menu tool provided by Adlink to install ROS, ROS 2,
and manage sourcing enviroments. In order to install ros_menu, follow the 
instructions under the "Install" heading at the following `ROS_MENU <https://github.com/Adlink-ROS/ros_menu>`_. 

Next, run the following command to update dependencies

.. code-block:: console

    rosdep update

The configurations for ROS 2 are controlled in the "config.yaml" file in the 
"ros_menu" directory. A few changes the default for "config.yaml" need to be 
made. Delete all of the lines below "cmds" unswer the ROS 2 heading, and before
the ROS 2/ROS 1 bridge heading. In their place add the source commands for your
workspace(s). The specific source command for our vehicle will be covered in
the section on downloading and building the needed ROS 2 packages.

Hardware Setup
--------------
The RPi takes external power from a 5V power supply, with a current capacity of 
up to 3A. We use a 5V, 3A battery elimination circuit (BEC) attached to the UAV
power distribution board (PDB). The pinout diagram for the RPi 40-pin header is 
found at the following `RPi_PINS <https://www.raspberrypi.com/documentation/computers/raspberry-pi.html>`_. 
The 5V and GND wires should be connected to pins 2/4 and 6 respectively.

Pixhawk Setup
=============
The first step in configuring the FCU is to follow the standard PX4 
installation and configuration for a quadcopter. The PX4 configuration guide
can be found at the following `PX4_CONFIG <https://docs.px4.io/main/en/config/>`_.
A specific configuration guide for the Holybro X500 v2 can be found at 
`X500_CONFIG <https://docs.px4.io/main/en/frames_multicopter/holybro_x500V2_pixhawk5x.html>`_.

After completing the standard PX4 setup, manual flights should be performed to 
verify the quadcopters overall operation. 

The next step is to connect the PX4 and companion computer over ethernet. A 
guide for this configuration can be found at `PX4_ETH <https://docs.px4.io/main/en/advanced_config/ethernet_setup.html>`_.
At this point, the RPi and Pixhawk should be connected with the included
ethernet cable. 

PX4 Parameters for VICON Camera Feedback
========================================
In order to configure the Pixhawk for state feedback from a motion capture
system (in our case VICON `GT_IFL <https://ifl.ae.gatech.edu>`_), some
parameters need to be modified. The list of modified parameters is as follows
(battery, logging parameters have been omitted):

* ``COM_DISARM_PRFLT``: 30 
* ``COM_FLTMODE1``: 0
* ``COM_FLTMODE4``: 2
* ``COM_FLTMODE6``: 7
* ``COM_RCL_EXCEPT``: 4
* ``COM_RC_IN_MODE``: 0
* ``COM_RC_OVERRIDE``: 3
* ``EKF2_EV_CTRL``: 3
* ``EKF2_EV_DELAY``: 0.1
* ``EKF2_EV_NOISE_MD``: 1
* ``EKF2_GPS_CHECK``: 240
* ``EKF2_GPS_CTRL``: 4
* ``EKF2_HGT_REF``: 3
* ``EKF2_IMU_CTRL``: 1
* ``MAN_ARM_GESTURE``: 0
* ``MAV_0_RATE```: 57600
* ``MAV_1_CONFIG``: 102
* ``MIS_TAKEOFF_ALT``: 0.9
* ``NAV_RCL_ACT``: 3
* ``PWM_AUX_FUNC1``: 101
* ``PWM_AUX_FUNC2``: 102
* ``PWM_AUX_FUNC3``: 103
* ``PWM_AUX_FUNC4``: 104
* ``RTL_RETURN_ALT``: 0.6
* ``SENS_BOARD_X_OFF``: 3.209
* ``SENS_BOARD_Y_OFF``: -0.204
* ``SYS_AUTOSTART``: 4015
* ``XRCE_DDS_0_CFG``: 1000

ROS 2 FCS Packages
==================

One of the solutions adopted by the lab for autonomous flight control is the 
ROS 2-PX4 framework. Therefore, the required ROS 2 packages for basic 
autonomous flight are detailed below. These are the packages needed for 
autonomous waypoint traversal, either from onboard sensor data or motion-
capture position feedback. Links to the detailed documentation pages for the 
various packages mentioned here are linked inline. All ROS 2 code is currently
assumed to be in Foxy.

Basic Autonomous Operation (Required for all)
---------------------------------------------
* ``areal_landing_uav_interfaces``: Provides interface definitions for the required actions and services
    * Documentation page: :ref:`areal_landing_uav_interfaces`
    * Software repository: `AREAL_INTERFACE <https://github.com/AREAL-GT/areal_landing_uav_interfaces>`_
* ``areal_landing_px4_communication``: Provides ROS 2 actions, servers, publishers, and subscribers for communication and control with a PX4 FMU 
    * Documentation page: :ref:`areal_landing_px4_communication`
    * Software repository: `AREAL_PX4_COMMS <https://github.com/AREAL-GT/areal_landing_px4_communication>`_

Motion Capture VICON Feedback (Required for Motion-Capture Flights)
-------------------------------------------------------------------
* ``ros2-vicon-receiver``: Written by the `OPT4SMART <http://opt4smart.dei.unibo.it/index.html>`_ group at The University of Bologna, publishes objects tracked by VICON to ROS 2 topics
    * AREAL lab-specific configuration details: :ref:`ros2-vicon-receiver`
    * Software repository: `ROS2_VICON <https://github.com/OPT4SMART/ros2-vicon-receiver>`_

Waypoint Behavior Tree (Required for all)
-----------------------------------------
* ``areal_new_bt``: Implementation file for the flight control behavior tree with ROS 2
    * Documentation page: :ref:`areal_new_bt`
    * Software repository: `AREAL_BT <https://github.com/AREAL-GT/areal_new_bt>`_

Packages from PX4 (Required for all)
------------------------------------
* ``px4_msgs``: Software repository `PX4_MSGS <https://github.com/PX4/px4_msgs>`_ 
* ``px4_ros_com``: Software repository `PX4_ROS_COM <https://github.com/PX4/px4_ros_com>`_ 
* ``micro_ros_agent``: Software repository `uROS_AGENT <https://github.com/micro-ROS/micro-ROS-Agent>`_

In order to perform autonomous UAV flights, clone or download the needed
repositories into a workspace onboard the RPi and build them. If using motion-
capture, make sure to download the ros2-vicon-receiver package into a seperate
workspace on a laptop to be used on the same WiFi network as the computer 
running the motion-capture (VICON) system. Also download another copy of the
areal_landing_px4_communication package to the same computer-based repository.
Both of these packages are needed to stream VICON positon feedback over 
appropriate ROS 2 topics.

Now that the UAV has been set up, operational instructions for running 
motion-capture autonomous flight experiments are given in :ref:`Motion-Capture Experimental Instructions`.




