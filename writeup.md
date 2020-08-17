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

[image1]: ./output_images/undistort_output.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/binary_combo_example.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.jpg "Output"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./advanced_lane_finding.ipynb".  

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.


I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cells 4 and 5 in the `advanced_lane_finding.ipynb` IPython notebook).  Here's an example of my output for this step. 
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 7th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner (you can find them in the 6th code cell of the Notebook):

```python
src = np.float32(
    [[(img_size[0] / 2 - 59), (img_size[1] / 2 + 100)],
    [(img_size[0] / 6 - 10), img_size[1]],
    [(img_size[0] * 5 / 6 + 40), img_size[1]],
    [(img_size[0] / 2 + 61), (img_size[1] / 2 + 100)]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 581, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1107, 720     | 960, 720      |
| 701, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial.

Then I created a binary warped image in cell 8, added a function to find peaks in a histogram from that binary warped image (code cell 9) and fit my lane lines with a 2nd order polynomial on cell 10. This is my result:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 11 in the `advanced_lane_finding.ipynb` IPython notebook. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cells 12 and 13 in the functions `draw()` and `pipeline()`. The `draw()` function draws the measurements back onto the road, while the `pipeline()` function applies all the methods described above to find the lines and apply the results. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced the first issue as I was trying to find appropriate threshold values and Sobel kernel size. 
Other problems I faced were finding source and destination points that would lead to a good warped image and when I had to find correct hyperparameters. 
Generally when these values weren't chosen carefully, it could lead to a very distorted image result, especially when fitting the lines. 

My pipeline would likely fail when the lines are not clearly visible or the light conditions change with respect to the test images. 
The hardcoded source points are also a weakness, as they only work if the camera position and the road don't change. 
Something that I could do if I were to continue working on this project would be to use algorithms to programmatically detect the source points and other image attributes. 
Another thing could be to make more trials on the coefficient tuning so that I could improve the program output (especially when there is a slight wobble of the line). 