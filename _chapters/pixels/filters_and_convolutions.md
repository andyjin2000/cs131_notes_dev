---
title: Filters and convolutions
keywords: (insert comma-separated keywords here)
order: 3 # Lecture number for 2020
---

**Authors:** AJ Arnolie, Andy Jin, Cynthia Liu, Edward Park, Daulet Tuleubayev, Jerry Wei

## Contents
1. Linear Systems
    - Images as Functions
    - Overview of Filtering
    - Moving Average Filter
    - Image Segmentation Filter
2. Properties of Linear Systems
    - Shift Invariance
    - Superposition
3. LSI Systems and Convolution
    - Impulse Functions and Responses
    - 2D Convolution with LSI Systems
4. Convolutions and Correlations
    - 2D Discrete Convolution
    - Examples of Simple Convolutions
    - Implementation of Convolutions
    - (Cross) Correlations
    - Examples of (Cross) Correlations

## 1. Linear Systems

### Images as Functions

Because images are digital, they are typically represented discretely as a 2D matrix of integer values which capture pixel intensity. Concretely, an image is represented as a function $f: \mathbb{R}^2 \rightarrow \mathbb{R}^M$, wherein $f[x,y]$ gives the intensity of the pixel at position $[x,y]$. For a grayscale image, the pixel intensity is an integer in the range $[0,255]$. For a color image, 

$$f[x,y] = \begin{bmatrix}
    r[x,y] \\
    g[x,y] \\
    b[x,y]
\end{bmatrix}$$ 

such that $r, g, b: [a,b] \times [c,d] \rightarrow [0,255]$.

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

<img src="https://i.imgur.com/GHci7zw.png" width="800"/>

### Overview of Filtering

The term **filtering** refers to the process by which a new image is formed by transforming pixel values of an original image. Filtering serves to extract useful information from images, such as edges, corners, blobs, or to modify and enhance certain image properties (e.g., super-resolution, in-painting, de-noising).

A **system**, denoted by system operator $\mathcal{S}$, converts an input function $f[n,m]$ into an output function $g[n,m]$, where $[n,m]$ are the independent variables (e.g., position in the image): 

$$f[n,m] \rightarrow \boxed{\text{System } \mathcal{S}} \rightarrow g[n,m]$$

Below are several equivalent notations to define a system transforming input $f$ into output $g$:

$$g = \mathcal{S}[f]$$

$$g[n,m] = \mathcal{S}\{f[n,m]\}$$

$$f[n,m] \overset{\mathcal{S}}\rightarrow g[n,m]$$
 
### Moving Average Filter

The moving average filter sets the output pixel value to be the average of its neighboring pixel values. For instance, a $3 \times 3$ moving average filter will set the pixel at position $[n,m]$ of the output image, $g$, to be:

$$g[n,m] = \frac{1}{9} \sum\limits_{k=n-1}^{n+1}\sum\limits_{l=m-1}^{m+1} f[k, l]$$

The outer summation sums over the rows and the inner summation sums over the columns. Using change of variables, this can be simplified to:

$$g[n,m] = \frac{1}{9} \sum\limits_{k=-1}^{1}\sum\limits_{l=-1}^{1} f[n-k, m-l]$$

The moving average filter has the visual effect of slightly blurring the image; it smooths out the image and removes sharp features by averaging those pixels with neighboring values that have smaller intensity.

<img src="https://i.imgur.com/Dp8ZGqc.png" width="800"/>

#### Filter Mask

Another way to think of the moving average filter is in terms applying the following moving average filter "kernel" or "mask" at every neighborhood of the input image, $f$, to compute $g$: 

$$h = \frac{1}{9} \begin{bmatrix}
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

<img src="https://i.imgur.com/oORn4FW.png" width="800"/>

## 2. Properties of Linear Systems


#### Systems can be Classified Based on their Properties

Filters can be quite different in how they are computed and how they affect images. So, how might we best categorize these different systems? There are both amplitude and spatial properties that systems are classified upon.

Some amplitude properties include:
* Additivity
* Homogeneity
* **Superposition**
* Stability
* Invertibility

Spatial properties include:
* Causality
* **Shift Invariance**
* Memory

There are two key categories that we focus on: **Shift Invariance** and **Superposition**.

### Shift-Invariant (SI) Systems

Given that $f[n,m] \xrightarrow[]{\mathcal{S}} g[n,m]$, $\mathcal{S}$ is considered shift-invariant if


$$f[n - n_0, m - m_0] \xrightarrow[]{\mathcal{S}} g[n - n_0,m - m_0]$$

for all input images $f[n,m]$ and shifts $(n_0,m_0)$.

In other words, if we shift the input image $f$ by some amount, then the output image will also be shifted by that same amount. If we know that $f$ produces $g$, then a shift in $f$ should produce a comparable shift in $g$.

#### Is the Moving Average System Shift-Invariant?

We can check this by considering the equation representing this system that we defined in the previous section. We know that $f[n,m] \xrightarrow[]{S} g[n,m]$. We also know that:

$$g[n,m] = \frac{1}{9} \sum_{k = - 1}^{1} \sum_{l = - 1}^{1} f[n-k,m-l]$$

Passing in a version of $f$ shifted by $(n_0, m_0)$ gives us the following result:

$$f[n - n_0,m-m_0] \xrightarrow[]{\mathcal{S}} \frac{1}{9} \sum_{k = - 1}^{1} \sum_{l = - 1}^{1} f[(n-n_0)-k,(m-m_0)-l] = g[n-n_0,m-m_0]$$

Therefore, we can conclude that that the moving average system *is* shift invariant.

### Superposition (Linear Systems)

When we look at linear filtering, our goal is to form a new image whose pixels are a weighted sum of the original pixel values. For our purposes, we use the same set of weights at each point.

So, we define $\mathcal{S}$ to be a **Linear System** if and only if it satisfies the **Superposition Property** as follows:

$$\mathcal{S}\{\alpha f_i[n,m] + \beta f_j[k,l]\} = \alpha \mathcal{S}\{f_i[n,m]\} + \beta \mathcal{S}\{f_h[k,l]\}$$

In other words, passing through a linear combination of two images through $\mathcal{S}$, is equivalent to first passing in each image and then building the linear combination afterwards.

Given this definition, we can conclude that the **Moving Average Filter** is a Linear System because its only operation involves calculating an average value, an operation that is intuitively linear. On the other hand, the **Image Segmentation Thresholding Filter** we discussed earlier is not a Linear System because it is possible that a linear combination of two images could result in pixel values over the threshold while the images' pixel values remain under the threshold when passed through the system individually. 

### Linear Shift-Invariant (LSI) Systems

Combining these two concepts, we define a Linear Shift-Invariant (LSI) System as a system that is both **Shift Invariant** and follows the **Superposition Property**. These Systems have a number of special properties, but most relevantly and critically, an LSI System is _completely specified by its impulse response_. The following section will discuss exactly what this statement means and how it is relevant in understanding how LSI Systems interact with images.

## 3. LSI Systems and Convolution

### Impulse Functions and Responses
To fully understand the relevance of an impulse response to an LSI System, we must first understand the impulse function behind it. In the field of Image Processing, 2D Impulse functions represent a very simple discrete signal in which one pixel holds a value of 1 while the others remain 0, and these functions are used to determine the output of a given system as will be discussed below.

The most common example of a discrete impulse function can be found in the 2D Delta ($\delta_2$) impulse function. This impulse function can be described by the piecewise function defined below, in which $\delta_2[0,0]$ exists at the center of the shown matrix.

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

When an impulse function is passed into a system, we get an impulse response as output, a critical piece in the calculation of convolution. Impulse responses are often referred to as **kernels** or **filters**. 

$$\delta_2[n,m] \rightarrow \boxed{\text{System } \mathcal{S}} \rightarrow h[n,m]$$

This conversion from impulse function to impulse response can be better understood using the Moving Average Filter example introduced previously:

Consider the 2D Delta ($\delta_2$) impulse function defined above. Here, we would like to find the impulse response corresponding to that impulse function. Recall that the expression used to represent the Moving Average Filter is as follows:

$$g[n,m] = \frac{1}{9} \sum\limits_{k=-1}^{1}\sum\limits_{l=-1}^{1} f[n-k, m-l]$$

We can repurpose this equation, replacing the input image $f$ with the $\delta_2$ function to generate an expression for the impulse response $h$ corresponding to the Moving Average Filter:

$$\delta_2[n,m] \overset{\mathcal{S}}\rightarrow h[n,m] = \frac{1}{9} \sum\limits_{k=-1}^{1}\sum\limits_{l=-1}^{1} \delta_2[n-k, m-l]$$

Running this function for each pixel in the input image and its surrounding neighbors, we get an impulse response $h$ of the following form:

$$\begin{bmatrix}
    0&0&0&0&0 \\
    0& \frac{1}{9} & \frac{1}{9} & \frac{1}{9}&0 \\
    0 & \frac{1}{9} & \frac{1}{9} & \frac{1}{9} & 0\\
       0 & \frac{1}{9} & \frac{1}{9} & \frac{1}{9}&0  \\
         0& 0& 0 &0 & 0
\end{bmatrix}$$

If this looks familiar, it is because this impulse response output $h$ is identical to the Moving Average filter discussed previously. Therefore, passing the $\delta_2$ impulse function into this LSI System produces a Moving Average filter that can be used for image smoothing.

As stated earlier, _an LSI System can be completely specified by its impulse response_. This means that the impulse response produced by passing some impulse function through an LSI system can be used to predict what a system's output image will look like. This statement also indicates that, given an input image and an impulse response for an LSI System, we can actually use this information to generate the corresponding output image.

### 2D Convolution with LSI Systems

Convolution is an operation that describes the output of an LSI System, a specific type of system with properties as discussed previously. Generally, convolution is an operation that involves transforming an input image by applying a defined kernel to each pixel in order to produce a resulting transformed output image. 2D Convolution is considered one of the most critical and most frequently used operations in image processing and computer vision.

We've discussed how the concept of a kernel might be used to apply a filter to an image using the _Moving Average Filter_ example, but now that we understand the concept of impulse functions and impulse responses, we can generalize this process to an arbitrary impulse response $h$.

We know that LSI Systems satisfy the **Superposition Property** and the **Shift-Invariance Property**. Additionally, we have just discussed that:

$$\delta_2[n,m] \overset{\mathcal{S}}\rightarrow h[n,m]$$

Using these three properties, we can derive an equation that describes an output image $g$ produced by an LSI system solely in terms of an input image $f$ and a kernel $h$. 

In order to do so, we must first imagine the input image $f$ as a sum of impulses or shifted $\delta_2$ functions. Let $f[n,m]$ represent a $3\times 3$ input image, and let $\delta_2[n,m]$ represent our delta impulse function as described previously. As you can see below, for each pixel in our image, we produce a new shifted $\delta_2$ function multiplied the initial pixels value.

<img src="https://i.imgur.com/QQuILc2.png" width="800"/>


$$f[n,m] = f[0,0] \cdot\delta_2[n,m] + f[0,1] \cdot\delta_2[n,m-1] + ... + f[2,2] \cdot\delta_2[n-2,m-2]$$

$$f[n,m] = \sum_{k=-\infty}^\infty\sum_{l=-\infty}^\infty f[k,l]\cdot \delta_2[n-k,m-l]$$


Now that we have described the input image as a sum of scaled and shifted $\delta_2$ functions, we can use this new representation for our image along with the previously defined properties to derive an equation for the output image in terms of $f[n,m]$ and $h[n,m]$.

First, we know the following to be true:

$$f[n,m] \overset{\mathcal{S}}\rightarrow g[n,m]\\
f[n,m] =\sum_{k=-\infty}^\infty\sum_{l=-\infty}^\infty f[k,l]\cdot \delta_2[n-k,m-l]$$

Therefore we can conclude that:

$$g[n,m]=\mathcal{S}\{\sum_{k=-\infty}^\infty\sum_{l=-\infty}^\infty f[k,l]\cdot \delta_2[n-k,m-l]\}$$

Next, by applying the Superposition Property, in which we treat the $f[k,l]$ terms as constants and only apply the System $\mathcal{S}$ to the shifted $\delta_2$ functions, we get the following:

$$g[n,m]=\sum_{k=-\infty}^\infty\sum_{l=-\infty}^\infty f[k,l]\cdot \mathcal{S}\{\delta_2[n-k,m-l]\}$$

Finally, we use our knowledge that $\delta_2[n,m] \overset{\mathcal{S}}\rightarrow h[n,m]$, and we apply the Shift-Invariance Property, which allows us to write the equation as follows:

$$f[n,m] * h[n,m]= (f*h)[n,m] = g[n,m]=\sum_{k=-\infty}^\infty\sum_{l=-\infty}^\infty f[k,l]\cdot h[n-k,m-l]$$

This operation is referred to as **2D discrete convolution** and can also be represented by $f[n,m] * h[n,m]$ or $(f * h)[m,n]$. In the case of the equation above, we can say that the function $f$ is being convolved with the kernel $h$.


## 4. Convolution and Correlation

### 2D Discrete Convolution
In computer vision, convolution is often performed 2D discrete signal, especially in image processing applications. The equation of 2D discrete convolution as derived above is defined as:

$$f[n,m] * h[n,m] = \sum\limits_{k=-\infty}^{\infty} \sum\limits_{l=-\infty}^{\infty} f[k, l] h[n-k, m-l]$$

The reasoning behind this equation can also be explained graphically. When we want to compute the convolution at a given location $(n, m)$, we need the input image $f$ as well as the impulse response $h$, the matrix we've previously referred to as a _kernel_.

<img src="https://i.imgur.com/sdWgUmz.png" width="800"/>

The following is the algorithm of computing convolution for 2D discrete functions like images:

- To compute convolution, first, we fold the kernel $h[k, l]$ such that the indexes become inverted as $h[-k, -l]$. In other words, we flip the kernel both vertically and horizontally to form $h[-k, -l]$.

<img src="https://i.imgur.com/Pd3JlWg.png" width="800"/>

- Next, we shift the folded kernel by $(n, m)$ to obtain $h[n-k, m-l]$. This is done so that the folded kernel now overlaps the pixel at $(n,m)$ and its surrounding neighbors. This will allow us to easily perform computations for convolution in the following steps. Please note that all the values outside our $3\times 3$ kernel are zero.

<img src="https://i.imgur.com/TPjmLYV.png" width="800"/>


- After that, we can multiply $h[n-k, m-l]$ by $f[k, l]$, which is overlapping the kernel and original input and then performing element-wise multiplication between the original input and the kernel in the cells within the neighborhood. This essentially allows us to compute a weighted sum of the pixel intensities within the neighborhood based on the values defined in the kernel $h$.

<img src="https://i.imgur.com/zfgCLrg.png" width="800"/>

- Finally, we sum up the results to obtain the output value at $(n, m)$. In order to fully convolve a given input image and build an output image, we must run this convolution process on every pixel $(n,m)$ in the input.

### Examples of Simple Convolutions
Now, let's look at a few basic examples of convolving a kernel with an actual image. 

Consider the effect of the following kernel on an image:

![](https://imgur.com/m0Kva3U.png)

There are a few things of note here. First, this kernel is perfectly symmetrical, which means that folding has no effect on it. Also, every coeffecient except for the center value is 0. Due to this, peforming an elementwise multiplication will return the value at the center pixel. As a result, this particular kernel is a sort of **Identity Kernel** that when applied to an image, will simply return the image itself.

For another example, consider the following kernel:

![](https://imgur.com/301khsT.png)

We first fold the kernel, relocating the 1 to the left and then perform an elementwise mutliplication. As we can see, this outputs the leftmost pixel for a given location. As a result, when applying the kernel to an image, the returned image will be shifted one pixel to the right.

Convolution also allows us to perform other effects, such as sharpening. This is the combination of kernels to sharpen an image:

![](https://imgur.com/JHzE0Zr.png)

First, we can break down the first kernel into a sum of two identity kernels, like so:

![](https://imgur.com/xZYpFJG.png)

The function of the last kernel is to blur an image. Together, the last 2 kernels take the original image (Identity Kernel) and remove the blur, leaving only the details. Then, those details are added to the original image again, sharpening it.


### Implementation of Convolutions
Computers will only convolve finite support signals. That is to say that any $(n, m)$ outside of some rectangular area are zero. In a practical sense, this just means we can't convolve images with infinite pixels in any diretion. NumPy's convolution functions provide support for this 2D Discrete Convolution of finite support signals (normal images).

Given an image with dimensions $N_1 \times M_1$ convolved with a kernel with dimensions $N_2 \times M_2$ where all dimensions are finite, the output of the convolution of the given input image and kernel would have dimensions of $(N_1 + N_2 - 1) \times (M_1 + M_2 - 1)$.

<img src="https://i.imgur.com/ese8Owm.webp?maxwidth=760&fidelity=grand" width="800"/>

To determine these dimensions of $(N_1 + N_2 - 1) \times (M_1 + M_2 - 1)$, we can imagine the worst edge case in which there is only a 1 pixel overlap between that image and the kernel for the convolution as shown in the image below. 

![](https://i.imgur.com/1D5r1gB.png)

Another such bounding edge case would be when the edges of the images themselves overlap as shown below.

![](https://i.imgur.com/ea1cNBv.png)

When we convolve around the edges of an image, the kernel invariably goes outside of the range of the image. To address this edge case, there are a variety of strategies that can be employed:
* **Zero "Padding"**: All values outside of the image/rectangular area are assumed to be zero.
* **Edge Value Replication**: Replicates the pixels closes to the edges of an image into the area outside of the image, “extending” the image.
* **Mirror Extension**: Similar to edge value replication, however, instead of replicating the edge pixels of the image, we copy reflections of the image; that is, for the values farther away from the image, values from further inside the image are replicated.

The specific padding technique selected for a given operation can change based on the specific application. Although **Zero Padding** is used most often, there are cases in which we would not want to use this padding technique. For example, in a situation in which we expect the background of the input image to be a certain, solid color, **Edge Value Replication** would be a better choice because it could replicate the current background.

### (Cross) Correlations

While **convolutions** express the amount of overlap of one function as it is shifted over another function, **correlations** compare the similarity of two sets of data, computing a measure of similarity of two input signals as they are shifted by one another. While convolution is a filtering operation, correlation is a measure of similarity.

(Cross) Correlations are denoted by the symbol $**$, and the cross-correlation of two 2D signals $f[n,m]$ and $h[n,m]$ is given by the equation:

$$f[n,m] ** h[n,m] = \sum_{k=-\infty}^{\infty} \sum_{l=-\infty}^{\infty} f[k,l] \cdot h[n+k,m+l]$$

As seen from the equation provided above, the equation for correlations is equivalent to that of convolutions except the $h[n-k,m-l]$ term is replaced with a $h[n+k,m+l]$ term. Conceptually, this means that correlations are equivalent to convolutions without the horizontal and vertical flipping of the kernel. However, due to the flipping, convolutions and correlations _**DO NOT**_ produce the same result.

<img src="https://imgur.com/3qS4Dw8.png" width="800"/>

The Convolution of $f * g$ and the Correlation of $f ** g$ are equivalent only when $g$ is symmetrical, as $g$ would remain unchanged after folding. 

### Examples of (Cross) Correlations

Let's imagine that there is an image $f$ that is our target image. 

![](https://i.imgur.com/lP8LuIx.png)

Now let us imagine that there is an image $g$ that equivalent to the intial image $f$ but with the addition of noise. 

![](https://i.imgur.com/0o9uQ9l.png)

Through evaluating image $g$, we wish to find the locations of the black squares in image $f$.

One method to do this would be to implement a threshold on the intensity values. However, there is too much noise present in $g$ that such a method is unviable.

![](https://i.imgur.com/mI2tkCU.png)

However, a more effective method for locating the black squares in the image would be to use correlation. Here, we can define a kernel $h$ that holds the pattern we aim to detect, a black square. By computing the correlation between the noisy image $g$ and the black square kernel $h$, we get the resulting image shown below in which the areas most similar to the pattern $h$ are the darkest.

![](https://imgur.com/SjzkuRj.png)

Now, applying a threshold to the output image $r$, we are given a closer approximation of where the black squares were originally.

![](https://imgur.com/tEjF4nc.png)

For a real life example, let us take the case of stereo vision, triangulating a position to estimate the depth of a point in a given scene. 

Imagine we have two optical sensors that are capturing a scene from slightly different points of view. To find the depth of a point in the scene, we must first know the location of the given point in both images. One way to achieve this is to take a position from one image and find a correlation in the second image, moving along a fixed axis, and the area where the correlation is the largest is the area with the greatest similarity.

<img src="https://imgur.com/Snaa68Q.png" width="800"/>
