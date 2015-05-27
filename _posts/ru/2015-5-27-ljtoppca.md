---
layout: post
title: PCA анализ топа русскоязычного ЖЖ
intname: pcaljtop1
permalink: pcaljtop1_ru
---
Провел небольшой анализ топа пользователей русскоязычного ЖЖ. Целью было выявить некоторые группы предпочитающих друг-друга для дружбы пользователей на основе анализа данных о взаимодружбе первой тысячи пользователей топа. К сожалению, очень явно выраженных групп не получилось - все пользователи в топе в целом склонны дружить примерно с одними и теми же людьми. Тем не менее кое какие тенденции прослеживаются.

Вот тут нарисованны точки, соответствующие позициям каждого пользователя в базисе, соответствующем двум наиболее значимым главным векторам. По горизонтальной оси виден явный кластер слева, к которому принадлежат абсолютное большинство точек, а так же менее выражденная группа справа. По вертикальной оси, в принципе, есть один четкий центр и некоторый разброс.

![](https://github.com/vzaguskin/sampleprojects/blob/master/LJClustering/all_unannotated.png?raw=true)

Вот тут подписаны имена для некоторых пользователей. Внизу слева - патриотические политики, вверху - либеральные тревел-блоггеры, в центре женщины-околопсихологи пишущие про отношения. Вот смысл горизонтальной оси понять сложнее, но именно она должна выражать разницу между, например aquateq_filips и zyalt либо sergeydolya, а также между miumau и bespridanitsa.

![](https://github.com/vzaguskin/sampleprojects/blob/master/LJClustering/first50_annotated.png?raw=true)