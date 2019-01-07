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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "pipeline.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function (line #23 of code cell 2).  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![cam cal](https://github.com/blanklist/CarND-Advanced-Lane-Lines/blob/master/cam_cal.png)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Once the camera is calibrated, one can use the same calibration values and replace the chessboard image with the more pertinent and 'real world' road image.
Here is an example of the same image calibration applied to a road image:

![road img cal](https://github.com/blanklist/CarND-Advanced-Lane-Lines/blob/master/road_img_cal.png)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines 3 through 20, cell #5, in `pipeline.ipynb`).  Here's an example of my output for this step.

![img transform](https://github.com/blanklist/CarND-Advanced-Lane-Lines/blob/master/img_transform.png)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform_matrix()`, which appears in lines 5 through 31, cell #6 in the file `pipeline.ipynb` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `perspective_transform_matrix()` function takes as input an image (`img`).  I chose to hardcode the source and destination points in the following manner:

```python
corners = np.float32(
    [[(img_size[0] / 2) - 54, img_size[1] / 2 + 95],
    [((img_size[0] / 6) - 15), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 60), img_size[1] / 2 + 95]])

warped_corners = np.float32(
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

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![warped](https://github.com/blanklist/CarND-Advanced-Lane-Lines/blob/master/warped.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I wrote two funcitons: find_lane_pixels() and fit_polynomial() found in cell #7 which attempt to find lane-line pixels and fit their positions with polynomials. The find_lane_pixels() function uses a histogram on the lowest part of the image to predict the most likely lane line pixels. The fit_polynomial() funciton leverages np.polyfit to calculate a second order polynomial based on the findings of find_lane_pixels() function.

![poly](https://github.com/blanklist/CarND-Advanced-Lane-Lines/blob/master/poly.png)

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I determined the radius of curvature of left and right lines in cell #9 in my code in `pipeline.ipynb`. The position of the vehicle is calculated with respect to the left and right lane lines in cell #11.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell #10 in my code in `pipeline.ipynb` in the function `redraw()`.  Here is an example of my result on a test image:

![redraw](https://github.com/blanklist/CarND-Advanced-Lane-Lines/blob/master/redraw.png)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://github.com/blanklist/CarND-Advanced-Lane-Lines/blob/master/output_images/project_video_processed.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The majority of this project was provided in the lecture section. The complications arose when putting multiple pieces together to create the pipeline() function. I was able to successfully implement the incremental steps in the earlier cell blocks. The pipeline() function then takes those steps and processes the images by leveraging early work. The main failing of my implementation is that each image is independent and does not refer to data found in the most recent images. If I were to spend more time on this project I would first create a Line() class for both the left and right lines. This class would enable me to keep track of the 'state' of both right and left lines and refer to the recent past of those lines in order to inform current measurements and make sanity checks throughout.
