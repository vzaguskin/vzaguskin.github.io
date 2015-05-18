---
layout: post
title: Using PCA for digits recognition in MNIST using python
intname: mnistpcaknn1
permalink: mnistpcaknn1
---

Here is a simple method for handwritten digits detection in python, still giving almost 97% success at MNIST.

We use [python-mnist](https://github.com/sorki/python-mnist/) to simplify working with MNIST,  [ PCA](http://en.wikipedia.org/wiki/Principal_component_analysis) for dimentionality reduction, and *KNeighborsClassifier* from *sklearn* for classification.

Here are 20 principal components for MNIST training set obtained with this method:

![](https://github.com/vzaguskin/sampleprojects/blob/master/mnist_knn/testres/allpcas.jpg?raw=true)

And this is the recognition result:

>tested  10000  digits

>correct:  9689 wrong:  311

>error rate:  3.11 %

>got correctly  96.89 %



Here is the code:

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

Usage:

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
