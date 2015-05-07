---
layout: post
title: Векторизируем график функции(python, skimage)
intname: chartvec1
permalink: chartvec1_ru
---

Пусть у нас есть картинка, содержащая график функции, где прямоугольниками обозначены её значения в отдельных точках, которые соединены прямыми линиями. Задача - определить координаты вершин и связи между ними и нарисовать векторизированное изображение того же самого. 

![](https://github.com/vzaguskin/sampleprojects/blob/master/chart%20vectorisation/laGK6.jpg?raw=true)  

Для начала, выделим интересующие нас линии и точки на основе цвета:

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

С помощью морфологических операций уберем линии, оставив только вершины, для чего последовательно применим несколько бинарных эрозий и затем дилляцию для восстановления связанности:

```python

    mask2 = morphology.binary_erosion(mask)
    mask2 = morphology.binary_erosion(mask2)
    mask2 = morphology.binary_erosion(mask2)
    mask2 = morphology.binary_erosion(mask2)
    
    mask2 = morphology.binary_dilation(mask2)

```

Найдем вершины и их центры масс:

```python

    label, numvertices = measurements.label(mask2)
    mc = measurements.center_of_mass(mask2, label, range(1,numvertices+1) )
    
```

Найдем линии, связывающие вершины между собой с ппомощью алгоритма, напоминающего [линии Хафа](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B5%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D0%A5%D0%B0%D1%84%D0%B0). Для этого, определим координаты точек, лежащих на каждой линии, соединяющей вершины и посчитаем долю тех, которые имеют значение True в нашей маске. Если две вершины действительно соединены линией, то их доля будет почти стопроцентной.

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

Получим следующий результат:

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


Ну и наконец, нарисуем картинку на основе полученных вершин и связей, чтобы убедиться в правильности работы алгоритма:

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


Полный исходный код примера [здесь](https://github.com/vzaguskin/sampleprojects/tree/master/chart%20vectorisation).