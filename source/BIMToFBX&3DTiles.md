---
title: BIM文件转FBX和3DTiles，并且应用Threejs和Cesium做展示
tags: ["Cesium", "BIM"]
date: 2023-01-20
---

网上有许多处理方法，但都或多或少存在一些问题。
1、推荐CesiumLab对于revit文件转clm再进行切片的处理方式，方便快捷但需要收费。
2、revit文件导出成nwc，通过Navisworks Manage导出FBX文件，虽然会保留着色效果，但纹理存在丢失，直接通过3dmax加载rvt文件导出FBX也存在类似问题。
3、模型结构上，出现同材质被归为一个模型结构，破坏了模型属性对应结构的相关信息，使得点击拾取查询出现一定问题。
为解决以上问题，本文分享一种基于3dmax材质转换器导出FBX再生成3dtiles的方法。
# 处理工具
本文处理工具有3dmax2020、CesiumLab，请自行参考网络安装。
# 材质转换
## 导入rvt
导入rvt发现，其材质为Autodesk材质，需要将其转成标准材质。
![在这里插入图片描述](https://img-blog.csdnimg.cn/22aa8b81455c460d8289f8b1446d61f6.png#pic_center)
## 材质编辑器
在Rendering（渲染）找到Scene Converter（场景转换器）
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d6addcad2b64ba0b3b7324c4ee9f346.png)
选择转换规则如图，点击转换
![在这里插入图片描述](https://img-blog.csdnimg.cn/7aa81b2ad23045c5974e287a30dee760.png)
材质就会从Autodesk材质转化为标准材质

![在这里插入图片描述](https://img-blog.csdnimg.cn/e22048d00a18461280269b748f4aad96.png)
## 文件导出
将模型导出成FBX文件，导入发现材质基本保留。FBX文件可直接用Threejs加载，代码可自行寻找。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e30236fbdf594bdaa594f5b7c9b52d74.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/a223f404c6e34853beae234162ca4933.png)
# CesiumLab切片展示

![在这里插入图片描述](https://img-blog.csdnimg.cn/6436d3339b124bbf8f6abf30bdbe160b.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1ebae89c64854c5fae382804a1e66f49.png)

# 结尾
参考：
http://www.cesiumlab.com/
若需转载请注明本文链接，感谢观看。
