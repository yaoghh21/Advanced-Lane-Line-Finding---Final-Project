# Advanced-Lane-Line-Finding---Final-Project
The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
* Implement video processing pipeline


* Note: Make sure you use the correct grayscale conversion depending on how you've read in your images. Use cv2.COLOR_RGB2GRAY if you've read in an image using mpimg.imread(). Use cv2.COLOR_BGR2GRAY if you've read in an image using cv2.imread().'

# Rubric points
Here I will consider the Rubric points individually and describe how I addressed each point in my implementation

# Camera Calibration
**1. Briefly describe how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image**

The code for this step is contained in the first two code cells in the iPython Jupyter codebook "Final_Project - Advanced Computer Vision.ipynb".

Using OpenCV functions "findChessboardCorners", "calibrateCamera" and "undistort", I achieved a good camera calibration and correection for distortion. Images of chessboards are provided, assuming different angles from a single camera point of view. I started by mapping "object points" using the function "findChessboardCorners", defined as the 3D real world coordinates of the corners in every image of the chessboards, then will transform "object points" to 2D "image points" assuming z = 0. Both object points and image points are appended in two different arrays and then passed to the "calibrateCamera" function to compute the camera calibration and distortion coefficients, which are passed to the "undistort" function to apply the correction to the image with the following result: 

![](Images/Undistorted%20Checkerboard.png)

# Pipeline

**1. Provide an example of a distortion-corrected images**

The attached image below is an example of the result of the distortion correction image by applying the OpenCV function "undistort" to one of the test images provided in the course. 

![](Images/Example1.png)

The distortion correction can be noticed from the difference of the white vehicle baiased to the right and also by difference in the shape of the hood. 

**2.Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a threshold binary image. Provide an example of a binary image result.**

To generate the binary image I started converting the undistorted image to HLS color space and separted the color channels to l_channel and s_channel. All these steps are labeled clearly in the Jupyter Notebooks file. 

![](Images/HLS.png)

I've decided to use the combination of S and L channels because they do a fairly robust job of picking up the white and yellow lines under very different color and contrast conditions, while other combinations that I also explored looked messy. Ultimately I decided to do a combination of gradient threshold and color threshold to combine them and achieve the final binary image. 

![](Images/ColorGradientThreshold.png)

Here is the result of the binary image after combining the two binary thresholds, which I also applied some smoothing. 

![](Images/BinaryImage1.png)
              
**3.Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.**

There is a function in my code called "Filter_Region" which create a mask using the vertices from the function "Select_Region" and apply them to the input image. To map the points in the input image to the different desired image points with the new perspective (bird's eye view in our case), I decided to manually select the region of interest by selecting 4 points that define a trapezoid in the input image to obtain the following image.   

![](Images/4points1.png)

![](Images/RegionOfInteres1.png)

Once the region of interest is defined I use the "warp" function clearly labeled in my Jupyter notebooks, to indicate 4 source points from our "region of interest" image and where we want those 4 points to appear and finally are transformed to the warped image.

Note: It is important to notice that at this point we also calculate the inverse perspective transform value "Minv", which will be utilized later on to unwarp the image. 

![](Images/4src4dst.png)

![](Images/WarpedImage.png)

**4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?**

In order to identify the lane line pixels and fit the positions with polynomial I used the functions "find_lane_pixels" which is the sliding windows method and "search_around_poly" which is the search from prior method and both are labeled in my Jupyter Notebook file. 

The first function "find_lane_pixels" computes the histogram of the half bottom of the image, then computes the max value for eah lane and use it as a starting point to draw the first of the windows (defined in the hyperparameters) which in the case of my function will draw 9 windows. Then the function identifies all the activated pixels for each line to use them to recenter every window and follow the line. Lastly it will extract the x and y pixels positions and it a second order polynomial to each using "np.polyfit" to then draw it also in the image. 

![](Images/Histogram.png)        ![](Images/SlidingWindows1.png)


Then the second function "find_lane_pixels" creates a margin around the previous polynomial to search the lane pixels of the new frame. This can be done because the lane lines don't change too much between frames. 


![](Images/Screen%20Shot%202019-06-09%20at%201.16.37%20PM.png)

**5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.**

The radius of the curvature and distance to the center are caluclated with the function "curv_rad_and_center_dist" clearly labeled in my Jupyter Notebook. I did it by using the following lines of code: 

**left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])**


In the aquations above for the left curve, **left_fit_cr[0]** is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and **left_fit_cr[1]** is the second coefficient, **y_eval** is the y position within the image upon which the curvature calculation is based (the bottom most y position in the image). Then **ym_per_pix** is the factor used for converting from pixels to meters. This convestion was also used to generate a new fit with coefficients in terms of meters. The same principle applies for right curve.  

After the radius of the curves are calculated I proceed to calculate the position of the vehicle with respect to the center with the following equation: 

![](Images/Center.png)   

In the equation above "r_fit_x_int" and "l_fit_x_int" are the x-intercepts of the right and left fits. The lane center position is the sum of the intercepts divided by 2. The Center distance is calculated by subtracting "car position" minus the "lane center position" times the meters per pixel to have the units in meters. 

**6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.**

For this step I used the functions "warp_b" to warp the detected lane boundaries back onto the original image, the function can be found clearly labeled and decribed step by step in my Jupyter Notebooks file. Note: Here is where "Minv" coefficient is utilized. 


![](Images/Warpb.png)

I finally use the function "visual display" to output a visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position 

![](Images/VisualOutput.png)

# Pipeline(video)

**1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).**

Here is a link to my video: https://youtu.be/jbsBxRbOjLM

# Discussion

Here I'll talk about the approach I took and the main issues I went through to achieve the result posted in the link above.
I didn't do anything different from the content in the course, actually I followed every step/goal as depicted in the project rubric/template. The most challenging part for me was to understand how to use the properties of the class lane to determine when to use the sliding windows method or the search around method. The pipeline has opportunities for improvements to determine the source points and the destination points when warping the image.
I was fascinated about the sliding windows method and how it all starts with the histogram peaks, but the hardest part was to really make sense of the code when using polyfit to fit a polynomial with the activated pixels for each line and then how to iterate going up to the lane sliding the widnows defined in the hypeparameters.
The other challenging part was to install moviepy in my computer, as the imageo lib had a version issue that took me a long time to resolve. I thought that Anaconda resolved most of the problems of the libraries versioning issues when it comes to libraries but it seems that this one was not the case.
This was a fun project and I learned a lot of computer vision. I can say I am delighted in developing a decent lane line detection algorithm.
Thanks,

