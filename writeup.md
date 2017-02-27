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

[image1]: ./output_images/undistorted_chessboard.jpg "Undistorted"
[image2]: ./output_images/undistorted_test6.jpg "Road Transformed"
[image3]: ./output_images/gradienttest6.jpg "Binary Example"
[image4]: ./output_images/binary_warped.jpg "Warp Example"
[image5]: ./output_images/fit_lane_lines.jpg "Fit Visual"
[image6]: ./output_images/detected_lanes_undidtorted.jpg "Output"
[video1]: ./test.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This document is the writeup file provided.
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell block.  

I first created a function, called 'get_object_image_points_dir_list', that takes a path of calibration images, a number of corners in the x direction and the number of corners in the y direction. THis function creates a numpy array of chess corner points that are undistorted in the real world. The ufnction then loops through each file in the specificed path reading hte image in, finding the chessbaord corners and appending the image corners to an image points list and appending the object points to an object point list. This unfction returns the object points, 'objpoints' and the image points, 'imgpoints'. This function assumes the chess board is on a flat x,y plan with no change aloth the z axis.

The lists of object point, 'objpoint' and image points, 'imgpoints' are passed to 'cv2.calibrateCamera to find the calibration matrix, 'mtx' and distortion coefficients, 'dist'. A calibration image, "camera_cal/calibration1.jpg", calibration matrix, 'mtx' and distortion coefficients, 'dist' are passed to function cal_distortion which returns the calibration image undistorted. Below is the saved result of the calibration image.

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.

A test image, "test_images/test6.jpg", calibration matrix, 'mtx' and distortion coefficients, 'dist' are passed to function cal_distortion which returns the test image undistorted. Below is the saved result of the test image.

![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. The code for this is in cell block 3. There are three gradient thresholding functions. The first is an absolute sobel threshold function, 'abs_sobel_thresh', that takes 4 inputs. The first is an image, 'img', to process, the oreintation of the sobel operator, 'orient', that takes either x or y as direction, a threshold min and max tuple, 'thresh' and a sobel kernel size, "sobel_kernel", that is an odd numnber. It outputs the binary image.
The second is an mangitude sobel threshold function, 'mag_sobel_thresh', that takes 3 inputs. The first is an image, 'img', to process, a threshold min and max tuple, 'thresh' and a sobel kernel size, "sobel_kernel", that is an odd numnber. It outputs the binary image based on the sobel magnitude of each pixel from the sobel operator with given kernel size between the specified threshold.
The third is an direction sobel threshold function, 'dir_sobel_thresh', that takes 3 inputs. The first is an image, 'img', to process, a threshold min and max tuple, 'thresh' and a sobel kernel size, "sobel_kernel", that is an odd numnber. It outputs the binary image based on the sobel direction of each pixel from the sobel operator with given kernel size between the specified threshold.
There are two color space functions: The first function, 'hls_select', comverts a RGB image to HLS color space and returns the binary image of a given HLS color channel. It takes 3 inputs, a image to process called 'img', a threshold min and max tuple called 'thres' and a channel to threshold. It returns the binary image of 'img'.
The second function, 'hsv_select', comverts a RGB image to HSV color space and returns the binary image of a given HLS color channel. It takes 3 inputs, a image to process called 'img', a threshold min and max tuple called 'thres' and a channel to threshold. It returns the binary image of 'img'.

These functions are used to build the pipeline, 'binary_pipline', to combine several thresholding images into one binary image. It takes 7 input: an image to process called 'img, an L channel threshold tuple of min and max values called 'l_thresh', an S channel threshold tuple of min and max values called 's_thresh', an x direction sobel threshold tuple of min and max values called 'sx_thresh', a y direction sobel threshold tuple of min and max values called 'sy_thresh', a magnitude sobel threshold tuple of min and max values called 'mag_thresh' and a direction sobel threshold tuple of min and max values called 'dir_thresh'. The pipeline uses the x direction sobel and direction sobel binary images to cobmine to one where there are both equal to one. It then combines this with the S channel binary threshold with the x direction and direction binary image where either is equal to 1. It returns two images, 'color_binary' and 'binary' where one is the  S channel and comdined gradients as the green and blue channels respectively and the other is the 1 channel combined binary image. 

The 'binary_pipeline' is tested with the undistorted test images. Below is an example of the undistorted image and the binary output.

![alt text][image3]

Cell block 4 was used for testing different gradient and color channel threshold combinations for the final 'binary_pipeline' design.

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in cell block 6. There are no helper functions for this portion. The first portion of this code undistorts the given test image, passed it to the 'binary_pipeline' function to get the binary image and then gets the perspective transformation matrix, 'M' and the inverse perspective transformation martix, 'Minv' using the following source poists, 'src, and destination points, 'dst':

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 256, 696      | 240, 720      | 
| 1067, 696     | 1040, 720     |
| 585, 457      | 240, 0        |
| 698, 457      | 1040, 0       |

I verified that my perspective transform was working by outputing the warped image of each test image. Below is an example of the perspective transformation output:

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In cell block 5 I declare 2 helper functions for fitting polynomials to lane lines, 'fit_lane_lines' and 'known_line_location_fit',and a function for visualizing the results, 'draw_known_fit_lines'.

The first function, 'fit_lane_lines', takes 3 inputs: a birds-eye view of a wapred binary image with lane lines on it called 'binary_warped', a scale for x for converting to another unit of measure called 'scale_x' and a scale for y for converting to another unit of measure called 'scale_y'. This function will create a historgram of the white pixels in the x direction and split into a left side and right side. It will then find the max frequency for the left and right historgrams to define an intial starting point to find pixels for the left and right lanes on the x axis. A window margin of +/-100 is set to find pixels and a minimum pixel threshold of 50 is set. The minimum pixle threshold is the minimum number of pixels needed in the window to recetner the window on those pixels. There are 9 windows to search for lane pixels that will move in the y direction. For each search window it will set the window size based on current window number. The positon of pixels in the left and right window are recorded spearately and are appended to a left lane indices and right lane indicies lists. If the number of detected points are greater than 50 for the left right (separately) then recenter the window to the new mean x value. Else leave the widnow in the current x position. The left and right indicies lists are merged together into a left and right indicies arrays. The left and right indicies are then used to limit the non-zero image points to left lane points and right  lane points into separate x and y position arrays. The left and right x and y arrays are passed to the numpy polyfit function to find the right and left polynomial fits. These fits ar also used to find the radius of curavture for the left lane and right lane based on the specified scale of x and y. This function returns the left fit, right fit, left lane indicies, right lane indicies, left lane cure and right lane curve.

The second function, 'known_line_location_fit', takes 5 inputs: a birds-eye view of a wapred binary image with lane lines on it called 'binary_warped', a scale for x for converting to another unit of measure called 'scale_x' and a scale for y for converting to another unit of measure called 'scale_y'. This function uses the previous frames left and right polynomial fits with a margin of */-100 limit the search area of non-zero pixels in 'binary_warped' for elft and right pixel indicies. The left and right indicies are then used to limit the non-zero image points to left lane points and right  lane points into separate x and y position arrays. The left and right x and y arrays are passed to the numpy polyfit function to find the right and left polynomial fits. These fits ar also used to find the radius of curavture for the left lane and right lane based on the specified scale of x and y. This function returns the left fit, right fit, left lane indicies, right lane indicies, left lane curve and right lane curve.

The visuaization function, 'drawn_known_fit_lines', takes 5 inputs: a binary warped image called 'binary_warped', a left and right polynomial fit called 'left_fit' and 'right_fit' repsectively, and the left and right image non-zero pixel indicies for 'binary_warped' called 'left_lane_inds' and 'right_lane_inds' respectively. It colors the left lane red, the right lane blue and draws the left and right ploynomails on the binary warped image to show the results of the fits and the detected lines. Below is an example of its output:

![alt text][image5]

These fucntions are used in cell block 6 to take test images, undistort them, use the 'binary_piple' on the them to get the binary image, warp the binary image to a birds eye view above the lane, then use 'fit_lane_lines' to find the polynomial fit for the left and right lane and 'drawn_known_fit_lines' to visualize the result.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The function 'radius_curvature' is at the bottom of cell block 5 and take 5 inputs: the y and x values called 'ploty' and 'x' respectively, the evaluation point for y called 'y_eval' amd the scale of x and y for unit conversions called 'scale_x' and "scale_y respectively. It is used in 'fit_lane_lines' and 'known_line_location_fit' to return the left and right lanes curvature from the left and right fit. It uses the following formula to find the radius of a curvature for a 2nd order polynomial: ((1+(2Ay+B)^2)^(3/2))/|2A|. It returns this result.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I tested this in cell block 7 and implemented it in the video processing pipeline.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I really followed the lessons to the Advanced land finding technqiues. I used the code I had completed in the quizzes to solve most of the rubric requirements in this project. I used trial and error to come up with what I believed to be the best binary image combinations between the gradients and color channels. I tried including the L channel from HLS but I found where there were shadows cast the road it caused a problem. In the end I used the S channel from HSV, the x sobel and direction sobel where the final result was S|(X&DIR) for the combination. I had a few issues with my initial persepctive transformation but once I used the interative matplotlib output I quickly solved the precision issue I was running into.

I also used the fit detection code from the lesson to solve that portion of the problem. I spent quite a bit of time following the steps to see what it was doing in the sliding window search and the previous fit to find the current frames lane lines. THat's part of the reason that I included  a lengthy explanation of those functions. I wanted to re-iterate line by line what it was doing to find the pixels for the left and right lane.

Once I had those issues solved finding the lane curvature and vehicle location was easy. I read some OpenCV documentation to find the cv2.putText function to have the radius and vehicle location results on the video.

This pipeline is likely to fail where road lines are not vibrant due to dust or fading. I live a Canada are during the spring time road lines are covered with dust and sand from sanding in the winter months. This gives them a road like color so using the gradient and S channel thresholding these lines are likely not be pciked up by these thresholding technqiues.

This pipeline is also assuming a fixed camera angle. This means it assumes the camera does not change angle with respect to the car but it also assumes it does not change anlge with respect to the road. So this is likely to have skewed results that could end up in failure when the altitude of the road changes significantly ie uphill or downhill. This also means this pipeline will have issues with bumpy roads where the vehicle is moving up and down due to the road conditions. IN this case the camera would be changing angle with respect to the road. This could again skew results from the pipeline causing failure.

This pipeline is designed to take 720x1280 images. Changing the image dimensions will cause it to fail. This could be solved by generalizing a formula for the source points in the perspective transformation.

This pipeline assumes unidirectional curves. I used a 2nd order polynomial so the detected curve will only be unidrectional. If the vehicle was on a windy 'S' shaped road it would not fit a proper polynomial to the shape of the road and would most likely end in failure. If a third order polynomial was used it would be able to fit an 'S' shaped curve. The sliding window used to detect lane pixels would be able to move the window in the proper direction to detect an shaped lane. It's really the implementation of the polynomial fit where it would fail. If the code was expanded to consdier 3rd order polynomial curves it would be able to detect 'S' shaped lines.

This pipeline assumes ideal weather conditions. Since the lane detection depends on the camera seeing the lines then poor weather conditions such as heavy snow or rain would likely reduce the lane detection and could result in failure. This is just the nature of using cameras and other sensors would have to be used to got over visibility challenges.