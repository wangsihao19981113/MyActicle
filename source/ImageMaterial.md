---
title: Cesium图片材质（解决billboard面对屏幕问题）
tags: ["Cesium"]
date: 2023-01-22
---

在Cesium展现图片的方式一般是billboard，这样展示的图片一般是一直面对这屏幕，假设现在有一个场景需要将
一张图片粘在地上，不随视角转动而变化，就需要用到图片材质做一个初步解决。
# 图片材质
添加一个geometry实例，一般为矩形或者多边形即可，然后为其附上图片材质。
```js
scene.primitives.add(new Cesium.Primitive({
    geometryInstances : instance,
    appearance : new Cesium.EllipsoidSurfaceAppearance({
        material : new Cesium.Material({
            fabric : {
                type : 'Image',
                uniforms : {
                    image : 'favicon.ico'
                }
            }
        })
    })
}));
```
# 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/4dfe6965ceba4771bb663441e243300d.png)

# 结尾
感觉修改Billboard的源码应该也能解决这个问题，以后会考虑看看。
新春快乐，感谢观看。