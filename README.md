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
1. Briefly describe how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image

The code for this step is contained in the first two code cells in the iPython Jupyter codebok  
