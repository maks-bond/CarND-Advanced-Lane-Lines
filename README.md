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
[image13]: ./writeup_images/image10.jpg "Image with activated pixels almost lost"
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

## 6. Pipeline steps
Here is the summary of pipeline steps (main function is 'update_lines'):
1. Undistort an image
2. Threshold an image and convert it to binary
3. Warp the image to have 'bird view'
4. Calculate polynomial coefficients for the left and right lane. If previous frame has produced valid polynomial coefficients, use them to search for new polynomials around old ones using activated pixels of the new frame, otherwise use window search.
5. Run sanity check on newly calculated polynomial coefficients to discard potentially incorrect polynomials. We only check that lane curvature of one lane is less than 5X curvature of another lane.
ed on the next step.
6. Calculate weighted average polynomial coefficients to achieve smoothing effect.
7. Use polynomial coefficients to draw lane lines on the original image by drawing them first on the warped image and then unwarping that image using inverse perspective transform matrix.
8. Calculate car center offset and draw it with lane curvatures on the image.

Example of the video frame after application of the pipeline functionality

![image12]


## Project video

Here's a [link to project video](./test_video_output/project_video.mp4)

## Challenge video
Here's a [link to challenge video](./test_video_output/challenge_video.mp4)

## Discussion

### Pipeline problems and ideas for improvement

Here is the list of weak points of the pipeline and discussoin on how can it be improved and discussion of some inaccuracies in the project and challenge videos:

1. Thresholding doesn't always identify pixels of lane lines which are further away from the car. That causes polynomial coefficients to be calculated incorrectly which causes incorrect curvature calculation. By watching project and challenge videos it might be visible that curvature of left and right line sometimes differs by more than 2X. Also in the project video last frames have right line drawn a bit off which is probably caused by lack of activated pixels to correctly estimate the polynomial.  
To improve thresholding I would experiment a bit more with thresholding technique. In current pipeline YCrCb and LAB color spaces have been chosen experimentally and also the 97th percentile for lower threshold is a variable which might have better value. Also magnitude and direction of gradient haven't been used at all in the current pipeline. Experiments didn't show the value of such thresholding approaches. But maybe gradient magnitude and direction might be combined with color thresholding approaches to improve the threshodlding algorithm.

2. Destination points of the perspective transform have been selected in a way to project the lane lines area of the image, but other image pixels are lost in the warped image. Since the perspective transform has been estimated using one test image, there is no gurantee that other images of the video will have lane lines projected correctly. For example in this image the activated pixels of the right lane lines are almost lost as they go outside projection area:  
![image13]  

The conclusion from this problem is that possibly we have to select destination points to have activated pixels be closer to the center in the warped image, however that might introduce additional lane lines of neighboring road lanes in the warped image which might confuse the 
'window search' algorithm.

Also this problem is probably the reason for majority of failures in the harder challenge video where the road is very curvy.

3. On the harder challenge video it is often the case when one of the lanes is out of the camera view. Current pipeline just fails to identify lane lines if we didn't find the polynomial for one of the lanes. In that case we could predict the location of the other lane using locaton and polynomial of the lane which is visible.