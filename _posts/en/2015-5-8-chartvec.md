---
layout: post
title: Vectorizing a chart(python, skimage)
intname: chartvec1
permalink: chartvec1
---

Here is an image with a function chart where squares give values at specific points and they are connected with  straight lines. We want to get the coordinates of vertices and connections between them and draw a vectorized image of that chart.  


![](https://github.com/vzaguskin/sampleprojects/blob/master/chart%20vectorisation/laGK6.jpg?raw=true)  

Let's start with the color segmentation.


```python

    import numpy as np
    from scipy.misc import imshow, imsave, imread
    from scipy.ndimage import filters, morphology, measurements
    from skimage.draw import line, circle
    
    img = imread("laGK6.jpg")
    
    r = img[:,:, 0]
    g = img[:,:, 1]
    b = img[:,:, 2]
    
    mask = (r.astype(np.float)-np.maximum(g,b) ) > 20


```

Then we apply image morphology to remove lines, leaving only vertices. We do that by performing several binary erosions to the mask followed by a binary dilation to restore full connectivity. 

```python

    mask2 = morphology.binary_erosion(mask)
    mask2 = morphology.binary_erosion(mask2)
    mask2 = morphology.binary_erosion(mask2)
    mask2 = morphology.binary_erosion(mask2)
    
    mask2 = morphology.binary_dilation(mask2)

```

Then we  can enumarate the vertices and get their centers of mass.

```python

    label, numvertices = measurements.label(mask2)
    mc = measurements.center_of_mass(mask2, label, range(1,numvertices+1) )
    
```

To find which of the vertices are connected we use a technique similar to [Hough Lines](http://en.wikipedia.org/wiki/Hough_transform). We calculate the coordinates of the points that would lay on the connecting lines and check how much of them are actually present in our mask. For the existing line their share would be close to 100% while for the missing lines it would be much smaller.  

```python

    arr = range(numvertices)
    
    connections=[]
    for i in range( numvertices):
        arr.remove(i)
        for j in arr:
            rr,cc = line(mc[i][0], mc[i][1], mc[j][0], mc[j][1])
            ms = np.sum(mask[rr,cc]).astype(np.float)/len(rr)
            if ms > 0.9:
                 connections.append((i,j))
    
    
    print "vertices: ", np.array(mc).astype(np.uint32)
    print "connections: ", connections

```

Here is what we get:

> vertices:  [[ 76 288]
>  [ 76 613]
>  [138 126]
>  [139 450]
>  [265 207]
>  [264 369]
>  [265 694]
>  [265  45]
>  [327 532]]
> 
> 
> connections:  [(0, 4), (0, 5), (1, 6), (1, 8), (2, 4), (2, 7), (3, 5), (3, 8)]


Finally, we can draw a new picture based on that data and make sure it matches the original.

```python

    mask3 = np.zeros(mask2.shape, dtype=np.uint8)
    mask3[:]=255

    for p in mc:
        rr, cc = circle(p[0], p[1], 5)
        #mask3[p[0], p[1]]=255
        mask3[rr,cc]=20
    
    for cn in connections:
        i, j = cn
        rr,cc = line(mc[i][0], mc[i][1], mc[j][0], mc[j][1])
        mask3[rr,cc]=12

```

![](https://github.com/vzaguskin/sampleprojects/blob/master/chart%20vectorisation/vectorized.jpg?raw=true)


The complete source of the sample is [here](https://github.com/vzaguskin/sampleprojects/tree/master/chart%20vectorisation).