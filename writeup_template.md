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
[image8]: ./writeup_images/image5.jpg "Original Thresholded image"
[image9]: ./writeup_images/image6.jpg "Warped Thresholded image"
[image10]: ./writeup_images/image7.jpg "Windows"
[image11]: ./writeup_images/image8.jpg "Polynomials drawn over warped image"
[image12]: ./writeup_images/image9.jpg "Polynomials drawn over warped image"
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

And here is an example of warped thresholded image:  
Original:  
![image8]

Warped:  
![image9]

## 4. Lane line pixel detection and polynomial fit

Next step of the pipeline is to identify lane line pixels and use them to fit second-order polynomial which estimates left and right lane lines. Warped binary image is used as an input.  

There are two ways to calculate lane line polynomials: to use window search by starting in the bottom at the places of highest concentration of activated pixels and moving up following the activated pixels of left and right line. Second way is to use polynomial calculated on previous video frame and search around pixels which correspond to this polynomial.

The code is located in the 'Extract Lane Pixels' and 'Calculate polynomial' sections of the notebook.

Here are examples of window search output and fitted polynomial.

Window search output. Windows define pixels which will be used to fit a polynomial. In this example all activated pixels correspond to lane lines and are within windows:
![image10]

Polynomial fit output:
![image11]

## 5. Radius of the curvature and vehicle center offset
Radius of the curvature has been calculated using the formula provided in the lecture. Since we need to calculate the radius in meters, I used the suggestion given in this post: https://knowledge.udacity.com/questions/21109  
Radius of the curvature is calculated using left and right lane polynomial coefficents. In the pipeline we apply smoothing where polynomial coefficients are the weighted average of polynomial coefficients on previous video frames.

Car center offset is calculated using the difference in pixels between center of the image and center of the lane at the bottom row of the warped image.

Curvature radius and car center offset are calculated in the `calculate_meters_curvature` and `calculate_offset` functions.

## 6. Example of the video frame after application of the pipeline functionality

![image12]

---

### Pipeline (video)

#TODO: Add details of pipeline functionality.

## 1. Project video

Here's a [link to project video](./test_video_output/project_video.mp4)

## 2. Project video
Here's a [link to challenge video](./test_video_output/challenge_video.mp4)

### Discussion

## 1. Pipeline problems and ideas for improvement

# TODO: Talk about curvature mismatch. Talk about problems on challenge video and problems on project video in the end. Talk about ptoblems on harder challenge related to lighting, having the line not always visible, perspective trasnform which might cause loss of lane line pixels. Ideally I would paste some pictures with identified problems found during debugging. But I can start without pictures.

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
