---
layout: post
title: Removing lines from ruled paper with Fourier transform.
intname: fourielines1
permalink: fourielines1
---

Here is an example of a ruled paper scan with digits we want to recognize. We would like to remove the rules but keep the digits. Luckily it is possible to do so. 

![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/digitchar.jpeg?raw=true)

The good news is that the rules are equally spaced so in the frequency domain they would look like a very bright line.

![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/visual_re1.jpg?raw=true)

In our case it is almost vertical, but can have some angle if the scanned image is slightly rotated. We want to detect the line and paint it out in the frequency domain and return back to space domain to get this result:

![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/result1.jpg?raw=true)

Or even this after more filtering:
![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/result_otsu1.jpg?raw=true)

Here is the code. Import everything we need:


```python

import numpy as np
from scipy.misc import imshow, imsave, imread, imresize
from scipy.ndimage import filters, morphology
from skimage import filter
from skimage.draw import line
from scipy import ndimage, spatial

```


helper function HoughLines, returning the brightest line crossing the center as a binary mask:


```python

def houghLines(img):
    w,h = img.shape
    acc=[]
    for i in range(h):
        rr,cc = line(0, i, w-1, h-i-1)
        acc.append(np.sum(img[rr, cc]))
    mi = np.argmax(acc)
    ret = np.zeros(img.shape, dtype=np.bool)
    rr,cc = line(0, mi, w-1, h-mi-1)
    ret[rr,cc]=True
    return ret

```


And the main function:



```python

def removeLines(img):
	imfft = np.fft.fft2(imggray)
	imffts = np.fft.fftshift(imfft)
    
    mags = np.abs(imffts)
    angles = np.angle(imffts)
    
    visual = np.log(mags)

    
    visual3 = np.abs(visual.astype(np.int16) - np.mean(visual))
    
    ret = houghLines(visual3)
    ret = morphology.binary_dilation(ret )
    ret = morphology.binary_dilation(ret )
    ret = morphology.binary_dilation(ret )
    ret = morphology.binary_dilation(ret )
    ret = morphology.binary_dilation(ret )
    w,h=ret.shape
    ret[w/2-3:w/2+3, h/2-3:h/2+3]=False
    
    
    delta = np.mean(visual[ret]) - np.mean(visual)
    
    
    visual_blured = ndimage.gaussian_filter(visual, sigma=5)
    
    	
    
    visual[ret] =visual_blured[ret]
    
    
    newmagsshift = np.exp(visual)
    
    newffts = newmagsshift * np.exp(1j*angles)
    
    newfft = np.fft.ifftshift(newffts)
    
    imrev = np.fft.ifft2(newfft)
    
    newim2 =  np.abs(imrev).astype(np.uint8)
    
    
    newim2 = np.maximum(newim2, img)
    
    return newim2

```