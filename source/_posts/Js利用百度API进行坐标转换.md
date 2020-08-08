---
title: Js利用百度地图API进行坐标转换
date: 2017-09-04 21:10:59
categories: 技术相关
tags: [JacaScript]
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/baidu-map.png" width="20%" height="20%">

在 JavaScript 中利用百度地图API对地理坐标系和投影坐标系(墨卡托)进行互转
<!-- more -->

# 导入js文件

在html文件中添加

```javascript
<script src="http://api.map.baidu.com/api?v=1.2"></script>
```

# 地理坐标转为平面坐标

```javascript
var projection = new BMap.MercatorProjection();
var mercatorPoint = projection.lngLatToPoint(new BMap.Point(114.3908, 30.4879));
alert("x = " + mercatorPoint.x + ", y = " + mercatorPoint.y);
```

# 平面坐标转为地理坐标

```javascript
var projection = new BMap.MercatorProjection();
var lngLat = projection.pointToLngLat(new BMap.Pixel(12734064.16, 3544542.8));
alert("lng = " + lngLat.lng + ", lat = " + lngLat.lat);
```
