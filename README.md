## Advanced Lane Finding

[//]: # (Image References)

[image1]: ./output_images/undistorted_chess_board.png "Undistorted Chessboard"
[image2]: ./output_images/undistorted_pipeline.png "Road Undistorted"
[image3]: ./output_images/binary_threshold_output.png "Binary Example"
[image4]: ./output_images/Perspective_Src_Dst_Pts_Mapping.png "Perspective Points Mapping"
[image5]: ./output_images/warp_output.png "Warping Result Output Visual"
[image6]: ./output_images/window_slide_output.png "Window slide operation Output Visual"
[image7]: ./output_images/polynomial_fit_output.png "PolynomialFit Lane approximation operation Output Visual"
[image8]: ./output_images/final_result_output.png "Final Result Output Visual"
[image9]: ./output_images/radius_curvature_formula.png "Radius of Curvature Formula"
[video1]: ./project_video.mp4 "Video"
---

### README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in "./Advanced-Lane-Finding.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of sobel x derivative sobel on l-channel and gradient thresholds on s-channel to generate a binary image (thresholding steps in cell index 7 inside method `binary_tranform`).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for perspective transform includes two functions `perpective_tranform_coords()` and `warp_image`, which appears in 9 and 10 code cells of the notebook file "./Advanced-Lane-Finding.ipynb".  The `perpective_tranform_coords()` function takes an image and returns the src and dst corners points for warping an image.

 The `warped()` method takes an image, it uses the `perpective_tranform_coords()` for src and dst points. It then calls OpenCV's `cv2.getPerspectiveTransform` and `cv2.getPerspectiveTransform` to get warping and inverse warping matrices. Finally, it calls `cv2.warpPerspective` to warp the input image using M matrix.

 Here's an example of my output for this step.
![alt text][image5]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a saved pipeline image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To find the lane pixels, I have made use of Line class to save the latest generated polynomial fit result. I have used methods `perform_window_slide()` and `update_polyfit_models()` during the pipeline to generate and update fit models. These methods are available in code cells 12 and 14 in notebool file "./Advanced-Lane-Finding.ipynb".

When the lane line fit models are not present (For the first frame or due to the outlier frames), we will use `perform_window_slide()` operation to generate a polynomial fit model. This functions makes use of histogram to capture peak values in the first and second halfs of the histogram and use those to estimate the best avg lanes pixels. It then moves above the avg lanes pixel points for the next frame until it reaches to the top of the image.

 Here's an example of my output for this step.
![alt text][image6]

Once the polynomial fit model is generated, we will make use it to estimate the lane pixel positions in the next video frame in method `update_polyfit_models()`.

 Here's an example of my output for this step.
![alt text][image7]

For various reasons, if the output from `update_polyfit_models()` is not valid, we will reset the last detected fit inside Line model class and perform `perform_window_slide()` again.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The method `calculate_curvature()` is used to calculate radius of curvature for a given image along with left and right polynomial fits.

I used the suggested derivate formula over second degree polynomial as shown below.

![alt text][image9]

Also, we made sure to convert pixels to real world values while generating left and right curvature radius as these gets mapped to the real world image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The method `process_image()` inside code cell 21, returns the final output image with lane polygon plotted onto the real world image.

To properly plot polyfill shape on to the actual real world image, i have used the method `convert_to_real_space()` which takes the inverse matrix value returned from `warp_img()` method to perform this task with `cv2.warpPerspective` method.

.  Here is the final result for one of the image from video pipeline:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I have noticed significant issues while running this pipeline on the challenge videos. 

I believe, it might be better to clearly extract the yellow and white lanes lines to discard border shadows etc.

I also should use the best_fit model computed over n polyfit models instead of using the most recent one. This might add more smoothness to plotted shaped.

I also should verify if the lane lines are parallel(approx).
