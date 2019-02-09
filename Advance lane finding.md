**Please find output images incuded in advanced lane finding.ipynb 
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


**###Camera Calibration

We start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. This example has 9x6 chessboard corners. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time we successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

We can then use the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function

**###Pipeline

####2. Apply distortion correction to test images I pickled the camera calibration transformation matrix and distortion coefficients mtx and dst and used these in cv2.undistort  to remove distortion in the test images. 

####3. Create a thresholded binary image I used a combination of color and gradient thresholds to generate a binary image 
####4. Perspective transform After the thresholding operation, we perform the perspective transform to change the image to bird's eye view. This is done because we can use this to identify the curvature of the lane and fit a polynomial to it. To perform the perspective transform, I identified 4 src points that form a trapezoid on the image and 4 dst points such that lane lines are parallel to each other after the transformation. The dst points were chosen by trial and error but once chosen works well for all images and the video since the camera is mounted in a fixed position. Finally I used the cv2.getPerspective to identify M and then cv2.warpPerspective to use the M matrix to warp an image
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 800,510       | 650,470       | 
| 1150,700      | 640,700       |
| 270,700       | 960, 720      |
| 510,510       | 270, 510      |


I verified that my perspective transform was working as expected by drawing the src and dst points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. 


####5.Identify lane-line pixels and fit their positions with a polynomial
In order to better estimate where the lane is, we use a histogram of the bottom half of image to identify potential left and right lane markings.I modified this function to narrow down the area in which left and right lanes can exist so that highway lane seperators or any other noise doesn't get identified as a lane. Once the initial left and right lane bottom points are identified, we divide the image in windows, and for each left and right window we find the mean of it, re-centering the window. We then feed the numpy polyfit function to find the best second order polynomial to represent the lanes


####6. Calculate the radius of curvature of the lane and the position of the vehicle with respect to the center The radius of curvature is given by following formula.

Radius of curvature=​​ (1 + (dy/dx)2)1.5 / abs(d2y /dx2)

We will calculate the radius of curvature for left and right lanes at the bottom of image.

x = ay2 + by + c

Taking derivatives, the formula is: radius = (1 + (2a y_eval+b)2)1.5 / abs(2a)

Also we need to convert the radius of curvature from pixels to meter. This is done by using a pixel conversion factor for x and y directions. The calculation of radius of curvature and offset from center is in code cell 14. A sanity check suggested in the lectures was to see if calculated radius of curvature ~ 1km for the track in project video.

Calculation of offset to the center of lane.

We assume the camera is mounted exactly in the center of the car. We first calculate the bottom of left and right lane and hence the center of the lane. The difference between the center of the image (1280 /2 = 640) and the center of the lanes is the offset (in pixels). The calculation was then converted to meters.

####7.Plot result back down onto tho road such that the lane area is identified clearly.

Once we have lane lines identified and polynomial fit to them, we can again use cv2.warpPerspective and the Minv matrix to warp lane lines back onto original image. We also do a weightedadd to show the lane lines on the undistorted image. The code for draw lines function is in code cell 18 . Here is an example of my result on a test image. I have used cv2.putText to display radius of curvature and offset from center on this image.

####8.Sanity Check I also implemented a function to sanity check if the lane lines were being properly identified. This function did the following 3 checks:

Left and right lane lines were identified (By checking if np.polyfit returned values for left and right lanes)
If left and right lanes were identified, there average seperation is in the range 150-430 pixels
If left and right lanes were identified then there are parallel to each other (difference in slope <0.1)
If an image fails any of the above checks, then identified lane lines are discarded and last good left and right lane values are used.

**###Pipeline (video)

###Pipeline (video) After creating all the required functions, I first tested a pipeline that combined all the functions on test images. Once I was satisfied with the results I created a process image pipeline for treating a sequence of images. I added the following checks to this pipeline:

If this is the first image then the lane lines are identified using a histogram search (checked using a counter variable)
Otherwise the previous left and right fit are used to narrow the search window and identify lanes
The left and right fits identified were stored in a variable and if sanity check described above was failed, then the last good values of left and right fit were used

Finally moviepy editor was used to process the project video image by image through the pipeline. The result was pretty good. Lane lines were identified very well through the entire video.


**###Discussion

Two problems that I faced were:

The left lane was getting confused with lane seperator (between going and oncoming traffic) since the histogram was identifying both of them. To solve this I set the condition that histogram identify leftx_base b/w 150 pixels and midpoint of the image. The 150 pixels ensured that lane seperators were discarded. Similarly for the right lane I made sure histogram limited the search from midpoint to mipoint + 500 pixels so any noise was not identified

np.polyfit sometimes resulted in curves that were a very bad fit. This was especially happening for the left lane in the challenge video. To address this, I introduced a condition in sanity check that left and right lane slopes are roughly parallel (delta in slope is less than 0.1). If this condition failed then, last good value of left and right lane fits were used. This was very useful in improving performance on the challenge video.

Here are a few other situations where the pipeline might fail:

*Presence of snow/debris on the road that makes it difficult to clearly see lane lines
*Roads that are wavy or S shaped where a second order polynomial may not be able to properly fit lane lines
*presence of combination of straight road and parallel road in high frequency


To make the code more robust we should try to incorporate the ability to test a wide range of parameters (for thresholding and warping) so that the code can generalize to a wider range of track and weather conditions. To do this we need a very powerful sanity check function that can identify lane lines that are impractical and discard them






