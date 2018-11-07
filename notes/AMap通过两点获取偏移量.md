### AMap通过两点获取偏移量

> 最近在用AMap做一个热点图，但是有些功能官方没法提供，所以自己封装了一些方法



- 两点确定偏移的米数

```js
/**
   * 
   * @param {Object | include longitude & latitude} point1 
   * @param {Object | include longitude & latitude} point2 
   * @return {Object | include x & y} obj
   */
  getOffset(point1, point2) {
    const p = new window.AMap.LngLat(point1.longitude, point1.latitude)
    const x2 = new window.AMap.LngLat(point2.longitude, point1.latitude)
    const x = Math.round(p.distance(x2))
    const y2 = new window.AMap.LngLat(point1.longitude, point2.latitude)
    const y = Math.round(p.distance(y2))
    const arr = [
      x,
      y,
    ]
    if (point1.longitude > point2.longitude) arr[0] = -x
    if (point1.latitude > point2.latitude) arr[1] = -y
    return {
      x: arr[0],
      y: arr[1],
    }
  }
```

- 确定偏移后的点的经纬度

```js
/**
   * 
   * @param {Object | include longitude, latitude, w, s} param0 
   * @param {Float | initial value : 1} rate 
   * @returns { Object | include longitude, latitude } obj
   */
  getOffsetLngLon({
    longitude,
    latitude,
    w,
    s,
  }, rate = 1) {
    const point = new window.AMap.LngLat(longitude, latitude)
    point.offset(w * rate, s * rate)
    return {
      longitude: point.getLng(),
      latitude: point.getLat(),
    }
  }
```

