---
layout: post
title: Очищаем скан листа от линеек с помощью фурье-преобразования.
intname: fourielines1
permalink: fourielines1_ru
---

На рисунке ниже пример скана тетрадного листа в линейку с цифрами, которые мы хотим распознать. Линейки нам довольно сильно мешают, особенно когда перечеркивают цифры. Хотелось бы их убрать, а цифры оставить. К счастью, это возможно.

![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/digitchar.jpeg?raw=true)


Воспользуемся тем фактом, что линейки нарисованы через равный интервал, поэтому в частотном пространстве они будут представлены как одна очень яркая линия. 

![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/visual_re1.jpg?raw=true)

В данном случае она почти вертикальна, но может быть и слегка наклонной. Наша задача в том, чтобы обнаружить и закрасить эту линию в частотном пространстве, после чего преобразовать изображение в исходный вид. В результате получим примерно следующее:

![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/result1.jpg?raw=true)

А после дополнительной очистки, даже такое:
![](https://github.com/vzaguskin/sampleprojects/blob/master/written_digits/result_otsu1.jpg?raw=true)

Ну, к коду. Импортируем все, что понадобится:

```python

    import numpy as np
    from scipy.misc import imshow, imsave, imread, imresize
    from scipy.ndimage import filters, morphology
    from skimage import filter
    from skimage.draw import line
    from scipy import ndimage, spatial

```


Вспомогательная функция HoughLines, возвращающая самую яркую линию, проходящую через центр, в виде бинарной маски:


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


И основная функция:



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