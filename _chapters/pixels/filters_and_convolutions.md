---
title: Filters and convolutions
keywords: systems, convolution, correlation
order: 3
---
# Filters and Convolutions

Table of contents:

- [Introduction](#topic1)
- [Images as Functions](#topic2)
- [Systems and Filters](#topic3)
- [System Properties](#topic4)
- [Linear Shift Invariant Systems](#topic5)
- [2D Discrete Convolutions](#topic6)
- [Covolution Implementation Details](#topic7)
- [Cross Correlation](#topic8)
- [Conclusion](#topic9)

[//]: # (Notice in the table of contents that [First Big Topic] matches #first-big-topic, except for all lowercase and spaces are replaced with dashes. This is important so that the table of contents links properly to the sections)

<a name='Topic 1'></a>
## Introduction
Linear systems and filters have several applications in computer vision. Some examples include 
* **de-noising**: removing noise from an image to get an improved version of the image, 
* **super-resolution**: producing a newer image with more detail from a lower resolution picture, and 
* **in-painting**: filling in pixels for which information may have been destroyed in some other process.

**Definition:** Filtering is a process through which we form a new image which has pixel values that are calculated or transformed from the original image's pixel values.

**Goals of filtering:**
1. Extract useful information from images (we can extract features, such as edges, corners, blobs, or other structures in the image).
2. Transform the image to improve them (transformations include super-resolution, in-painting, and de-noising).


<a name='Topic 2'></a>
## Images as Functions
Images are usually digital and discrete, and are formed by sampling the 2-dimensional space on a regular grid. We can therefore represent them as matrices of integer values in computer memory (such that every element in that matrix is the representation of a pixel).

Cartesian coordinates: We can also think of an image as being present in a Cartesian plane. We can then assign every pixel to a coordinate.

**Images as functions:** We can consider images as a function mapping from $R^2$ to $R^M$.
* For example, in a grayscale image, $f[x, y]$ gives us the intensity at position $[x, y]$
* Input is  defined over a rectangle with a finite range: $f:[a, b] x [c, d] \rightarrow [0, 255]$
* For color images, we have the following function: $\begin{align}
f(x, y) = \Big[r(x, y) \\
													g(x, y) \\
													b(x, y)\Big] 
                                                    \end{align}$
                                

<a name='Topic 3'></a>
## Systems and Filters
We define a system as a unit that converts an input function $f[n,m]$ into an
output function $g[n,m]$, where $[n, m]$ are the independent variables. In the case of images, $[n, m]$ represents the **spatial position** in the image.
$$
f[n,m]\xrightarrow{System S}g[n,m]
$$
$S$ is the **system operator** defined as a mapping/assignment that transforms the
input $f$ into the output $g$.
$$
g = S[f], g[n,m] = S\{f[n,m]\}
\\
f[n,m]\xrightarrow{S}g[n,m]
$$
Now let's look at two examples.
The first example is **moving average**, which uses a 2D moving average over a 3 × 3 neighborhood window. This filter "transforms" each pixel value into the average value of its neighborhood and achieves smoothing effect on the image.
$$
g[n,m]=\frac{1}{9}\sum_{k=n-1}^{n+1}\sum_{l=n-1}^{n+1}f[k,l]
\\
=\frac{1}{9}\sum_{k=-1}^{1}\sum_{l=-1}^{1}f[n-k,m-l]
$$
Filter result:
<div class="fig figcenter fighighlight">
  <img width="700" src="https://i.imgur.com/4TmuGop.png">
</div>

The second example is **image segmentation**.
Base on a simple threshold, we are able to get
$$
g[n,m] =  \left\{\begin{matrix}
255 & f[n,m]>100\\ 
0 & \text{otherwise.} 
\end{matrix}\right.
$$
Filter result:
<div class="fig figcenter fighighlight">
  <img width="700" src="https://i.imgur.com/BnzITX5.png">
</div>


<a name='Topic 4'></a>
## System Properties
A useful way of thinking about systems is in terms of their properties. Two catgories of properties include **amplitude properties** and **spatial properties**. 

Examples of amplitude properties include:
1. Additivity: the output corresponding to the sum of any two inputs is the sum of the two outputs.
2. Homogeneity: scaling any input scales the output by the same factor.
3. Stability: every bounded input produces a bounded output.
4. Invertibility: produces distinct output signals for distinct input signals.
5. Superposition: first, we note that the definition of a **linear system/filter** is a system that forms a new image whose pixels are a weighted sum of the original pixel values, using the same weights at each point. Given a system $S$ $$f[n,m]\xrightarrow{S}g[n,m],$$ we say $S$ is a linear system (function) if and only if it satisfies $$S[\alpha f_i[n, m] + \beta f_j[k, l]] = \alpha S[f_i[n, m]] + \beta S[f_j[k, l]],$$ otherwise known as the **superposition property**. In words, this property states that passing in the linear combination of images $f_i, f_j$ to our system yields the same output as passing $f_i, f_j$ to our system individually, then taking the linear combination of the resulting images.

Examples of Spatial Properties include:
1. Causality: present value of the output depends only on the present or past values of the input.
2. Memory:  output depends on past or future values of the input.
3. Shift Invariance: given a system $S$, $$f[n,m]\xrightarrow{S}g[n,m],$$ we say $S$ is **shift invariant** if $$f[n - n_0,m - m_0]\xrightarrow{S}g[n - n_0,m - m_0]$$ for every input image $[n, m]$ and shifts $n_0$, $m_0$. Essentially, this means that if we shift the input $f$ by a given amount, then our output will be shifted by the same amount. In the context of an image, shifting entails moving an image in the cartesian plane. 

Let's take a look at the properties of the Moving Average System introduced above. 
![](https://i.imgur.com/SsCDScD.png)

***Is the moving average system shift invariant?*** Yes!
To see this, we start with passing $f$ through our system $S$ to obtain $g$, as before: $$f[n,m]\xrightarrow{S}g[n,m]$$ We also know that in the case of the moving average system, $g$ can be written down as: $$g[n, m] = \frac{1}{9}\sum_{k=n-1}^{n+1}\sum_{l=m-1}^{m+1}f[k,l] = \frac{1}{9}\sum_{k=-1}^{1}\sum_{l=-1}^{1}f[n-k,m-l]$$ Now, passing in the shifted version of $f$, $$f[n - n_0, m - m_0]\xrightarrow{S}\frac{1}{9}\sum_{k=-1}^{1}\sum_{l=-1}^{1}f[(n-n_0)-k, (m-m_0)-l]$$ we note that we get a shifted version of $g$, $g[n-n_0, m-m_0]$, that is shifted by exactly the same amount.

***Is the moving average system a linear system?*** Yes!
To see this, we take an image $f[x, y] = \alpha f_i[n, m] + \beta f_j[k, l]$, and note that since $$S[f[x, y]] = \frac{1}{9}\sum_{a=x-1}^{x+1}\sum_{b=y-1}^{y+1}f[a, b] $$ we can split this into $$= \frac{1}{9}\sum_{a=n-1}^{n+1}\sum_{b=m-1}^{m+1}\alpha f_i[a, b] + \frac{1}{9}\sum_{a=k-1}^{k+1}\sum_{b=l-1}^{l+1}\beta f_j[a, b]$$ $$ = \alpha (\frac{1}{9}\sum_{a=n-1}^{n+1}\sum_{b=m-1}^{m+1}f_i[a, b]) + \beta (\frac{1}{9}\sum_{a=k-1}^{k+1}\sum_{b=l-1}^{l+1}f_j[a, b])$$ $$= \alpha S[f_i[n, m]] + \beta S[f_j[k, l]]$$ which fulfills the superposition property. 

***Is the thresholding system a linear system?*** No! 
Note that we can have $f_i[n, m] + f_j[n, m] > T$ when both $f_i[n, m] < T$ and $f_j[n, m] < T$. 





<a name='Topic 5'></a>
## Linear Shift Invariant Systems
We’ve now seen two properties of systems: superposition and shift invariance.  A system that satisfies both of these properties is a special system called a *linear shift invariant (LSI) system*.  Formally, an LSI system satisfies the following two properties:

$$
S\{\alpha f_1[n,m] + \beta f_2[n,m]\} = \alpha S\{f_1[n,m]\} + \beta S\{f_2[n,m]\}
$$

$$
f[n-n_0, m-m_0]\xrightarrow{S}g[n-n_0, m-m_0]
$$

LSI systems have important applications in computer vision.  One example of an LSI system is the moving average filter.

How does an LSI system act on an input image $f$?  To understand this, we can analyze the impulse response.  For a two-dimensional image, there is a single impulse at the origin, and the impulse function $\delta_2$ is given by:
$$
\delta_2 = \left\{\begin{matrix}
1 & \text{at }[0,0]\\ 
0 & \text{everywhere else} 
\end{matrix}\right.
$$

<div class="fig figcenter fighighlight">
  <img width="350" src="https://i.imgur.com/LKXtTP2.jpg">
</div>

We can pass in an impulse function into the LSI system and record its response:
$$
\delta_2[n,m]\xrightarrow{S}h[n,m]
$$
where $\delta_2$ is an impulse function and $h$ is the response.


The key idea is that an image $f$ can be represented as a sum of impulses:
$$
f[n,m] = \sum_{k = -\infty}^\infty \sum_{l = -\infty}^\infty f[k,l] \cdot \delta_2[n-k, m-l]
$$
An example of the 3x3 case is shown below:
<div class="fig figcenter fighighlight">
  <img width="700" src="https://i.imgur.com/bdsQ7IS.jpg">
</div>

An LSI system is completely specified by its impulse response.  This means that for any input $f$, we can compute the output $g$ in terms of the impulse response $h$:
\begin{align*}
f[n,m]&\xrightarrow{S}g[n,m] \\
f[n,m]&=\sum_{k = -\infty}^\infty \sum_{l = -\infty}^\infty f[k,l] \cdot \delta_2[n-k, m-l]\\
&\xrightarrow{S}\sum_{k = -\infty}^\infty \sum_{l = -\infty}^\infty f[k,l] \cdot S\{\delta_2[n-k, m-l]\}
\end{align*}
By the shift invariance property, we know that 
$$
S\{\delta_2[n-k, m-l]\} = h[n-k,m-l]
$$

Therefore,
$$
f[n,m]\xrightarrow{S}\sum_{k = -\infty}^\infty \sum_{l = -\infty}^\infty f[k,l] \cdot h[n-k, m-l]
$$
This can also be written as 
$$
f[n,m] * h[n,m] = \sum_{k = -\infty}^\infty \sum_{l = -\infty}^\infty f[k,l] \cdot h[n-k, m-l]
$$
which is known as a *discrete convolution*.

**Example.**  We can understand how LSI systems relate to the impulse response with an example of a 3x3 moving average filter.  Recall that the 3x3 moving average filter is given by:
$$
f[n,m]\xrightarrow{s}\frac{1}{9}\sum_{k=-1}^1 \sum_{l=-1}^1 f[n-k, m-l]
$$
From this, we have the following as an expression for the impulse response:
$$
\delta_2[n,m]\xrightarrow{s}h[n,m] = \frac{1}{9}\sum_{k=-1}^1 \sum_{l=-1}^1 \delta_2[n-k, m-l]
$$
If we calculate the values for $n = -2, \cdots, 2$ and $m = -2, \cdots, 2$, we get the following:
<div class="fig figcenter fighighlight">
  <img width="275" src="https://i.imgur.com/FHdfNNf.jpg">
</div>

This can be represented by the matrix:
$$
\begin{bmatrix}
\frac{1}{9} & \frac{1}{9} & \frac{1}{9}\\ 
\frac{1}{9} & \frac{1}{9} & \frac{1}{9}\\ 
\frac{1}{9} & \frac{1}{9} & \frac{1}{9}
\end{bmatrix}
$$

This is the same filter that we derived for the moving average.

<a name='Topic 6'></a>
## 2D Discrete Convolutions
Using the moving average filter from the previous section we can manipulate images to apply various filters and effects.

To do this we must identify a kernel, flip it horizontally and vertically, or "fold" it, and shift the kernel to the desired location (n,m). We then add all the values in the kernal to find the value of the pizel at (n,m)

Examples-

If we have 1 3x3 kernel with a 1 in the center location, surrounded by zeros, we will get back the same image. This can be refered to as the identity matrix.

If we have a 3x3 matrix with a 1 in the (3,2) location,  the image is shifted by 1 pixel to the right.

Finally, if we have 3x3 kernel with all 1s, all divided by 9,it will blur the image as it is a moving average of surrounding pixels.

Bringing it all together, if we then have a 3x3 matrix with a 2 in the center surrounded by zeros, minus the matrix from the previous example, we can split the first term into 2 identity kernels. We then see the identity kernel minus the blur kernel which we can call the details of the image. We then add the original image again.

This essentially enhances the details of the image and this process can be repeated to add more detail to the image. This adds a sharpening effect, to amplify the differences.
<a name='Topic 7'></a>
## Covolution Implementation Details

In digital processing, images are typically represented as the finite support signals, meaning that outside of the $(n,m)$ rectangular region covered by the image, pixel values are zero. As such, numpy library in Python will perform convolution of a finite support signal. 

The issue that arises from this approach is the edge effect -- when the convolution kernel exceeds boundaries of the image at the edge or close-to-edge nodes. Several strategies have been devised to handle image edge issue, as outlined below:

- *Zero "padding"* -- extends finite support signal of the image to the necessary range with pixels with the value of zero, thereby providing necessary information for the convolution kernel.
- *Edge value replication* -- extrapolates the image by copying the edge pixel values.
- *Mirroring* -- reflects the image around the edge (e.g. extending $n$ pixels outside of the image is equivalent to moving $n$ pixel inside from the edge).
- *Other* -- beyond the scope of this class (e.g. wrapping, kernel cropping, clamping, etc.)

With all of the above techniques, there is no one-for-all solution and, typically, performance will vary depending on the task at hand and the image analyzed. However, if no technique to deal with the boundary effect is specified, zero "padding" is usually the default option in most of the convolution engines.

<a name='Topic 8'></a>
## Cross Correlation

At this point, we have covered everthing through convolutions. We will now introduce the method of cross correlation. Cross coorelation is equivalent to covulution without the flip.

A main use of cross correlation is to find the similarities between $f$ and $h$. We represent cross correlation with $**$

Say we have two 2d signals $f[n,m]$ and $h[n,m]$. Then, 

$$
\begin{align}
f[n,m]**h[n,m]= \sum_{k = -\infty}^\infty \sum_{l = -\infty}^\infty f[k,l] \cdot h[n+k, m+l]
\end{align}
$$
Below, we see the slight but important distinctions between convolution and cross-correlation. Note how the lack of flipping mirrors our convuluted signal output.
<div class="fig figcenter fighighlight">
  <img width="275" src="https://i.imgur.com/9c3DUab.png">
</div>

When would cross correlation and convultion give the same output? When flipping our kernel results in the same kernel as before (it is symmetric).

Further explaining this difference, we define convultion to be a filtering operation while cross correlation is a measure of relatedness between two signals. 

<a name='Topic 9'></a>
## Conclusion
In this lecture, we have seen that images can be thought of as functions. That definition is useful when looking at systems and filters. In particular, we have shown the relation between linear shift invariant (LSI) systems and the operation we call convolution.
