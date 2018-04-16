## Writeup

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

[image0]: camera_cal/calibration1.jpg "Distorted"
[image1]: camera_cal_undistored_images/calibration1.jpg "Undistorted"
[image2]: test_images_undistored/test4.jpg "Undistored Image"
[image3]: test_threshold/test3_thresholding.png "Binary Example"
[image4]: test_warped/straight_lines1_unwarped_to_warped_with_dots.png "Warp Example"
[image5]: test_polyfit/straight_lines1_warped_polyfit.png "Fit Visual"
[image6]: test_polyfit/final_image_radcurvature_and_center_offset.png "Output"
[image7]: test_images/test4.jpg "Distored Image"
[video1]: test_videos_output/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained the method `calibrate_camera()` in cell `In [446]` of the IPython notebook located in `P4.ipynb`

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Before undistorting:
![alt text][image0]

After undistorting:
![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, here is an actual image of the road that I undistorted (before and after fixing the distortion).

Before undistorting:
![alt text][image7]

After fixing distortion:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding is done in the method `threshold(img, s_thresh=(170, 255), sx_thresh=(20, 100))` in `P4.ipynb` in code cell `In [26]`).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_image(undist_img)`, which appears in the file `P4.ipynb` (in cell `In [115]`).  The `warp_image(undist_img)` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image (see the red dots) and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used the code provided in the class to find lane lines by creating a histogram of the binary warped image and fit my lane lines with a 2nd order polynomial. The code for that is in the method `get_lane_line_info(binary_warped)` in the file `P4.ipynb` and an example output looks like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this after identifiying lane lines and fitting with a polynomial in the method `pipeline(img)` in the file `P4.ipynb`. The calculation for centure of curvature was provided during the class. For the center offset, I calculated the center of the lane and subtracted the center of the image as suggested during the class.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in a method called `pipeline(img)` (right after calculating the radius of curvature and center offset) in file `P4.ipynb` (in cell `In [496]`). Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

Here's a [link to my video result](test_videos_output/project_video_output.mp4)

---

### Discussion

I basically followed the approach suggested in the class lectures. First I did the camera calibration and used the information from the calibration to undistort the image. Then I applied thresholding to the undistorted image. Then performed a perspective transform on the thresholded image. Using the latter, I was able to compute the lane lines and fit polynomials thru them. I then computed the radius of curvature and center offset. And finally I unwarped the image to display the lane detection on the original image. One area where my pipeline fails is when the car drives over the brightest part of the road the lane detection extends all the way to the left railing. To make it more robust I may have to keep tweaking the thresholding.
  
