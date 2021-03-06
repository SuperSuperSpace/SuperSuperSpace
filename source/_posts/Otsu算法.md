---
title: Otsu算法
date: 2018-06-02 22:04:42
tags: OpenCV
---

---

背景：最大类间方差法是日本学者大津于1979年提出的，是一种自适应阈值确定的方法，又叫大津法，简称Otsu，是一种基于全局的二值化算法。

原理：Otsu算法是根据图像的灰度特性，将图像分为前景和背景两个部分，当取最佳阈值时，两部分之间的差别应是最大的，在Otsu算法中衡量这个差别的标准就是较为常见的最大类间方差。前景和背景之间的类间方差如果越大，说明构成图像的两个部分之间的差别越大，当部分目标被错分为背景或部分背景被错分为目标，将会导致这两个部分差别变小，当所取阈值的分割使类间方差最大时就意味着错分概率为最小。

记 ：  
> ①T为前景与背景的分割阈值
> ②W₀为前景像素点数占图像比例
> ③U₀为前景像素平均灰度
> ④W₁为背景像素点数占图像比例
> ⑤U₁为背景像素平均灰度
> ⑥U为图像总平均灰度

则有：
> U = W₀ X U₀ + W₁ X U₁

> g = W₀ X (U₀ - U)² + W₁ X (U₁ - U)²

结合上面两式可得：

> g = W₀ X W₁ X(U₀ - U₁)² 

或  
> g =  W₀ / 1 - W₀ X (U₀ - U)²

当方差g最大时，可以认为此时前景和背景差异最大，此时的灰度T是最佳阈值。类间方差法对噪声以及目标大小十分敏感，它仅对类间方差为单峰的图像有较好的分割效果。如果光照不均，反光或者背景复杂等因素影响，类间方差可能呈现双峰或多峰，此时效果不理想。

参考链接：http://blog.csdn.net/u011285477/article/details/52004513



---