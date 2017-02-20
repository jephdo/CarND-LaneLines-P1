#**Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./output_images/debug_steps.png "Debug Steps"

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:

1. Color selection

2. Canny edge detection

3. Region of interest masking using a triangle shape

4. Hough Lines

5. Line extrapolation by clustering lines and averaging their slopes.

![Steps from my pipeline][image2]

First, I used color selection to filter for yellow and white objects. For white objects 
I used very similar code to the lecture i.e. setting an RGB threshold to (200, 200, 200).
However, to extract yellow from the image, I converted my image from RGB space to HSV space. This
made it easier to define a hue range for yellow.

Secondly, I applied Canny edge detection on the color selected image from Step 1 using
a lower threshold of 50 and an upper threshold of 150.

Then I applied a region of interest mask using a triangle shape and the hough
lines function.

The fifth and last step to modify the `draw_lines` function was the most involved
step. I took the list of lines I got from the Hough transform and grouped the
lines by similar slope. Then for each group (cluster), I took the average slope
weighted by each line segments length. The Hough transform returns an array of line segments, 
some longer than others. Longer contiguous lines give us more confidence that it is actually a lane line
so a weighted average instead of a simple average will be more robust to lines
detected from noise.

Then I found the  "average" midpoint, I found the midpoint of each line segment and took
a simple average of their X and Y coordinates. Given this "average" point and 
average slope I extended the lines across a Y range of [300,600] and drew that 
onto the image. 

I repeat this step for each group of lines on two conditions: (1) the average
slope of the cluster needs to be between a certain range and (2)  the total length
of the line segments must exceed some threshold.

For condition (1): some of the lines returned by the Hough transform are horizontal
lines. Since we know the lane lines should be sloped in a certain direction we can
add an extra condition that the slope should be somewhere between 0.5 to 1.

For condition (2): each cluster of lines has however many line segments. If this
cluster represents a true lane line, then most likely we would've detected some minimum number of
line segments (edges from Canny edge detection). So I added the condition that
the sum of the lengths of the line segments must exceed about 90 pixels (guessed 
from trial and error).


###2. Identify potential shortcomings with your current pipeline

One of the biggest shortcomings is that the pipeline is dependent on the viewport
of the camera. For example, the region of interest masking of the pipeline (Step 3)
is a hardcoded triangle. That means the image camera has to be a particular dimension
and the camera has to be angled exactly. This is not particularly robust. There
needs to be a better way to find the horizon of the road.

Secondly, the pipeline expects the lane lines to be perfectly straight. Early 
in the process I ran into problems with solidYellowLeft.jpg because the left
yellow lane in the horizon is actually slightly curved. So I had to tolerate that
by relaxing some conditions in my pipeline that grouped lines together. In real road
conditions, it's not just turns that are curved, some lane lines will look
curve because of distortions in the distance.

###3. Suggest possible improvements to your pipeline

One possible improvement is to use a polynomial regression to extrapolate 
lane lines. Right now I extrapolate the lines by taking the line segments from the 
Hough transform and "averaging" them. Instead, however, I could use these line
segments (and points) to fit a polynomial regression that could deal with 
curved lane lines.

Another potential improvement would be to make color selection more robust.
In the challenge video, I lose the lane lines on a patch of road where the 
contrast of the lines with the pavement is very low. I know the correct
hue values for yellow and white under HSV spaces, but I would need to do more
trial and error to find good ranges for saturation and value.
