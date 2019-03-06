# RISE-Q
## Test 2 Written Report

# 6GPE: 6DOF GATE POSE ESTIMATION
# ALGORITHM SUBMISSION FOR FLYABLE AREA DETECTION

## I. Introduction

The human vision system  performs several complex functions at once:  among others, allows us to detect and identify objects of interest, have an idea of their position and orientation in space, as well as its relative and absolute scale. These functions are fundamental in daylife (for walking, using tools, cooking food, etc) and are leveraged in drone racing by human pilots. Here, the object of interest are gates that are to be traversed while performing high-speed high-accuracy maneuvers against time, in accordance to the track layout, and avoiding possible obstacles and collisions. 

For an autonomous drone to have a fair race againts human pilots, it is logical to think that it must display similar skills, namely being capable of object detection and classification, to estimate the object's position, orientation and scale. This would allow it to detect gates, know where they are and which direction they are facing, and with its scale, the best way to traverse them.  This is also the basic idea behind our approach to navigation and we adapt our algorithm for the specific task of flyable area detection for Test 2. In this sense, the output of our algorithm is the rotation and translation of the gate with respect to the drone's camera frame.

While our score for Test 2 is 78%, a) we consider our approach  provides sound information for navigation when compared to flyable area detection itself, b) given more training time and the correct camera calibration matrix during training, our algorithm could achieve >90% score, c) can extend our algorithm to multiple object detection, d) do not depend on gate fiducials, e) can achieve near real-time performance on an Nvidia Xavier Platform and f) . 

## II. High level decisions and reasoning
In the context of Test 2, the objective is to recognize the flyable area in images containing the ADRL gate. However, as explained before, the broader objective in drone racing is to traverse those gates fast, accurately, and robustly while avoiding obstacles and collisions. For this reason, we decided to implement an algorithm that detects and classifies gates in images, estimates the 6DoF position and orientation of the gate, and finally, calculates the position, orientation and extent of the flyable area in 3D space. This idea was proven by Team RPG during  IROS 2018's Autonomous Drone Competition [1], in which they got 1st place. For the purpose of Test2, the projection of the flyable area into the camera plane is calculated and the 4 points defining the bounding box are reported. Moreover, this approach relaxes further processing for trajectory planning. 

Although for Test 2 and ADRL gates are provided with clear, always visible fiducials, this is not the case in other racing contexts[2] nor real scenarios.  In general, objects do not have clear fiducials marks for identification, and for racing in particular other drones and obstacles must be avoided even if they are not provided with them. In this sense, we opted for an algorithm that could be generalized to any object, without the need for fiducials or total visibility. This is a key characteristic of our algorithm and also the reason why we approached it using neural networks -NNs-. NNs can be trained to detect, classify and estimate the 6DoF pose of an object in short time and reasonable precision. For our Test 2 submission, we leveraged the MIT-licensed singleshot6Dpose[3] NN. We discuss details in Section III.

To train our NN, large, accurately labeled datasets are required. Due to the approach we took, 2D bounding box ground thruth is not enough. Our network requires ground thruth rotation, translation and bounding "cube" labels. Since this labels are not readily available, we approached the creation of our training dataset using domain-randomized and photorealistic synthetic images, created in Unreal Engine 4. This approach has been proven in [4]. In total, 25,000 images with labels were generated, and the dataset is augmented with varying hue, noise, shape, jitter, saturation and exposure during training.



## III. Technical Description

As previously mentioned, we made use of  singleshot6Dpose [3] NN. This is a Convolutional NN based -CNN- algorithm for general 6DoF pose estimation. It is based on the YOLOv2 [5] detector which achieves state-of-the-art precision and real time speed. The algorithm takes a single RGB image as input and predicts an object 6DoF pose as its output. To predict 6DoF pose, the algorithm works in 2 steps: First, the network predicts the 2D projection on the camera plane of the 3D bounding cube of the object. Second, it uses the predicted points to estimate 6DoF pose using a PnP algorithm and camera calibration parameterers. At training time, the network is given the 2D projection on the camera plane of the 3D bounding cube, which the network will predict, along with class id. The implementation is done in Python 3.5.2 and using pyTorch 0.4. 


## a. APPROACH DESCRIPTION

The problem of predicting the 2D projection of the 3D bounding cube of an object is modelled as regression problem from image pixels to bounding cube coordinates and class probabilities. The input image is seen once (not by regions or sliding windows) and predictions are obtained from a single neural network. This allows for high speed execution. The 6DoF gate pose estimation problem is formulated as the estimation of the 2D projection (over the camera plane) of the 3D bounding cube corners (and its centroid) associated with the gate. In total 9 control points. Given these 2D projections, a PnP algorithm estimates the 6DoF pose of the gate.

The input image is divided as an S x S grid (S = 7). It is processed with a CNN, and the output is a 13x13xD (D = 9*2 + 1 + C) tensor, which contains the estimation of the 2D projections of the 9 control points (9x2), an object confidence value and C class probabilities. The network architecture is presented in Figure 3. The object confidence score should reflect the presence or absence of an object. This is, higher values if the gate is present and smaller ones if it is not. In practice, the confidence function c(x) (1) is applied to each 2D control point estimation relative to the ground truth and the mean value of the 9 points is used as final object confidence score. Dt(x) represents the euclidean distance between the predicted 2D projections and ground truth ones, d_th = 30 pixel is a cutoff value for the distance, after which the confidence is set to zero (meaning no object present).

During training, to optimize the parameters of the network, the loss funciton (3) is used. Mean-squared error is used for both confidence and coordinate losses, and cross entropy for classification loss.


At test time, the class specific confidence value (reported as the confidence for Test 2, the 9th value in the required output for each image), is calculated by multiplying the class probabilities with the object confidence as in (2). This is done for each cell in the SxS grid. Since several cells will predict the presence of the same object (albeit with difference class probabilities), and since the object can in fact occupy various cells, the 3x3 neigborhood to each cell is inspected. For an accurate prediction, the weighted average of the 2D predictions in the neighborhood is taken and reported as final result. The weights are the confidence scores from each cell. These final 2D predictions are then passed to a PnP algorithm which estimates their 3D position, and finaly the rotation and translation of the gate.

## Network Design

The network architecture is shown in Fig. 1. The initial convolutional layers extract features from the image (inspired in GoogLeNet) and the fully connected layers predict the output probabilites (P(Object) * IOU, P(Class|Object)) and control points. 

## Loss Function

The loss function the system optimizes for is the sum-squared error of the output.  

