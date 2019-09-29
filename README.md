[//]: # (Image References)
[image_0]: ./misc/rover_image.jpg
[perception_V1]: output/perception_V1.gif
[perception_V2]: output/perception_V2.gif
[rocks_filter]: ./misc/rocks_filter.jpg
[navigable]: ./misc/navigable_space_V2.jpg
[![Udacity - Robotics NanoDegree Program](https://s3-us-west-1.amazonaws.com/udacity-robotics/Extra+Images/RoboND_flag.png)](https://www.udacity.com/robotics)
# Perception, decision making and actuation for Roversim


![alt text][image_0] 

This project is based on the [RoboND-Rover-Project](https://github.com/udacity/RoboND-Rover-Project) and it will be used to illustrate my solution for perception, decision making and actuation for Roversim.
This file will only document my solution and improvement for this project. For more information about the initial project please check [RoboND-Rover-Project](https://github.com/udacity/RoboND-Rover-Project) 



## 1. Perception 
### Basic project
We have a Jupyter notebook file in the code directory where we can test different functions written for perception.
* Perspective transformation: Transform the Rover camera captured image to bird-eye view
    * Select 4 source and 4 destination point to define the matrix of the transformation (getPerspectiveTransform)
    * Transform all the pixels from the source to destination view (warpPerspective)  
* Apply a binary transformation to extract traversal vs obstacle area
    * Define a threshold and select pixels
* Coordinate transformation: extract the non zero pixels and apply transformations
    * rover_coords: Calculate free space pixel positions with reference to the rover position being at the position (0,0).
    * Calculating the polar coordinate using the result of rover_coords  (useful to extract a vector that represents the orientation of the movement)
* Pixel to world transformation: map the free space to the world map.
    * Rotation of the pixels using the yaw of the rover giving by the simulator
    * Scale the coordinate with respect to the size of the ground_truth map(1pixel->1m) and the current coordinates (1pixel-> 0.1m) 
    * Translate the pixels using the position of the rover giving by the simulator
    
    
### Perception version 1 
#### Result
<p>

![Alt Text][perception_V1]
<br>
<em>Fig. 1: The result of the perception version 1</em>

</p>


#### Implementation
The work for perception version 1 consists of combining the functions to map the navigable space in the world. 

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

In version 2 of perception, we will try to improve the perception to get a better mapping for the ground truth.

### Perception version 2
#### Result 

<p>

![Alt Text][perception_V2]
<br>
<em>The result of the perception version 2</em>

</p>

When we compare the ground truth and the results from version 1 and 2 we find that version 2 does a better mapping of the navigable space 
```python
# Code used for the comparison
created=np.array(data.worldmap[:, :, 1],dtype=np.int32)
created[created>0]=1
truth=np.array(ground_truth_BW,dtype=np.int32)
print(np.sqrt(np.mean((created- truth) **2)))
print(np.count_nonzero((created- truth)))
```

* Version 1
    * 0.24 distance between the ground truth and the created map 
    * 2449 wrong pixel
* Version 2
    * 0.139 distance between the ground truth and the created navigable area
    * 780 wrong mapped pixel
    
#### Implementation 
* Extract the mask for precious rocks
    * All the rocks are yellow which is represented by a high value for the red and green colors and a low value for the blue color
    * Closing morphological operation clean any noise pixels that belong to the rock. (for more information about morphological operation see resources)

<p>

![Alt Text][rocks_filter]
<br>
<em>Fig. 3: Precious Rocks extraction</em>
</p>

---
* Extract navigable area 
    * mask1: Binarization using the threshold 120 on the grayscale image
    * Equalize the Image (useful for images with low contrast)
    * mask2: Binarization using the threshold 200 on the Equalized image  (The result will depend on the brightness of the image)
    * Combine two mask1 and mask2 
    * Remove grids: remove the small separation between free space area when the grid is displayed for example
    * Connected component algorithm
    * Identify the component that belongs to the navigable area and create the  mask
    * Closing morphological operation to clean any noise pixels that should be belonging to the navigable space (for more information about morphological operation see resources)
   
<p>

![Alt Text][navigable]
<br>
<em>Fig. 4: Navigable space extraction</em>
</p>

---   

* Extract obstacle
    * The precious rocks and the navigable area masks are combined and inversed to get obstacle mask.
---
* Common Process
    * Identifying (navigable, obstacle, precious rocks) by comparing the number of pixels that have been identified for each area.
---

#### Future Improvement
* Optimise the process by using threads or combine matrix calculation 


### Source
* [MorphologicalProcessing](http://inside.mines.edu/~whoff/courses/EENG510/lectures/10-MorphologicalProcessing.pdf)

## Contributors 
 
| [<img src="https://avatars1.githubusercontent.com/u/20454717?s=460&v=4" width="100px;"/><br /><sub><b>Mohmaed Amine Ouali</b></sub>](https://github.com/MohamedAmineOuali)<br /> 
| :---: |  