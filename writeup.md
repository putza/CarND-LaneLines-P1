# **Finding Lane Lines on the Road**

This writeup is intended for submission to the Udacity Self-Driving car nano-degree.

The source code for this prokect can be found in my [Github repository](https://github.com/putza/CarND-LaneLines-P1).

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[fig_roi]: ./examples/fig_roi.png
[fig_colors]: ./examples/fig_colors.png
[fig_pipeline_full]: ./examples/fig_pipeline_full.png
[fig_pipeline_comp]: ./examples/fig_pipeline_comp.png

---

# 0. Reflection

## Initial Image observations

### Region of interest

The lane markings are in very consistent regions of the image. If the car is driving nicely in the lane, the following mask works well:

![Region of interest][fig_roi]

For lane switching, more ROIs have to be considered.

### Line color

The lines are either white or yellow. The RGB codes for pure white and pure yellow are [255,255,255] and [255,255,0]. The blue channel is therefore not helpfu in the lane detection. Averaing green and yellow did not seem to give any benefits, so I focussed on the red channel.

![Color Channel Investigation][fig_colors]

## 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

### Pipeline

My pipeline consisted of the following steps:

1. Gray scaling: User red channel
1. Apply Gaussian Blur
1. Image Enhancement.
    1. scikit image histogram intensity re-scaling to make the yellow and    white lines more intense.
    1. A few other methods for the challenge question
1. Canny Edge Detector
1. Cut to region of interest
1. Hough transfrom
1. Overlay lanes onto original image


### Draw Line Modification

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by as follows:

1. I converted the list of coordinates for the lined into a pandas dataframe.
2. I calculated the slope, y intercept and length of all lines in the dataframes and appended these values as columns to the dataframe
3. I removed all rows which contained infinity in the slope field due to a potential division by zero.
4. I find the minimum y value to plot the lane up to the edge of the ROI
5. I calculated the minimum and maximum slope.
6. Left lane:
    1. I only calculate the left lane if the minimum slope was below -0.4.
    2. If the minimum slope is below -0.4, then I select all rows with a slope of less then 80% of the minimum slope.
    3. I averaged the slope and the intercept
    4. I calculate the new coordinates of the line.
7. Right lane:
    1. Same as left, except replace minimum with maximum amd less with greater.

I implemented try blocks to catch any exceptions such as:

* Hough transform does not return any lines.
* No left or right lanes


### The pipeline in operation

Below is the pipeline applied to a single image

![Pipeline][fig_pipeline_full]

and the beginning and end result for all test images:

![Pipleine comparison][fig_pipeline_comp]

### And here are the movies

Sadly Markdown does not support avi files, so here are some good old boring links:

* [White Lane Detection](solidWhiteRight.mp4)
* [Yellow Lane Detection](solidYellowLeft.mp4)
* [Challenge Video](challenge.mp4)


## 2. Identify potential shortcomings with your current pipeline


1. Coding standards:
    1. The pipeline contains a lot of parameters, but a really lousy API to access all tuning parameters.
    2. A lot of hard coded parameters
    3. Not a great diagnostic pipeline
2. My image enhancement algorithm is not very good. It works well enough for the supplied test images, but some frames in the movies are poorly detected.
3. Parameterization: I really did not tune the parameters. I mostly chose the default from the lectures.
4. Lane prediction on a single image only.
    1. Jittering between frames
    2. Missing lanes when I filtered everything out
5. Simplistic lane: straight line
6. Change of image format throws things off.



## 3. Suggest possible improvements to your pipeline

### Addressing the issues above:

1. Use Object Oriented Programming
    1. A nice class for the pipeline, to register new algorithms and handle the parameters.
    2. A nice parameter interface
    3. A easy way to monitor the algorithm while it executes
2. Use the diagnostic tools above to find the offending frames in the movie and adapt the image enhancement accordingly.
    1. Use the full spectrum of OpenCV and scikit-image functions
    2. Automatically report frames which do not find a lane
    3. Automatically report outliers
3. OOP would make this easier. Spend some more time on this.
4. Averaging scheme between frames
    1. Detect outliers and jitter beween frames
    2. Interpolation, prediction scheme
    3. Incorporate other sensor data for a Kalman filter
5. Higher order polynomial for the lane
6. MOre robustness w.r.t. format changes.


### General remarks

Adapting the image contrast to make the lines stand out more would be worth the effort:

* Look at local adaptive methods
* Convert from HSV to HSV and cluster colors close to white and yellow.

We could also search for white and yellow lines separately and then combine the two images.