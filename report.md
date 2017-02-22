#**Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)
[image1]: ./report/image1.png "Color Filter"
[image2]: ./report/image2.png "Blur"
[image3]: ./report/image3.png "Canny"
[image4]: ./report/image4.png "ROI"
[image5]: ./report/image5.png "Hough"
[image6]: ./report/image6.png "Cluster"
[image7]: ./report/image7.png "Merge"
---

### Reflection

###1. Pipeline description

####1.1 Pipeline
Starting e.g. with this figure: 
<img src="./test_images/whiteCarLaneSwitch.jpg " height="150" />

The *draw_lane_lines()* was changed to include the following steps:

1. **Color filtering** - For the first action, the image is filtered in such a way that only the interesting colors remain (white and yellow), the rest is blacked out.
This is accomplished with *cv2.inRange()*  for a range close to white color and other for a range close to yellow color. Each *cv2.inRange()* returns a mask that it's bitwise ORed to have a mask including white and yellow and used to mask the original image with *cv2.bitwise_and()*.
The color segmentation is done in HSV (Hue Saturation Value) color space where proximity of color makes more sense (as in human perspective) and segmenting colors is easier. [link](http://docs.opencv.org/3.0-beta/doc/py_tutorials/py_imgproc/py_colorspaces/py_colorspaces.html) 

![][image1]

2. **Blur with Gaussian** - Gaussian blur to smooth image avoiding some noise in the canny edge detection

![][image2]

3. **Canny Edges** - Detect regions with high gradient. Thresholds were kept as in classes, they seem to do OK.

![][image3]

4. **Region of interest** - Selects the region of interest dynamically depending on image size, e.g. *extra.mp4* has different shaped frames.

![][image4]

5. **Hough** - Bonds together the edge points detected by Canny, outputting line segments. Parameters were also kept as in classes, they look OK for all tested scenarios.

![][image5]

6. **Cluster lane marking** - The cluster lane marking is the peace of code that separates the left and right lanes. They're separated based on their slope, A line segment (output of Hough) can belong to the left lane, to the right lane or to None.
Only lines with slope <-0.5 or >0.5 are considered, the rest are almost horizontal and, are typically outliers.
Negative slope will belong to left lane and positive to right lane.
Several methods to estimate the correct line parameters from all line segments belonging to a lane were considered.
 **Simple average of parameters**, not good, this would only work if there were no outliers, but sometimes the edge detected as a slight different slope for example, averaging over all gives same importance to all line segments which is a bad idea, the parameters will easily drift away from intended value.
 **Least Squares Line fit**, since the output of Hough are points defining the segments, and the segments theoretically fall on the lane line, this method gives the line that best fits all those way points. Even if each segment has slightly wrong slope it will be averaged out when fitting with other segments. 
**IIR Filtering** The dashed lane lines are sometimes not detected in every frame, this would lead to unknown lane line location. Storing the previous parameters of the lane line, and making the reasonable assumption that they do not change abruptly, an IIR filter allows smoothness and robustness to frames where lines are not detected.

![][image6]

7. **Merge pictures** - The end of the pipeline is just composing a picture with the original image overlayed with the estimated lines.

![][image7]

####1.2 How it got there
The process was trial and error, moving from the original idea given in the classes.
- All the Canny and Hough parameters were kept and they look fine.
- Adaptation of ROI, was done with fixed values and then adapted to dynamic due to third video.
- Clustering the lines into right and left lane started by dividing positive and negative slopes but, has there were some horizontal lines detected, segments with low slope were removed.
- Estimation of lane line parameters with polyfit was seen as the more reliable one as it does not depend on individual segment parameters, rather on their end points which is more reliable information.
- IIR filtering was seen as necessary to cope with the frames where no line was detected. It also introduces the advantage of smoothing out the estimation.
- Due to shadows on last video, the grayscale image was not enough to get a good result, further filtering on the interest colors needed to be done. Investigating the color segmentation functions purposed, a nice solution was coded and it seems very nice on the last video.


###2. Identify potential shortcomings with your current pipeline

Potential shortcoming could be:
- The color detection which is now fine tuned for these white and yellow colors.
It is fairly robust but under other light, at night, artificial light,...
- Steeper curves, as the algorithm is prepared to track straight lines only.
- Road without lanes
- Lane change or other vehicle covering the lane
- ...

###3. Suggest possible improvements to your pipeline

- Possibility to track curved lines
- Better filtering, maybe a Kalman that can also have knowledge about more sensors or data
- Dynamically adaptation of interest area, since when facing a steep road the horizon/Vanishing point will be in different place
- Better identify lane color under other light, maybe with some kind of white balance processing

