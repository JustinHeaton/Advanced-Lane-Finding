# Advanced Lane Finding

## Udacity Self Driving Car Engineer Nanodegree - Project 4

![Final Result Gif] (https://github.com/JustinHeaton/Advanced-Lane-Finding/blob/master/images/project_vid.gif)

The goal of this project is to develop a pipeline to process a video stream from a forward-facing camera mounted on the front of a car, and output an annotated video which identifies:
- The positions of the lane lines 
- The location of the vehicle relative to the center of the lane
- The radius of curvature of the road

The pipeline created for this project processes images in the following steps:
- Step 1: Apply distortion correction using a calculated camera calibration matrix and distortion coefficients.
- Step 2: Apply a perspective transformation to warp the image to a birds eye view perspective of the lane lines.
- Step 3: Apply color thresholds to create a binary image which isolates the pixels representing lane lines.
- Step 4: Identify the lane line pixels and fit polynomials to the lane boundaries.
- Step 5: Determine curvature of the lane and vehicle position with respect to center.
- Step 6: Warp the detected lane boundaries back onto the original image.
- Step 7: Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Code:
This project requires python 3.5 and the following dependencies:
- [NumPy] (http://www.numpy.org/)
- [Pandas] (http://pandas.pydata.org/)
- [matplotlib] (http://matplotlib.org/)
- [OpenCV] (http://opencv.org/)
- [MoviePy] (http://zulko.github.io/moviepy/)

### Step 1: Distortion Correction
In this step, I used the OpenCV functions `findChessboardCorners` and `drawChessboardCorners` to identify the locations of corners on a series of pictures of a chessboard taken from different angles.

Next, the locations of the chessboard corners were used as input to the OpenCV function `calibrateCamera` to compute the camera calibration matrix and distortion coefficients. 

![Corners Image] (https://github.com/JustinHeaton/Advanced-Lane-Finding/blob/master/images/corners.png)

Finally, the camera calibration matrix and distortion coefficients were used with the OpenCV function `undistort` to remove distortion from highway driving images.

![Undistorted Image] (https://github.com/JustinHeaton/Advanced-Lane-Finding/blob/master/images/undistorted.png)

Notice that if you compare the two images, especially around the edges, there are definitely differences between the original and undistorted image.

### Step 2: Perspective Transform
The goal of this step is to transform the undistorted image to a "birds eye view" of the road which focuses only on the lane lines and displays them in such a way that they appear to be relatively parallel to eachother (as opposed to the converging lines you would normally see). To achieve the perspective transformation I first applied the OpenCV functions `getPerspectiveTransform` and `warpPerspective` which take a matrix of four source points on the undistorted image and remaps them to four destination points on the warped image. The source and destination points were selected manually by visualizing the locations of the lane lines on a series of images.

![Birds Eye Image] (https://github.com/JustinHeaton/Advanced-Lane-Finding/blob/master/images/warped.png)

### Step 3: Apply Binary Thresholds
In this step I attempted to convert the warped image to different color spaces on create binary thresholded images which highlight only the lane lines and ignore everything else. 
I found that the following color channels and thresholds did a good job of identifying the lane lines in the provided test images:
- The S Channel from the HLS color space, with a min threshold of 180 and a max threshold of 255, did a fairly good job of identifying both the white and yellow lane lines, but did not pick up 100% of the pixels in either one.
- The U Channel from the YUV color space, with a min threshold of 110 and a max threshold of 255, did an almost perfect job of picking up the yellow lane lines, but completely ignored the white lines.
- The Blue channel from the RGB color space, with a min threshold of 220 and an upper threshold of 255, did a better job than the S channel in identifying the white lines, but completely ignored the yellow lines. 

I chose to create a combined binary threshold based on the three above mentioned binary thresholds, to create one thresholded images which does a great job of highlighting almost all of the white and yellow lane lines.

![Binary Thresholds] (https://github.com/JustinHeaton/Advanced-Lane-Finding/blob/master/images/thresholds.png)

### Steps 4, 5 and 6: Fitting a polynomial to the lane lines, calculating vehicle position and radius of curvature:
At this point I was able to use the combined binary image to isolate only the pixels belonging to lane lines. The next step was to fit a polynomial to each lane line, which was done by:
- Identifying all non zero pixels in the image using the numpy function `numpy.nonzero()`.
- Fitting a polynomial to each lane using the numpy function `numpy.polyfit()`.

After fitting the polynomials I was able to calculate the position of the vehicle with respect to center with the following calculations:
- Calculated the average of the x intercepts from each of the two polynomials `position = (rightx_int+leftx_int)/2`
- Calculated the distance from center by taking the absolute value of the vehicle position minus the halfway point along the horizontal axis `distance_from_center = abs(image_width/2 - position)`
- If the horizontal position of the car was greater than `(image_width/2` than the car was considered to be left of center, otherwise right of center.
- Finally, the distance from center was converted from pixels to meters by multiplying the number of pixels by `3.7/700`.

Next I used the following code to calculate the radius of curvature for each lane line in meters:
```
ym_per_pix = 30./720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meteres per pixel in x dimension
left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
left_curverad = ((1 + (2*left_fit_cr[0]*np.max(lefty) + left_fit_cr[1])**2)**1.5) \
                             /np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*np.max(lefty) + right_fit_cr[1])**2)**1.5) \
                                /np.absolute(2*right_fit_cr[0])
```
The final radius of curvature was taken by average the left and right curve radiuses.

### Step 7: Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
The final step in processing the images was to plot the polynomial back on to the warped image, fill the space between the polynomials to highlight the lane that the car is in, use another perspective trasformation to unwarp the image from birds eye back to its original perspective, and print the distance from center and radius of curvature on to the final annotated image.

![Filled Image] (https://github.com/JustinHeaton/Advanced-Lane-Finding/blob/master/images/filled.png)
