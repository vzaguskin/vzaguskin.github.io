---
layout: post
title: Histogram matching(согласование гистограмм) на python
intname: histmatching1
permalink: histmatching1_ru
---

Навеяло [вот этим вопросом](http://stackoverflow.com/questions/30284137/how-to-extract-color-shade-from-a-given-sample-image-to-convert-another-image-us) на стэковерфлоу.
Вопрошающий интересуется, как сделать цвета картинки похожими на цвета другой картинки. В его примере, вот исходная картинка:

![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/source.jpg?raw=true)

Он хочет взять оттенок из вот этой картинки:

![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/tint_target.jpg?raw=true)

и применить его к исходной.

Предлагаемый алгоритм согласования гистограмм дает на этом примере следующий результат:

до: ![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/source.jpg?raw=true)
после: ![](https://github.com/vzaguskin/sampleprojects/blob/master/hist_norm/histnormresult.jpg?raw=true)

Применяемый алгоритм [согласования гистограмм](http://en.wikipedia.org/wiki/Histogram_matching) по сути состоит из применения
алгоритма [нормализации гистограммы](http://en.wikipedia.org/wiki/Histogram_equalization) к обеим картинкам
после чего можно построить функцию преобразования значения цвета из двух полученных функций преобразования.

За основу возьмем реализацию метода нормализации гистограммы от [Jan Erik Solem](http://www.janeriksolem.net/2009/06/histogram-equalization-with-python-and.html)


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

И сделаем из нее функцию согласования, работающую как с одноканальными, так и с многоканальными изображениями:


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
