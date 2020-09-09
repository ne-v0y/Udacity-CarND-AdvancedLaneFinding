## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[undistoredOG]: ./output_images/undistorted_og.png "Undistorted OG"
[undistortedImage]: ./output_images/undistorted.png "Undistorted"
[roadTransformed]: ./output_images/roi_transform.png "Road Transformed"
[allColorSpaces]: ./output_images/all_color_space.png "All Color Spaces"
[colorSelected]: ./output_images/color_selected.png "Color Selected"
[threshCombined]: ./output_images/thresh_combined.png "Thresh Combined"
[warpedROI]: ./output_images/warped_roi.png "Warped ROI"
[fitPlot]: ./output_images/plot_fitting.png "PFit Plot"
[searchPrev]: ./output_images/search_region.png "Search Previous Frame"
[videoFrame]: ./output_images/sample_video_frame.png "Sample Video Frame"
[errorLane]: ./output_images/s

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./AdvancedLineFinding.ipynb", bundled in a class called "CameraCalibration".  

The class instance stores the camera matrix, distortion coefficient for `cal_undistort` method.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][undistortedImage]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the `CameraClibration` class, I applied `cal_undistored` method with the previously calibrated parameters to calculate the undistorted original image.
![alt text][undistoredOG]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. To see what color space to use, I converted images to HLS and HSV spaces separetely, and observed that the lanes are more visible in S channel of HLS image and V channel of HSV image.
![alt text][allColorSpaces]

I combined the result of S and V spaces.
![alt_text][colorSelected]

I also combined a set of gradient thresholding filters, and combined the result of them,
![alt_text][threshCombined]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

For region of interest transformation, I chose the hardcode the source and destination points.
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 450      | 200, 0        | 
| 150, 720      | 200, 720      |
| 1150, 720     | 1080, 720      |
| 740, 450      | 1080, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][roadTransformed]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

With the combined color and gradient filtering, and region of interest, my input image for finding lanes is like this
![alt_text][warpedROI]

Applying sliding windows searching method, I get a result of lane fitting in a 2nd order polynomial like this
![alt text][fitPlot]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

the curvature of the lane is calculated based on the knowledge here https://www.intmath.com/applications-differentiation/8-radius-curvature.php

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Combining all results and apply the detected lane area back to video frame, I get a result like this

![alt text][videoFrame]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./process_pipline_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had issues when trying to get adequate pixels from the lane detection. At the beginning, I used an `AND` method on the HLS and HSV pixel selections, but that didn't give a lot of pixels on the dash lane, and resulted something like this:  
![alt_text][./output_images/error_with_fit_plot.png]
Then I changed it to an `OR` operation on the two color channel, slightly decreased the range of the color, and that solves the issue. 
I had a few bad frames where the road is bright, and that messes with the color thresholding. But due to the continunity on the road width and search area, even though the binary image got false positives, it still maintained the most of it. I can make the detection pipeline into a class and keep track of the medium lane width to keep the car from driving off the road, or drop the frame if too many false positives occurred comparing to the previous frame.