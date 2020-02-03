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

[image1]: ./output_images/correction.png "Undistorted"
[image2]: ./output_images/correction2.png "Road Transformed"
[image3]: ./output_images/sobel.png "Gradient Tresholding"
[image4]: ./output_images/hls.png "Color Tresholding (HLS color space)"
[image5]: ./output_images/lab.png "Color Tresholding (LAB color space)"
[image6]: ./output_images/perspective.png "Perspective transform"
[image7]: ./output_images/perspective1.png "Perspective transform and Gradient tresholding"
[image8]: ./output_images/_test1.jpg "Output"
[image9]: ./output_images/bug1.png "Bug"
[image10]: ./output_images/rectangles.png "Find lane line"
[image11]: ./output_images/lab_hls.png "HLS & LAB"
[image12]: ./output_images/Capture2.jpg "warped"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
Camera calibration is an important step in analyzing camera images .We need to to distortion correction so we can get useful information out of camera images.Distorted images can change the apparent size of an object or can make objects appear closer or farther away that they actually are. Therefore to avoid confusions like that ,we need to  to calculate camera matrix, distortion matrix, and others coefficients for apply distortion correction on raw images.
The code for this step is contained in the first   nine code cells of the IPython notebook located in ".Project P2.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
To apply distortion correction you need to:
-find the corners of a chessboard image `cv2.findChessboardCorners()`. 
-drawing detected corners on an test image 
-camera calibration, given object points, image points, and the shape of the grayscale image`cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None)`
-undistorting a test image using (mtx) and  (dist) coefficients`cv2.undistort(img, mtx, dist, None, mtx)`

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)
##### 2.1. Gradient Thresholding (Sobel X,Y,Magnitude,Direction)
![alt text][image3]
##### 2.2 Color Thresholding (color extraction for yellow and white, using the HLS and HSV)
![alt text][image4]
##### 2.2.1 Color Thresholding (color extraction for B and L channel using the LAB color space)
![alt text][image5]
#### 2.2.1  vs 2.2
![alt text][image11]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()` in the  IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:
![alt text][image12]
```python
points = np.array([[580,469],
                   [280,679],
                   [750,467], 
                   [1070, 676]], np.int32)
src = points.astype(np.float32)
(x_len, y_len) = (720,1280) 
dst = np.array([[200, 0], 
                [200, 680], 
                [1000, 0],
                [1000, 680]], np.float32)
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 469      | 200, 0        | 
| 280, 679      | 200, 680      |
| 750, 467      | 1000, 0       |
| 1070, 676     | 1000, 680     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image10]
Steps:
- 0.Splid the image in nwindows
- 1.Loop through each window in nwindows
- 2.Find the boundaries of our current window. 
- 3.Use cv2.rectangle to draw these window boundaries onto our visualization image out_img. 
- 4.Find out which activated pixels from nonzeroy and nonzerox above actually fall into the window.
- 5.Append these to our lists left_lane_inds and right_lane_inds.
If the number of pixels you found in Step 4 are greater than your hyperparameter minpix, re-center our window (i.e. leftx_current or rightx_current) based on the mean position of these pixels.
- 6.Fit a polynomial

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines :
```python
    ym_per_px = 30/720 # meters per pixel in y dimension
    xm_per_px = 3.7/900 # meters per pixel in x dimension

    # Fit new polynomials to x,y in world space
    left_fit_cr = np.polyfit(lefty*ym_per_px, leftx*xm_per_px, 2)
    right_fit_cr = np.polyfit(righty*ym_per_px, rightx*xm_per_px, 2)
    
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_px + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_px + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
    
    #distance from center: (min X point from the left+ max point x from the right )/2 -middle line 
    car_position = warped_image.shape[1]//2
    center_line = (min(leftx) + max(rightx))//2
    distance = np.abs(car_position - center_line) * xm_per_px
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:
I saved the output images in the folder ./output_images/.

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
##### What problems / issues did you face in your implementation of this project

After I did all steps from  above I tried my pipeline project_video and I noticed it had some problems:
![alt text][image9]
So I returned to step 2 with gradient transformation to search the solution for my problem.I tried  different color transformation:
1.selecting yellow and white color from HSL and HSV color space:![alt text][image4]
2.extracting L and B channel from LAB color space:
![alt text][image5]
3.applying Sobel X,Y ,magnitude and sobel direction
:![alt text][image3]
I choose extracting L  and B channel from LAB color space and I got better results.[alt text][image4]


##### Where will your pipeline likely fail?

1.One of the case  when  my pipeline failed was when my detection algorithm didn't fit perfectly  with the actual lane from some frames.

My pipeline performs quite well on the project_video.mp4  video file but  it might not perform well on others videos that have a lot of unseen elements such are in harder_challenge_video.mp4 or challenge_video.mp4.
Overall the project was interesting and I 'm  excited about the next projects and courses. :) 

##### what could you do to make it more robust?

This project was more challenging than the previous one and I think there is a lot of improvements that can be implemented to detect lane line with high accuracy.

1.Smoothing the right line which can reduce jittering.
2.Include a  dynamic thresholding depending the lighting conditions of the road.
3.Optimization of the algorithm for the detection of 45 or 90 degree curves









