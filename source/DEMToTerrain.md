---
title: DEM到三维可视化地形
tags: ["Cesium", "maptalks" ,"DEM"]
date: 2023-04-14
---
# 前言
前端时间详细讲了一下高程点到DEM，再到热力图的一连串自动化处理步骤，热力图是一种可视化高程的方法，如果想要立体三维的效果，请细看以下内容。本文我将用两种方法来三维可视化DEM，将会用到cesium和maptalks两种地图引擎。
# 处理流程
## （一）cesium
### Tif转Terrain
直接用cesiumlab转，参数如图，直接提交生成地形文件。
![image](https://user-images.githubusercontent.com/53400642/232002453-f8c2b3e5-aa9a-407e-a267-4df1003620ec.png)
![image](https://user-images.githubusercontent.com/53400642/232002553-3f692742-d79a-4133-8629-2ac27c03969e.png)
### 开源代码处理
参考github上一个地形处理工具，生成地形。
 [https://github.com/geo-data/cesium-terrain-builder](https://github.com/geo-data/cesium-terrain-builder/)
### cesium地形加载
用cesium以下代码加载一下。
```js
var terrainProvider = new Cesium.CesiumTerrainProvider({
          url:"/Terrian/2",
          minimumLevel: 0,
          maximumLevel: 20,
    })
 window.viewer.terrainProvider = terrainProvider;
 window.viewer.scene.globe.terrainExaggeration = 20.0;//地形夸张
```
### 效果
红线是我描绘的以防看不清。
![image](https://user-images.githubusercontent.com/53400642/232002657-f1cfe436-a6d8-4b55-b900-4d954de8945a.png)
## （二）Maptalks
maptalks是一个国内开发的地图引擎，其对三维模型、3dtiles等的拓展使其兼顾着二三维的渲染效果，如果了解threejs还可以根据其插件做threejs相关开发，是一款挺不错的地图引擎。最近也是有用到做一个项目，开发起来很便捷。那maptalks是如何渲染地形，我们只需要参考其例子即可。
[https://maptalks.org/maptalks.three/demo/terrain.html](https://maptalks.org/maptalks.three/demo/terrain.html)
### 代码分析
主要看这个函数，其用到两个服务
服务1：https://api.mapbox.com/v4/mapbox.satellite/${z}/${x}/${y}.png?access_token=${accesstoken}
mapbox影像服务，用于对地形做纹理
服务2：https://a.tiles.mapbox.com/v4/mapbox.terrain-rgb/${z}/${x}/${y}.pngraw?access_token=${accesstoken}
mapbox地形RGB服务，用于生成地形高度
```js
        function addTerrain() {
            const TILESIZE = 256;
            const minx = 3268, maxx = 3277, miny = 1632, maxy = 1641;
            const tiles = [];
            for (let x = minx; x <= maxx; x++) {
                for (let y = miny; y <= maxy; y++) {
                    tiles.push([x, y, 12]);
                }
            }
            tiles.forEach(tile => {
                const [x, y, z] = tile;
                const bbox = tilebelt.tileToBBOX(tile);
                const texture = `https://api.mapbox.com/v4/mapbox.satellite/${z}/${x}/${y}.png?access_token=${accesstoken}`;
                const terrain = threeLayer.toTerrain(bbox, { texture, imageWidth: TILESIZE, imageHeight: TILESIZE }, new THREE.MeshBasicMaterial());
                // const sclae = 1.005;
                // terrain.getObject3d().scale.set(sclae, sclae, 1);
                threeLayer.addMesh(terrain);
                tileData.push({
                    tile,
                    terrain
                });
            });
            setTimeout(() => {
                updateTerrainData();
            }, 2000);

        }
```

```js
function updateTerrainData() {
            tileData.forEach(d => {
                const { tile, terrain } = d;
                const [x, y, z] = tile;
                const url = `https://a.tiles.mapbox.com/v4/mapbox.terrain-rgb/${z}/${x}/${y}.pngraw?access_token=${accesstoken}`;
                actor.test({ tile, url }, (message) => {
                    if (message.data) {
                        taskQueue.push({
                            data: new Uint32Array(message.data),
                            terrain
                        })
                    }
                })
            });
        }
```
需要关注几个值如下，这几个值要和你生成的瓦片数据一一对应，或许你只能获得某一个地区的DEM数据，这个地区的瓦片对应在哪里，找到你想要的层级和对应范围。（说得有点抽象，不懂可以研究下瓦片地图金字塔模型）
![image](https://user-images.githubusercontent.com/53400642/232002743-d8fe4317-4b1d-4978-a61d-9f74f34962d6.png)
### 制作Mapbox Terrain-RGB（mapbox地形RGB服务）
经过代码分析发现我们主要需要的就是一个类似MapboxTerrain-RGB的服务，以下就是这个服务的做法：
#### 1、安装处理程序
安装前必须还需要安装python环境
[gdal](https://gdal.org/) 栅格和矢量地理空间数据处理
[rasterio](https://github.com/rasterio/rasterio) 读写栅格数据；
[rio-rgbify](https://github.com/mapbox/rio-rgbify/) 将 dem 栅格编码为 rgb 栅格的 rasterio 插件；
#### 2、tif数据处理
在数据存储目录cmd

第一步旨在将坐标系转化为3857，同时将无数据的值化为null。
```cmd
gdalwarp -t_srs EPSG:3857 -dstnodata None -co TILED=YES -co COMPRESS=DEFLATE 
-co BIGTIFF=IF_NEEDED  输入.tif 输出.tif
```
第二步旨在制作出符合Mapbox Terrain-RGB模型的tif数据
具体模型可以参考[Mapbox Terrain-RGB](https://docs.mapbox.com/data/tilesets/reference/mapbox-terrain-rgb-v1/)
```cmd
rio rgbify -b -10000 -i 0.1 输出.tif 第二次输出.tif
```
#### 3、服务发布
geoserver发布栅格tif即可
![image](https://user-images.githubusercontent.com/53400642/232002848-9fc0ee62-189d-4c3d-a491-f7048dfb3339.png)
### 完整代码
```js
<template>
  <div id="map"></div>
</template>

<script>
import * as maptalks from 'maptalks';
import * as THREE from 'three';
import { GroupGLLayer } from "@maptalks/gl-layers"
import { ThreeLayer } from 'maptalks.three'
import tilebelt from '../lib/map/layer/tilebelt'
let accesstoken = '你的token';
export default {
  name: "TerrainView",
  mounted() {
    const workerKey = 'terraindata'
    maptalks.registerWorkerAdapter(workerKey, function (exports, global) {

      let canvas;
      const TILESIZE = 256;

      function getCanvas() {
        if (canvas) {
          return canvas;
        }
        canvas = new OffscreenCanvas(TILESIZE, TILESIZE);
        return canvas;
      }
      //will be called only for once when loaded in worker thread
      exports.initialize = function () {
        // console.log('terraindata worker initialized');
      };
      //to receive message from main thread sent by maptalks.worker.Actor
      exports.onmessage = function (message, postResponse) {
        const data = message.data;
        const { url } = data;
        fetch(url).then(res => res.blob()).then(blob => createImageBitmap(blob)).then(bitmap => {
          const canvas = getCanvas();
          const offCtx = canvas.getContext('2d');
          offCtx.drawImage(bitmap, 0, 0, canvas.width, canvas.height);
          var imgData = offCtx.getImageData(0, 0, canvas.width, canvas.height).data;
          bitmap.close();
          const data = new Uint32Array(imgData).buffer;
          postResponse(null, { data: data }, [data]);
        }).catch(error => {
          console.log(error)
          postResponse(null, {}, []);
        });
      };
    })
    const actor = new maptalks.worker.Actor(workerKey);
    actor.test = function (params, callback) {
      const { url, tile } = params;
      this.send({ url }, null, (err, message) => {
        message.tile = tile;
        callback(message);
      });
    }


    var baseLayer = new maptalks.TileLayer('tile', {
      // urlTemplate: 'https://mt2.google.cn/maps/vt?lyrs=s&hl=zh-CN&gl=CN&x={x}&y={y}&z={z}',
      urlTemplate: 'https://api.mapbox.com/v4/mapbox.satellite/{z}/{x}/{y}.png?access_token=' + accesstoken,
      subdomains: ['a', 'b', 'c', 'd'],
      debug: true,
      debugOutline: 'red'
      // attribution: '&copy; <a href="http://osm.org">OpenStreetMap</a> contributors, &copy; <a href="https://carto.com/">CARTO</a>'
    });


    var map = new maptalks.Map("map", {
      "center": [109.16032190999756, 19.81946033063278], "zoom": 16, "pitch": 58.40000000000002, "bearing": -0.6000000000000227,
      // bearing: 180,

      centerCross: false,
      doubleClickZoom: false,
      baseLayer: baseLayer,
      zoomControl: true,
      heightFactor: 1.4
    });
    // baseLayer.hide();

    // the ThreeLayer to draw buildings
    var threeLayer = new ThreeLayer('t', {
      forceRenderOnMoving: true,
      forceRenderOnRotating: true
      // animation: true
    });

    threeLayer.prepareToDraw = function (gl, scene, camera) {

      var light = new THREE.DirectionalLight(0xffffff);
      light.position.set(0, -10, 10).normalize();
      scene.add(light);
      addTerrain();
      animation();
    }
    const tileData = [];
    function addTerrain() {
      const TILESIZE = 256;
      const minx = 52637, maxx = 52641, miny = 29085, maxy = 29089;
      const tiles = [];
      for (let x = minx; x <= maxx; x++) {
        for (let y = miny; y <= maxy; y++) {
          tiles.push([x, y, 16]);
        }
      }
      tiles.forEach(tile => {
        const [x, y, z] = tile;
        const bbox = tilebelt.tileToBBOX(tile);
        const texture = `https://api.mapbox.com/v4/mapbox.satellite/${z}/${x}/${y}.png?access_token=${accesstoken}`;
        //let texture = `/geoserverAddress/geoserver/cite/gwc/service/wmts?layer=cite:3857_gd_dem_n_rgb&style=&tilematrixset=EPSG:900913&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix=EPSG:900913:${z}&TileCol=${x}&TileRow=${y}`
        const terrain = threeLayer.toTerrain(bbox, { texture, imageWidth: TILESIZE, imageHeight: TILESIZE }, new THREE.MeshBasicMaterial());
        // const sclae = 1.005;
        // terrain.getObject3d().scale.set(sclae, sclae, 1);
        threeLayer.addMesh(terrain);
        tileData.push({
          tile,
          terrain
        });
      });
      setTimeout(() => {
        updateTerrainData();
      }, 10);

    }

    const taskQueue = [];
    function updateTerrainData() {
      tileData.forEach(d => {
        const { tile, terrain } = d;
        const [x, y, z] = tile;
        const url = `http://localhost:8888/geoserverAddress/geoserver/cite/gwc/service/wmts?layer=cite%3A3857_gd_dem_n_rgb&style=&tilematrixset=EPSG%3A900913&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image%2Fpng&TileMatrix=EPSG%3A900913%3A${z}&TileCol=${x}&TileRow=${y}`
        actor.test({ tile, url }, (message) => {
          console.log(message.data)
          if (message.data) {
            taskQueue.push({
              data: new Uint32Array(message.data),
              terrain
            })
          }
        })
      });
    }

    function loopQueue() {
      if (taskQueue.length) {
        const { data, terrain } = taskQueue[0];
        terrain.updateData(data);
        taskQueue.splice(0, 1);
      }
    }
    const sceneConfig = {
      postProcess: {
        enable: true,
        antialias: { enable: true }
      }
    };
    const groupLayer = new GroupGLLayer('group', [threeLayer], { sceneConfig });
    groupLayer.addTo(map);

    function animation() {
      // layer animation support Skipping frames
      threeLayer._needsUpdate = !threeLayer._needsUpdate;
      if (threeLayer._needsUpdate && !threeLayer.isRendering()) {
        threeLayer.redraw();
        loopQueue();
      }
      requestAnimationFrame(animation);

    }

    function getTerrains() {
      return tileData.map(d => {
        return d.terrain;
      })
    }
  }
}
</script>

<style scoped>
#map{
  width: 100%;
  height: 100%;
}
</style>

```
### 效果
![image](https://user-images.githubusercontent.com/53400642/232003017-d1a7a4fe-9cdb-43f5-8226-b75883acb754.png)
# 结尾
对我来说，以前想到地形三维展示就想到cesium，但像maptalks这样的地图引擎同样也可以做到类似的事情，这对于技术选型来说又多了一种选择。
# 参考
[张志敏的技术专栏——以 Mapbox Terrain-RGB 模型发布高程数据](https://beginor.github.io/2021/01/11/publish-dem-data-with-mapbox-terrain-rgb.html)

[MapBox文档](https://docs.mapbox.com/)
