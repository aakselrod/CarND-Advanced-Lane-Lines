##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/calibration2_undistorted.jpg "Undistorted"
[image2]: ./output_images/straight_lines1_pipeline-undistorted.jpg "Road Transformed"
[image3]: ./output_images/straight_lines1_pipeline-comb_binary.jpg "Binary Example"
[image4]: ./output_images/straight_lines1_pipeline-warped.jpg "Warp Example"
[image5]: ./output_images/straight_lines1_pipeline-warped_with_windows.jpg "Fit Visual"
[image6]: ./output_images/straight_lines1_pipeline-fully-processed.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./adv_lane_lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To generate a distortion-corrected image like the one below, I used `cv2.undistort()` with the previously discovered parameters to correct the distortion of the `straight_lines1.jpg` image:
![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in the `pipeline` function of the above-named Python notebook, starting with the `# Sobel x` comment and ending where `comb_binary` is created). The parameters I found to work well, I set as the function defaults for the `pipeline` function. Here's an example of my output for this step for the same image as above. I used the OpenCV function `Sobel` to detect gradients in X and Y, and then thresholded both the X and Y gradients as well as the S channel.

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is also in the `pipeline` function in the 3rd code cell of the IPython notebook. It's a single line below the `# Warp the perspective...` comment right after the color/gradient thresholds using the OpenCV `warpPerspective` function. I chose to hardcode the source and destination points in the following manner:

```
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identifying lane-line pixels is also in the `pipeline` function in the 3rd cell. It starts with the `success = False` code and continues to the `out_img[ploty, right_fitx] = [255, 255, 0]` line. In that space, the code tries to use the previous fit weighted average to find the lane line pixels if the previous fit exists, fits the pixels to a polynomial as described in section 5, and then adds the current fit to the weighted average. If the sanity check fails, it tries to use the previous fit weighted average without the new polynomial fit. If that doesn't work either, or if there's no previous fit weighted average, it attempts to re-detect the lane line pixels using a histogram and windows. A sample image using windows to detect the lane lines is below:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The position of the vehicle is calculated in the lines starting with `y_eval = np.int(np.max(ploty))` and ending at `offset = mppx * ...`, which calculates how off-center the lane lines are in the image and infers the camera position from that. The curve radius of the lanes is calculated in the lines right after that, starting with `left_fit_cr = ...` and ending at `right_curverad = ...` by fitting to a polynomial in world-space and calculating the curve radius from that as described in the class materials. These are calculated within the loops that identify the lane-line pixels and fit their positions with a polynomial because the values are used for a sanity check (assuming the vehicle is in a lane and the curve radius isn't too small, creating too tight a curve to be navigable).

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The result is plotted back down onto the road in the `pipeline` function in the third code cell. The lines where this is done start at `warp_zero = ...` and end at `result = ...`. The lines are plotted in birds' eye view perspective, then `warpPerspective` is used with the `Minv` perspective transform calculated using the flipped source and destination points calculatd above, and then that's combined using the `addWeighted` OpenCV function with the original undistorted image. Then, the curve radius and offset are printed in the top left of the image starting at `curve = ...` and ending at `cv2.putText(result, ...)`. The final image looks like this:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I originally tried to use only windows for lane line pixel detection, but there were difficult frames that caused the detected lane to disappear or spiral out of control for several frames at a time. I then tried to use a weighted average of previous fits, but the difficult frames caused the detected lane to disappear and stay disappeared because the offset was too high. I then added sanity checks for the offset and curve radius and attempted, in order, to use the weighted average, then just the last fit, and then start over with windows. This made it so that the detected lane stayed true. The pipeline is still likely to fail when switching lanes, or in situations where none of the above techniques work correctly, such as lighting conditions that don't work well with the default gradient/color thresholds I used. One technique that might help is to set gradient and color thresholds using a convolutional neural network trained on multiple roads in multiple lighting conditions with a large amount of augmentation.