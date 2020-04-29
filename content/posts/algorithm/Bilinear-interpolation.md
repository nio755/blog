---
title: 双线性图像缩放(Bilinear Image Scaling)
date: 2020-04-29T16:09:54+08:00
lastmod: 2020-04-29T16:09:54+08:00
author: nio
cover: /img/algorithm-cover.png
categories: ["算法"]
tags: ["图像", "算法"]
# showcase: true
draft: true
---

## 介绍

对比最近邻插值，双线性插值是基于周围像素来产生更平滑的缩放比例，通过取原图像中距离目标像素点最近的4个点，并对这4个点与其对应权值的乘积求和，获得最终像素值。

### 线性插值

线性插值用于估计两点间的任意点的方法。如下图示：

![linear](../img/linear.png)

![lieq](../img/lieq.gif)

## 算法原理

![放大图像引入“空白”](../img/enlarge.png)

首先找到A，i和B的关系：

![A，i和B的关系](../img/eq1.gif)

相同的，找到C，j和D的关系：

![C，j和D的关系](../img/eq2.gif)

将得到的两个线性插值方程组合在一起，形成一个双线性函数的方程式。

![双线性函数的方程式](../img/eq3.gif)

将等式1和等式2代入3，得到：

![代入求解](../img/bilineareq.gif)

使用最后的等式，对图像进行插值。

## 代码实现

[代码链接](https://github.com/nio755/LearnLearnLearn/blob/master/Image/code/nearestNeighbor/nearestNeighbor.go)

```go
func resizeBilinear(img image.Image, width, height int) image.Image {
	tmp := image.NewRGBA(image.Rect(0, 0, width, height))
	oldWidth, oldHeight := img.Bounds().Dx(), img.Bounds().Dy()
	x_ratio := float64(oldWidth) / float64(width)
	y_ratio := float64(oldHeight) / float64(height)
	for i := 0; i < height; i++ {
		for j := 0; j < width; j++ {
			x := int(x_ratio * float64(j))
			y := int(y_ratio * float64(i))
			x_diff := x_ratio*float64(j) - float64(x)
			y_diff := y_ratio*float64(i) - float64(y)
			r, g, b, a := img.At(x, y).RGBA()
			r1, g1, b1, a1 := img.At(x, y+1).RGBA()
			r2, g2, b2, a2 := img.At(x+1, y).RGBA()
			r3, g3, b3, a3 := img.At(x+1, y+1).RGBA()
			
			// red element
			// Yr = Ar(1-w)(1-h) + Br(w)(1-h) + Cr(h)(1-w) + Dr(wh)
			red := float64(r>>8)*(1-x_diff)*(1-y_diff) + float64(r1>>8)*(
				x_diff)*(1-y_diff) + float64(r2>>8)*(y_diff)*(1-x_diff) + float64(r3>>8)*(x_diff*y_diff)
			// green element
			// Yg = Ag(1-w)(1-h) + Bg(w)(1-h) + Cg(h)(1-w) + Dg(wh)
			green := float64(g>>8)*(1-x_diff)*(1-y_diff) + float64(g1>>8)*(x_diff)*(1-y_diff) + float64(g2>>8)*(y_diff)*(1-x_diff) + float64(g3>>8)*(x_diff*y_diff)
			// red element
			// Yr = Ar(1-w)(1-h) + Br(w)(1-h) + Cr(h)(1-w) + Dr(wh)
			blue := float64(b>>8)*(1-x_diff)*(1-y_diff) + float64(b1>>8)*(x_diff)*(1-y_diff) + float64(b2>>8)*(y_diff)*(1-x_diff) + float64(b3>>8)*(x_diff*y_diff)

			alpha := float64(a>>8)*(1-x_diff)*(1-y_diff) + float64(a1>>8)*(x_diff)*(1-y_diff) + float64(a2>>8)*(y_diff)*(1-x_diff) + float64(a3>>8)*(x_diff*y_diff)

			col := color.RGBA{
				R: uint8(red),
				G: uint8(green),
				B: uint8(blue),
				A: uint8(alpha),
			}
			tmp.Set(j, i, col)
		}
	}
	return tmp
}
```

## 参考

- [Bilinear Image Scaling](http://tech-algorithm.com/articles/bilinear-image-scaling/)
