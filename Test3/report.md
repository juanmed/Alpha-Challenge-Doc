# Alpha Challenge Test 3: Technical Report. MAX 2 PAGES

##TODO

|Section|Topic|Detail|
|---|---|---|
|Introduction|||
|High Level Decisions and Reasoning| Subsections| Add 1 subsection per subsystem, indicating in each the most important decisions and reasoning.|
|Technical Description| Equations| 지호 please see my message to you in this section|
|Trajectory Generation|||
|Trajectory Tracking: Control|||
|State Estimation| Method| Eugene, are you using the derivative-free kalman filter from the differential flatness theory? using extended Kalman filter. We are not using our inputs(T, M) at prediction of state, using values from IMU.|
|State Estimation| Gate Pose Estimation| @Eugene, you now update drone's pose using gate pose and IR Marker. Is the inverse posible? Update gate pose from drone pose estimation? If not, need to delete that from Introduction   We can because we have rotation and translation vectors, but I don't know why we have to do it. Because we are not generating trajectory based on vision and predicting gates' positions. If it is necessary, please tell me.|
|Results|||
|Future Development|||
|References|||

## Introduction

Fast, autonomous flight of quadrotors requires algorithms that leverage the dynamics of the drone, fuses sensory information precisely for accurate navigation, and adapt to uncertainty. We introduce our differential-flatness (DF) based controller and state estimator, the preamble of our future RL-DFMPC controller (described in Test 1 Report). DF theory allows us to generate trajectories that make best use of drone dynamics, thus relaxing both control effort and computational requirements, and improving agility for fast maneuvers. Our Kalman-Filter-based state estimator fuses signals from the IMU, Camera and Range Finder to update ***both the state and the gate position estimates***. 

We got X% result for Test 3 Leaderboad. 


## High Level Decisions and Reasoning

### Trajectory Generation

### Controller
Just like high performance athletes learn to leverage all strenghts and weaknesses in their bodies through extensive training, our controller computes reference inputs that leverage drone dynamics, through differential flatness. Error feedback, through PID and LQR controllers, is used to calculate correction terms for the reference inputs, allowing accurate tracking. Added to this, we use ***rotation matrix representation of the attitude,*** which affords us very agile maneuvers like inverted flight, without singularities. This controller will be dramatically improved with the inclusion of an MPC. This will be the basis for our Reinforcement Learning based controller, which will improve speed, robustness and agility of our algorithm

### State Estimator

## Technical Description
We model our drone using Newton-Euler formalism. Define an inertial frame *W* with origin *Ow* and a body-fixed frame *B* with origin *Ob* attached to the CoM of the drone. The vector *x* represents the position of *Ob* relative to *W*. The dynamic equations for movement are in (1) to (5).

***지호 please insert here equations 1 to 5 from: https://jfmy.wordpress.com/2019/01/24/proof-of-differential-flatness-for-3d-quadrotor/***

## Trajectory Generation

## Trajectory Tracking: Control

The input to our controller are the reference states, reference inputs and estimated states. Reference inputs are obtained from a DF-based trajectory generator and a PID controller is used to generation correction terms for them, ensuring asymptotical convergence of the state error to zero. The desired thrust and angular velocities are calculated using an inverse mapping from the inputs, as in [3]. Figure 1 presents a block diagram of our controller. 

             * Insert thrust and angular velociticy equations here *

<img src = "Fig X. Alpha Pilot Control Diagram.png">

Figure 1. Block diagram of the controller.

A cascaded control architecture was followed: position control in the outer loop and orientation control in the inner loop. For position control, PID feedback was selected for its tuneability. Aggresive behaviour has higher priority than safety during racing, and PID control is a good fit that allows a good compromise between aggressiveness and stability. This controller calculates the thrust command, as well as the desired orientation and angular velocity which are passed to the inner loop. For orientation control, we selected an LQR controller for faster convergence. This is necessary as the position controller requires convergence of the desired orientation for tracking. Finally, the outer loop runs at 30Hz, while the inner loop runs much faster at 200Hz.


## State Estimation

## Results

## Future Development

## References
[1]Faessler, M., Franchi, A., & Scaramuzza, D. (2017). Differential Flatness of Quadrotor Dynamics Subject to Rotor Drag for Accurate Tracking of High-Speed Trajectories. https://doi.org/10.1109/LRA.2017.2776353.
[2]Lee, T., Leok, M., & McClamroch, N. H. (2010). Geometric tracking control of a quadrotor UAV on SE(3). Proceedings of the IEEE Conference on Decision and Control, 5420–5425. https://doi.org/10.1109/CDC.2010.5717652
[3]Scholarsarchive, B., Mclain, T., Beard, R. W., Mclain, T. ;, Beard, R. W. ;, Leishman, R. C. ;, … Mclain, T. (2011). Differential Flatness Based Control of a Rotorcraft For Aggressive Maneuvers BYU ScholarsArchive Citation Differential Flatness Based Control of a Rotorcraft For Aggressive Maneuvers, (September), 2688–2693. Retrieved from https://scholarsarchive.byu.edu/facpub%0Ahttps://scholarsarchive.byu.edu/facpub/1949
[4]Levine, S., & Koltun, V. (2013). Guided Policy Search - gps_full.pdf, 28. https://doi.org/10.1109/ICRA.2015.7138994




