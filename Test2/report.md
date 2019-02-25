# RISE-Q
## Test 2 Written Report

# 6DoF 
# ALGORITHM SUBMISSION FOR FLYABLE AREA DETECTION

## Introduction

The human vision system  performs several complex functions at once:  among others, allows us to detect and identify objects of interest, 
and have an idea of their position and orientation in space, as well as its relative and absolute scale. These *** are fundamental
in daylife (for walking, using tools, cooking food, etc) and are leveraged in drone racing by human pilots, a **** in which the object of interest are gates that are to be traversed, and doing so performing high-speed high-accuracy maneuvers in accordance to the track layout, while avoiding possible obstacles and collisions. 

For an autonomous drone to have a fair race againts human pilots, it is logical to think that it must display similar skills, 
namely gate detection and classification, and means to estimate its position, orientation and scale. This would allow it to detect gates, know where they are and how to approach them, while providing robustness to gate perturbations.  This is also the basic idea behind our work.

## High level decisions and reasoning
In the context of Test 2, the objective is to recognize the flyable area in images containing the ADRL gate. However, as explained before, the broader objective in drone racing is to traverse those gates fast, accurately, and robustly while avoiding obstacles and collisions. For this reason, we decided to implement an algorithm that detects and classifies gates in images, estimates the 6DoF position and orientation of the gate, and finally, calculates the position, orientation and extent of the flyable area in 3D space. This idea was proven by Team RPG during  IROS 2018's Autonomous Drone Competition, in which they got 1st place. For the purpose of Test2, the projection of the flyable area into the camera plane is calculated and the 4 points defining the bounding box are reported. Moreover, this relaxes further processing for trajectory planning. 

Although for Test 2 and ADRL gates are provided with clear, always visible fiducials, this is not the case in other racing contexts. Even the training dataset for Test 2 had initially fiducial marks and/or flyable areas that were blurred or not totally visible. In general, objects do not have clear fiducials marks for identification, and for racing in particular other drones and obstacles must be avoided even if they are not provided with them. In this sense, we opted for an algorithm that could be generalized to any object, without the need for fiducials or total visibility. This is a key characteristic of our algorithm and also the reason why we approached it using neural networks or NNs. NNs can be trained to detect, classify and estimate the pose of an object in short time and reasonable precision. One of such algorithms is the MIT-licensed singleshot6Dpose [3], which we leveraged for our Test 2 submission and discuss in Section X.

To train NNs, large, accurately labeled datasets are required. Due to the approach we took, bounding box ground thruth is not enough and was not readily available from Test 2 resources, and ground thruth rotation, translation and bounding "cube" labels are necessary. We approached the creation of our training dataset using domain-randomized and photorealistic synthetic images, created in Unreal Engine 4. This approach has been proven in []. In total, 25,000 images with labels were generated, and the dataset is augmented with varying hue, noise, shape, jitter, saturation and exposure during training.

## Detailed explanation

As previously mentioned, we made use of the open-sourced MIT Licensed singleshot6Dpose [3]. This is a Convolutional NN based algorithm for general 6DoF pose estimation. It is based on the YOLOv2 [] detector which achieves state-of-the-art precision and real time speed. The algorithm takes a single RGB image as input and predicts an object 6DoF pose as its output. To predict 6DoF pose, the algorithm works in 2 steps: First, the network predicts the 2D projection on the camera plane of the 3D bounding cube of the object. Second, it uses the predicted points to estimate 6DoF pose using a PnP algorithm. At training time, the network is given the 2D projection on the camera plane of the 3D bounding cube, which the network will predict, along with class id. 