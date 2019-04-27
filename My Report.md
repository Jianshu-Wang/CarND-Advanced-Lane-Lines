
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

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test4.jpg "Road Transformed"
[image3]: ./output_images/undistort_output.png "Binary Example"
[image4]: ./output_images/gradient_binary_result.png "Binary Example"
[image5]: ./output_images/perspective_result.png "Binary Example"
[image6]: ./output_images/perspective_result2.png "Binary Example"
[image7]: ./output_images/Lane_Pixel_Result.png "Binary Example"
[image8]: ./output_images/Example_Result.png "Binary Example"
[image9]: ./examples/warped_straight_lines.jpg "Warp Example"
[image10]: ./examples/color_fit_lines.jpg "Fit Visual"
[image11]: ./examples/example_output.jpg "Output"
[video1]: ./output_images/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "report.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

Here we have to get the original image first and then use the `objpoints` and `imgpoints` like what I did in the previous step then I get the comparison figure which including the original and undistorted images:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used HLS color space and get the S image, and get the B image from LAB colorspace, and get the L image from the LUV image. Adjust their range, the reason why I use these is because I can get yellow lines by using S image and B image, and it is very easy to find white lines by using L image. I set `Sthresh=(90, 150)`, `Lthresh = (220,255)`, `Bthresh = (135,200)`. The combine logic is `(S&B)|L`, by using this combination it can filter the effect of the tree shade and some unusual road colors. Here is the combined output:
![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Now it is time to use `cv2.getPerspectiveTransform` to get the perspective transform from our original image, and we will get:
![alt text][image5]

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `report.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 100, 0        | 
| 280, 670      | 100, 720      |
| 1150, 670     | 1150, 720      |
| 730, 460      | 1150, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I find the 2 sides of the lane by finding peaks of histogram, and find the lane line pixels, and fit those pixel position by using `cv2.fillpoly()`, fill them with green color and I got what I need.
![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines by using `fit_polynomial_curve`, the same method, I got the radius of both sides and then get a mean value of them, you can check it in my `report.ipynb`.
I get `1488.6332453796426 m 2879.66577725656 m` for left and right side respectively.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in my code in `report.ipynb` in the part 6. Use the inverse matrix to get the perspective view and add it back on the original undistorted image, use`cv2.addWeighted(undistorted, 1, wrap_img, 1, 0)`, Here is an example of my result on a test image:
![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As you can see in the output video, the method described above works very well on the project video, by it can not handle the challenge videos, I have tried many possible threshold value ranges, but I still can not find a good thresholds' combination to make it works on both. I think we can try to the previous image frame to predict the area of the next frame and then to detect the target line in the next image frame, this would be a good way to avoid large errors. I will try after I finish all the projects.


