## Advanced Lane Finding Project
---

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

[image1]: ./camera_cal/calibration1.jpg "Original chessboard"
[image2]: ./writeup_images/image1.jpg "Undistorted chessboard"
[image3]: ./test_images/test1.jpg "Road Original"
[image4]: ./writeup_images/image2.jpg "Road Undistorted"
[image5]: ./writeup_images/image3.jpg "Road Thresholded"
[image6]: ./test_images/straight_lines1_undist.jpg "Image used for perspective transform"
[image7]: ./writeup_images/image4.jpg "Warped image"
[video1]: ./project_video.mp4 "Video"

---

Below are steps of the pipeline with description of each step. Note that the code for each step is located in the iPython notebook. The sequence of the steps in the notebook is the same.

## 1. Camera Calibration

To calibrate camera we use chessboard images provided in the project folder.
We expect chessboard to have 9 corners in each row and 6 corners in each column.
`cv2.findChessboardCorners` finds chessboard corners in each chessboard image which satisfies 9X6 pattern.
Chessboard corners are used as image points input to `cv2.calibrateCamera`. As object points (world coordinates of chessboard corners) we simply use grid-like coordinates with Z coordinate being 0.
Distortion coefficients and camera matrix obtained from `cv2.calibrateCamera` call are used to undistort each image in the pipeline. The code can be found in 'Calibrate Camera' section in the iPython notebook.

Here is an example of original chessboard image and undistorted image:
Original   
![image1]
  
Undistorted  
![image2]

And here is an example of original road image and undistorted road image.
Original   
![image3]
  
Undistorted  
![image4]  

Undistortion is the first step in the pipeline.

## 2. Image thresholding
Next step of our pipeline is to convert undistorted RGB road image to binary image with lane lines being highlighted relative to its background.

To detect yellow line we use B channel of LAB color space. We select all pixels which are above 97th percentile of B channel values. Such a 97th percentile threshold is calculated in the lower half of the image to eliminate 'sky' area and increase influence of the road on percentile calculation.

To detect white line we use Y channel of YCrCb color space. The lower threshold is 97th percentile, upper threshold is max value of Y channel.

Such color space and color channel selection has been decided after experimentation with different color spaces and their color channels. It was identified that yellow lane line has high values in B (LAB) channel and white color has high value in Y (YCrCb) color channel. Percentile is used as a threshold instead of raw value as images with different brightness have different distribution of values in each color channel. Using high percentile allows to identify areas of the image with high intensity of that color channel (and in our case such areas of the image are road lanes).

Here is an example of image thresholding:
Original:  
![image3]  

Thresholded image (blue color shows white areas and green color shows areas selected after yellow color thresholding).
![image5]

## 3. Perspective Transform

At this step we are finding direct and inverse perspective transform using source and destination points on some test image. Source points are four points on lane lines where two top points and two bottom points have same Y coordinate. Destination points create rectangle.

Such transform allows to view road image from bird view which is convenient to estimate lane line polynomials.

The follwoing source and destination points have been used:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 230, 700      | 200, 720      | 
| 502, 514      | 200, 500      |
| 783, 514      | 1080, 500     |
| 1066, 700     | 1080, 720     |

Here is an example of the original image and warped image where it is visible that the lane lines are parallel.

Original:    
![image6]  

Warped:  
![image7]  

# TODO: Add warped thresholded image.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
