## Advanced Lane Finding
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
[image3]: ./output_images/Undistorted_Image.png "Undistorted Test Image"
[image4]: ./output_images/Gradient_X_Threshold.png "Gradient X Threshold"
[image5]: ./output_images/Gradient_Y_Threshold.png "Gradient Y Threshold"
[image6]: ./output_images/Gradient_Mag_Threshold.png "Gradient Magnitude Threshold"
[image7]: ./output_images/Gradient_Dir_Threshold.png "Gradient Direction Threshold"
[image8]: ./output_images/Final_Grayscale.png "Combined Gradient Threshold"
[image9]: ./output_images/HLS_SL_Color_Filter.png "S Filter"
[image10]: ./output_images/LAB_B_Color_Filter.png "B Filter"
[image11]: ./output_images/Combined_Color_Filters.png "Combined Color Filtering"
[image12]: ./output_images/Final_Gradient_Image.png "Final Grayscale Image"
[image13]: ./output_images/Stacked_Thresholds.png "Stacked both Thresholds"
[image14]: ./output_images/Warped_Image.png "Warped Test Image"
[image15]: ./output_images/Histogram.png "Histogram of warped Lanes"
[image16]: ./output_images/Sliding_Window_Visualization.png "Sliding Window Visualization"
[image17]: ./output_images/Panel_Output.png "Panel Output"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration

The code for this step is contained in the second code cell of the IPython notebook located in `./ALF_Pipeilne.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result [Source: Udacity Lecture Notes]:
![alt text][image1]
This undistorted image was given a perspective transform using `cv2.getPerspectiveTransform()` and warped using `cv2.warpPerspective()`, the code for this is in `Cell 4 & 5` of jupyter notebook. . Rectangular co-ordinates from the source (undistorted image) were selected and destination (undistorted and warped image) co-ordinates were also given, so that the function can map each points on the destination image. Using the camera matrix and distortion coefficients from the first step, a Map matrix was found along with its inverse matrix to undistorted and warped the images as shown below:
![alt text][image2]

### Pipeline (Test Image)
#### 1. Distortion Correction.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one below. I used `cv2.undistort()` function for this and it is observable from the edges of the image. the code for this is in `Cell 2 & 5` of jupyter notebook. 
![alt text][image3]
#### 2. Thresholding.

I used a combination of color and gradient thresholds to generate a binary image. The test image selected for this is "test5.jpg"
First step includes, gradient thresholding using Sobel Transform along X and Y axis as shown below, using  `abs_sobel_thresh()` function.
![alt text][image4]
![alt text][image5]
Gradient Magnitude and Direction threshold were also implemented using `mag_threshold()` and `dir_threshold()` functions, as shown below. 
![alt text][image6]
![alt text][image7]
Combining these gradient thresholding techniques, the output is shown below.
![alt text][image8]
To detect yellow lane I used color filters, I have used two color spaces, HSL & Lab, to filter out yellow lane with accuracy. I used Saturation threshold from HSL colorspace using function `hls_select()` (I tried with both Saturation and Lightness thresholding but it was better with just Saturation threshold) and B threshold from Lab colorspace using function `lab_bthresh()`, which works pretty well with blue-yellow color range. both thresholding outputs are shown below.
![alt text][image9]
![alt text][image10]
![alt text][image11]
Combining both the gradient threshold and color filtered images, the final output is shown below along with stacked image of both gradient threshold and color filters seperately, `blue - Color Filter` & `green - Gradient Threshold`
![alt text][image12]
![alt text][image13]

The code for all these images are provided in `Cell 8` (need to uncomment) of the Jupyter Notebook

#### 3. Perspective Transform

The code for my perspective transform includes a function called `warping()`, which appears in `lines 13 - 24` in `Cell 9` of jupyter notebook.  The `warping()` function takes as inputs an image (`img`) and outputs warped image.  I chose the hardcode source and destination points in the following manner:

```python
src = np.float32(np.array([[200,720],[600,450],[700,450],[1100,720]]))
dest = np.float32(np.array([[350,720],[350,0],[950,0],[950,720]]))
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200,720      | 350, 720        | 
| 600,450      | 350, 0      |
| 700,450     | 950, 0      |
| 1100,720      | 950, 7200        |

I used mask on both sides to eliminate other lane or edge detection with hardcoded points within `warping()`

```python
pt1 = np.float32(np.array([[0,720],[0,0],[200,0],[200,720]]))
pt2 = np.float32(np.array([[1100,720],[1100,0],[1280,0],[1280,720]]))
```

The final warped image is shown below. the code for this is commented at the bottom of `Cell 9`.
![alt text][image14]


#### 4. Lane Line Fitting

The lane finding algorithm ( `Cell 10` ) is based on the 'window search' methodology shown in the Udacity lectures. The starting point of the search is identified using a histogram plot, defined in `DrawHisto()` function, of the non-zero data points in the binary image. Two peaks, one in left half and right half each, represent the centre of left and right lane roots respectively. Starting here, a window search is done going from bottom to top in the image, tracking co-ordinates of the pixels and storing them seperately to perform a second order polynomial fit to find the desired lane lines, this is done in the `DrawSlidingWindow()` function. The images below shows the histogram and the final output of the function `Visualization()`, which shows the lane lines along with the left lane pixels (red) and right lane pixels (blue).

![alt text][image15] ![alt text][image16]

#### 5. Radius of Curvature and Vehicle Offset.
##### Radius of Curvature
`Cell 11` in the jupyter notebook shows the radius of curvature and vehicle offset calculation. Lane width in pixels was measured in the warped image and then it was equated to pixel count to the lane width in meters, which is assumed as a standard `3.7 m` here. Similarly, the length of the road patch used for perspective transform is assumed to be `30 m` and the corresponding pixel count of `600 (width of warped destination image)` is used to calibrate the pixel/m conversion. The standard radius of curvature expression for a second degree polynomial as shown in the Udacity lectures is used to find the radius. For video implementation, the curvature of the lane with more data points is used as the lane curvature as that lane line is more 'dominating'.

##### Vehicle Offset
The vehicle position with respect to the lane center is calculated. The idea is to use the image center (assuming the camera is mounted at the center) and the mid point of left and right lanes and use these points to identify whether the vehicle is right or left of the lane center.

both these calculations are carried out in the `FindROC()` function in `Cell 11`.

#### 6. Final Panel Output

I have merged four images to get a better understanding and visualization of each important step of the pipeline. I have stacked four images, `top left - input image`, `bottom left - warped with lanel lines`, `bottom right - unwarped lane area and lane pixels`, `top right - final output with lane area, lane pixels, radius of curvature and vehicle offset information`. `DisplayPanel()` is the function which carries out this task, it's in `Cell 12`. Output of the "test5.jpg" is shown below.

![alt text][image17]
---

### Pipeline (video)

Cell 16-17 are the one where the whole pipeline is defined along with some extra logic to reduce wobbly lines. It checks for the dominant line based on histogram plot and based on that information it decides the current dominating lane (left/right) and assigns the radius of curvature to that line's radius of curvature. Along with this a quality check of lane lines was incorporated to eliminate lines which are outside of standard deviation window using `meanLaneWidth()` function. In that case of elimination, it would consider the previous frame value and move forward.

Here's a [link to my video result](https://youtu.be/Zw9oFyG_rzs)

---

### Discussion

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
