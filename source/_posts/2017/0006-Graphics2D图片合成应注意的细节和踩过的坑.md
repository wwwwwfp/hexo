---
title: Graphics2D图片合成应注意的细节和踩过的坑
index_img: /img/cover/06.jpg
categories:
  - 图片文字处理
tags:
  - Graphics2D
  - 图片处理
abbrlink: fa3b7566
date: 2017-12-29 11:27:01
---
Graphics2D 2d https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html
## 图片处理
### 首先画布肯定是需要的，可以新建一个空白画布，也可以以图片做画布。
```java
B ufferedImage  bi = new BufferedImage(width,height,type);
2d = bi.createGraphics();
```
### 如果需要生成RGB格式，需要做如下配置
```java
bi = 2d.getDeviceConfiguration().createCompatibleImage(width,height,Transparency.TRANSLUCENT);
```
    注：参数width 和 height 要和是前面画布的对应。
    Transparency透明度设置
### 画图 g.drawImage(img,x,y,width,hight);
注：参数x，y为图片左上角坐标
### 旋转处理 AffineTransform atf.rotate(theta,x,y)
    注：theta这儿的角度需要转换成弧度数
    x,y为旋转中心坐标，图片旋转参考点为图片的中心点
    同时有偏移、缩放、旋转操作时，画图顺序为：缩放-->偏移-->旋转
### 图片抗锯齿设置
    2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING,RenderingHints.VALUE_ANTIALAS_ON);
    或
    image = image.getScaledInstance(width,height,BufferedImage.SCALE_SMOOTH)
    g.drawImage(image,x,y,observer)

## 字体处理
### Graphics2D 处理字体的做法和处理图片的大体一致
    1、最需要注意的一点就是 在画字体的时候 x,y坐标为字体左左左左下角
    2、旋转中心可以通过获取字体的行高和字字符串宽度对应的api计算获得
    3、最好用同一包中的字体ttf。如果混用，图片在处理缩放时会存在差异，即使用的字体类型、大小、样式都一致，同样可能会存在差异