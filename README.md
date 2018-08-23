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
[thresh_warp]: ./writeup_images/thresh_and_warp.png "Thresholded and Warped"
[painted_lane]: ./writeup_images/painted_lane.png "Painted Lane"
[final]:./writeup_images/final.png
[video1]: ./test_videos_output/result.mp4 "Video"

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

The entire pipeline so far looks like this:

![alt_text][thresh_warp]

#### 4. Identify lane-line pixels and fitting their positions with a polynomial

This method is found in Step 4 of the jupyter notebook. To find the lanes, first I took a histogram of the thresholded and warped image. The I used a sliding window approach, centered around the two histogram peaks (one on each side of the image) to collect the pixel coordinates that are non-zero. After collecting non-zero pixel coordinates, I fed those points into the numpy polyfit function to get a polynomial. I then drew each polynomial onto the image with separate colors and filled the space in the middle with green. This is painted onto an image that still has transformed perspective, and overlayed onto the original later. At this point, the processed image looks like this:

![alt text][painted_lane]

In the same function, I calculated the center distance in the same function by determining the offset between the center of the image and the average location of the two lanes.

#### 5. Calculate the radius of curvature of the lane

In Step 6 I calculated the radius of curvature of the lane at the bottom of the image (i.e. the current location of the car) for each lane, averaged them together, and wrote the result on the image.

#### 6. The result plotted back down onto the road

This is done in Step 5. To add the result back onto the original image, I simply do a perspective transform again, but with the inverse matrix. Then I overlay the transformed image onto the original using `cv2.addWeighted()` and write the center distance onto the image. This is the final result:

![alt text][final]

---

### Pipeline

Here's a [link to my video result](./test_videos_output/result.mp4)

---

### Discussion

#### 1. Where will the pipeline likely fail?  How could it be more robust?

I spent too much time getting the binary thresholding to work well for the image I chose to test on and forgot to test it out on different images with different challenges, such as one where the pavement color changes in a line along the middle of the lane. So the pipeline works pretty well when the pavement changes to concrete and in shadowy conditions due to the color thresholding and gradient thresholding technique I used, but there are plenty of different scenarios where it breaks down. Second, I had no method to drop the lanes and start from scratch if it appeared to fit a lane that didn't make sense. For the project video this was fine, but for more challenging videos, if the lane position is drawn way off in one frame, it has a tendency to stay there for quite awhile since I look around the previously detected lane for the lane in the next frame. This would of course be catastrophic. Luckily, engineers spend more than just a couple weeks refining their techniques before it is considered ready to run on a real autonomous vehicle.
