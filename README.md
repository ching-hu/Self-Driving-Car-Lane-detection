# **Finding Lane Lines on the Road** 

## Ching Hu

### Lane detection using various OpenCV algorithms
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Convert from RGB to YUV image space
* Histogram equalization to maximize contrast
* Convert from YUV back to RGB
* Convert to grayscale
* Apply Gaussian blur 
* Apply adaptive threshold
* Create a mask for region of interest (triangle)
* Draw Hough lines based on masked threshold image
* Usge average of slope intercept of multiple Hough line to create virtual guiding lane 
* For video processing or series of images, averaging of multiple virtual guiding lane
* Overlay virtual guiding lane with original image using alpha channel blend
* Write output images and videos to output directory


---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 12 steps as listed above.  
The first five steps is preprocess image for most robust adaptive threshold:
* Convert from RGB to YUV image space
* Histogram equalization to maximize contrast: 
		side effect of this step is introducing quite a bit of noise
		But the benefit of this step is to handle very low contrast images such as a fog condition
* Convert from YUV back to RGB
* Convert to grayscale
* Apply Gaussian blur 
		Blurring is just an attempt to remove random noise, the actual kernal size is emperically determined

Once the image is ready for adaptive threshold, the OpenCV adaptive threshold was used. 
The Canny threshold was also tested, but the overall test result comparison was more favorable for adaptive threshold.

Next, a triagular mask region was created.  The triangle size is emperically determined based on the field of view,
which should be fairly consistent per car per camera.  

The most time consuming part of this project is to actually draw the virtual lines overlaying the actual lane markings.
The simplist approach is to start with a slope-intercept line equation y = mx + b, where m is the slope and b is the
intercept.  To determine if the line is on the left or right, the sign of the slope is used.  Negative slope is 
associated with the right lane, and positive slope is associated with the left lane.  To further filter down noise,
the slope angles are constrained based on emperical testing.  The array of left and right lane found by Hough line is 
then averaged respective to form a true line that draws through majority of the lines, with the longer lines have
more weights.  For video, the virtual lines will jump rapidly as I found out, and I had to device a global variable 
to remember the previous left and right lane set to average with current lane set to smooth out the jitter.

The rest of the steps is to follow through the mechanics of overlaying the virtual line with the original image and 
then write the output image or video to file.


### 2. Identify potential shortcomings with your current pipeline

The static definition of the triangular region may become an issue during lane change.  The lane markings for a few
frames of the video will be outside of the triangular mask region.  The current algorithms does not cover this case.

The other cases are when a large vehicle is fairly close, i.e. most of the lane markings may not be visible, especially 
during a turn.

The virtual line calculation is definitely not optimal, this was very clearly seen in the challenge video, where the 
virtual lines jumped eradiclly over a white section of the road.  I believe this has to do with the threshold used
here is not configured specifically for this scene.  This goes back to the lack of robustness in my pipeline, which
definitely requires more improvement.

I modified some of the original images to test how my pipeline would handle them, the object in the middle of the road
was the hardest one to handle.  The simmulated fog condition was easily handled by the histogram equalization.  However,
this may not be realistic enough.  Fog does not have linear density, and may blend in more with the white lane markings.

There are many other situations where the current lane detection pipeline would fail.  More analytics is required to 
perform adaptive lane detection.


### 3. Suggest possible improvements to your pipeline

As a coding principle, any hard coded numbers is not desired.  Ideally, I would like to have adaptive look up table or
even a real-time adaptive set to best find the lanes.  This is where neural net may possibly be of a major help.  However
neural net outputs also needs to be double or triple checked agains outputs of atleast two other models to be trusted.

I'm not as familiar with OpenCV and definitely new to python and the entire development environment, so I spend quite a 
bit of time setting them up and learning while doing this project.  Much of the pipeline steps can be easily experimented
in photoshop.  The development environment definitely is limited for debugging, and I really wished there were intelli-sense
and real-time debugging with break points, step, monitor, etc.  Yes, I'm spoiled by visual studio IDE.