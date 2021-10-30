# CarND-pj2_AdvanceLaneFinding

## Udacity Self-Driving Car Engineer - Project2: Advance Lane Finding (Computer Vision)

[//]: # (Image References)
[image1]: ./output_images/img_original.jpg "Original"
[image2]: ./output_images/Distortion.jpg "Undistorted"
[image3]: ./output_images/Conbined_Threshold.jpg "Binary"
[image4]: ./output_images/masked_image.jpg "Masked"
[image5]: ./output_images/transoform.jpg "Transform"
[image6]: ./output_images/warped_image.jpg "Warped"
[image7]: ./output_images/sliding_windows.jpg "Sliding Windows"
[image8]: ./output_images/search_around.jpg "Search Around"
[image9]: ./output_images/r_curve.png "Curve.png"
[image10]: ./output_images/rectangle.jpg "Rectangle"
[image11]: ./output_images/output.jpg "Output"
[gif1]: ./output/test_binary.gif "test_binary"
[gif2]: ./output/test_warp.gif "test_warp"
[gif3]: ./output/output.gif "output"

![output][image11]

# Advanced Lane Finding Project

The goals steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## Pipeline:

I will explain step by step the processing of each frame in the video. The numbering of the following sections corresponds to the numbering in `Advanced_Lane_Finding.ipynb` notebook. I show results over this original image:

![Original][image1]

### 1.- Camera Calibration:

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistorted][image2]

### 2.- Get Binary Image:

I used a combination of color and gradient thresholds to generate a binary image. I particularly combined:
- Gaussian Blur
- S channel of HSV (with with thresholds)
- Maasked yellow color in RGB (with with thresholds)
- Masked white color in RGB (with with threshold)
- Sobel gradient in x,y, magitude and direction.

Here's an example of my output for this step:

![Binary][image3]
arrart to verify that the lines appear parallel in the warped image.

![Warped][image6]

A piece of the video showing the output at this stage:

![warped_gif][gif2]

### 5.- Find Lines.
#### 5.1.- Sliding windows.

Once I have the transformed image I used sliding windows method to find lines. The line is searched at the bottom and then the image is traversed upwards by moving a "window" and looking for the continuation of the line in that region. This is the result:

![sliding_windows][image7]

#### 5.2.- Search from prior

Once I found the lines I use previous line information to search around this region. Results are the following:

![Search_around][image8]

If the lines are lost for more than `MAX_CORRUPTED_FRAMES` frames, I come back to the Sliding Windows method again. To detect loss of lines, compare previous radius and offset with with new ones.

### 6.- Measuring Curvature and Offset

The radius of curvature of each line are calculated using the following formula (given by Udacity).

![curvature][image9]

Then both radius are averaged and printed on each frame. 

To calculate offset, I evaluated the polynomial of each line at the bottom of the image and took the average of both. Then I got the difference between that point and the midpoint of the frame. This result is the offset.

The following parameters are used to transform measurements from pixels to meters:

- ym_per_pix = 30/720 # meters per pixel in y dimension
- xm_per_pix = 3.7/700 # meters per pixel in x dimension


### 7.- Return to original image

I implemented `get rectangle` to print a rectangle between lines. Then I used the inverse transform matrix to get back to the original perspective.

![output_rectangle][image10]

---
## Applying pipeline to video.

The results of processing the video through the pipeline are shown in the next gif:

---

### Discussion

I would like to mention some problems of using this pipeline:
- At one point in the video the image has low contrast and the lines are lost. Maybe it could be improved by applying some technique to improve the contrast in those frames.
- The bend radius measurement appears to fluctuate very fast. This can make a true line in a frame appear false compared to the previous frame.
