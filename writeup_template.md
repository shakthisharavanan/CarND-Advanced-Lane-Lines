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

[image1]: ./calibration.png "Calibration"
[image2]: ./test.png "Road Transformed"
[image3]: ./binary.png "Binary Example"
[image4]: ./example.png "unwarped"
[image5]: ./transform.png "Warped"
[image6]: ./fit_lines.png "Polynomial fitting"
[image7]: ./frames.png "subsequent frames"
[image8]: ./radius.png "Radius of curvature"
[image9]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd and 3rd code cell of the IPython notebook "adv-lane-lines.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds (as defined in code cell 5 in "adv-lane-lines.ipynb") to generate a binary image as seen below. I use the sobel filter and take the gradient in x direction and then apply a threshold to it. Then I use the s-channel from the HSV color space and apply a threshold to it. Then I combine the 2 thresholded images to give the final binary image.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in code cell 8 of the ipython notebook. The `warp()` function takes as inputs an image (`img`) and also if the inverse transform (i.e. from destination to source) should be performed. It also uses source (`src`) and destination (`dst`) points that were previously defined.  I chose the hardcode the source and destination points.


| Source        | Destination   | 
|:-------------:|:-------------:| 
| (600,450)      | (200,0)       | 
| (220,700)      | (200,700)      |
| (1080,700)     | (1000,700)      |
| (700,450)     | (1000,0)        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear somewhat parallel in the warped image. It is not perfect, but this will do.

Test

![alt text][image4]


Warped

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I took the histogram of the lower half of the warped binary image. The peaks of the histogram corresponded to the initial left and right lane positions. Then I used a sliding window approch to find all the pixels corresponding to left and right lanes. With these stored pixel values I fit a 2nd order polynomial as seen in cell 21 of the ipython notebook. 

![alt text][image6]

Once the initial left and right lanes are fitted with a polynomial, then I just search near these postions to find the lane lines of the subsequent frames. It is implemented in cell 23 of the ipython notebook.

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented this in the code cell 28 of the ipython notebook. I used the Radius of Curvature formula explained in the lecture notes. I assumed the camera is mounted at the center of the car and the deviation of the midpoint of the lane from the center of the image is the offset. 


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 29th and 30th cell of the ipython notebook. Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had to play around with the source and the destination points required for the perspective transform before finding the correct ones. I also had to play around with some thresholding parameters. Also I had to reduce the margin with which lanes in subsequent frames were bwing detected. Otherwise the algorithm failed at the parts where there were shadows on the road.

I believe there must be a better way of defining source and the destination points which make it more robust than my current method of manually defining the points. Also sometimes the histogram used to detect the intial lanes pixels may fail if there is a stronger edge created by the divider. The algorithm may fail in those cases.

I could take the average of starting lane pixels the previous 10 frames to make it more robust.
