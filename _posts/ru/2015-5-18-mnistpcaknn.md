---
layout: post
title: Распознавание рукописных цифр из базы MNIST с помощью PCA
intname: mnistpcaknn1
permalink: mnistpcaknn1_ru
---

Простой метод распознавания рукописных цифр на python, дающий, тем не менее почти 97% результат на базе MNIST.

Для упрощения работы с базой используется пакет [python-mnist](https://github.com/sorki/python-mnist/), для уменьшения размерности - [метод главных компонент(он же PCA)](https://ru.wikipedia.org/wiki/%D0%9C%D0%B5%D1%82%D0%BE%D0%B4_%D0%B3%D0%BB%D0%B0%D0%B2%D0%BD%D1%8B%D1%85_%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82), а для классификации - *KNeighborsClassifier* из пакета *sklearn*.

Вот 20 главных компонент, полученных алгоритмом:

![](https://github.com/vzaguskin/sampleprojects/blob/master/mnist_knn/testres/allpcas.jpg?raw=true)

А вот результат работы:

>tested  10000  digits

>correct:  9689 wrong:  311

>error rate:  3.11 %

>got correctly  96.89 %



Собственно, код:

```python
def recognizePCA(train, trainlab, test, labels, num=None):

    if num is None:
        num=len(test)

    train4pca = np.array(train)
    test4pca = np.array(test)

    n_components = 20

    #fitting pca
    pca = RandomizedPCA(n_components=n_components).fit(train4pca)



    xtrain = pca.transform(train4pca)

    xtest = pca.transform(test4pca)

    clf = KNeighborsClassifier()

    #fitting knn
    clf = clf.fit(xtrain, trainlab)

    #predicting
    y_pred = clf.predict(xtest[:num])



#checking the result and printing output
    r=0
    w=0
    for i in range(num):
        if y_pred[i] == labels[i]:
            r+=1
        else:
            w+=1
    print "tested ", num, " digits"
    print "correct: ", r, "wrong: ", w, "error rate: ", float(w)*100/(r+w), "%"
    print "got correctly ", float(r)*100/(r+w), "%"

    return pca.components_

```

Использование:

```python

from mnist import MNIST
import numpy as np
from sklearn.decomposition import RandomizedPCA
from sklearn.neighbors import KNeighborsClassifier

mndata = MNIST('path_to_mnist')
trainims, trainlabels = mndata.load_training()
ims, labels = mndata.load_testing()

recognizePCA(trainims, trainlabels, ims, labels)
```
