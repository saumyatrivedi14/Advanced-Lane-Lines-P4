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

[image1]: ./output_images/Test_Undistorted.png "Undistorted"
[image2]: ./output_images/Test_Undistortion_Warping.png "Undistorted and Warped"
[image3]: ./output_images/Undistorted Image.png "Undistorted Test Image"
[image4]: ./output_images/Gradient X Threshold.png "Gradient X Threshold"
[image5]: ./output_images/Gradient Y Threshold.png "Gradient Y Threshold"
[image6]: ./output_images/Gradient Mag_Threshold.png "Gradient Magnitude Threshold"
[image7]: ./output_images/Gradient Dir_Threshold.png "Gradient Direction Threshold"
[image8]: ./output_images/Final Grayscale.png "Combined Gradient Threshold"
[image9]: ./output_images/HLS S Color Filter.png "S Filter"
[image10]: ./output_images/Final Grayscale.png "Combined Gradient Threshold"
[image11]: ./output_images/Final Grayscale.png "Combined Gradient Threshold"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./ALF_Pipeilne.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result [Source: Udacity Lecture Notes]: 

![alt text][image1]

This undistorted image was given a perspective transform using cv2.getPerspectiveTransform() and warped using cv2.warpPerspective(), the code for this is in Cell 4 & 5 of jupyter notebook. . Rectangular co-ordinates from the source (undistorted image) were selected and destination (undistorted and warped image) co-ordinates were also given, so that the function can map each points on the destination image. Using the camera matrix and distortion coefficients from the first step, a Map matrix was found along with its inverse matrix to undistorted and warped the images as shown below:

![alt text][image2]

### Pipeline (Test Image)

#### 1. Distortion Correction.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one below. I used cv2.undistort() function for this and it is observable from the edges of the image. the code for this is in Cell 2 & 5 of jupyter notebook. 

![alt text][image3]

#### 2. Thresholding.

I used a combination of color and gradient thresholds to generate a binary image. The test image selected for this is "test5.jpg"
First step includes, gradient thresholding using Sobel Transform along X and Y axis as shown below, the part of this code is in Cell 5 using  abs_sobel_thresh() function.

![alt text][image4]
![alt text][image5]

Gradient Magnitude and Direction threshold were also implemented using mag_threshold() and dir_threshold() functions, as shown below. The part of this code is in Cell 5

![alt text][image6]
![alt text][image7]

Combining these gradient thresholding techniques, the output is shown below.

![alt text][image8]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:



---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
