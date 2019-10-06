## Writeup

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

All code is located in the juypter notebook _advanced_lane_finding.ipynb_.

[//]: # (Image References)

[distorted]: ./examples/calibration1.jpg
[undistorted]: ./examples/calibration1_undistorted.jpg
[standard]: ./examples/straight_lines1_standard.jpg
[distorted_test]: ./examples/straight_lines_distort.jpg
[undistorted_test]: ./examples/straight_lines_undistort.jpg
[color_selection]: ./examples/straight_lines1_color_selection.jpg
[warp]: ./examples/straight_lines1_warped.jpg
[histo]: ./examples/straight_lines1_histo.jpg
[windows]: ./examples/straight_lines1_windows.png
[polyfit]: ./examples/color_fit_lines.jpg
[unwarped]: ./examples/straight_lines1_unwarped.jpg
[final]: ./examples/straight_lines1_final.jpg
[project_video]: ./examples/out_project_video.gif
[challenge_video]: ./examples/out_challenge_video.gif
[fail]: ./examples/tunnel_fail.jpg

### Camera Calibration

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. 
Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  
Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  
`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 
I used the sub-pixel accurate location of the corners.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the calibration images using the `cv2.undistort()` function and obtained this result: 


Distorted        |  Undistorted
:-------------------------:|:-------------------------:
![alt text][distorted] |  ![alt text][undistorted]

### Pipeline (Single Images)

The different steps on the pipeline are demonstrated based on the following test image:

![alt text][standard]

#### 1. Distortion-Corrected Image.

Using the camera matrix and distortion coefficient calculated during the camera calibration we can undistort our test image:

![alt text][undistorted_test]

This functionality is implemented in the function `undistort_image()`.

#### 2.  Thresholded Binary Image

The thresholding is performed in the `color_selection()` function.

I used a combination of the HSL and YUV colorspace. Lane lines can be yellow or white. Therefore, I extracted these two colors from the image.

HSL (white and yellow): 
For the white color, I chose high light value (205 - 255). I did not filter hue and saturation values.
For the yellow color, I chose hue between 10 and 40. I chose relatively high saturation and did not filter light value for the yellow color selection.

YUV (only yellow):
I only used the U-channel to extend the yellow color selection with a threshold of 105.

Moreover, I assumed that all yellow pixels are in the left half and all white pixels in the right half of the image.

The result of the color selection is shown below.

![alt text][color_selection]

#### 3. Persepctive Transform

The perspective transform is performed by a function called `warp_image()`.  The `warp_image()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. 

I chose to hardcode the source and destination points in the following manner:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 280, 700     | 250, 700        | 
| 595,  460     | 250,    0      |
| 725,  460     | 1065,   0      |
| 1125, 700   | 1065, 700       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear approximately parallel in the warped image:

![alt text][warp]

#### 4. Lane Finding


The first step is to create a histogram of lower half of the image. With this histogram we are adding up the pixel values along each column in the image. 
The two most prominent peaks in this histogram will be good indicators of the x-position of the base of the left and right lane line. The histogram of the example image is shown below.

![alt text][histo]


The next step is to initiate a Sliding Window Search (see figure below) in the left and right parts which we got from the histogram.

![alt text][windows]

The sliding window is applied by following steps:

1. The left and right start point is calculated based on the histogram.
2. Calculate the position of all non-zero x- and non-zero y-pixels.
3. Start iterating over the windows where we start from points calculate in step 1.
4. Identify the non zero pixels in the window just defined.
5. Collect all the indices in the list and set the center of next window using these points.
6. Separate the points to left and right positions.
7. Fit a second degree polynomial using np.polyfit for the points of the left and right boundary respectively.

The theory of the polynomial fitting is shown below:

![alt text][polyfit]

Finally, the image is unwarped again:

![alt text][unwarped]

This functionality is implemented in the function`fit_polynomial()`.

#### 5. Add Metrics

The following code shows the calculation of the curvature and offset:

```python
def measure_radius_of_curvature(fit_cr, img):
    y_eval = img.shape[1]
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
I assume that the projected section of the lane is about 30 meters long and 3.7 meters wide. 

I calculated the curvature of the right and left lane boundary separately and took the mean of both values. For a mean radius higher than 5000m I concluded that the road is straight.

For the offset I assumed that the center of the image is the center of the car.

#### 6. Final Result

I annotated the intermediate result from step 4 with the metrics from the last step and we finally achieve the following result:

![alt text][final]

---

### Pipeline (video)

The pipeline for videos include the following improvements:

1. Search from Prior: In the next frame of video we don't need to do a blind search again for the sliding window approach, but instead we can just search in a margin around the previous lane line position, 
like in the above image. This only works as long we found lane boundaries in the previous step which passed the sanity checks.

2. Sanity Checks: The sanity checks investigate the plausibility of the found lane lines. 
For example it is assumed that between frames the curvature and offset only changes slightly. 
Moreover, the lane width is checked. Too small and too wide lanes are omitted. Furthermore, the left and right lane line are not allowed to intersect each other.

3. In case, our current solution failed the sanity checks or we were not able to find a solution, we use the history to construct lane lines.

4. Smoothing: Based on the history and the current solution a weighted average is performed for smoothing.

The code to this changes can be found in the class `ProcessVideo`.

#### Video Examples

#### Project Video

![alt text][project_video]

Link to the project video mp4 [link to my video result](./output_videos/project_video_output.mp4)

#### Challenge Video

![alt text][challenge_video]

Link to the challenge video mp4  [link to my video result](./output_videos/challenge_video_output.mp4)

---

### Discussion

The biggest issue I faced during this task was the selection or extraction of the relevant pixels for lane finding. Finally, I decided to rely on colorspaces and 
do not use any gradient method. The gradient methods extracted too much not relevant information which makes it very difficult to find the correct position of the lane lines.

While the presented selection works very well for the project video, it fails in the tunnel section of the challenge video:

![alt text][fail]

I had to use the information from previous frames to construct lane lines for this frames. Here this works quiet good. Nevertheless, it might be dangerous to rely on old frames for more than a few frames.

Methods like histogram equalization or contrast limited adaptive histogram equalization (CLAHE) could solve this problem. Nonetheless, it might be quiet difficult to implement this in a robust fashion.

Moreover, the pipeline performs not well on unseen elements, such as bends, uneven lighting, or lane changing situations.

To get a more robust lane detection one could use deep neural networks. For example a convolutional neural network (CNN) or a recurrent neural network (RNN). Also a hybrid deep architecture combining CNN and Rnn can be very successful.
