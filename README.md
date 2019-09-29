[//]: # (Image References)
[image_0]: ./misc/rover_image.jpg
[perception_V1]: output/perception_V1.gif
[![Udacity - Robotics NanoDegree Program](https://s3-us-west-1.amazonaws.com/udacity-robotics/Extra+Images/RoboND_flag.png)](https://www.udacity.com/robotics)
# Perception, decision making and actuation for Roversim


![alt text][image_0] 

This project is based on the [RoboND-Rover-Project](https://github.com/udacity/RoboND-Rover-Project) and it will be used to illustrate my solution for perception, decision making and actuation for Roversim.
This file will only document my solution and improvement for this project. For more information about the initial project please check [RoboND-Rover-Project](https://github.com/udacity/RoboND-Rover-Project) 



## 1. Perception 
### Basic project
We have a jupyter notebook file in code directory where we can test and experiment different functions written for perception.
* Prespective transformation: Tronsform the Rover camera capture image to bird-view image
    * Select 4 source and 4 destination point in order to define the matrix of the transformation (getPerspectiveTransform)
    * Transform all the pixels from the source to destination view (warpPerspective)  
* Apply binary transformation to extract traversal vs obstacle eara
    * define a threashhold and select pixels
* Corrdinate transformation : extract the non zero pixels and apply transformations
    * rover_coords (useful for calcualting ploar cordinate with respect to the rover): Calculate pixel positions with reference to the rover position being at the "center bottom of the image".
    * caulating the polar corindate using rover coords (usful to extract a vector that represent the oriantation of the deplacement arrow)
* pixel to world transformation: map the free space to the world map.
    * rotation of the pixels using the yaw of the rover giving by the simulator
    * translate the pixels using the position of the rover giving by the simulator
    
    
### Perception version 1 

![Alt Text](perception_V1)
*The result of the perception version 1*

The work for perception version one consist in integrating all the function and combine them in order to show the free sapces detected and its mapping to the ground truth 

```python
    # 1) Define source and destination points for perspective transform
        #we already have the source and destination points from above we will be using them
    # 2) Apply perspective transform
    warped = perspect_transform(img, source, destination)
    # 3) Apply color threshold to identify navigable terrain/obstacles/rock samples
    threshed = color_thresh(warped)
    # 4) Convert thresholded image pixel values to rover-centric coords
    xpix, ypix = rover_coords(threshed)
    # 5) Convert rover-centric pixel values to world coords
    navigable_x_world, navigable_y_world=pix_to_world(xpix, ypix, data.xpos[data.count], data.ypos[data.count], data.yaw[data.count],data.worldmap.shape[0]*1.0,10)
    navigable_x_world, navigable_y_world=np.asarray(navigable_x_world, dtype=int), np.asarray(navigable_y_world, dtype=int)
    data.worldmap[navigable_y_world, navigable_x_world, 2] = 255  
``` 

In the version 2 of perception we will try to improve the perception in order to get a better mapping comparing to the ground truth.

