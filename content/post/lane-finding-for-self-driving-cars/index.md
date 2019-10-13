---
title: Lane Finding for Self-Driving Cars
subtitle: Using computer vision techniques for lane finding for self driving cars.
summary: Using computer vision techniques for lane finding for self driving cars.
date: 2017-08-21
math: true
diagram: true
markup: mmark
tags:
- Self-driving 
- Python
categories:
- Computer-Vision
image:
  placement: 1
---

Lane detection is one of the most important aspects of autonomous navigation. It is the first step in enabling the vehicle to visualize and then make sense of its environment.

The goal of this project was to create a pipeline that finds lane lines on the road.

<!-- ![Detected Lanes on road](/assets/images/udacity-SDCNDP/term1/lane-finding-project/laneLines_thirdPass.jpg) -->


### 1. The Approach

1 . The original image is first converted to ~~grayscale~~ HSV colorspace.

**Why, you ask ?**

Whilst the RGB colorspace is more conventional and intuitive in terms of describing colors, it isn't the best option around when the illumination conditions change.

[HSV](https://msdn.microsoft.com/en-us/library/windows/desktop/dd372106(v=vs.85).aspx) colorspace is good at this in a way that all the color information is encoded in single channel H. This way, the information in this channel doesnt change with varying lighting conditions.

![Conversion to HSV color space](/assets/images/udacity-SDCNDP/term1/lane-finding-project/Selection_023.png)

2 . The ~~grayed out~~ HSV converted image is then smoothed out using the opencv library's gaussianBlur function.

This helps remove any noise from the image. It's an important pre-processing step for canny edge detection.

3 . Masks for yellow and white colors are applied on the HSV image to extract lane lines from the image.

![Yellow and White masking](/assets/images/udacity-SDCNDP/term1/lane-finding-project/Selection_024.png)

**Why ?**

Even after conversion to HSV color space and application of Gaussian Blur, there can still be noise in the image that can interfere with the identification of lane lines. An effective way to get past this is to look for specific colors in the image, filtering out the noise in the image.
In our case, we know the lane lines are primarily yellow and white in color, we can make use of this information to apply yellow and white masks on our HSV image and try to remove everything else, other than the lane lines.

4 . Then the edges in the image are detected by passing the smoothed out gray image into the opencv's canny function. 

![Canny Edge Detection](/assets/images/udacity-SDCNDP/term1/lane-finding-project/Selection_025.png)

5 . After the edges have been detected, only that portion of the image needs to be considered for further processing that has road lane lines in view.

![Polygon shape to take the portion from the image having lane lines](/assets/images/udacity-SDCNDP/term1/lane-finding-project/polygon.png)

![Region of interest](/assets/images/udacity-SDCNDP/term1/lane-finding-project/Selection_026.png)

6 . Now that the image only has the canny edges for the lane lines portion and the rest of it blacked out, hough lines are drawn from those detected edges.

![Hough lines drawn for the lane lines](/assets/images/udacity-SDCNDP/term1/lane-finding-project/Selection_027.png)

Drawing lines from the detected edges is one of the most important aspects for this project. For two single lines to be drawn over the edges, one for each line of the lane (left and right), the draw_line() function is modified in the following way:

###DRAW-LINE ALGORITHM

<img src="/assets/images/udacity-SDCNDP/term1/lane-finding-project/algorithm.jpg" width="640" alt="Combined Image" />
<br><br>


  
 a. To extrapolate the lines returned from the hough lines and create two single lines, two endpoints (x_min, y_min, x_max. y_max) for the two lane lines and their corresponding  slope values (positive_slope, negative_slope) and intercepts (positive_intercept, negative_intercept) are needed. This forms the problem statement for the algorithm.
  
  b. The array of lines returned from the hough_lines() function is iterated upon and for each line, the slope is calculated and stored in spearate arrays for positive and negative slopes.
  
  c. From whether the slope is positive or negative, it can be inferred if the line belongs to the left line of the lane or the right line.
  
  d. To remove noisy lines pertaining to edges not belonging to any lane, but still within the lane area, a threshold is defined for the value of the slope acceptable for the algorithm.
  
  e. For each line, the intercept is also calculated and stored.
  
  f. The y_max for both lane lines is the point at the bottom of the image, from where the lines start.
  
  g. The slope values for both lines can be calculated by taking the averages of the corresponding positive and negative slope values. This also helps in removing some noise pertaining to the slope values.
  
  h. Similarly, the intercept for both lane lines can be calculated by taking the mean of the stored intercepts.
  
  i. The y_min can be found out by comparing all the y values in all the lines and taking the minimum value.
  
  j. Now x_min and x_max can be found out by just fitting all the peviously found parameters in an equation of the line.
  
  k. The lines can now be drawn over the lane lines.


7 . This image is now combined with the original image and the returned image would be the original one, with the algorithm generated lane lines drawn over it.

![Hough lines drawn on original image](/assets/images/udacity-SDCNDP/term1/lane-finding-project/Selection_028.png)


8 . The final result looks like this:

![Test Video](/assets/images/udacity-SDCNDP/term1/lane-finding-project/normal.gif)

![Challenge Video](/assets/images/udacity-SDCNDP/term1/lane-finding-project/challenge_hsv.gif)

### 2. Reflections

The conversion to HSV colorspace makes the pipeline robust to varying lighting conditions, but still the lines are a bit jittery. Improvements can be made to the `draw_line` algorithm to make better use of information from previous frames of the video while processing.


P.S - Here's the link to the [Github Repo](https://github.com/peps0791/Udacity-SDCNDP-Lane-Finding).