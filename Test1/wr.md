# Alpha Challenge Test 1: Written Response

##TODO

|Question No|Topic|Detail|
|---|---|---|
|4|LINC Project| Describe how is RISE Lab involved in this. |
|2||Describe what is our team's state estimation approach|
|3|Simulator and control algorithm|Describe how simulator will be used in regard to control algorithm|
|4|Details| Add details on research grants and industry collaboration|
|5|All pending||
|NA|Review of answers by team|Yonghee, Eugene please review this file and add info where corresponds.|
|NA|Information Disclose| If we mention support from private or public companies, we must disclose who those companies are and state specifics on what support they provide. Most importantly, is it ok for us and/or these companies?|


# V0.1

## 1) What are the big technical challenges an autonomous drone needs to overcome in order to beat a human pilot flying the same drone? Why?

The challenges comprise various areas and can be understood by how a real race evolves. 

### Race Track Mapping 
Before the race, human and autonomous drones (simply -drone- herein after) need some practice runs.  The challenge for the drone here is accurate mapping of the gates pose with respect to an inertial frame. This gate mapping is done by having practice runs. If the mapping error is big, this errors will  propagate to motion planning, which if not corrected will lead to missing gates in the best case, and to the drone going out of the track in the worst case. If the mapping errors are corrected while flying, some succes will be achieved but at the cost of flying non time-optimal paths, since the drone will need to be searching for gates to fly. This challenge is avoided if the drone is given the precise pose of all gates before the race.

### Perception
After a map is built, and the drone knows the checkpoints (gates) it needs to traverse, the next challenge is perception. This challenge refers to the ability of the drone to understand its environment (position of walls, objects, gates, other drones, etc) and its own state (position, velocities and orientation) within it. Perception has two components: sensors and algorithms. Because the purpose of racing is to travel fast, fast and sophisticated sensors are necessary. Having such sensors is the first challenge.

In state estimation, the challenge is estimating the state of drone with only onboard sensors and processor. Visual Odometry techniques, in which this state is estimated through with images from a camera, will be particularly challenging. The latter because, if not combined with other specialized sensors (IMU), errors in position, attitude etc, estimation are accumulated until there is lost of the state. High rate, real time state estimation is still a challenge even with the most advanced hardware. Accurate but slow visual odometry has to be combined with inaccurate but fast motion sensors (like IMU), for reasonable performance. 

### Computation

Assuming good enough sensors and algorithms are available, the next challenge is having enough computational power on-board and fast algorithms to process the input from these and accurately estimate where obstacles (walls, objects, other drones, etc) and gates are. The challenge is double: first processing large amounts of data from sensors in real time, and second extracting meaningful information from these data. Processing camera streams at 60Hz with limited on-board computational power is a notoriously demanding task for modern processors.

### Trajectory Generation

After this computation is done, the drone will have enough knowledge of the checkpoints and the environment to decide where and how to move. The drone now can compute a trajectory that takes it from its current position to the next gate, in the minimum possible time. This is the planning challenge as must be done while flying. As UAV is nonlinear system retrained by nonholonomic constraints(underactuated characteristic) that have 4 inputs and 6 DOF, it would be challenge to generate trajectory for UAV. Trajectory needs to be dynamically feasible and satisfy conditions like minimum or maximum rotor speed. Also considering nonlinear controller for tracking, time-optimal trajectory should be calculated on board in real-time.

### Tracking Control 

If a successful trajectory was computed, the next challenge is to actually fly through that trajectory, or the tracking control challenge. This challenge also comes with its own computation challenge. At high speeds, aerodynamic effects, propeller flapping, vibration, etc. are of considerable magnitude and if ignored will lead to sub-optimal flying performance. The controller needs to compensate for this dynamics in order to track the computed trajectory, or otherwise will end up crashing or missing gates. This effects are however non-linear and challenges in modeling them and controlling the drone are outstanding. Moreover, large linear velocity (>10m/s) and fast rotational dynamics (>200degree/s) imply that the control inputs must be computed at very high speeds. Again, having enough on-board computational power and fast algorithms is imperative for success.

<div style="page-break-after: always;"></div>

## 2) Describe your team's planned technical approach to AlphaPilot. Why do you think your team could win with this approach?

### Race Track Mapping Challenge Approach

In order to map a DRL style race track, it is necessary to do it by flying the drone itself. We have in our team an skilled human pilot. By leveraging the pilot skills, practice runs in which mapping of the gates and its covariance is obtained becomes feasible. 

### Perception, Computation and Trajetory Generation Challenges Approach
We will be given 60fps cameras and onboard IMU, so the sensor limitation is diminished. To process the camera input fast and efficiently, we will use one-shot convolutional neural network approach, based on state-of-the-art YOLO network. Such approach can process and provide gate detection and gate pose estimation at >60fps.

We will use SVO(Semi-Direct Visual Odometry) as our algorithm for localization and mapping of the environment. SVO is an algorithm gathering advantages of direct and feature-based visual odometry. It extracts features of selected keyframes in a parallel thread. Also it requires a sparse (not full) reconstruction of the environment. This allows for fast and accurate drone state estimation. We predict state estimation speeds greater than 20Hz in an NVIDIA Jetson Xavier.

Following, using the pre-recorded layout of the track and as gate detections become available from the perception sub-system, first a correction of the state layout and correction for state estimation drift will be performed. With a corrected estimation of the track layout and the state of the drone, a time-optimal trajectory planning problem will be solved by using the differential-flatness property of the drone dynamics. This property allows to get a linear system from the non linear dynamics, which also alliviates computational power constraints.

Then trajectories can be designed in the flatness space by parameterizing the flatness outputs as a series of basis functions with time as the variable. For example, we can set basis functions as Bezier Curve, B-spline, or piecewise polynomials. These polynomials can ensure enforceability and reduce difficulty of the problem. Moreover, these curves will be constructed by solving a minimization problem of an objective function parameterized on the time variable. With this objective function and constraints, problem is transformed into a nonlinear programming like quadratic programming.

Although both previous algorithms provide efficiency gains, the limited computational power has to be solved. HOW TO SOLVE LIMITED COMPUTATIONAL POWER.... (PENDING)  

### Tracking Control 

The trajectory tracking challenge will be solved by using a combination of 2 controllers that will be switched online: one differential-flatness based Model Predictive Controller -DFMPC- controller designed around a high-fidelity high-speed model, and a controller policy learned using reinforcement learning and DFMPC guided policy search -RL/DFMPC-. 

The DFMPC controller provides 4 big advantages: First, time-optimal trajectory planning is fast and efficient to compute for  differentially flat systems. Second, the non-linear model prediction problem  in MPC, becomes a linear model prediction problem, which is much simple and faster to solve. Third, perturbances and non-modelled effects can be compensated for by the MPC controller stage. This, combined with the fact that our drone model for control accounts for rotor drag, apparent thrust, battery and rotor dynamics, will allow us considerable gains in tracking performance.  Fourth, since the controller respects the dynamics of the system, these implies computed control inputs are natural to the system, and require lower energy to be performed. This equates to longer fly times. 

The RL/DFMPC controller will be designed around the DFMPC controller, using well known Guided Policy Search. This policy will then learn to -imitate- DFMPC. The important advantage of such a policy is that its computation can be much faster than DFMPC. Since both controllers have similar behaviour and will use the same input, it is possible to switch between one and the other in real time. This switching will be done based on the planned maneuver: for  straight parts of the track, in which minimum time travel is critical for winning, but controller commands are not extremely dynamic, the slower but optimal DFMPC controller will be used. But when hard-curves, flips and twists need to be done, and highly-dynamic inputs are required, the system will switch to RLDFMPC controller, which has very high throughput. The switching conditions will be planned for by the motion planning sub-system.

<div style="page-break-after: always;"></div>

## 3) How does your team plan to use the simulator and development kit provided by DRL? How does your team plan to handle real-world variations that are difficult to capture in simulation? *

### General Approch
The simulator usage will be maximized in the following manners. First, it will be used as test tool for analyzing and improving perception, control and state estimation algorithms. Second, it will be used to collect data for model learning (state and inputs of the drone). We explain our approach as follows:

### Specific advantages
We exploit two advantages of simulation: First, unlike real settings, in a simulation ground-truth data can be obtained. Second, the simulation environment characteristics (lighting, occlusions, place, position of objects, winds, object materials etc ) and drone properties (mass, inertia tensor, rotor characteristics, sensor noise, etc ) can be programmatically controlled. These advantages will be maximized as follows. 

### Perception and State Estimation Algorithms

Regarding perception algorithms, the gate pose estimation, provided by the perception sub-system, will be compared against ground-thruth, obtained in simulation. This serves as indispensable feedback to know the estimation error and improve our algorithms. Following, by modifying the environment and drone sensor's parameters, the robustness of our algorithms can be assesed. On the environment side, by adding occlusions, various lighting, and changing object materials (simple job in simulation) we get means to understand the weaknesses of our algorithms to changes in the environment and conditions of the gates to be detected. On the drone side, by adding sensor noise (in camera image, IMU, etc.) it will be possible to find cases in which our algorithm fail to provide accurate gate pose estimations. Finally, by exploiting the very same sensor and environment/drone modifications, we can get new training data with which the performance of our algorithms can be improved to overcome the weaknesses found.

In short, the availability of ground truth and flexibility to change the parameters of the simulator, provides means to make fast iterations to improve our algorithm and extend our training data set to include exactly those settings in which our algorithm is weak.

### Trajectory Generation

State-of-the-art autonomous drone racing assumes gates are at very similar height and can be traversed in the horizontal direction. This makes construction of a testing environment a relatively simple task.  Also linear velocities have been slower compared to human drone racing. However Alpha Pilot Challenge  will require flying gates with very different poses, at high speeds. This includes very different gate heights. Setting a real environment for such testing is a challenge. We will use the simulator to create such testing environments. We will then be able to test our trajectory generation algorithm beyond the requirement in state-of-the-art autonomous drone racing. We will improve our algorithms to ensure trajectories make full use of the dynamic movement the drone is capable of, and making sure these trajectories can be tracked accurately enough by our controller.

### Tracking Control
Regarding our control algorithms, the simulator, together with the ROS nodes provided for communication, will be exploited as a tool to asses its tracking performance. More especifically, the tracking error for the parts of the track before and after gates will be of great interest. In the same sense, the lap time can be assesed and thus improved. A very important set of tests for which the use of the simulator will be maximized is controller switching. Since we will implement two controllers that switch based on the maneuver, the timing and smoothness of the switching becomes a critical property. In simulation it is possible to test various approaches at no cost.

###  Model learning
We will obtain drone state ground truth data from the simulator to learn the parameters of a first order Markov Decision Process whose state-transition model is differentially flat. This will allow us to identify the parameter values for a drone flying with the speed and aggressiveness seen in human drone racing. In other words, we will obtain a high-speed high-performance model for the drone dynamics. The differentiating advantage of the simulator is that ground thruth data for the state of the drone can be obtained directly without space or accuracy limitations. In contrast, previous work using similar techniques have one of two limitations: the speed at which the data is recorded or the accuracy with which it is obtained. For the former, ground thruth drone state data is obtained in a lab environment using motion tracking sensors but limited to relatively slow speeds and maneuvers (due to the space constraints). For the latter, sensor data is obtained from high speed and aggressive maneuvers performed outdoors, but the state of the drone is estimated from sensor data using optimized Kalman filtering. Our approach however can make use of ground-truth aggressive high-speed data obtained from the simulator, which avoids both previous limitations. The disadvantage is of course non modelled effects in simulator that are not taken into account. Nonetheless, modern simulators take into account complex high-speed dynamics, and, like the DRL Simulator, are already good enough that are used for inital classification rounds in human drone racing competition. In conclusion, the non-modelled dynamics in simulation is not a barrier for its use anymore, and our MPC controller will help compensating for them.

<div style="page-break-after: always;"></div>


## 4) What support, resources, and tools does your team plan to use for the competition to supplement the hardware, software, and training provided? How do you plan to support necessary travel should you proceed in the competition? Please include any hardware, software, people, data, mentoring, sponsorships, etc. *

We are a group of Mechanical Engineering Graduate School students at Sungkyunkwan University. We are doing research on autonomous flying robots at the Robotics and Intelligent Systems Laboratory -RISE Lab- lead by our team's Captian Prof. Dr. Hyung Pil Moon. The autonomous flying robots area is a line of research in this lab. Thus, we count with access to a host of tools present in a robotics lab, and for drone research specifically: Motion Tracking Hardware and Software from Vicon Company, 4 different drones with integrated sensors and computers for research, along with their tools and parts (batteries, monocular cameras, infrared sensors, GPS, barometers, etc), several Nvidia Jetson TX2 development boards, and access to computers with enough power for neural network training and algorithm development.  

Financing for travelling expenses, extra sw/hw tools, is secured by several research grants RISE Lab is part of, and industry-collaboration projects. Among the research grants obtained so far, BK21+ Project and LINC Project have the greater importance. BK21+ (Brain Korean 21 Plus) is a Korean Government program aimed to foster creativity in academic research in korean universities. SKK University and RISE Lab are part of this program from year 2017 to 2021 to support various research projects. The LINC Project (Leaders in Industry-University Collaboration) aims to foster industry-university collaboration to strengthen korean economy and support smart and bio industries.  

On the industry-collaboration side, we are in talks with a South Korean and U.S. companies developing services with drones. IF WE HAVE SUPPORT FROM COMPANIES, WE MUST DISCLOSE IT.... PENDING 

<div style="page-break-after: always;"></div>

## 5) What do you see as the biggest challenges specific to your team that you will face during the AlphaPilot competition? What is your initial plan for overcoming this challenge? For example: missing skills, financial challenges, team cohesiveness, time commitment, etc. *

The members of our team have experience with the tools, algorithms and hardware that we foresee will be used during the competition. From python and C++ programming in ROS, working with NVidia Jetson kits, developing vision based SLAM and robuts control algorithms using various sensors, to simulation and deployment in real robots, a broad range of experience is available within our team members. However this will be the first time we put all this experience together for drone racing. In this sense, the biggest challenges for our team will be to tranfer this accumulated knowledge to the problem of flying a drone autonomously and doing so in a coherent manner. A second challenge, related to the student status of the team members, will be to find the right balance between the academic workload and the competition. 

The first of the challenges will be overcomed by a team management approach. With the overview of team's Captain Prof. Dr. HyungPil Moon, the issue of coherently leveraging each of team's member abilities and experience and apply these to autonomous drone flying can be overcomed. In particular his 15+ years of research experience in robotics, his 10+ years as head of RISE Lab, combined with his 3 years experience as creator and organizer of the IROS ADR Challenge, will allow each team member to focus his/her efforts and experience on the appropriate challenges and subsystem, and that make sense to autonomously fly a drone. This is critical to our success given the time constraints, and to maximize the outcome of every effort.

Moreover, regarding our second challenge, academic and competition workload balance will be  obtained by combining both efforts. We will focus the academic efforts to what is required in the competition. In other words, the courses and projects that the team members will take and do for the duration of the competition, will be closely related to the challenges arising in the competition. This way, single effort will serve double purpose: develop our drone for the competition and getting the coursework done. This will be to the team members as a single and meaningful workload, rather than double unrelated workload.





