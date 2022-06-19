---
title: Java awt生成图片消除锯齿
index_img: /img/cover/01.jpg
categories:
  - Java基础
tags:
  - 抗锯齿
abbrlink: da05d629
date: 2017-12-18 11:31:15
---


### java-awt生成图片消除锯齿
```java
//抗锯齿
Graphics2D.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
//g.drawImage(image, x, y, w,h, null);  

//利用iamge.getScaledInstance API
g.drawImage(image.getScaledInstance(w,h),BufferedImage.SCALE_SMOOTH),x,y,null);

ImageIO.write(big,formateName, file);
```

