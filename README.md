# Advanced-Lane-Finding
#### A pipeline to detect lanes in a video stream using perspective transform, histogram peaks, and a polynomial curve fit.
---

The goals / steps of this project are the following:

* Compute a camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[undistorted]: ./writeup_images/undistorted.png "Undistorted"
[original]: ./test_images/straight_lines1.jpg "Original"
[thresholded]: ./writeup_images/thresholded.png "Binary Threshold"
[warped]: ./writeup_images/warped.png "Perspective Transform"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration

#### 1. Computing the camera matrix and distortion coefficients.

The code for this step is contained in the first section of the jupyter notebook `advanced_lane_finding.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][undistorted]

### Pipeline (single images)

#### 1. Starting point

To demonstrate this step, I will describe the image processing pipeline using one of the test images like this one:
![alt text][original]

#### 2. Using color transforms, gradients or other methods to create a thresholded binary image

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at in Step 3 of the jupyter notebook). I found success by converting the image to HLS colorspace and thresholding the x-gradient of the L-channel and directly thresholding the S-channel.  Here's an example of my output for this step.

![alt text][thresholded]


#### 3. Performing a perspective transform to warp the image into a bird eye view

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src_pts = np.float32([(594,450),
                      (692,450), 
                      (250,680), 
                      (1050,680)])

loc = 380
dst_pts = np.float32([(loc,0),
                      (w-loc,0),
                      (loc,h),
                      (w-loc,h)])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 594, 450      | 380, 0        | 
| 250, 680      | 380, 720      |
| 1050, 680     | 900, 720      |
| 692, 450      | 900, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][warped]

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
