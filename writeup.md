## Writeup

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

[02_Undistort_Camera_CalibImg]: ./output_images/02_Undistort_Camera_CalibImg.png "02_Undistort_Camera_CalibImg"
[02_Undistort_Driving_Curved]: ./output_images/02_Undistort_Driving_Curved.png "02_Undistort_Driving_Curved"
[02_Undistort_Driving_Straight]: ./output_images/02_Undistort_Driving_Straight.png "02_Undistort_Driving_Straight"
[03_GradientThreshold_Curved_Bridge]: ./output_images/03_GradientThreshold_Curved_Bridge.png "03_GradientThreshold_Curved_Bridge"
[03_GradientThreshold_Curved_Shadow]: ./output_images/03_GradientThreshold_Curved_Shadow.png "03_GradientThreshold_Curved_Shadow"
[03_GradientThreshold_Straight]: ./output_images/03_GradientThreshold_Straight.png "03_GradientThreshold_Straight"
[04_PerspectiveTransform_Curved]: ./output_images/04_PerspectiveTransform_Curved.png "04_PerspectiveTransform_Curved"
[04_PerspectiveTransform_Straight]: ./output_images/04_PerspectiveTransform_Straight.png "04_PerspectiveTransform_Straight"
[05_06_LanePolynomialFit_Curved]: ./output_images/05_06_LanePolynomialFit_Curved.png "05_06_LanePolynomialFit_Curved"
[05_06_LanePolynomialFit_Straight]: ./output_images/05_06_LanePolynomialFit_Straight.png "05_06_LanePolynomialFit_Straight"
[07_08_LaneOverlayOnImage_Curved]: ./output_images/07_08_LaneOverlayOnImage_Curved.png "07_08_LaneOverlayOnImage_Curved"
[07_08_LaneOverlayOnImage_Straight]: ./output_images/07_08_LaneOverlayOnImage_Straight.png "07_08_LaneOverlayOnImage_Straight"
[09_Final_Radius_Position_Overlay_Curved]: ./output_images/09_Final_Radius_Position_Overlay_Curved.png "09_Final_Radius_Position_Overlay_Curved"
[09_Final_Radius_Position_Overlay_Straight]: ./output_images/09_Final_Radius_Position_Overlay_Straight.png  "09_Final_Radius_Position_Overlay_Straight"
[output_video]: ./project_video_output.mp4 "output_video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd code cell of the IPython notebook located in "./P4_AdvancedLaneLines.ipynb", with the comment block:

```python
########################################################
# 1. Compute the camera calibration matrix
#    and distortion coefficients given a set of chessboard images.
########################################################
```

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function, as in the beginning of 4th code cell, with the comment:


```python
###############################################################
# 2. Apply a distortion correction to raw images.
########################################################
```

and obtained this result: 
![02_Undistort_Camera_CalibImg][02_Undistort_Camera_CalibImg]
It is very obvious by looking at the bottom-right chess squares that there was some undistortion.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to two of the test images like these 2:

Straight Lanes: It is a bit hard to tell if there was any distortion in this image, because it seems already undistorted, but the next image is conducive to undistortion.
![02_Undistort_Driving_Straight][02_Undistort_Driving_Straight]

Curved Lanes: It is quite obvious from the 'Deer Crossing' sign that there was some undistortion.
![02_Undistort_Driving_Straight][02_Undistort_Driving_Curved]

The distortion correction was applied simply by calling `cv2.undistort()` function, with the Matrix and the distortion coefficients that were obtained in the previous step, using the Camera Calibration images.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image - thresholding steps in the 5th code cell, starting with

```python
###############################################################`
# 3. Use color transforms, gradients, etc.,
#    to create a thresholded binary image.
########################################################.  
```

I actually experimented with the thresholds, and tried 3 different combinations, as follows, and picked the one that gave me the best result. "Best result" was one which was still giving me proper lane-markings despite the lighter bridge, or a darker shadow in the image.
The 3 combinations of transforms, gradients, and thresholding I used were:

1. Gradient Thresholds Only: This used a (gradient in 'x', ANDed with a gradient in 'y'), which were ORed with a (gradient magnitude thresholded ANDed with a gradient direction thresholded)
2. Combination of two thresholds, a Gradient in 'x' ORed with an S-channel (color transformation) thresholded
3. Finally, this one combined the Gradient Threshold in "1." ORed with S-channel thresholding.

Since #3. was the most complex, I was hoping it would give me the best results. However, after trying with all the test-images, #2. gave me the best results. So I went with #2.: Combination of two thresholds, a Gradient in 'x' ORed with an S-channel (color transformation) thresholded.

Here are the outputs Straight and Curved Driving images conveted to thresholded binary images.

Just straight: Both #2. and #3. work great
![03_GradientThreshold_Straight][03_GradientThreshold_Straight]

Curved with bridge: Here #1. totally loses the lane on the bridge. However, both #2. and #3. work OK on the bridge, but 2. is slightly better.
![03_GradientThreshold_Curved_Bridge][03_GradientThreshold_Curved_Bridge]

Curved with bridge, and tree shadow: Again, both #2. and #3. work OK, but #2 is better.
![03_GradientThreshold_Curved_Shadow][03_GradientThreshold_Curved_Shadow]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `binary_transform_top_down()`, which appears in the 6th code cell of the IPython notebook, starting with the comment:

```python
########################################################
# 4. Apply a perspective transform 
#    to rectify binary image ("birds-eye view").
########################################################
```

The `binary_transform_top_down()` function takes as an image (`img`), and then it first gets the #2. gradient tranform, and then calls `cv2. getPerspectiveTransform()`. I chose the hardcoded the source and destination points in the following code:

```python
	# Source points
    topleft = (595,450)
    topright = (687,450)
    bottomleft = (200,720)
    bottomright = (1100,720)
    src = np.float32([topleft,topright,bottomleft,bottomright])
    
    # Destination points
    dst = np.float32([[200,0],[1100,0],[200,720],[1100,720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 450      | 200, 0        | 
| 687, 450      | 1100, 0      |
| 200, 720     | 200, 720      |
| 1100, 720      | 1100, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
Here are 2 examples, of straight, and curved driving:

![04_PerspectiveTransform_Straight][04_PerspectiveTransform_Straight]
![04_PerspectiveTransform_Curved][04_PerspectiveTransform_Curved]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The lane-line fitting is done in the 7th code cell, starting with:

```python
############################################################### 
# 5. Detect lane pixels and fit to find the lane boundary.
# 6. Determine the curvature of the lane and vehicle position 
#    with respect to center.
########################################################
```
The code is very similar to as provided in the lectures.
Essentially, it starts by creating a distribution of white pixels in one horizontal line, and continues to the top, in the binary top-down image. It identifies them, and tries to do a 2nd-degree polynomial fit. The results are as follows for both straight and curved lanes.

![05_06_LanePolynomialFit_Straight][05_06_LanePolynomialFit_Straight]
![05_06_LanePolynomialFit_Curved][05_06_LanePolynomialFit_Curved]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This is done in the 10th cell, in the section starting with:

```python
########################################################
# 9. Output numerical estimation of lane curvature 
#    and vehicle position.
########################################################
```

This is in function: `get_curve_m_and_position()`

Again, this is also code very similar from the lectures, where it just tries to obtain the radius based on the polynomial fit - after converting it to meters-space, from pixel space.
For the curvature, I average the left-lane and right-lane curvature found in meters.
For obtaining the vehicle position, I find its difference from the center of the image to the center of the lanes. 
To get the center of the lanes, I just evaluate the polynomial fit at the lowest y point for each lane: left and right.

Here are the examples of the values displayed for straight-lane and curved-lane test images
![09_Final_Radius_Position_Overlay_Straight][09_Final_Radius_Position_Overlay_Straight]
![09_Final_Radius_Position_Overlay_Curved][09_Final_Radius_Position_Overlay_Curved]

For the staight-lane, my curvature is very high: 159km, which means the polynomial fitting has detected it as almost stright.
My curvature here is ~700m, which is very close to the 1000m (1km) value given in the project outline, which tells me the lane-finding pipeline is good, but not great.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I actually did this step before finding the 'radius of curvature' and 'vehicle position', which is why the images in the previous section already contain the found-lanes already overlayed into the test images.

In any case, I did this in the 8th code-cell, starting with:

```python
########################################################
# 7. Warp the detected lane boundaries back onto the original image.
# 8. Output visual display of the lane boundaries 
########################################################
```

This is quite simple, as it is just a reversal of the warping we did before to get the top-down image.
In addition, for this section, I created a special mask, which is the negative of the trapezoidal mask, as visible in the images below. I then overlay that onto the test-images.
From top-left to bottom right:

* Top-left: Original undistorted image
* Top-right: Unwarped image (reverse of warping done before)
* Bottom-left: White mask onto the trapezoidal section of the lane-markings detected in 'Top-right' image
* Bottom-right: Original undistorted image overlayed with the mask created in 'Bottom-left' image

Straight-lanes:
![07_08_LaneOverlayOnImage_Straight][07_08_LaneOverlayOnImage_Straight]

Curved-lanes:
![07_08_LaneOverlayOnImage_Curved][07_08_LaneOverlayOnImage_Curved]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [output_video](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced 2 major issues:
1. Getting smaller radii of curvature in the final step. To correct this, I realized that I had not done the undistortion. After undistortion, I started getting values close to 1km on the curved-lane test-images.
2. Deciding on a good thresholds to generate the binary thresholded images. I started with the values provided in the lectures, and changed them slightly. In addition, I had to try 3 combinations to determine which one to go with. This was slightly hard open-ended problem.

Finally, I still think my binary-thresholded image could use more improvement. I wonder how it would work in the morning and evening conditions, and what color transforms may be required, because both dawn and dusk have more red in the light than white. 
So the pipeline could be setup where it uses multiple transforms based on some quantified measure, as opposed to just visual inspection.
