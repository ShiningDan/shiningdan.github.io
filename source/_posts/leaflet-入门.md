---
title: leaflet 入门
date: 2017-07-27 20:35:30
categories: coding
tags:
  - JavaScript
  - Leaflet
  - Map
---

leaflet是领先的开源JavaScript库为移动设备设计的互动地图，下面介绍的是我的 leaflet 学习笔记

<!--more-->

## 开始使用 Leaflet

## GeoJSON 介绍

GeoJSON是一种地理数据的描述格式。GeoJSON可以描述的对象包括：几何体，要素和要素集

这里几何体(Geometry)的类型有我们熟悉的点(Point),线(LineString),面(Polygon), 多点(MultiPoint),多线(MultiLineString),多面( MultiPolygon)和几何体集合(GeometryCollection)。

要素(Feature)包含了几何体信息以及附加的一些属性信息。

* [The GeoJSON Format](https://tools.ietf.org/html/rfc7946)
* [geojson-spec](http://geojson.org/geojson-spec.html#examples)

```
{
   "type": "FeatureCollection",
   "features": [{
       "type": "Feature",
       "geometry": {
           "type": "Point",
           "coordinates": [102.0, 0.5]
       },
       "properties": {
           "prop0": "value0"
       }
   }, {
       "type": "Feature",
       "geometry": {
           "type": "LineString",
           "coordinates": [
               [102.0, 0.0],
               [103.0, 1.0],
               [104.0, 0.0],
               [105.0, 1.0]
           ]
       },
       "properties": {
           "prop0": "value0",
           "prop1": 0.0
       }
   }, {
       "type": "Feature",
       "geometry": {
           "type": "Polygon",
           "coordinates": [
               [
                   [100.0, 0.0],
                   [101.0, 0.0],
                   [101.0, 1.0],
                   [100.0, 1.0],
                   [100.0, 0.0]
               ]
           ]
       },
       "properties": {
           "prop0": "value0",
           "prop1": {
               "this": "that"
           }
       }
   }]
}
```


