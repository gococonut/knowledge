# Map Projections

* 因有项目需要根据经纬度在世界地图标点，了解了一下世界地图的投影方式，总结如下:

## [常用世界地图](https://en.wikipedia.org/wiki/List_of_map_projections)

* [Equirectangular projection 等距圆柱投影](https://en.wikipedia.org/wiki/Equirectangular_projection)

地图上的距离与经纬度的差值成比例，所以可以通过简单的线性计算得到世界地图的坐标

* [Gall Stereographic Gall 立体投影](https://en.wikipedia.org/wiki/Gall_stereographic_projection)

```php
    /**
     * 世界地图(Gall 立体投影)经纬度转换为坐标, 转换后坐标为以地图 0°, 0° 为原点的坐标系
     *
     * For example,
     *
     * ```php
     * $position = [39.90469, 116.40717]] // 北京
     * $r = 117.4912;
     *
     * $result = MathHelper::transformLatAndLng2Coordinate($position, $r);
     * // the result is: [54.54, 106.08]`
     *
     * @param array $array
      * @return array the list of coordinate
      */
    function transformLatAndLng2Coordinate(array $position, $r)
    {
        $x = $r * deg2rad($position[1]) / sqrt(2);
        $y = $r * (1 + sqrt(2) / 2) * tan(deg2rad($position[0] / 2));

        return [$x, $y];
    }
```

