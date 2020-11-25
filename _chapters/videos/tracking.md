---
title: Tracking
keywords: feature tracking, klt algorithm
order: 16 # Lecture number for 2020
---


The process of detecting and tracking objects over multiple video frames is an interesting research area in computer vision. In this lecture, we will explore the problem of feature tracking and two different versions of the KLT feature tracker algorithm.


* [Feature Tracking](#feature-tracking)
    * [Problem Statement](#problem-statement)
    * [Challenges in Feature Tracking](#challenges-in-feature-tracking)
    * [Quality of Good Features](#quality-of-good-features)
    * [Motion Estimation Techniques](#motion-estimation-techniques)
* [Simple KLT Tracker](#simple-klt-tracker)
    * [Pipeline](#pipeline)
    * [Results](#results)
* [2D Transformations: Recap](#2d-transformations-recap)
    * [Types of 2D Transformations](#types-of-2d-transformations)
    * [Translation](#translation)
    * [Similarity Motion](#similarity-motion)
    * [Affine Motion](#affine-motion)
* [Iterative KLT Tracker](#iterative-klt-tracker)
    * [Problem Setting](#problem-setting)
    * [KLT Objective](#klt-objective)
    * [Mathematical Solution](#mathematical-solution)
    * [Interpretation of the H Matrix](#interpretation-of-the-h-matrix)
    * [Overall KLT Tracker Algorithm](#overall-klt-tracker-algorithm)
    * [KLT over Multiple Frames](#klt-over-multiple-frames)
    * [Challenges in Iterative KLT Tracker](#challenges-in-iterative-klt-tracker)



## Feature Tracking

### Problem Statement

Feature tracking takes an input a sequence of images, representing a video taken by a camera. The goal of feature tracking algorithms is not only to detect feature points in each frame, but also to track the points through the sequence of frames. Specifically, instead of treating each frame as a brand new image, we would like to keep track of which points from the previous frame represent the same feature points in the current frame.

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/feature-tracking-2.jpg?raw=true" width="600">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/feature-tracking.jpg?raw=true" width="600">
  <br />
  <em>
    Figure 1. Feature point detection and tracking (Credit: Yonsei University [2])
  </em>
  <br />
</p>

_**Single object tracking**_ applies when we can make an assumption that each frame will contain the object we want to track exactly once. Meanehile, _**multiple object tracking**_ applies when there could be any number of objects in each frame, such as a crowd of people on a busy street.

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/single-object-tracking.gif?raw=true" width="400">
  <br />
  <em>
    Figure 2. Single object tracking <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

<p align="center">
  <br />
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/multiple-object-tracking.gif?raw=true" width="400">
  <br />
  <em>
    Figure 3. Multiple object tracking <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

Assumptions can also be made regarding whether the camera is fixed or moving. Feature tracking with a moving camera is a more difficult problem, because we need to account for the motions of both the objects and the backgrounds. Another possibility is tracking with multiple cameras, with some assumptions about where the cameras are in physical space relative to each other.

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/fixed-camera-object-tracking.gif?raw=true" width="600">
  <br />
  <em>
    Figure 4. Tracking with a fixed camera <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

<p align="center">
  <br />
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/moving-camera-object-tracking.gif?raw=true" width="600">
  <br />
  <em>
    Figure 5. Tracking with a moving camera <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

<p align="center">
  <br />
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/multiple-cameras-object-tracking.gif?raw=true" width="500">
  <br />
  <em>
    Figure 6. Tracking with multiple cameras <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

<!-- ![feature tracking img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/feature-tracking.jpg?raw=true)  -->

### Challenges in Feature Tracking

The process of tracking feature points present a number of challenges.

* We must determine which features can be tracked, so that our algorithms can efficiently track them accross multiple frames.
* We must account for objects that are not moving, but have their appearance changing over time due to other factors like rotation, moving into shadows, and so on.
  * For example, a stationary object could appear to be moving if a shadow is being cast over it, darkening it from one side to the other.
* We must correct drift, which is when small errors in the location of key points can accumulate over time as appearance model is updated.
* We need to be able to introduce and remove some or all of the tracked points, as objects are allowed to leave the frame entirely or reenter at a later stage.

### Quality of Good Features

To account for these challenges, the features from the images that we decide to use must be resilient in those ways. In particlar, we need a technique that can measure the quality of features from just a single image, and we want to avoid smooth regions and edges.

For example, we do not want to track "ambiguous" locations, meaning large undifferentiated regions of the image that are hard to "anchor" to a specific location, such as arbitrary points along edges, faces, or smooth color gradients. We want to look at corners and sharp changes.

A feasible solution is using Harris Corners. We can leverage the Harris method to detect corners as key points and determine the quality of our key points from single images. For example, we can use Harris to get the corners of the very first frame, then track them from there. This practice guarantees small error sensitivity.


### Motion Estimation Techniques

Feature tracking is similar to, but not the same as optical flow. In fact, optical flow takes the pixels of the image frames and recover their motion based on image brightness changes across time and space simultaneously. However, feature tracking only attempts to estimate the motion of a small number of visual feature points over multiple frames. We can also achieve this goal by combining keypoint detection on each frame with optical flow algorithms to track the feature points.

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/optical-flow.jpg?raw=true" width="400">
  <br />
  <em>
    Figure 7. Application of optical flow in tracking feature points <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

Feature tracking has many practical applications. One example is using the combined key point locations and motions to solve for all locations in 3d space, for simultaneous location and mapping in robotics.

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/seq_initial.gif?raw=true" width="200">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/seq_tracked_1.gif?raw=true" width="200">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/seq_features_1.gif?raw=true" width="200">
  <br />
  <em>
    Figure 8. An application of feature tracking for visual navigation of a vehicle. <br /> (Courtesy of Jean-Yves Bouguet – Vision Lab, California Institute of Technology [3])
  </em>
  <br />
</p>

<!-- ![feature tracking 2 img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/seq_initial.gif?raw=true) 

![feature tracking 3 img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/seq_tracked_1.gif?raw=true) 

![feature tracking 4 img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/seq_features_1.gif?raw=true)  -->


## Simple KLT Tracker

The name “KLT” is derived from the names of its authors Kenade, Lucas, and Tomasi. This algorithm is a simple approach to feature extraction. 

### Pipeline

These are the steps for the algorithm:

1. Find a good point to track. For instance, we can accomplish this by running a Harris corner detector and selecting some corners in the image.
2. For every detected Harris corner, compute the motion of that corner across consecutive frames. We can accomplish this by using optical flow to calculate how the point is moving through time.
3. Link the motion vectors we obtain from optical flow in successive frames to get a full track for each Harris point. 
4. Introduce new Harris points by re-running the Harris detector every $10 - 15$ frames (because we may get new points or lose some points due to occlusion).
5. Keep track of new and old Harris points using steps $1-3$.

### Results

This section illustrates some results of the simple KLT tracking algorithm.

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/klt_fish.gif?raw=true" width="500">
  <br />
  <em>
    Figure 9. Results of the KLT algorithm for tracking fish (Courtesy of Kanade [4, 5])
  </em>
  <br />
</p>

<p align="center">
  <br />
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/klt_cars.gif?raw=true" width="400">
  <br />
  <em>
    Figure 10. Results of the KLT algorithm for tracking cars (Courtesy of Kanade [4, 5])
  </em>
  <br />
</p>

<p align="center">
  <br />
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/klt_movement.gif?raw=true" width="400">
  <br />
  <em>
    Figure 11. Results of the KLT algorithm for tracking human movements (Courtesy of Kanade [4, 5])
  </em>
  <br />
</p>

<!-- ![klt img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/klt_cars.gif?raw=true) 

![klt 2 img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/klt_movement.gif?raw=true)  -->


## 2D Transformations: Recap

This section reviews some fundamental concepts of 2D transformations that are necessary for understanding the next section.

### Types of 2D Transformations

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/2d_transforms.jpg?raw=true" width="600">
  <br />
  <em>
    Figure 12. Types of 2D transformations <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
</p>

<!-- ![2d transformations img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/2d_transforms.jpg?raw=true) -->

There are different types of 2D transformations, as shown in Figure 12. Some examples include:

* Translation transformations.
* Euclidean transformations.
* Similarity transformations.
* Affine transformations.
* Projective transformations.


### Translation

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/translation.jpg?raw=true" width="300">
  <br />
  <em>
    Figure 13. Translation motion <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

<!-- ![translation img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/translation.jpg?raw=true) -->

Translation is a motion that shifts an object from one point to another. In particular, translation motion maps one point $(x, y)$ to a new point $(x^\prime, y^\prime)$ in the next frame such that:

$$
\begin{eqnarray*}
x^{\prime}&=& x+b_{1} \\
y^{\prime}&=&y+b_{2}\\
\end{eqnarray*}
$$

We can rewrite this equation in matrix notation using homogeneous coordinates:

$$
\begin{bmatrix}
x^{\prime}\\
y^{\prime}
\end{bmatrix} = \begin{bmatrix}
1 & 0 & b_{1} \\
0 & 1 & b_{2}
\end{bmatrix}\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
$$

Let $W$ denote the above translation transformation. We have:

$$
W(x ; p)=\begin{bmatrix}
1 & 0 & b_{1} \\
0 & 1 & b_{2}
\end{bmatrix}\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
$$

in which there are only two parameters $\displaystyle p=\begin{bmatrix} b_{1} & b_{2} \end{bmatrix}^T$.

The derivative of the trasformation $W$ with respect to $p$ is:

$$
\frac{\partial W}{\partial p}(x ; p)=\begin{bmatrix}
1 & 0 \\
0 & 1
\end{bmatrix}
$$

This is the Jacobian of the translation motion.

### Similarity Motion

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/similarity.png?raw=true" width="500">
  <br />
  <em>
    Figure 14. Similarity motion <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

<!-- ![similarity img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/similarity.png?raw=true) -->

Similarity motion is a rigid motion that includes scaling and translation. In particular, similarity transformation maps one point $(x, y)$ to a new point $(x^\prime, y^\prime)$ in the next frame such that:

$$
\begin{eqnarray*}
x^{\prime}&=&a x+b_{1} \\
y^{\prime}&=&a y+b_{2}
\end{eqnarray*}
$$

We can rewrite the equations as a matrix transformation:

$$
W(x ; p)=\begin{bmatrix}
a & 0 & b_{1} \\
0 & a & b_{2}
\end{bmatrix}\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
$$

in which the parameters are $\displaystyle p=\begin{bmatrix}
a & b_{1} & b_{2}
\end{bmatrix}^T$.

<!-- The similarity transformation matrix W and the parameters p are described as follows:

$$
\begin{eqnarray*}
W&=&\begin{bmatrix}
a & 0 & b_{1} \\
0 & a & b_{2}
\end{bmatrix}\\
p&=&\begin{bmatrix}
a & b_{1} & b_{2}
\end{bmatrix}^T
\end{eqnarray*}
$$ -->

The derivative of the trasformation $W$ with respect to $p$ is:

$$
\frac{\partial W}{\partial p}(x ; p)=\begin{bmatrix}
x & 1 & 0 \\
y & 0 & 1
\end{bmatrix}
$$

This is the Jacobian of the similarity motion.

### Affine Motion

<p align="center">
  <img src="https://github.com/Visininjr/cs131_notes_dev/blob/master/images/affine.png?raw=true" width="500">
  <br />
  <em>
    Figure 15. Affine motion <br /> (Courtesy of Juan Carlos Niebles & Jiajun Wu – Stanford Vision and Learning Lab [1])
  </em>
  <br />
</p>

<!-- ![affine img](https://github.com/Visininjr/cs131_notes_dev/blob/master/images/affine.png?raw=true) -->

Affine motion includes scaling, translation, and rotation. In particular, affine transformation maps one point $(x, y)$ to a new point $(x^\prime, y^\prime)$ in the next frame such that:

$$
\begin{eqnarray*}
x^{\prime}&=&a_{1} x+a_{2} y+b_{1} \\
y^{\prime}&=&a_{3} x+a_{4} y+b_{2}
\end{eqnarray*}
$$

We can rewrite the equations as a matrix transformation:

$$
W(x ; p)=\begin{bmatrix}
a_{1} & a_{2} & b_{1} \\
a_{3} & a_{4} & b_{2}
\end{bmatrix}\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
$$

in which the parameters are $\displaystyle p=\begin{bmatrix}
a_{1} & a_{2} & b_{1} & a_{3} & a_{4} & b_{2}
\end{bmatrix}^T$.

<!-- The affine transformation matrix W and parameters p are described as follows:

$$
\begin{eqnarray*}
W&=&\begin{bmatrix}
a_{1} & a_{2} & b_{1} \\
a_{3} & a_{4} & b_{2}
\end{bmatrix}\\
p&=&\begin{bmatrix}
a_{1} & a_{2} & b_{1} & a_{3} & a_{4} & b_{2}
\end{bmatrix}^T
\end{eqnarray*}
$$ -->

The derivative of the trasformation $W$ with respect to $p$ is:

$$
\frac{\partial W}{\partial p}(x ; p)=\begin{bmatrix}
x & y & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & x & y & 1
\end{bmatrix}
$$

This is the Jacobian of the affine motion.

## Iterative KLT Tracker

### Problem Setting

The iterative KLT tracker is a more advanced version of the above simple KLT tracker.

Given a video sequence, we want to find all the features and track them across the video. Using Harris corner detection, we can obtain the current location of each feature in a given frame, and then find their new locations in the next frame.

For each feature at location $x = \begin{bmatrix} i_x & i_y \end{bmatrix}^T$, we want to create an initial template $T(x)$ for that feature, which is typically an image patch around $x$. In addition, for a location $x$, we will assume that $x$ undergoes a transformation (translation, affine, etc.) parameterized by $p$ to reach its new location $W(x; p)$.

Compared to the simple KLT tracker, this iterative technique employs a different way of connecting sequential frames. Instead of harnessing optical flow to compute and track motions, we directly estimate the transformations based on input feature vectors and linear approximations. This practice enables us to handle more complex motions and make our feature tracking pipeline more robust.

### KLT Objective

Given this problem setting, the objective of the iterative KLT tracker is to find the parameter of transformation $p$ that minimizes the difference between the original template $T(x)$ and the image patch around the new location of $x$ in the next frame (after the transformation). This idea is represented by the following KLT objective, which is what we want to minimize:

  $$L = \sum_x [I(W(x; p)) - T(x)]^2 $$

where
* $W(x; p)$ is the new location of feature $x$.
* $I(W(x; p))$ is the image intensity at the new location.
* $p$ represents the vector of parameters that define the transformation that moved $x$ to its new location $W(x; p)$.
* The sum is over the image patch around $x$.

### Mathematical Solution

In this section, we will derive a closed form approximation of $p$ that minimizes the KLT objective.

Since $p$ may be large, directly finding the minimum of $\displaystyle L = \sum_x \left[I(W(x;p)) - T(x)\right]^2$ may be quite challenging. 

Instead, we will break down $p$ into two components $p = p_0 + \Delta p$. In this case, $p_0$ and $\Delta p$ represents large and small (residual) motions respectively. We can fix $p_0$ by initializing it with our best guess of the motion, then solve for the small value $\Delta p$.

Now, we can use the Taylor series to approximate $L$. We know that for a small value of $\Delta x$:

$$
f(x + \Delta x) = f(x) + \Delta x \frac{\partial f}{\partial x} + \Delta x^2 \frac{\partial^2 f}{\partial x^2} + \dots
$$

By applying this Taylor approximation with the first two terms, we learn that:

$$ 
\begin{align*}
L &= \sum_x \left[I(W(x; p_0 + \Delta p)) - T(x)\right]^2 \\
&\approx \sum_x \left[I(W(x;p_0)) + \nabla I \frac{\partial W}{\partial p} \Delta p - T(x)\right]^2 \\
\end{align*}
$$

in which $\displaystyle \nabla I = \begin{bmatrix} I_x & I_y \end{bmatrix}$ and $\displaystyle \frac{\partial W}{\partial p}$ can be pre-computed for affine motions, translation motions, and other transformations.

Afterwards, we aim to find $\displaystyle \arg\min_{\Delta p} \overline{L}$ where $\displaystyle \overline{L} = \sum_x \left[I(W(x;p_0)) + \nabla I \frac{\partial W}{\partial p} \Delta p - T(x)\right]^2$.

Computing its derivative with respect to $\Delta p$ and setting it equal to $0$, we get:

$$ 
\frac{\partial \overline{L}}{\partial \Delta p} = \sum_x \left[\nabla I \frac{\partial W}{\partial p}\right]^T \left[I(W(x;p_0)) + \nabla I \frac{\partial W}{\partial p} \Delta p - T(x)\right] = 0
$$

By solving for $\Delta p$, we learn that:

$$
\Delta p = H^{-1} \sum_x \left[\nabla I \frac{\partial W}{\partial p}\right]^T \left[T(x) - I(W(x;p_0))\right]
$$

in which $\displaystyle H = \sum_x \left[\nabla I \frac{\partial W}{\partial p}\right]^T \left[\nabla I \frac{\partial W}{\partial p}\right]$ must be invertible.

### Interpretation of the H Matrix

Let's consider translation motions. We know that:

$$
\begin{eqnarray*}
\nabla I &=& \begin{bmatrix} I_x & I_y \end{bmatrix} \\
\frac{\partial W}{\partial p} &=& \begin{bmatrix} 1 & 0 \\ 0 & 1\end{bmatrix}
\end{eqnarray*}
$$

Thus, the $H$ matrix becomes:

$$ 
\begin{align*}
H &= \sum_x \left[\nabla I \frac{\partial W}{\partial p}\right]^T \left[\nabla I \frac{\partial W}{\partial p}\right] \\
&= \sum_x \left[\begin{bmatrix} I_x & I_y \end{bmatrix} \begin{bmatrix} 1 & 0 \\ 0 & 1\end{bmatrix}\right]^T \left[\begin{bmatrix} I_x & I_y \end{bmatrix} \begin{bmatrix} 1 & 0 \\ 0 & 1\end{bmatrix}\right] \\
&= \sum_x \begin{bmatrix} I_x^2 & I_xI_y \\ I_xI_y & I_y^2\end{bmatrix}
\end{align*}
$$

This is the matrix for the Harris corner detector. We can see that $H$ is easily invertible when both of its eigenvalues are large, which represents a corner in the input image. Therefore, corners in images are good features for calculating translation motions.

### Overall KLT Tracker Algorithm

Given our formula for $\Delta p$, these are the steps for the iterative KLT tracking algorithm.

Given the features from Harris detector:

1. Initialize $p_0$.
2. Compute the initial templates $T(x)$ for each feature.
3. Transform the features in the image $I$ with $W(x;p_0)$.
4. Measure the error $\displaystyle I(W(x;p_0))-T(x)$.
5. Compute the image gradients $\displaystyle \nabla I = \begin{bmatrix} I_x & I_y \end{bmatrix}$.
6. Evaluate the Jacobian $\displaystyle \frac{\delta W}{\delta p}$.
7. Compute the steepest descent $\displaystyle \nabla I \frac{\delta W}{\delta p}$.
8. Compute the inverse Hessian $H^{-1}$.
9. Calculate the change in parameters $\Delta p$.
10. Update parameters $\displaystyle p_0 = p_0+\Delta p$.
11. Repeat steps $3-10$ until $\Delta p$ is sufficiently small.


### KLT over Multiple Frames

We can extend this algorithm over multiple frames. This is because once we find a transformation between two consecutive frames, you can repeat this process for every new frame that comes in. Run the Harris detector every so often ($15-20$ frames) to replenish feature points.

### Challenges in Iterative KLT Tracker

When implementing this algorithm, there are a few key issues to consider:

* Window size (the size of neighborhood/template around each location $x$):
    * A small window tend to be more sensitive to noise and may miss larger motions (if we do not use a pyramid technique).
    * A large window is more likely to cross an occlusion boundary, making the signal confusing to the algorithm. In addition, a large window size adds more computational costs to our program.
    * Most implementations stay between the range of $15 \times 15$ to $31 \times 31$ for neighborhoods.
* Weighting the window:
    - A common approach is applying weights so that center pixels get higher weights. For example, using Gaussian weighting gives more emphasis to center pixels.




## References

* [1] Niebles, J. C., & Wu, J. (2020). CS 131 Lecture: Tracking. 
* [2] Suhr, J. K. (2009). Kanade-Lucas-Tomasi (KLT) Feature Tracker Feature Tracker. 
* [3] Bouguet, J.-Y., & Perona, P. (1995). 3D Motion and Structure Estimation. ICCV 95 Proceedings. Department of Electrical Engineering, California Institute of Technology. 
* [4] Lucas, B. D., & Kanade, T. (1981). An Iterative Image Registration Technique with an Application to Stereo Vision. Computer Science Department, Carnegie Mellon University.
* [5] Tomasi, C., & Kanade, T. (1991). Detection and Tracking of Point Features. Computer Science Department, Carnegie Mellon University.


<!-- &emsp; [1] http://vision.stanford.edu/teaching/cs131_fall1920/slides/18_tracking.pdf

&emsp; [2] https://web.yonsei.ac.kr/jksuhr/articles/Kanade-Lucas-Tomasi%20Tracker.pdf

&emsp; [3] http://www.vision.caltech.edu/bouguetj/Motion/navigation.html

&emsp; [4] https://cecas.clemson.edu/~stb/klt/lucas_bruce_d_1981_1.pdf

&emsp; [5] https://cecas.clemson.edu/~stb/klt/tomasi-kanade-techreport-1991.pdf -->
