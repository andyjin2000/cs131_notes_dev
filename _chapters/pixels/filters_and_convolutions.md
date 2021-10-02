---
title: CS 131 Lecture 3 Notes: Filtering
keywords: (insert comma-separated keywords here)
order: 3 # Lecture number for 2020
---

## Linear Systems

### Images as Functions

Because images are digital, they are typically represented discretely as a 2D matrix of integer values which capture pixel intensity. Concretely, an image is represented as a function $f: \mathbb{R}^2 \rightarrow \mathbb{R}^M$, wherein $f[x,y]$ gives the intensity of the pixel at position $[x,y]$. For a grayscale image, the pixel intensity is an integer in the range $[0,255]$. For a color image, $$f[x,y] = \begin{bmatrix}
    r[x,y] \\
    g[x,y] \\
    b[x,y]
\end{bmatrix}$$ such that $r, g, b: [a,b] \times [c,d] \rightarrow [0,255]$.

We can think of the image matrix as having infinite width and height, centered at (0,0), as shown below. Note that (0,0) is the top-left corner of the image.

$$\begin{bmatrix}
    \ddots &  & \vdots & & \\
    & f[-1, 1] & f[0, 1] & f[1, 1] \\
    \dots & f[-1, 0] & f[0, 0] & f[0, 1] & \dots\\
        & f[-1, -1] & f[0, -1] & f[1, -1]  \\
         & & \vdots & & \ddots
\end{bmatrix}$$

However, we typically only consider a finite region within this matrix for a specific image. That is, defined over a rectangle with a finite range, the function representation of a grayscale image is: $f: [a,b] \times [c,d] \rightarrow [0,255]$, where $[a,b] \times [c,d]$ is the domain support which defines the range of valid inputs to $f$. Concretely, $[a,b]$ specify the horizontal range of the image, and $[c,d]$ the vertical range. 

Furthermore, we can plot images as a surface, such as the one shown below. The 2D plane indicates the coordinates of the images, and the vertical height indicates the intensity of each pixel.

![](https://i.imgur.com/GHci7zw.png)

### Overview of Filtering

The term **filtering** refers to the process by which a new image is formed by transforming pixel values of an original image. Filtering serves to extract useful information from images, such as edges, corners, blobs, or to modify and enhance certain image properties (e.g., super-resolution, in-painting, de-noising).

A **system**, denoted by system operator $\mathcal{S}$, converts an input function $f[n,m]$ into an output function $g[n,m]$, where $[n,m]$ are the independent variables (e.g., position in the image): 
$$f[n,m] \rightarrow \boxed{\text{System } \mathcal{S}} \rightarrow g[n,m]$$

Below are several equivalent notations to define a system transforming input $f$ into output $g$:
- $g = \mathcal{S}[f]$
- $g[n,m] = \mathcal{S}\{f[n,m]\}$
- $f[n,m] \overset{\mathcal{S}}\rightarrow g[n,m]$
 
### Moving Average Filter

The moving average filter sets the output pixel value to be the average of its neighboring pixel values. For instance, a $3 \times 3$ moving average filter will set the pixel at position $[n,m]$ of the output image, $g$, to be:
$$g[n,m] = \frac{1}{9} \sum\limits_{k=n-1}^{n+1}\sum\limits_{l=m-1}^{m+1} f[k, l]$$

The outer summation sums over the rows and the inner summation sums over the columns. Using change of variables, this can be simplified to:

$$g[n,m] = \frac{1}{9} \sum\limits_{k=-1}^{1}\sum\limits_{l=-1}^{1} f[n-k, m-l]$$

The moving average filter has the visual effect of slightly blurring the image; it smooths out the image and removes sharp features by averaging those pixels with neighboring values that have smaller intensity.

![](https://i.imgur.com/Dp8ZGqc.png)

#### Filter Mask

Another way to think of the moving average filter is in terms applying the following moving average filter "kernel" or "mask" at every neighborhood of the input image, $f$, to compute $g$: $$h = \frac{1}{9} \begin{bmatrix}
    1 & 1 & 1 \\
    1 & 1 & 1 \\
    1 & 1 & 1
\end{bmatrix} = \begin{bmatrix}
    \frac{1}{9} & \frac{1}{9} & \frac{1}{9} \\
    \frac{1}{9} & \frac{1}{9} & \frac{1}{9} \\
    \frac{1}{9} & \frac{1}{9} & \frac{1}{9}
\end{bmatrix}$$

Each element in the kernel is the coefficient we are multiplying the corresponding element in the neighborhood by. After performing element-wise multiplication, we then sum all the results in the neighborhood, and then take the average.

### Image Segmentation Filter

Filtering can also be used to perform image segmentation, setting the value of an output pixel to either black or white based on whether the corresponding input pixel is above or below a threshold *t*, respectively. This can be represented as a piecewise function:

$$g[n,m] = \begin{cases} 255, & f[n,m] > t \\ 0, & \text{otherwise}\end{cases}$$

This image segmentation filter has the visual effect of transforming the image into contrasting dark and light regions, as seen below:

![](https://i.imgur.com/oORn4FW.png)

### Properties of Linear Systems

### Linear Shift-Invariant (LSI) Systems <Does this go here?>
_<Jerry, fill this introductory info in>_


Additionally and critically, an LSI System is completely specified by its impulse response. The following section will discuss exactly what this statement means and how it is relevant in understanding how LSI Systems 

## LSI Systems and Convolution

### Impulse Functions and Responses
To fully understand the relevance of an impulse response to an LSI System, we must first understand the impulse function behind it. In the field of Image Processing, 2D Impulse functions are 


One of the most commonly used examples of impulse functions can be found in the 2D Delta ($\delta_2$) impulse function. This impulse function can be described by the piecewise function defined below, in which $f[0,0]$ exists at the center of the shown matrix.

$$ \delta_2[n,m]=   \left\{
\begin{array}{ll}
      1, & n=0,m=0 \\
      0, & otherwise \\
\end{array} 
\right.  $$

$$\begin{bmatrix}
    \ddots &  & \vdots & & \\
    & 0 & 0 & 0 \\
    \dots & 0 & 1 & 0 & \dots\\
        & 0 & 0 & 0  \\
         & & \vdots & & \ddots
\end{bmatrix}$$

When an impulse function is passed into a system, we get an impulse response as output, a ____ that does this. Impulse responses are often referred to as kernels or filters. 

$$\delta_2[n,m] \rightarrow \boxed{\text{System } \mathcal{S}} \rightarrow h[n,m]$$

This conversion from impulse function to impulse response can be better understood using the Moving Average Filter example introduced previously:

_<Insert Example Here>_

As stated earlier, _an LSI System can be completely specified by its impulse response_. This means that the impulse response produced by passing some impulse function through an LSI system can be used to predict what a system's output image will look like. This statement also indicates that, given an input image and an impulse response for an LSI System, we can actually use this information to generate the corresponding output image.

### 2D Convolution with LSI Systems

_<Is it bad that I briefly introduce convolution here? Not sure how else to order the info>_ 
Convolution is an operation that describes the output of an LSI System, a specific type of system with properties as discussed previously.

We discussed how the concept of a kernel might be used to apply a filter to an image using the _Moving Average Filter_ example, but now that we understand the concept of impulse functions and impulse responses, we can generalize this process to an arbirary impulse response $h$.

We know that LSI Systems satisfy the **Superposition Property** and the **Shift-Invariance Property**. Additionally, we have just discussed that:
$$\delta_2[n,m] \overset{\mathcal{S}}\rightarrow h[n,m]$$

Using these three properties, we can derive an equation that describes an output image $g$ produced by an LSI system solely in terms of an input image $f$ and a kernel $h$. 

In order to do so, we must first imagine the input image $f$ as a sum of impulses or shifted $\delta_2$ functions. Let $f[n,m]$ represent a 3X3 input image, and let $\delta_2[n,m]$ represent our delta impulse function as described previously. As you can see below, for each pixel in our image, we produce a new shifted $\delta_2$ function multiplied the initial pixels value.

_<Insert Visual Representation of this here>_

\begin{align}
f[n,m] &= f[0,0] \cdot\delta_2[n,m] + f[0,1] \cdot\delta_2[n,m-1] + ... + f[2,2] \cdot\delta_2[n-2,m-2]\\
&= \sum_{i=-\infty}^\infty\sum_{j=-\infty}^\infty f[i,j]\cdot \delta_2[m-i,n-j]\\
\end{align}

Now that we have described the input image as a sum of individual pixels, we can imagine that the output will simply be equal to the sum of the systems output for each cell.

$$f[n,m] \overset{\mathcal{S}}\rightarrow g[n,m]\\
f[n,m] \overset{\mathcal{S}}\rightarrow g[n,m]=\sum_{i=-\infty}^\infty\sum_{j=-\infty}^\infty f[i,j]\cdot S\{\delta_2[m-i,n-j]\}\\
$$

$$
f[n,m] \overset{\mathcal{S}}\rightarrow g[n,m]=\sum_{i=-\infty}^\infty\sum_{j=-\infty}^\infty f[i,j]\cdot h[m-i,n-j]=(f*h)[m,n]
$$

This operation is also referred to as 2D discrete convolution and can be represented by $f[n,m]*h[n,m]$ or $(f*h)[m,n]$. In the case of the image above, we can say that the function $f$ is being convolved with the kernel $h$.

