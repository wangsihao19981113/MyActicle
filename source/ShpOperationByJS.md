---
title: shp文件纯前端的上传、解析、编辑、下载
tags: ["shpwrite", "shpjs"]
date: 2023-01-20
---
本文主要讲述一种体量较小的shp文件纯前端的上传、解析、编辑、下载的技术流程，适用于要素量少的shp文件修改操作。
# 准备工作
下载一下几个包，详细用法请见结尾参考。
```cmd
npm install file-save
下载文件

npm install shpjs
解析shapefile的压缩包

npm install shp-write
生成shapefile的压缩包
```
# 流程
## 上传
使用elementui上传。
```html
 <el-upload
     class="upload-demo uploadShape"
     action="#"
     :on-preview="handlePreview"
     :on-remove="handleRemove"
     :before-remove="beforeRemove"
     :http-request="handleUpload"
     :on-change="handleOnChange"
     multiple
     :limit="1"
     :file-list="FileList"
     :on-exceed="handleExceed">
   <el-button slot="trigger" size="small" type="primary">点击上传</el-button>
   <el-button style="margin-left: 10px;" size="small" type="success" @click="parsingZip">解析</el-button>
   <el-button size="small" type="success" @click="startEdit">开始编辑</el-button>
   <el-button size="small" type="success" @click="endEdit">结束编辑</el-button>
   <el-button size="small" type="success" @click="download">下载</el-button>
   <div slot="tip" class="el-upload__tip">请上传zip文件</div>
 </el-upload>
```
## 解析
获取文件类并且解析出GeoJSON数据，使用的是shpjs.parseZip，解压的是zip文件，res就为GeoJSON数据。
```js
 parsingZip(){
   let self = this;
   let file = this.FileList[0].raw
   if(!file){return}
   let reader = new FileReader();
   reader.readAsArrayBuffer(file)
   reader.onload = function(e) {
     let res = e.target.result;//ArrayBuffer
     shpjs.parseZip(res).then(function (res){
       self.addGeometry(res)
       self.geojson = res
     }).catch(function (e){
       console.log(e)
     })
   }
 },
```
## 展示
这里前端展示界面和修改使用maptalks的接口，创建地图的代码可参考maptalk官网例子，详细用法请见结尾参考，亦可用其他的前端渲染引擎。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e48702ad0d2046e38e66ddaf8847a55a.png)
```js
addGeometry(geojson){
      let map = window.app.MTmap
      if(geojson.type == "FeatureCollection") {
        for(let i = 0 ; i < geojson.features.length ; i++){
          let geometry = maptalks.GeoJSON.toGeometry(geojson.features[i]).addTo(map.getLayer('v'));
          this.ShapeCollection.push(geometry)
        }
      }
      else{
        let geometry = maptalks.GeoJSON.toGeometry(geojson).addTo(map.getLayer('v'));
        this.ShapeCollection.push(geometry)
      }
    }
```
## 编辑
![在这里插入图片描述](https://img-blog.csdnimg.cn/26beae21a8b0492ea5567f8affa2ee2a.png)


```js
 startEdit(){
   for(let i = 0 ; i < this.ShapeCollection.length ; i++){
     this.ShapeCollection[i].startEdit();
   }
 }
 endEdit(){
   for(let i = 0 ; i < this.ShapeCollection.length ; i++){
     this.ShapeCollection[i].endEdit();
   }
 }
```
## 下载
### shp-write问题
因为初用shp-write的时候出现报错了，报错截图如下。
![在这里插入图片描述](https://img-blog.csdnimg.cn/16dceecd85704a759e087303c3e6218a.png)
我也不懂这个是啥，看上去好像版本新了，方法没了，本人也没有试过降低版本，但参考了一些github提问的中解决的方法，找到了一个办法，具体解决方法截图如下。十分感谢这位哥。
![在这里插入图片描述](https://img-blog.csdnimg.cn/64ca95e4cedb45e8a9f7b9a36402e854.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/93f2cd39490d4bde9caea531ecc6ffca.png)
### 代码
folder为存放的文件夹，point、polygon、line为点线面shapefile的名字，涉及一个base64转blob的方法Base64ToBlob，可参考网上代码。用file-save类去下载这个zip文件。
```js
    download(){
      let self = this;
      let geojson = this.getGeoJSONFromLayer(window.app.MTmap.getLayer('v'))
      if(geojson){
        var options = {
          folder: 'myshapes',
          types: {
            point: 'mypoints',
            polygon: 'mypolygons',
            line: 'mylines'
          }
        }
        shpwrite.zip(geojson,options).then(function(content) {
          let blob = self.Base64ToBlob('data:application/zip;base64,' + content)
          saveAs(blob, 'export.zip');
        });
      }
    },
```
```js
 getGeoJSONFromLayer(layer){
   let geolist = layer._geoList
   let features = []
   for(let i = 0 ; i < geolist.length ; i++){
     features.push(geolist[i].toGeoJSON())
   }
   return({
     type:"FeatureCollection",
     features:features
   })
 },
```
### 结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/785e09209aa6491d93a17542bcbede7e.png)
# 结尾
此方法目前在大体量数据解析过程中可能会出错或者时间过长，因此如果数据量大的话还是推荐上传到服务器进行解析。
本文代码是vue中写的，并且截取片段，望不会造成太大的阅读困扰。
参考：
链接: [maptalks](https://maptalks.org/maptalks.js/api/1.x/Map.html)
链接: [shp-write](https://github.com/mapbox/shp-write/issues/48)
链接: [shpjs](https://github.com/sitna/shapefile-js/pkgs/npm/shpjs)
链接: [file-save](https://github.com/eligrey/FileSaver.js)
若需转载请注明本文链接，感谢观看。


