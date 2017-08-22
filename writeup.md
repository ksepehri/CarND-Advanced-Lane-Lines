## Writeup Template

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

[image1]: ./writeup_media/binary_birds_eye_luv.png "Binary Birds Eye"
[image2]: ./writeup_media/binary_luv.png "Binary"
[image3]: ./writeup_media/birds_eye_curved.png "Birds Eye Curved"
[image4]: ./writeup_media/birds_eye.png "Birds Eye"
[image5]: ./writeup_media/first_pass_success.png "First Pass Success"
[image6]: ./writeup_media/good_bad_lane_lines.png "Good and Bad Lane Lines"
[image7]: ./writeup_media/lane_lines_look_ahead.png "Lane Lines Look Ahead"
[image8]: ./writeup_media/lane_lines.png "Lane Lines"
[image9]: ./writeup_media/perspective_transform.png "Perspective Transform"
[image10]: ./writeup_media/undistort.png "Undistort"
[image11]: ./writeup_media/undistort_road.png "Undistort Road"

[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in code cells 2-7 of the IPython notebook located in "./lane_line_detection.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image10]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the checkerboard images like this one:
![alt text][image9]

Here I apply it to a test image:
![alt text][image11]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (Step 5 in notebook).  Here's an example of my output for this step. I used the LUV color space as it removed extraneous noise and displayed lanes more clearly than HLS.

![alt text][image2]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `road_pers_transform()`, which appears in Step 6 of the notebook.  The `road_pers_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
xff1 = 60
xff2 = 60
yff = 95
src = np.float32(
    [[(img_size[0] / 2) - xff1, img_size[1] / 2 + yff],
     [((img_size[0] / 6) - xff2), img_size[1]],
     [(img_size[0] * 5 / 6) + xff2, img_size[1]],
     [(img_size[0] / 2 + xff1), img_size[1] / 2 + yff]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
     [(img_size[0] / 4), img_size[1]],
     [(img_size[0] * 3 / 4), img_size[1]],
     [(img_size[0] * 3 / 4), 0]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image1]
![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then in steps 8 & 9 I identified lane line pixels in the rectified binary image. The left and right line have been identified and fit with a curved polynomial form. 

![alt text][image8]

In step 10 I took this further by applying a look ahead filter to limit searching for a lane. This improved my detection as well.

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in step 10 as well. The radius of curvature is based upon the [this site](http://www.intmath.com/applications-differentiation/8-radius-curvature.php). For the position I use the last item in the left and right fit to get the points at the bottom of the image. From there I calculate the width of the lane, then I half it and add it to one intercept and compare it to the middle of the image to get the offset.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in step 12.  Here is an example of my result on a test image:

![alt text][image5]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I took a very incremental approach to this project, i tried to get each step well implemented before adding complexities which may affect the outcome. My pipeline will fail in low contrast lighting conditions as it has not been fine tuned for that. I could make the pipeline more robust by adding additional color transforms/gradients.