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
![image](https://user-images.githubusercontent.com/53400642/231701731-e5ca84ab-bcb6-48fb-8611-58e4e05d949f.png)
## 材质编辑器
在Rendering（渲染）找到Scene Converter（场景转换器）
![image](https://user-images.githubusercontent.com/53400642/231701849-51c00718-920a-4be8-bb90-2c862014d4c6.png)
选择转换规则如图，点击转换
![image](https://user-images.githubusercontent.com/53400642/231701915-cd7e5104-8be8-4003-bbe3-d358ed980e60.png)
材质就会从Autodesk材质转化为标准材质
![image](https://user-images.githubusercontent.com/53400642/231702006-ead56c49-3918-42b2-b49c-a979144de1df.png)
## 文件导出
将模型导出成FBX文件，导入发现材质基本保留。FBX文件可直接用Threejs加载，代码可自行寻找。
![image](https://user-images.githubusercontent.com/53400642/231702055-995fddba-8c50-419a-a473-56530074bd19.png)

![image](https://user-images.githubusercontent.com/53400642/231702102-f9f45aea-4b7d-475b-b8da-b9ec96e75ebd.png)
# CesiumLab切片展示

![image](https://user-images.githubusercontent.com/53400642/231702145-379ed2f8-0ce2-4d3d-a23a-68f0fafe0f5a.png)

![image](https://user-images.githubusercontent.com/53400642/231702198-ba9a1182-acd4-4c25-8495-48c805a04716.png)

# 结尾
参考：
http://www.cesiumlab.com/
若需转载请注明本文链接，感谢观看。
