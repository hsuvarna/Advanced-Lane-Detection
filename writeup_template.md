## Writeup Template

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

[image1]: ./examples/calib2_orig.png "Calibration Input Output"
[image2]: ./examples/undist2.png "Undistorted Input Output"
[image4]: ./examples/undist1_out.png "Undistorted Input Output"
[image5]: ./examples/undist4_persp_out.png "Perspective Example"
[image6]: ./examples/threshold_b_out.png "Threshold hsb B Output"
[image7]: ./examples/threshold_l_out.png "Threshold l hls Output"
[image8]: ./examples/pipeline_in_out.png "Image pipeline Output"
[image9]: ./examples/rects.png "Sliding windows Output"
[image10]: ./examples/histogram.png "Histogram Output"
[image11]: ./examples/prev_poly_fit.png "Previous polynomial fitting Output"
[image12]: ./examples/warp_fill_out.png "Lane detected and filled"
[video1]: ./ai_car.mp4 "Video"

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The calibration of camera output is done by detecting the chess board corners which are 9x6. The input calibration images are converted to gray level for this purpose. The detected image points and corners are recorded for further usage in global lists.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the calibrated camera parameters, and the cv2.undistort() method, input images distortion is removed. The distortion is introduced by cameras which take a 2D picture of 3D real world objects. 

![alt text][image2]
![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The thresholding is done using the l-channel of HLS version of input image and also using the b-channel of HSB version. Then both are combined to apply right thresholding. The methods hls_l_threshold(), hsb_b_threshold(0 do the thresholding.

![alt text][image6]
![alt text][image7]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.



| Source        | Destination   | 
|:-------------:|:-------------:| 
| 450, 470      | 430, 0        | 
| 660, 470      | 830,0         |
| 250, 720      | 430, 720      |
| 1200, 720     | 830, 720      |

This took quite a bit of time to granualrly adjust but very important in aligning the highlighted lane edges. The code for this is in method unwarp_image()

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The routines sliding_window_poly_fit(), poly_fit_using_prev_fit() accomplish this. The sliding windows method identifies lane boundaries for the first time by identifying the left/right lanes using histogram. Then for each edge, it slides windows to detect the curve and finally fits a 2nd order polygon on these pixels. 
Later on the poly_fit_using_prev_fit() takes the sliding_windows_poly_fit() output and does lane edge detection.
The image pipeline output,sliding windows fitting, histogram and detected lanes are shown below. 

![alt text][image8]
![alt text][image9]
![alt text][image10]
![alt text][image11]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code get_lane_radius_car_pos() does this. It takes the left, right edge points and the corresponding pixel indices. Then it computes the radius and center using 2nd order polynomials.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code draw_curved_warped_highlighted_polyfit_lane() does this.

![alt text][image12]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's it ![alt text][image5]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. Test on challenge video to adjust the src/dst numbers for unwarping. The numbers work good on freeway but not on curved roads like in the challenge video.

2. There were sudden jumps in the highlighted lane edges as the history of previous frames is not enough. Initially I had only 3 frames in the history. But by debugging, I found that 8 frames is common. I made the history frames as 7 to avergae from. This fixed the sudden jump of high;ight region in sharp curves.

3. The number to know which frame is same as previous frame was also experimental. I printed the numbers of left, right edges polynomial coefficients. The 2nd gegree coefficiient was the sensitive one , followed by 1st order coefficient. The constant coeff was not changing much for this video.

4. The number to identify left or right curve is not perfect. Currently I put 0.02 to distinguish left vs right curve. But that is not working fully.

5.In between, program had problems because of None type of objects being not checked.
