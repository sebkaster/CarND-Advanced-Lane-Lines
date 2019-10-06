## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

All code is located in the juypter notebook _advanced_lane_finding.ipynb_

[//]: # (Image References)

[distorted]: ./examples/calibration1.jpg "Undistorted"
[undistorted]: ./examples/calibration1_undistorted.jpg "Road Transformed"
[distorted_test]: ./examples/straight_lines_distort.jpg "Undistorted"
[undistorted_test]: ./examples/straight_lines_undistort.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. 
Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  
Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  
`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 
I used the sub-pixel accurate location of the corners.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the calibration images using the `cv2.undistort()` function and obtained this result: 


Distorted        |  Undistorted
:-------------------------:|:-------------------------:
![alt text][distorted] |  ![alt text][undistorted]

### Pipeline (Single Images)

#### 1. Distortion-Corrected Image.

Using the camera matrix and distortion coefficient calculated during the camera calibration we can undistort our test image.

![alt text][image2]

#### 2.  Thresholded Binary Image

I used a combination of the HSL and YUC colorspace. Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Persepctive Transform

The perspective transform is performed by a function called `warp_image()`.  The `warp_image()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. 

I chose the hardcode the source and destination points in the following manner:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 280, 700     | 250, 700        | 
| 595,  460     | 250,    0      |
| 725,  460     | 1065,   0      |
| 1125, 700   | 1065, 700       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?


The first step is to create a histogram of lower half of the image. With this histogram we are adding up the pixel values along each column in the image. 
The two most prominent peaks in this histogram will be good indicators of the x-position of the base of the left and right lane line.

The next step is to initiate a Sliding Window Search in the left and right parts which we got from the histogram.

The sliding window is applied in following steps:

1. The left and right start point is calculated based on the histogram.
2. We then calculate the position of all non-zero x- and non-zero y-pixels.
3. We then start iterating over the windows where we start from points calculate in step 1.
4. We then identify the non zero pixels in the window we just defined.
5. We then collect all the indices in the list and decide the center of next window using these points.
6. Once we are done, we separate the points to left and right positions.
7. We then fit a second degree polynomial using np.polyfit for the points of the left and right boundary respectively.

The result of the polymial fitting is shown below:

![alt text][image5]

Finally, the image  is unwarped again:

![alt text][image5]

#### 5. Add Metrics

The following code show the calculation of the curvature and offset

```python
def measure_radius_of_curvature(fit_cr, img):
    y_eval = img.shape[1] * config.ym_per_pix
    curverad = ((1 + (2*fit_cr[0]*y_eval*config.ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
    return curverad

def measure_offset(img, left_fit, right_fit):
    x_max_real = img.shape[1]*config.xm_per_pix
    y_max_real = img.shape[0]*config.ym_per_pix
    
    vehicle_center = x_max_real / 2
    line_left = left_fit[0]*y_max_real**2 + left_fit[1]*y_max_real + left_fit[2]
    line_right = right_fit[0]*y_max_real**2 + right_fit[1]*y_max_real + right_fit[2]
    line_center = line_left + (line_right - line_left)/2
    offset = line_center - vehicle_center
    return offset
```

I calculated the curvature of the right and left lane boundary separately and took the mean of both values.

For the offset I assumed that the center of the image is the center of the car.

#### 6. Final Result

I annotated the intermdiate result from step 4 with the metrics from the last steps and we finally achieve the following result:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video_output.mp4)

Here's a [link to my video result](./output_videos/challenge_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
