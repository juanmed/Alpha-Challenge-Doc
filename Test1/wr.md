#TODOHELLO

|Question No|Topic|Detail|
|---|---|---|
|1|State Estimation Challenges| Describe what are the technological challenges regarding state estimation|
|2|Our State Estimation Approach|Describe what is our team's state estimation approach|
|3|Simulator and control algorithm|Describe how simulator will be used in regard to control algorithm|
|4|Details| Add details on research grants and industry collaboration|
|5|All pending||
|NA|Review of answers by team||
|NA|||


## 1) What are the big technical challenges an autonomous drone needs to overcome in order to beat a human pilot flying the same drone? Why?

The challenges comprise various areas and can be understood from a race perspective. Before the race, human and autonomous drone (simply “drone” herein after) need some practice runs. This challenge is avoided if the drone is given the precise pose of all gates before the race. Otherwise, the challenge for the drone here is “accurate mapping” of the gates pose with respect to an inertial frame, by doing practice runs. If the mapping error is big, this errors will  propagate to motion planning, which if not corrected will lead to missing gates, or if corrected will lead to non time-optimal paths. After a map is built, and the drones start to race, the next challenge is motion planning. This can be divided in two challenges: perception, planning and computation. The challenge in perception is for the drone to be able to accurately detect where the gates are as it flies through the race track, using a camera and other sensors. Because the speed are very high, a fast and sophisticated camera is necessary. Assuming such a camera is available, the next challenge is having enough computational power on-board and fast algorithms to process the input from the camera to accurately estimate where obstacles (walls, objects, other drones, etc) and gates are. When this computation is done, the drone now can compute a trajectory that takes it from its current position to the next gate, in the minimum possible time. All this while flying! If a successful trajectory can be computed, the next challenge is to actually fly through that trajectory, or the tracking control challenge. This challenge also comes with its own computation challenge. At high speeds, aerodynamic effects, propeller flapping, vibration etc are of considerable magnitude. The controller needs to compensate for this dynamics in order to track the computed trajectory, or otherwise will end up crashing or missing the next gate. This effects are however non-linear and challenges in modeling and controlling are outstanding. Moreover, as if a velocity of 30m/s was not enough, the rotation dynamics are even faster (>200degree/s). This implies that the control inputs must be computed at very high speeds. Again, having enough on-board computational power and fast algorithms is imperative.  THE STATE ESTIMATION CHALLENGE.

Tracking mapping
Motion planning
Tracking Control
Vision
Computation 
Tracking Drones

## 2) Describe your team’s planned technical approach to AlphaPilot. Why do you think your team could win with this approach?

In order to map a DRL style race track, it is necessary to do it by flying the drone itself. We have in our team an skilled human pilot. By leveraging the pilot skills, practice runs in which mapping of the gates and its covariance is obtained becomes feasible. The motion planning challenge will be approached as follows: We will be given 60fps cameras, so the sensor limitation is diminished. To process the camera input fast, we will use one-shot convolutional neural network approach, based on state-of-the-art YOLO network. Such approach can process and provide gate detection and gate pose estimation at >60fps. Using the pre-recorded layout of the track and gate detections provided by the perception sub-system, a time-optimal trajectory planning problem will be solved by using the differential-flatness property of the drone dynamics WRITE APPROACH.     Although both previous algorithms are efficient, the limited computational power has to be solved. HOW TO SOLVE COMPUTATIONAL POWER. The trajectory tracking challenge will be solved by using a combination of 2 controllers that will be switched online: one differential-flatness based MPC -DFMPC- controller designed around a high-fidelity high-speed model, and a controller policy learned using reinforcement learning and DFMPC guided policy search -RLDFMPC-. The DFMPC controller provides 4 big advantages: First, time-optimal trajectory planning is fast and efficient to compute for  differentially flat systems. Second, the non-linear model prediction problem  in MPC, becomes a linear model prediction problem, which provides big efficiency gains in computation. Third, perturbance and non-modelled effects can be compensated for the the MPC controller stage. Fourth, the controller respects the dynamics of the system, which can then preciselly follow the trajectory. The RLDFMPC controller will designed around the DFMPC controller. Using well know Guided Policy Search algorithm in RL, we will learn a control policy which will be guided by DFMPC. This policy will then “imitate” DFMPC. The important advantage of such a policy is that its computation time is very fast, in the order of 10s of microseconds. Since both controllers have similar behaviour and will use the same input, it is possible to switch between one and the other in real time. This switching will be done based on the planned maneuver: for  straight parts of the track, in which minimum time travel is critical for winning, but controller commands are not extremely dynamic, the slower but optimal DFMPC controller will be used. But when hard-curves, flips and twists need to be done, and highly-dynamic inputs are required, the system will switch to RLDFMPC controller, which has very high throughput. STATE ESTIMATION APPROACH.



## 3) How does your team plan to use the simulator and development kit provided by DRL? How does your team plan to handle real-world variations that are difficult to capture in simulation? *

The simulator provided usage will be maximized in the following manners. First, it will be used as test tool for perception, control and state estimation algorithms. Second, it will be used to collect data for model learning (state and inputs of the drone). We explain our approach as follows:

We exploit two advantages of simulation: First, unlike real settings, in a simulation ground-truth data can be obtained. Second, the simulation environment characteristics (lighting, occlusions, place, position of objects, winds, object materials etc ) and drone properties (mass, inertia tensor, rotor characteristics, sensor noise, etc ) can be programmatically controlled. This advantage will be maximized as follows. 

Regarding perception algorithms, the gate pose estimation, provided by the perception sub-system, will be compared against ground-thruth, which can be obtained in simulation. This serves as indispensable feedback to know the estimation error and improve our algorithms. Following, by modifying the environment and drone sensor’s parameters, the robustness of our algorithms can be assesed. First, by adding occlusions, various lighting, and changing object materials (a simple job in simulation) we get means to understand the weaknesses of our algorithms to changes in the environment and conditions of the gates to be detected. Next, by adding sensor noise both in camera and (add other sensors) it will be possible to find cases in which our algorithm fail to provide gate pose estimations. Finally, by exploiting the very same camera and environment/drone modifications, we can get new training data with which the performance of our algorithms can be improved to overcome the weaknesses found.

In short, the availability of ground truth and flexibility to change the parameters of the simulator, provides means to make fast iterations to improve our algorithm and extend our training data set to include exactly those settings in which our algorithm is weak.

Regarding our control algorithms, the simulator, together with the ROS nodes provided for communication, will be exploited as a tool to asses its tracking performance. 

Our estimation algorithms will be highly benefited from the simulator. Since drone pose estimation algorithms will fuse information from several sensors, these algorithms will be thoroughly tested. First, ground-truth data availability and the possibility to add noise to sensors provides means to asses state estimation error for several noise levels. Since the controller works on state estimation data provided by the estimator, this analysis is critical. Second, estimation error and drift, characteristic of long-term state estimation algorithms using IMU only,  will be corrected by gate pose estimation data from the perception system. 

Finally, 

## 4) What support, resources, and tools does your team plan to use for the competition to supplement the hardware, software, and training provided? How do you plan to support necessary travel should you proceed in the competition? Please include any hardware, software, people, data, mentoring, sponsorships, etc. *

We are a group of Mechanical Engineering Graduate School at Sungkyunkwan University. We are doing research on autonomous flying robots at the Robotics and Intelligent Systems Laboratory -RISE Lab- lead by Prof. Dr. Hyung Pil Moon. The autonomous flying robots area is a line of research in this lab. Thus, we count with access to a host of tools present in a robotics lab, and for drone research specifically: Motion Tracking Hardware and Software from Vicon Company, 4 different quadrotors and hexarotors with integrated sensors and computers for research, along with their tools and parts (batteries, monocular cameras, infrared sensors, GPS, barometers, etc), several Nvidia Jetson TX2 development boards, access to computers with Nvidia Titan X Pascal GPUs for neural network training, 

Financing for travelling expenses, extra sw/hw tools, is secured by several research grants and industry-collaboration projects being developed internally by the lab. Among the research grants obtaines so far, BK+ Project, LINC Program have the greater importance

On the industry-collaboration side, we are in talks with a South Korean company developing services with drones. 


SKKU
BK+ Project
LINC program
Drone companies 
USA Company

## 5) What do you see as the biggest challenges specific to your team that you will face during the AlphaPilot competition? What is your initial plan for overcoming this challenge? For example: missing skills, financial challenges, team cohesiveness, time commitment, etc. *

Missing skills


## Trajectory


Trajectory Generation.

1. Chaojie Zhang, Trajectory Generation for Aircraft Based on Differential Flatness and Spline Theory.  ICINA 
In this paper, they make trajectory by using differential flatness spline theory. The spline is based on Bernstein basis Bezier curve. In future, They will use other curves such as B-spline. 

2. Zhen He … A Time-optimal Trajectory Generation Algorithm for Quadrotors with Various States Constraints. ICARM
3. Mellinger D, Kumar V. Minimum Snap Trajectory Generation and Control for Quadrotors in Proceedings of the IEEE international Conference Robotics and Automation. 

In paper 2 and 3, trajectory is calculated by dividing many segments. By minimizing cost function(or objective function), trajectory can be time optimal. 

4. E.Kahale. Minimum Time Reference Trajectroy Generation for an Autonomous Quadrotor. ICUAS
The trajectory generation algorithm proposed in this paper is based on the optimality notion.

Paper 1 seems easier than other things but has no simulation results.
Paper 2,3 would be more time-optimal.
They all use nonlinear programming.
If I can use nonlinear programming in Matlab, can simulate some trajectory.


