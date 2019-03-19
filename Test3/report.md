# Alpha Challenge Test 3: Technical Report. MAX 2 PAGES

##TODO

|Section|Topic|Detail|
|---|---|---|
|Introduction|||
|High Level Decisions and Reasoning| Subsections| Add 1 subsection per subsystem, indicating in each the most important decisions and reasoning.|
|Technical Description|||
|Trajectory Generation|||
|Trajectory Tracking: Control|||
|State Estimation| Method| Eugene, are you using the derivative-free kalman filter from the differential flatness theory? using extended Kalman filter. We are not using our inputs(T, M) at prediction of state, using values from IMU.|
|State Estimation| Gate Pose Estimation| @Eugene, you now update drone's pose using gate pose and IR Marker. Is the inverse posible? Update gate pose from drone pose estimation? If not, need to delete that from Introduction   We can because we have rotation and translation vectors, but I don't know why we have to do it. Because we are not generating trajectory based on vision and predicting gates' positions. If it is necessary, please tell me.|
|Results|||
|Future Development|||
|References|||

## Introduction

Fast, autonomous flight of quadrotors requires algorithms that leverage the dynamics of the drone, fuses sensory information precisely for accurate navigation, and adapt to uncertainty. We introduce our differential-flatness (DF) based controller and state estimator, the preamble of our future RL controller. DF theory allows us to generate trajectories that make best use of drone dynamics, thus relaxing both control effort and computational requirements, and improving agility for fast maneuvers. Our Kalman-Filter-based state estimator fuses signals from the IMU, Camera and Range Finder to update ***both the state and the gate position estimates***.       


## High Level Decisions and Reasoning

### Trajectory Generation

### Controller
High performance athletes learn to leverage all strenghts and weaknesses in their bodies through extensive training, and in this way achieve world-class performance. We believe in such an approach for fast/agile drone fly, and thus leverage drone dynamics for control. Our controller computes the inputs that would naturally arise when tracking a desired trajectory, thus respecting its dynamics. Correction terms are calcuted only for the reference input, and not only from the reference states, and permit accurate tracking. Added to this, we use ***rotation matrix representation of the attitude,*** which affords us very agile maneuvers like inverted flight, without singularities. 

### State Estimator

## Technical Description



## Trajectory Generation

## Trajectory Tracking: Control

The input to our controller are the error and reference states. The error states are calculated from feedback terms from the state estimator, and the reference states are obtained from the differential-flatness based trajectory. A PID controller is used for asymptotical convergence of the states to the reference states. Since the input to the quadrotor are the body-frame Thrust and angular velocities, these are calculated using an inverse mapping from reference inputs to desired quadrotor inputs, as in (1). Figure X presents a block diagram of our controller. 

             * Insert thrust and angular velociticy equations here *
             * Insert Controller Block Diagram Figure Here *






## State Estimation

## Results

## Future Development

## References
Faessler, M., Franchi, A., & Scaramuzza, D. (2017). Differential Flatness of Quadrotor Dynamics Subject to Rotor Drag for Accurate Tracking of High-Speed Trajectories. https://doi.org/10.1109/LRA.2017.2776353.
Lee, T., Leok, M., & McClamroch, N. H. (2010). Geometric tracking control of a quadrotor UAV on SE(3). Proceedings of the IEEE Conference on Decision and Control, 5420–5425. https://doi.org/10.1109/CDC.2010.5717652
Scholarsarchive, B., Mclain, T., Beard, R. W., Mclain, T. ;, Beard, R. W. ;, Leishman, R. C. ;, … Mclain, T. (2011). Differential Flatness Based Control of a Rotorcraft For Aggressive Maneuvers BYU ScholarsArchive Citation Differential Flatness Based Control of a Rotorcraft For Aggressive Maneuvers, (September), 2688–2693. Retrieved from https://scholarsarchive.byu.edu/facpub%0Ahttps://scholarsarchive.byu.edu/facpub/1949
Levine, S., & Koltun, V. (2013). Guided Policy Search - gps_full.pdf, 28. https://doi.org/10.1109/ICRA.2015.7138994




