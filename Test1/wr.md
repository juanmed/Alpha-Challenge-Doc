# Alpha Challenge Test 1: Written Response

##TODO

|Question No|Topic|Detail|
|---|---|---|
|1|State Estimation Challenges| Describe what are the technological challenges regarding state estimation|
|2|Our State Estimation Approach|Describe what is our team's state estimation approach|
|3|Simulator and control algorithm|Describe how simulator will be used in regard to control algorithm|
|4|Details| Add details on research grants and industry collaboration|
|5|All pending||
|NA|Review of answers by team|Yonghee, Eugene please review this file and add info where corresponds.|
|NA|Information Disclose| If we mention support from private or public companies, we must disclose who those companies are and state specifics on what support they provide. Most importantly, is it ok for us and/or these companies?|


## 1) What are the big technical challenges an autonomous drone needs to overcome in order to beat a human pilot flying the same drone? Why?

The challenges comprise various areas and can be understood by how a real race evolves. 

### Race Track Mapping 
Before the race, human and autonomous drones (simply “drone” herein after) need some practice runs.  The challenge for the drone here is “accurate mapping” of the gates pose with respect to an inertial frame, by doing practice runs. If the mapping error is big, this errors will  propagate to motion planning, which if not corrected will lead in the best case to missing gates, and to the drone gone out of the track in the worst case. If the mapping errors are corrected while flying, some succes will be achieved but at the cost of flying non time-optimal paths. This challenge is avoided if the drone is given the precise pose of all gates before the race.

### Perception
After a map is built, and the drone "knows" the checkpoints (gates) it needs to traverse, the next challenge is perception. This challenge refers to the ability of the drone to understand its environment (position of walls, objects, gates, other drones, etc) and its own state (position, velocities and orientation) within it. 

The drone should maximize the use of sensors. Because the purpose of racing is to travel fast, fast and sophisticated sensors are necessary.

In state estimation, the challenge is estimating the state of drone with only onboard sensors and processor. Visual Odometry techniques, in which this state is estimated through with images from a camera, will be particularly challenging. The latter because, if not combined with other specialized sensors (IMU) errors in position, attitude etc, estimation are accumulated until there is lost of the state. High rate, real time state estimation is still a challenge even with the most advanced hardware. Accurate but slow visual odometry has to be combined with inaccurate but fast motion sensors (like IMU), for reasonable performance.

### Computation

Assuming good enough sensors are available, the next challenge is having enough computational power on-board and fast algorithms to process the input from these and accurately estimate where obstacles (walls, objects, other drones, etc) and gates are. The challenge is double: first processing large amounts of data from sensors in real time, and second extracting meaningful information from these data. Processing camera streams at 60Hz with limited on-board computational power is a notoriously demanding.

### Motion Planning 

After this computation is done, the drone will have enough knowledge of the checkpoints and the environment to decide where and how to move. 

The drone now can compute a trajectory that takes it from its current position to the next gate, in the minimum possible time. This is the planning challenge as must be done while flying. 

### Trajectory Generation

As UAV is nonlinear system retrained by nonholonomic constraints(underactuated characteristic) that have 4 inputs and 6 DOF, it would be challenge to generate trajectory for UAV. Trajectory needs to be dynamically feasible and satisfy conditions like minimum or maximum speed. Also considering nonlinear controller for tracking, time-optimal trajectory should be calculated on board in real-time.

### Tracking Control 

If a successful trajectory was computed, the next challenge is to actually fly through that trajectory, or the tracking control challenge. This challenge also comes with its own computation challenge. At high speeds, aerodynamic effects, propeller flapping, vibration etc are of considerable magnitude and if ignored will lead to bad flying performance. The controller needs to compensate for this dynamics in order to track the computed trajectory, or otherwise will end up crashing or missing the next gate. This effects are however non-linear and challenges in modeling them and controlling the drone are outstanding. Moreover, large linear velocity (>10m/s) and fast rotational dynamics (>200degree/s) imply that the control inputs must be computed at very high speeds. Again, having enough on-board computational power and fast algorithms is imperative for success.




## 2) Describe your team’s planned technical approach to AlphaPilot. Why do you think your team could win with this approach?

### Race Track Mapping Challenge Approach

In order to map a DRL style race track, it is necessary to do it by flying the drone itself. We have in our team an skilled human pilot. By leveraging the pilot skills, practice runs in which mapping of the gates and its covariance is obtained becomes feasible. 

### Perception and Computation Challenges Approach
We will be given 60fps cameras and onboard IMU, so the sensor limitation is diminished. To process the camera input fast and efficiently, we will use one-shot convolutional neural network approach, based on state-of-the-art YOLO network. Such approach can process and provide gate detection and gate pose estimation at >60fps.

The algorithmic we will use is SVO(Semi-Direct Visual Odometry). We will be able to have state estimations at rates grater than 20Hz in an NVIDIA Jetson Xavier. 
Our state estimation approach will ....(PENDING)

### Motion Planning 

The motion planning challenge will be approached as follows:  Using the pre-recorded layout of the track and as gate detections become available from the perception sub-system, first a correction of the state layout and correction for state estimation drift will be performed. 

With a corrected estimation of the track layout and the state of the drone, a time-optimal trajectory planning problem will be solved by using the differential-flatness property of the drone dynamics. More especifically we will... (PENDING)  

Although both previous algorithms provide efficiency gains, the limited computational power has to be solved. HOW TO SOLVE COMPUTATIONAL POWER. 

### Trajectory Generation

To reduce computation for trajectory generation, differential flatness theory can be used. Nonlinear dynamics systems could be linearized by selecting the appropriate flat output. If it is possible to find a set of state and input variables as function of output and its finite-order derivative, it is differential flatness system. 

Then Trajectories can be designed in the flatness space by parameterizing the flatness outputs as a series of basis functions with time as the variable. For example, we can set basis functions as Bezier Curve, B-spline, or piecewise polynomials. This polynomials can satisfy the constraints and reduce difficulty of the problem.  And this problem is transformed into a nonlinear programming like quadratic programming

### Tracking Control 

The trajectory tracking challenge will be solved by using a combination of 2 controllers that will be switched online: one differential-flatness based Model Predictive Controller -DFMPC- controller designed around a high-fidelity high-speed model, and a controller policy learned using reinforcement learning and DFMPC guided policy search -RL/DFMPC-. 

The DFMPC controller provides 4 big advantages: First, time-optimal trajectory planning is fast and efficient to compute for  differentially flat systems. Second, the non-linear model prediction problem  in MPC, becomes a linear model prediction problem, which is much simple and faster to solve. Third, perturbance and non-modelled effects can be compensated for by the MPC controller stage. This, combined with the fact that our drone model for control accounts for rotor drag, apparent thrust, battery and rotor dynamics, will allow us considerable gains in tracking performance.  Fourth, since the controller respects the dynamics of the system, these implies computed control inputs are natural to the system, and require lower energy to be performed. This equates to longer fly times. 

The RL/DFMPC controller will be designed around the DFMPC controller, using well known Guided Policy Search. This policy will then learn “imitate” DFMPC. The important advantage of such a policy is that its computation can be much faster than DFMPC. Since both controllers have similar behaviour and will use the same input, it is possible to switch between one and the other in real time. This switching will be done based on the planned maneuver: for  straight parts of the track, in which minimum time travel is critical for winning, but controller commands are not extremely dynamic, the slower but optimal DFMPC controller will be used. But when hard-curves, flips and twists need to be done, and highly-dynamic inputs are required, the system will switch to RLDFMPC controller, which has very high throughput. The switching conditions will be planned for by the motion planning sub-system.


## 3) How does your team plan to use the simulator and development kit provided by DRL? How does your team plan to handle real-world variations that are difficult to capture in simulation? *

### General Approch
The simulator usage will be maximized in the following manners. First, it will be used as test tool for perception, control and state estimation algorithms. Second, it will be used to collect data for model learning (state and inputs of the drone). We explain our approach as follows:

### Specific advantages
We exploit two advantages of simulation: First, unlike real settings, in a simulation ground-truth data can be obtained. Second, the simulation environment characteristics (lighting, occlusions, place, position of objects, winds, object materials etc ) and drone properties (mass, inertia tensor, rotor characteristics, sensor noise, etc ) can be programmatically controlled. These advantages will be maximized as follows. 

### Perception and State Estimation Algorithms

Regarding perception algorithms, the gate pose estimation, provided by the perception sub-system, will be compared against ground-thruth, which can be obtained in simulation. This serves as indispensable feedback to know the estimation error and improve our algorithms. Following, by modifying the environment and drone sensor’s parameters, the robustness of our algorithms can be assesed. On the environment side, by adding occlusions, various lighting, and changing object materials (a simple job in simulation) we get means to understand the weaknesses of our algorithms to changes in the environment and conditions of the gates to be detected. On the drone side, by adding sensor noise both in camera and (add other sensors) it will be possible to find cases in which our algorithm fail to provide gate pose estimations. Finally, by exploiting the very same camera and environment/drone modifications, we can get new training data with which the performance of our algorithms can be improved to overcome the weaknesses found.

In short, the availability of ground truth and flexibility to change the parameters of the simulator, provides means to make fast iterations to improve our algorithm and extend our training data set to include exactly those settings in which our algorithm is weak.

### Trajectory Generation

It is easier to change environment in simulation than real world, so it is efficient to test whether UAV generates trajectory well in given environment. When the competition assumes that most gates are in same altitude and there is no aggressive change in path, it is not that difficulty for trajectory generation. But we change some environment such as gate altitude or angle of path that means level-up. Now then we can check our technical defect or limitation of drone. Also trajectory generation is highly relevant to tracking control so we can check trajectory generation and tracking control simultaneously.

### Tracking Control
Regarding our control algorithms, the simulator, together with the ROS nodes provided for communication, will be exploited as a tool to asses its tracking performance. 

Our estimation algorithms will be highly benefited from the simulator. Since drone pose estimation algorithms will fuse information from several sensors, these algorithms will be thoroughly tested. First, ground-truth data availability and the possibility to add noise to sensors provides means to asses state estimation error for several noise levels. Since the controller works on state estimation data provided by the estimator, this analysis is critical. Second, estimation error and drift, characteristic of long-term state estimation algorithms using IMU only,  will be corrected by gate pose estimation data from the perception system. The algorithms that correct for track layout and state estimation drift can then be improved.

###  Model learning
Still PENDING....


## 4) What support, resources, and tools does your team plan to use for the competition to supplement the hardware, software, and training provided? How do you plan to support necessary travel should you proceed in the competition? Please include any hardware, software, people, data, mentoring, sponsorships, etc. *

We are a group of Mechanical Engineering Graduate School at Sungkyunkwan University. We are doing research on autonomous flying robots at the Robotics and Intelligent Systems Laboratory -RISE Lab- lead by Prof. Dr. Hyung Pil Moon. The autonomous flying robots area is a line of research in this lab. Thus, we count with access to a host of tools present in a robotics lab, and for drone research specifically: Motion Tracking Hardware and Software from Vicon Company, 4 different quadrotors and hexarotors with integrated sensors and computers for research, along with their tools and parts (batteries, monocular cameras, infrared sensors, GPS, barometers, etc), several Nvidia Jetson TX2 development boards, access to computers with Nvidia Titan X Pascal GPUs for neural network training, 

Financing for travelling expenses, extra sw/hw tools, is secured by several research grants and industry-collaboration projects being developed internally by the lab. Among the research grants obtaines so far, BK+ Project and LINC Program have the greater importance. PENDING PROGRAM DETAILS

On the industry-collaboration side, we are in talks with a South Korean and U.S. companies developing services with drones. IF WE HAVE SUPPORT FROM COMPANIES, WE MUST DISCLOSE IT. 


SKKU
BK+ Project
LINC program
Drone companies 
USA Company

## 5) What do you see as the biggest challenges specific to your team that you will face during the AlphaPilot competition? What is your initial plan for overcoming this challenge? For example: missing skills, financial challenges, team cohesiveness, time commitment, etc. *

PENDING
a) Missing skills


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


