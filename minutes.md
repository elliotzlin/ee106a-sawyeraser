# Minutes
A log of all of our meetings

## 3 November 2017
### 1000-1500
* Set up git repository
* Figured out how to track AR tags using Baxter's head camera
  * Edit the ```pr2_indiv_no_kinect.launch``` file
  * Set ```cam_image_topic``` to ```/cameras/head_camera/image```
  * Set ```cam_info_topic``` to ```/cameras/head_camera/camera_info```
  * Set ```output_frame``` to ```/head_camera```
* Did a ```rostopic echo``` on the ```/ar_pose_marker``` to see the positions
  * When you hold a tag in front of the camera, you see the Cartesian coordinates publishe to this topic

## 6 November 2017
### 1000-1600
* Followed Lab 4 to git clone the ar_track_alvar directory into our workspace and and the appropriate changes that we discovered on Nov.3 2017 
* Also decided to do some research on how to move the arms of Baxter and discovered that 
* It would be best to first plan with MoveIt and see if we can gain any helpful resources. 
* We came into the lab and started to test out the ar_track_alvar files and check to see if everything we did was working correctly. The when we echoed the ar_track pose, we seemed to be getting the position of the AR tag with repect to head_camera
* Although the MoveIt interface provided a useful insight to what we need to do, we realized that the best approacht that we could do is refer back to Lab 5 and use Inverse Kinematics. We borrowed code form the lab and want use the infromation that we gather from the AR tags in order to make the Baxter move into a specified location. That location is believed to be the location of the AR tag in space. However, over inverse kinematics is still a work in progress. 

### 1930-2115
* Figured out how to debug tf
* Running ```rosrun tf tf_echo base reference/head_camera``` works
  * Emphasis on the ```reference/head_camera``` instead of just ```head_camera```
* Took a dinner break at Snack Shack

### 2200-0130
* Rebooted Baxter because we thought there was something off with the configuration
  * Rebooting seemed to help with the RViz; can display RobotModel w/out errors
* NOTE: Baxter can only run two cameras at once due to bandwidth limitations
  * Ran into this issue after rebooting Baxter
  * Run ```rosrun baxter_tools camera_control.py -l``` to list cameras
  * Run ```rosrun baxter_tools camera_control.py -o <camera name>``` to open ```<camera name>```
  * Run ```rosrun baxter_tools camera_control.py -c <camera name>``` to close ```<camera name>```
* Decided to switch to Sawyer robot due to wonkiness of Baxter's head camera
  * NOTE: Sawyer can only run one camera at a time due to bandwidth limitations
  * Still unsure how to use head_camera; ran ```rosrun intera_examples camera_display.py -c head_camera``` to force the head_camera to work
* Tested ar_track_alvar on the Sawyer
  * Dim lights introduced noise in the tracking, but the translation was otherwise accurate
  * Very accurate detection; we'll probably switch to the Sawyer
* Started testing the inverse kinematics
  * Not going well so far; keeps aborting

## 7 November 2017
### 2100-2359
* NOTE Sawyer cameras are on topics ```/io/internal_camera/*```
* Because this is getting annoying, cheat sheet for setting up inverse kinematics
  * Make sure robot is enabled
  * Run ```rosrun intera_interface joint_trajectory_action_server.py```
  * Run ```roslaunch sawyer_moveit_config move_group.launch electric_gripper:=true```
  * Use ```rosrun tf tf_echo head_camera right_wrist``` or something to check positions
* Discovered consistent f-up in inverse kinematics
  * Always get error ```/sdk_position_w_id_joint_trajectory_action_server_right: Exceeded Error Threshold on right_j5: <some number>```
* Yuge bug, found that our planning frame was actually ```/base``` and not desired ```/head_camera```
  * Put ```print(group.get_planning_frame())``` in ik node
* CORRECTION we actually had to set ```group.set_pose_reference_frame('head_camera')```

## 8 November 2017
### 0000-0115
* Overflow from yesterday night's session
* 0025, we have successfully verified our inverse kinematics solution
* 0033, we have successfully passed checkpoint 1; inverse kinematics to a pose specified by the AR tag
* Found a straight edge; AR tags actually 5.5 centimeters
* Recorded a demo of checkpoint 1

## 9 November 2017
### 1415-1700
* Identified 'torso' joint to constrain, ~~```right_torso_itb```~~ ```right_l0```
* When echoing joint states, joint of interest in ```right_j0```
* Determined two main changes from talking with Valmik:
  1. Instead of constraining the links of the Sawyer we will be constraining the angle in which the joint could rotate. 
  1. Instead of calculating the transformation with respect to the head_camera we will now be calculating with respect to the body.
* We are now creating a node and package that will tie everything together nicely. 

## 10 November 2017 
### 0000-0400
* Successfully completed AR tag transformations in```ik_node```
  * Creates a service to MoveIt
  * Creates a subscriber to the ```ar_pose_marker``` topic
  * Calculates and prints the most recent transformed marker pose on user command
  * Finished running all nodes and launch files into one launch file

## 10 November 2017 
### 1500-1900 
* Working locally on making the IK prefrom the entire process at the push of the enter key
* Making sure that GIT properly shows the name of the individual whom commited.
* Finally made a file that runs everything and reached CHECKPOINT 1 ... AGAIN ... 
* Alan is down. Long live Ada!
* Need to continue our work, making sure that the robot does not spin out of control 

## 12 November 2017 
### 2300-0100
* Confirmed that Ada has serious error in finding the depth of an AR tag. Task at hand is to stop Ada from rotating camera away from tag.
* https://docs.ros.org/kinetic/api/moveit_tutorials/html/doc/pr2_tutorials/planning/src/doc/move_group_interface_tutorial.html#planning-with-path-constraints This document might help us understand how to set some path constraints.
* The IK is off by approxiamtely the legnth of the wrist 

## 13 November 2017 
### 1900-0200
* Will begin testing whether or no the changes had an effect 
* Found a huge problem when trying to run any type of motion planning with sawyer. You need to make sure that the electric_gripper:=true or else you will get a collision error. This error exsisted in the launch file. 
* Decided to remove the following from the launch file 
* ```<arg name="electric_gripper" value="true" />```
* ```<include file="$(find sawyer_moveit_config)/launch/sawyer_moveit.launch" />```
* Trying to figure out how to set some path constraints
* Successfully reached CHECKPOINT 1 (actually)
  * Torso joint successfully constrained
  * Still notice path planning is difficult, but to be expected

## 14 November 2017
### 1900-0200
* Figured out how to add scene objects to further constraint path
* Created a SolidPrimitive block to model board
* Verified that Sawyer is indeed able to wipe the board with MoveIt GUI
* Switched to using a larger AR tag to improve tracking accuracy
  * Noted an improvement in our path planning
* "Fixed" orientation of gripper (actually it's only good for near vertical boards)
* Began attempting to transform board objects dynamically

## 15 November 2017
### 1030-1700
* Arrived at decision to afix AR tag to wall
  * Biggest implication being we are assuming a static wall
  * Assuming a static wall, we don't have to update our board collision object
  * In addition, we can affix the desired orientation of our gripper, potentially improving our IK performance
* Noted bias of +0.07 for our head_camera's depth perception
* Magic numbers set for wall
  * Achieves very close results
* Talk with Laura
  * Global variables into parameter server (lab 8)
  * Logic code goes outside of code that defines publishers and subscribers (look at Occupancy Grid)
* We set the bias of new AR Tag, set Cartesian Path to make sure that the gripper move parallel to the floor, and more importantly it moves in a straight line
* Bottom Line: The robot can swipe.

### 2300-0145
* Programmed routine to swipe an area bounded by a rectangle

## 16-17 November 2017
### 1900-0800
* Commenced work on modularizing board swiping function
* Commenced work on openCV module of project
* Whipped up FAST feature detection to detect AR tag; ended up abandoning it in favor of a square finding routing from an OpenCV example
* Broke for lunch at 0430 at Denny's
  * Sophia was able to join us for the meal as well

## 22 November 2017
### 0415-1910
* Successfully calculated homography transformation matrix
  * By successfully I mean the function call didn't error out
* Checked the accuracy of tf transformations from base
  * Concluded that it's "good enough"
  * Also deduced that a tape measure would really come in handy
* Started working on separate callback that would detect markers on the whiteboard and bound it with rectange
* Successfully implemented routine that would bound a collection of points with a rectangle
* Action items for next work session:
  * Verify edge lengths of rectangle to verify homography
  * Publish coordinates of this bounding rectangle
  * Read from topic in ```ik_node```
