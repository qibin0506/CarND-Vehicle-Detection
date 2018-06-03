## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/CarNonCar.png
[image2]: ./output_images/CarHog.png
[image8]: ./output_images/NonCarHog.png
[image3]: ./output_images/multi_view_sliding_windows.png
[image4]: ./output_images/findCarHeat.png
[image5]: ./output_images/findCarHeat2.png
[image6]: ./output_images/findCarHeat3.png
[image7]: ./output_images/findCarHeat4.png
[image9]: ./output_images/findCarHeat5.png
[image10]: ./output_images/findCarHeat6.png


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 4th code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images from the dataset provided within the class.  
Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`color_space`, `hog_channel`, `orientations`, `pixels_per_cell`, `cells_per_block` and `hist_bins`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `hog_channel='ALL'`,`orientations=8`, `pixels_per_cell=(8, 8)`,`cells_per_block=(2, 2)` and `hist_bins=32`:

![alt text][image2]
![alt text][image8]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters especially with the color space `YCrCb` and `YUV` .I also experimented with the `hog_channel=0`,`hog_channel=1` and `hog_channel='ALL'`.
Main goal was to maximize the model accuracy and achieve 99% test accuracy. At the same time reduce the fitting time (~3s). The confusion matrix proved to be helpful as well.

true negative (TN), false positive (FP)
false negative (FN), true positive (TP)

[[7002 1966]
 [ 855 7937]]

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM while extracting HOG features from all color channels by using a combination of parameters which gave the best results.
Model accuracy, fitting time and confusion matrix were the indicators I used to determine the final choice. 
After extracting the features I created an array stack of feature vectors and defined the labels vector. 
Afterwards I split the data into randomized training and test sets (test size 20%).
Next step I fit a per-column scaler only on the training data while applying the scaler to both the training and the test sets.
The code for this step is contained starting with the 4th up to the 8th code cells of the IPython notebook.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

For this step I used a slide window which takes an image, start and stop positions in both x and y, window size (x and y dimensions), and overlap fraction (for both x and y). 
I used 0.5 overlap and no defined positions on the x dimension (didn't mind having car detections on the oncoming traffic as well). 
Instead I defined y start and stop by using a multi-scale windows search accounting how far or how close the car will be within the field of view.
The code for this step is contained in the 11th and 13th code cells of the IPython notebook.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

The pipeline will take each frame, extract features using hog sub-sampling of YCrCb color space plus spatially binned color and histograms of color in the feature vector.
I extracted the image patch, the color and scale features and made a prediction using the linear svc classifier.
The step was repeated for a combination of hard coded windows size scale and start/stop positions in y, in order to account how far/close the car is within the field of view.
The code for this step is contained in the 13th code cells of the IPython notebook.
 
Here's a [link to my pipeline video result](./output_images/test_video_out_final.mp4)

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output_images/project_video_out_final.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video and I added a heat map to each box in the box list, applied threshold to remove false positives and searched for boxes to identify individual blobs in the heatmap.
I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. 
All detection were added into a history class with a buffer of 3 frames.

Here's an example result showing the heatmap from a series of frames of video:

![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]
![alt text][image9]
![alt text][image10]

--

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most challenging part was to find the best HOG parameters. However, using the model accuracy, the fitting time and the confusion matrix as indicators made things easier.
Also hard-coding an accurate combination for the window size and the start/stop positions in y, required a series of measurements while saving key video frames.
There could be many improvements added on the detection side and a better approach for removing false positives if a Kalman filter is used instead.
