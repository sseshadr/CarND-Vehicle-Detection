**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, apply a color transform and append binned color features, as well as histograms of color, to the HOG feature vector. 
* Normalize features and randomize a selection for training and testing.
* Implement a sliding-window technique and use trained classifier to search for vehicles in images.
* Run pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/egcarimg.png
[image2]: ./output_images/egnotcarimg.png
[image3]: ./output_images/rgbFeature.png
[image4]: ./output_images/ycbcrFeature.png
[image5]: ./output_images/hsvFeature.png
[image6]: ./output_images/slidingResult.png
[image7]: ./output_images/hogSubsamplingResult.png
[image8]: ./output_images/hogSubsamplingFiltered.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the code cell named as 'Define Features' the IPython notebook [vehDetTrack.ipynb](https://github.com/sseshadr/CarND-Vehicle-Detection/blob/master/vehDetTrack.ipynb).  

I started by reading in all the `vehicle` and `non-vehicle` images. I used glob to loop through all the folders and append the filenames of cars and not cars. Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

![alt text][image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=6`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image3]

Here is an example using the `YCbCr` color space and HOG parameters of `orientations=6`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image4]

Here is an example using the `HSV` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image5]

#### 2. Explain how you settled on your final choice of HOG parameters.

As shown above, I tried the entire algorithm on multiple color spaces first and settled on the HSV. Intuition says this is probably because the car colors are a little more saturated than the background. One of the lectures also mentioned this which validates the intuition. I also watched a project walkthrough that suggested that orientations be set to 9. The reason being that the original HoG paper suggests that the performance of the features tapers off after this number. I also computed features on all 3 channels of the image to get as many features as possible and not omit any valuable information. Finally, I also used color binning features and color histogram features. I used `spatial_size=(32,32)` and `histbins=32`. I increased these values from 16 in the starter code to again have a longer feature vector.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using sklearn's SVM function. This can be found in the 'Train Classifier' cell of  the IPython notebook [vehDetTrack.ipynb](https://github.com/sseshadr/CarND-Vehicle-Detection/blob/master/vehDetTrack.ipynb). 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I tried 2 different approaches here similar to the lectures. 

The first one is a sliding window search where HoG features are separately extracted for every window. This can be found in the 'Perform Sliding Window Search' and 'Test Sliding Window Search' cells of  the IPython notebook [vehDetTrack.ipynb](https://github.com/sseshadr/CarND-Vehicle-Detection/blob/master/vehDetTrack.ipynb). The result of this technique on the provided 6 test images are shown below.

![alt text][image6]

The second one is based on HoG subsampling. Here we extract the HoG features of the entire image once and then index into every window to get their HoG features. This is to speed up the code execution time.  This can be found in the 'HoG Subsampling' cell of  the IPython notebook [vehDetTrack.ipynb](https://github.com/sseshadr/CarND-Vehicle-Detection/blob/master/vehDetTrack.ipynb). The result of this technique on the last test image is shown below.

![alt text][image7]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

As explained above in respective sections, ultimately I searched using HSV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector.  Example images are already listed above. I first tried the straight sliding window search which was slow. Subsampling improved the performance. I also used a scale factor of 1.5 for the same reason.

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./test_videos_output/project_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  This threshold value was obtained empirically to have a reasonable balance between number of false positives and lack of actual detections (false negatives). I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. Example frame, raw detections, corresponding heat map and the filtered bounding box based on the thresholded heat map is shown below.

![alt text][image8]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I see 3 distinct scenarios where the algorithm is lacking:

* Tracking mid to far white car: Seems like HSV is unable to reliably identify features for a white car that is some distance from the ego car. Perhaps there are untested colorspaces which might be better.
* Detections near shadows: There are a lot of false detections under shade of the tree. Need to investigate which feature (color, histogram or HoG) is contributing to this and see if some intuition can be developed.
* General false detections: There are a couple of ways in which this can be mitigated. (1) A number of the noisy false detections seem to be smaller bounding boxes. We could write a condition that avoids smaller bounding boxes. (2) Write a tracking algorithm that averages bounding boxes over multiple frames. This will have a history effect and result in less changes frame to frame.

