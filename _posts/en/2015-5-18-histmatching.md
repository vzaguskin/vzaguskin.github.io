---
layout: post
title: Histogram matching in python
intname: histmatching1
permalink: histmatching1
---

Inspired by [ths question](http://stackoverflow.com/questions/30284137/how-to-extract-color-shade-from-a-given-sample-image-to-convert-another-image-us) on stackoverflow.

The author wants to make the colors in one picture similar to the colors in another. Here is his source picture:

![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/source.jpg?raw=true)

He wants to get the tint from that picture:

![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/tint_target.jpg?raw=true)

and apply it to the source.

The algorithm I suggest gives the following result:

before: ![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/source.jpg?raw=true)
after: ![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/histnormresult.jpg?raw=true)

The algorithm is called [histogram matching](http://en.wikipedia.org/wiki/Histogram_matching) and essentially means applying [histogram equalization](http://en.wikipedia.org/wiki/Histogram_equalization) to both pictures, and then creating the pixel value translation function from the two equalization functions.

Let's use the code from  [Jan Erik Solem](http://www.janeriksolem.net/2009/06/histogram-equalization-with-python-and.html) as a base.


```python
   def histeq(im,nbr_bins=256):

   #get image histogram
   imhist,bins = histogram(im.flatten(),nbr_bins,normed=True)
   cdf = imhist.cumsum() #cumulative distribution function
   cdf = 255 * cdf / cdf[-1] #normalize

   #use linear interpolation of cdf to find new pixel values
   im2 = interp(im.flatten(),bins[:-1],cdf)

   return im2.reshape(im.shape), cdf


```

And modify it so it becomes a histogram matching function working with both single-channel and multichannel(like RGB) images:


```python

from scipy.misc import imsave, imread
import numpy as np

imsrc = imread("source.jpg")
imtint = imread("tint_target.jpg")

nbr_bins=255
if len(imsrc.shape) < 3:
    imsrc = imsrc[:,:,np.newaxis]
    imtint = imtint[:,:,np.newaxis]

imres = imsrc.copy()
for d in range(imsrc.shape[2]):
    imhist,bins = np.histogram(imsrc[:,:,d].flatten(),nbr_bins,normed=True)
    tinthist,bins = np.histogram(imtint[:,:,d].flatten(),nbr_bins,normed=True)

    cdfsrc = imhist.cumsum() #cumulative distribution function
    cdfsrc = (255 * cdfsrc / cdfsrc[-1]).astype(np.uint8) #normalize

    cdftint = tinthist.cumsum() #cumulative distribution function
    cdftint = (255 * cdftint / cdftint[-1]).astype(np.uint8) #normalize


    im2 = np.interp(imsrc[:,:,d].flatten(),bins[:-1],cdfsrc)



    im3 = np.interp(im2,cdftint, bins[:-1])

    imres[:,:,d] = im3.reshape((imsrc.shape[0],imsrc.shape[1] ))

try:
    imsave("histnormresult.jpg", imres)
except:
    imsave("histnormresult.jpg", imres.reshape((imsrc.shape[0],imsrc.shape[1] )))


```
