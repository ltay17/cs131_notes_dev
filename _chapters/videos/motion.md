---
title: Optical Flow
keywords:
order: 0
---

- [7.1 Optical Flow](#7.1-optical-flow)
- [7.2 Lukas-Kanade Method](#7.2-lukas-kanade method)
- [7.3 Pyramids for Large Motion](#7.3-pyramids-for-large-motion)
	- [7.3.1 Previous Assumptions and Large Motion](#7.3.1-previous-assumptions-and-large-motion) 
	- [7.3.2 Optical Flow Estimation with Pyramids](#7.3.2-optical-flow-estimation-with-pyramids) 
	- [7.3.3 Optical Flow Results](#7.3.3-optical-flow-results) 
- [7.4 Horn-Schunk Method](#7.4-horn-schunk-method)
- [7.5 Applications](#7.5-applications)

[//]: # (This is how you can make a comment that won't appear in the web page! It might be visible on some machines/browsers so use this only for development.)

[//]: # (Notice in the table of contents that [First Big Topic] matches #first-big-topic, except for all lowercase and spaces are replaced with dashes. This is important so that the table of contents links properly to the sections)

[//]: # (Leave this line here, but you can replace the name field with anything! It's used in the HTML structure of the page but isn't visible to users)
<a name='Topic 1'></a>

## 7.0 LEAVE THIS UNTIL WE ARE DONE so everyone has instructions
	
Here you can start to talk about the first topic of your notes. You can bold text like **this**, or italicize text like *this*. If you want to make a numbered list it's as easy as
1.  
2. 
3. 

- Bullet
- points
- are
- similar 

For a more detailed cheatsheet on the most important functionality of Markdown, check out this link https://wordpress.com/support/markdown-quick-reference/, which you can format in Markdown with your own [link title](https://wordpress.com/support/markdown-quick-reference/)


<a name='Subtopic 1-1'></a>
### Subtopic 1-1
You might want to include images in your notes, since Computer Vision as a field is blessed with tons of cool visualizations. Here's an example from the CS 231N notes page we included as a reference for you:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/examples/classify.png">
  <div class="figcaption">Put your informative caption here! If you really want to mess around with the classes in this div container then feel free, but inserting images just like this should work great!</div>
</div>

<a name='Subtopic 1-2'></a>
### Subtopic 1-2
Sometimes you might want to insert some code snippets into your notes. As an example, here's a snippet of python code taken from the CS 231N notes:
```python
Xtr, Ytr, Xte, Yte = load_CIFAR10('data/cifar10/') # a magic function we provide
# flatten out all images to be one-dimensional
Xtr_rows = Xtr.reshape(Xtr.shape[0], 32 * 32 * 3) # Xtr_rows becomes 50000 x 3072
Xte_rows = Xte.reshape(Xte.shape[0], 32 * 32 * 3) # Xte_rows becomes 10000 x 3072
```

<a name='Subtopic 1-3'></a>
### Subtopic 1-3
Sometimes you might want to write some mathematical equations, and LaTeX is a great tool for that! You can write an inline equation like this \\( a^2 = b^2 \\), or you can display an equation on its own line like this! \\[ a^2 = b^2 + c^2 \\]

You can also apply LaTeX syntax to label your equations and refer to them later! Here's the equation:

$$ \begin{equation} \label{your_label} a^2 = b^2 + c^2 + d^2 + e^2 \end{equation} $$

and here's a linked reference to it: \eqref{your_label}. For now, this configuration likes the \\"\\$\\$ equation stuff ... \\$\\$\\" syntax to have an empty line above and below it, but it displays the same anyway.

**For a guide on LaTeX syntax and how to write mathematical equations and formulas with it, check out [this link](https://www.overleaf.com/learn/latex/mathematical_expressions)** 

**Here's a short guide on how to use the basics of LaTeX**
- You've seen above the syntax to start and end an equation, so now let's work on what you fill in the middle
- You can make variables and expressions **bold** in equations too: \\(\mathbf{x} + y\\)
- Superscripts and subscripts are easy: use the ^ and _ symbol and bound your super/sub script by {} if it's more than one character. For example: \\(e^{-x+10}\\)
- Greek letters are also simple, use the \ character with their written name with optional capitalization, such as alpha or Alpha. For example: \\(\alpha + \beta + \gamma + \delta + \Gamma + \Delta\\). Not all capital greek letters work like this, but you can search online for solutions if this trick fails or reach out to the CA's. In general the \ character in LaTeX is the gateway to all kinds of special characters and functionalities.
- Sums and Products are really useful in Latex. You can use both superscripts and subscripts to mark the bounds: \\(\log(\prod_{i=0}^{2n}i^2) = \sum_{i=0}^{2n}\log (i^2)\\)
- Another useful trick is to write out a matrix or a vector in LaTex. There's a lot of customization you can do with this, so check out this [page](https://www.overleaf.com/learn/latex/Matrices) for more details. Here's some examples in our Markdown environment: 


$$\begin{bmatrix}
1 & 2 & 3\\
a & b & c
\end{bmatrix}$$

$$\begin{bmatrix}
1\\
2\\
3\\
\end{bmatrix}$$

$$\begin{bmatrix}
1 & 2 & 3\\
\end{bmatrix}$$

As with the labelled equations, it makes a difference whether the lines above and below the equation are blank, so keep that in mind while debugging! 

## 7.1 Optical Flow
This should give you the primary tools to develop your notes. Check out the [markdown quick reference](https://wordpress.com/support/markdown-quick-reference/) for any further Markdown functionality that you may find useful, and reach out to the teaching team on Piazza if you have any questions about how to create your lecture notes

## 7.2 Lukas-Kanade Method

### 7.2.1 Motivation for Lucas-Kanade

From 7.1, we were left with some ambiguity: we have 2 unknowns (u, v) at every pixel in the image, where u(x, y) represents x-direction motion and v(x, y) represents y-direction motion. Specifically, we have the equation:

$$ \nabla{I} \cdot 
\begin{bmatrix} 
u\\ 
v\\ 
\end{bmatrix} 
+ I_t = 0 $$

Where \\( \nabla{I} \\) is the intensity gradient of the image with respect to x and y, and \\( I_t \\) is the intensity gradient with respect to t, or the "temporal derivative." As you can see, we are underconstrained because we have one equation and two unknowns (the gradients are scalar).

To uncover more constraint equations, we introduce the **spatial coherence contraint,** which assumes that pixel neighbors have the same optical flow value (u, v). This lets us redefine our previous equation as:

$$ \nabla{I(p_{i})} \cdot 
\begin{bmatrix} 
u\\ 
v\\ 
\end{bmatrix} 
+ I_{t}(p_i) = 0 $$

Where \\( \nabla{I(p_{i})} \\) is the intensity gradient at pixel i and \\( I_{t}(p_i) \\) is the gradient of I with respect to t at pixel i.

Now we have an **overconstraining** problem, because if we use a window of size m x m to evaluate intensity, we now have m * m equations for only 2 unknowns. For example, given a 5 x 5 window, we have 25 equations for 2 unknowns, and our sequence of equations is:

$$ 
\begin{bmatrix} 
I_{x}(p_1) & I_{y}(p_1)\\ 
... & ...\\
I_{x}(p_{25}) & I_{y}(p_{25})\\
\end{bmatrix}
\cdot
\begin{bmatrix} 
u\\ 
v\\ 
\end{bmatrix}
=
-\begin{bmatrix}
I_{t}(p_1)\\
...\\
I_{t}(p_{25})\\
\end{bmatrix}
$$

In 7.2.2, we will resolve this overconstraint issue.

### 7.2.2 Lucas-Kanade Flow

In 7.2.1, we showed an example where we were overconstrained with 25 equations solving 2 unknowns. To resolve this issue, we will use least-squares to solve for the vector \\(\begin{bmatrix} u\\ v\\ \end{bmatrix} \\).

Referring to our previous system of equations of form \\( A \cdot d = b \\), we will instead use the least-squares form \\( (A^{T}A)d = A^{T}b \\). This new system of equations is equivalent to:

$$
\begin{bmatrix}
\Sigma I_xI_x & \Sigma I_xI_y\\
\Sigma I_xI_y & \Sigma I_yI_y\\
\end{bmatrix} 
\cdot 
\begin{bmatrix}
u\\
v\\
\end{bmatrix} =
-\begin{bmatrix}
\Sigma I_xI_t\\
\Sigma I_yI_t\\
\end{bmatrix}
$$

The dimensions of resulting matrix are (2 x 2) x (2 x 1) = 2 x 1, exactly what we wanted.

Some requirements for implementing Lucas-Kanade:

- \\( A^TA \\) should be invertible
- \\( A^TA \\) should be large enough to minimize noise (e.g. eigenvalues \\( \lambda_1 \\) and \\( \lambda_2 \\) should be not too small
- \\( A^TA \\) should be well-conditioned (e.g. \\( \lambda_1/ \lambda_2 \\) not too large for \\( \lambda_1 > \lambda_2 \\)

What does this matrix \\( A^TA \\) remind you of?

That's right, the second moment matrix M from the Harris corner detector! Both deal with eigenvalues and a 2 x 2 matrix of summations of gradients of an image.

Note that the eigenvectors and eigenvalues of \\( A^TA \\) determine the edge direction and magnitude:

- The eigenvector with the larger eigenvalue contains the direction of fastest intensity change
- The other eigenvector is perpendicular to the first

### 7.2.3 Interpreting the Eigenvalues

As noted in 7.2.2, eigenvalues should have a certain size and proportion to each other in order to provide reliable corner detection. The following image outlines the effects of each eigenvalue scenario and their pitfalls.

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/videos/picture_7.3/LK_eig_interpretation.png">
</div>

1. **Edges:** Around edges, there is an aperture problem. We can measure optical flow in the direction perpendicular to the edge, but along the edge it is hard to discern change.
2. **Flats:** Flat areas are regions of low texture. These are challenging because of their homogeneity and are not easily distinguishable.
3. **Corners:** Corners are the regions we would like to capture in our Lucas-Kanade study. These provide relatively large eigenvalues for each eigenvector, and are relatively distinguishable features.

### 7.2.4 Improving Accuracy

In this section, we will evaluate areas for accuracy improvement.

Recall the small motion assumption from 7.1:

$$
0 = I(x + u, y + v) - I_t(x, y) \approx I(x, y) + I_xu + I_yv - I_t(x, y)
$$

This is an approximation, but if we add back higher order terms, we can achieve even greater accuracy:

\\( I(x, y) + I_xu + I_yv + \\) higher order terms \\( - I_t(x, y)\\)

The process for finding these higher terms is a polynomial root finding problem. This can be approached in a few ways:

1. Using Newton's method (out of the scope of this class)
2. More iterations of Lucas-Kanade (increases computational complexity)

**Iterative Refinement:**

Another approach to improving accuracy:

1. Estimate the velocity at each pixel by solving the Lucas-Kanade equations (find (u, v))
2. Warp the previous image (I(t - 1)) toward the next image (I(t)) using the estimated flow field found in Step 1. This creates a predicted next frame based on (u, v).
3. Repeat until convergence (this handles residuals not captured in Step 2)

**Potential Sources of Error in Lucas-Kanade:**

Recall that in order to perform Lucas-Kanade, we made a few assumptions:

- Assumed \\( A^TA \\) is easily invertible
- Assumed there is not much noise (eigenvalues of \\( A^TA \\) not too small)

When these assumptions are violated,

- Our small motion assumption is invalid (see 7.1 and 7.2.4)
- Brightness constancy is not satisfied (can't rely on intensity values)
- A given pixel doesn't necessarily move like its neighbors (window size too large)

## 7.3 Pyramids for Large Motion
### 7.3.1 Previous Assumptions and Large Motion
We made some key assumptions in the previous section about Lukas-Kanade Method, such as: 

**Small motion** - points do not move very far

**Brightness constancy** - projection of the same point is the same in every frame

**Spacial coherence** - points move like their neighbors

However we still need to consider cases with larger motion, more than one pixel. One potential idea that can help us is reducing resolution of the video, such that motion size is one pixel. 
<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/videos/picture_7.3/IMG_3774.jpg">
</div>
Using that we can detect optical flow at the lower resolution image, and use that to build up back to higher resolution video. 

### 7.3.2 Optical Flow Estimation with Pyramids
We want to calculate optical flow in a coarse-to-fine approach. We first calculate the pyramid of the image, by converting image in lower resolution versions in multiple layers. As a result, motion that was 10 pixels in the high-resolution image, results in motion of 1.25 pixels in a lower resolution version. 
<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/videos/picture_7.3/IMG_BECD9E44C44F-1.jpeg">
</div>

We run Lukas-Kanade on the low resolution image, because we can do it as now we satisfy small motion requirment. We use solution of that algorithm to upsample to the next layer with a little higher resolution. There we use L-K again but to detect residual movements that were not detected as a result of resolution decrease. We keep repeating that process until we get back to the original high resolution layer. 
<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/videos/picture_7.3/IMG_AEBE3D9581A9-1.jpeg">
</div>

### 7.3.3 Optical Flow Results
We can see what happens when we don't use pyramids. Vectors in areas of large motion are very innacurate and fail to represent reality. 
<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/videos/picture_7.3/IMG_78D6637B7B4A-1.jpeg">
</div>
Whereas on the picture where we use pyramids, we can see that vectors are properly alligned and calculated. 
<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/videos/picture_7.3/IMG_8F39FB500A2B-1.jpeg">
</div>

## 7.4 Horn-Schunk Method
This should give you the primary tools to develop your notes. Check out the [markdown quick reference](https://wordpress.com/support/markdown-quick-reference/) for any further Markdown functionality that you may find useful, and reach out to the teaching team on Piazza if you have any questions about how to create your lecture notes

## 7.5 Applications
This should give you the primary tools to develop your notes. Check out the [markdown quick reference](https://wordpress.com/support/markdown-quick-reference/) for any further Markdown functionality that you may find useful, and reach out to the teaching team on Piazza if you have any questions about how to create your lecture notes
