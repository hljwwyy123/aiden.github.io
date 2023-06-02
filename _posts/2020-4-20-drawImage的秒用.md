---
layout: page
title: drawImage的妙用
date:   2022-10-20 17:36:43 +0800
tags:
  - 绘座
comments: true
jekyll-theme-WuK:
  default:
    sidebar:
      open: true
  archive:
    group_by: "%b %Y" # 见<https://liquid.bootcss.com/filters/date/>
    vega_lite: # 显示一个统计图，需要引入 vega-lite
      enable: true
---
## 场景: 在GridCanvas的背景下跨canvas进行截图

如图：

![image.png](https://img.alicdn.com/imgextra/i2/O1CN01R688SD1copgsriKri_!!6000000003648-1-tps-958-547.gif)

座位矩阵被分散到 Grid 1-4 4个Grid中。

首先确定offScreenCanvas的大小，大小由座位矩阵决定，跟Grid 无关，根据Seats 计算出Seats的BBox，但是由于数据协议的原因，只能获取到Seats ，获取不到Label所以需要再加上padding，预留出Label的空间。

offScreenCanvas 大小确定好了以后，再分别把4个Grid依次通过drawImage 绘制到同一个 offScreenCanvas 中。但并不是需要把整个grid 都draw进去，根据drawImage 第二、第三个参数可以设置drawImage的偏移量。就是真正座位+Label的位置。

## drawImage API

![image.png](https://img.alicdn.com/imgextra/i2/O1CN01Y1KLCn1IWFIt7iVsG_!!6000000000900-0-tps-720-349.jpg)

## 实现思路

```javascript

const LabelPadding = 35; // 35为一个座位的尺寸
gridsCanvas.forEach(g => {
  offScreenCanvasCtx.drawImage(
    g.rootNode, 
    g.rootNodeData.x - frameImage.x + LabelPadding, 
    g.rootNodeData.y - frameImage.y + LabelPadding
  );
});

```

离屏canvas绘制好以后把数据同步给data-proxy，这时候给data-proxy的image其实是偏大的(加了padding)，所以也要修改image的 {x, y} 为 {x - padding, y - padding}

```javascript

const LabelPadding = 35; // 35为一个座位的尺寸
seatImage.url = ossObj.model.ossObjectUrl;
seatImage.width += LabelPadding * 2;
seatImage.height += LabelPadding * 2;
seatImage.x -= LabelPadding;
seatImage.y -= LabelPadding;
dataProxy.setImageInArea(area.id, seatImage);

```
