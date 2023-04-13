---
title: UE5+Cesium for Unreal+Pixel Streaming实现基础的天空地形Web展示
tags: ["UE5"]
date: 2023-01-20
---
近期有在研究UE5游戏引擎在GIS上的应用，Cesium刚好有相关的插件能生成一个场景，但又想到不知如何应用到Web，于是参考网上一些大神的参考流程，结合Pixel Streaming提供的两个重要代码，实现了在Vue前端展示嵌入，借此分享记录。
# 软件和插件
下载安装UE5.1，VS2022
安装插件 Cesium for Unreal，Pixel Streaming
![image](https://user-images.githubusercontent.com/53400642/231702777-5355cba9-946c-4aa3-9ade-0b40d0950ccd.png)
# 处理流程
## UE处理
### 加载插件
先打开UE，新建一个项目（最好不要有中文，尽量不包含test字样），勾选插件，选择确定后，会让你重启一下软件，选择重启。
![image](https://user-images.githubusercontent.com/53400642/231702817-1e11f6fb-faab-43ba-ae8d-ddd53254017b.png)
![image](https://user-images.githubusercontent.com/53400642/231702856-b333aba8-e878-4b0d-8b26-7e733f531366.png)
### Cesium认证
先加载天空试试效果，然后点击Connect to Cesium ion去认证信息。
![image](https://user-images.githubusercontent.com/53400642/231702902-f8493f33-7a45-43a1-8538-c5c107ef7a73.png)
没有账号的需要先注册一个，点击Allow，网络问题可能会加载有点慢，可以多试几次。
![image](https://user-images.githubusercontent.com/53400642/231702947-a4c3dfca-e439-4f3c-99ef-7d433abe0d60.png)
出现以下提示代表认证可以了。

![image](https://user-images.githubusercontent.com/53400642/231703253-3de2cc52-d3ec-48d1-8d56-66c72f9dbccb.png)

添加一下世界地形和底图，第一次加载会提醒你使用密钥，申请一个就好，可以在实例中添加自己的底图，不过好像只能叠加3个，分别对应Marterial Layer Key属性Overlay0，Overlay1，Overlay2（本来想用作地图服务加载，发现只有3个不够用），这里就不作修改了。

![image](https://user-images.githubusercontent.com/53400642/231703392-33a59602-a144-43e0-8e75-3442d84539b5.png)
![image](https://user-images.githubusercontent.com/53400642/231703510-cd181d55-3bfd-4b4d-a26e-73d7ba795949.png)
![image](https://user-images.githubusercontent.com/53400642/231703577-12ba381f-813d-428a-9a30-c8a45923d0a5.png)

### 保存关卡，打包
保存一下关卡，编辑打开项目设置，改一下编辑器开始地图和默认地图，改成刚才保存那个。
![image](https://user-images.githubusercontent.com/53400642/231703652-7144177a-829d-4743-946e-2ef78b97e0ac.png)
![image](https://user-images.githubusercontent.com/53400642/231703699-48b8661f-a70f-4737-942e-2c35a1c578ed.png)
同样是编辑打开编辑器偏好设置，修改一下额外启动参数。
-AudioMixer -PixelStreamingIP=localhost -PixelStreamingPort=8888。
![image](https://user-images.githubusercontent.com/53400642/231703750-4dd25785-804e-4915-b922-fd8487e9d293.png)
然后打包项目。
![image](https://user-images.githubusercontent.com/53400642/231703808-b7c5a143-80a6-4ea8-b927-38a541696856.png)
## 文件处理
### 创建快捷方式
找到打包的目录首页，给原本exe文件拉个快捷方式，属性目标后写上
-AudioMixer -PixelStreamingIP=localhost -PixelStreamingPort=8888
![image](https://user-images.githubusercontent.com/53400642/231703865-268717da-9960-460a-a394-a7aca4e34d91.png)
![image](https://user-images.githubusercontent.com/53400642/231703908-861e9b4a-dae2-4c9e-8d37-686b1f3aed0e.png)
### 生成代码并运行
然后找到这个路径，项目名称\Samples\PixelStreaming\WebServers，运行一下get_ps_server.sh文件，就会生成几个文件夹和md文件（bat也点过，没反应，如果有人点bat成功的，那请自行操作）。然后点击，项目名称\Samples\PixelStreaming\WebServers\SignallingWebServer\platform_scripts\cmd 文件目录中的run_local.bat。
![image](https://user-images.githubusercontent.com/53400642/231703968-7c889600-1353-49e8-9b94-4388f7e035ce.png)
![image](https://user-images.githubusercontent.com/53400642/231704005-cc3d3279-94f1-475a-b074-d9ed6cc13baf.png)
### 运行exe
运行一下刚创建的快捷方式，然后在80端口看看结果。
![image](https://user-images.githubusercontent.com/53400642/231704040-384ecaa4-bf6d-4f74-a7ba-9a4304fdd972.png)
## 前端处理
### 两个文件
首先我们找到打包目录下，我的项目\Samples\PixelStreaming\WebServers\SignallingWebServer\scripts中这两个文件复制一下。

![image](https://user-images.githubusercontent.com/53400642/231704138-229bb1c4-f892-466f-b2a1-a3eba5db4026.png)
然后复制一下到Vue脚手架，app.js然后改个名字（别问为什么，问就是别人也改）。

![image](https://user-images.githubusercontent.com/53400642/231704193-9a6585ef-6222-48fa-83d2-aa35a5233498.png)

webRtcPlayer.js最后加上
```js
export default webRtcPlayer
```
webRtcVideo.js
前面加上
```js
import webRtcPlayer from './webRtcPlayer';
let connection_Url = 'ws://localhost/'//默认连接
```
中间修改
```js
//let connectionUrl = window.location.href.replace('http://', 'ws://').replace('https://', 'wss://');
let connectionUrl = connection_Url
```
后面加上
```js
function initLoad(options) {
    if(options.url) {
        connection_Url = options.url;
    }
    populateDefaultProtocol()
    registerMessageHandlers()
    registerKeyboardEvents()
    connect()
    startAfkWarningTimer()
}


var webRtcVideo = {
    initLoad: initLoad
}

export default webRtcVideo
```
App.vue
```js
<template>
  <div id="app">
    <div   ref="video" id="player"></div>
  </div>
</template>

<script>
import webrtc from "./webRtcVideo";
export default {
  name: 'App',
  mounted() {
    webrtc.initLoad({url:'ws://localhost/'})
  }
}
</script>

<style>
html{
  width: 100%;
  height: 100%;
}
body{
  width: 100%;
  height: 100%;
  margin: 0;
  overflow:hidden
}
#app {
  width: 100%;
  height: 100%;
}

</style>


```

### 网站声音修改
网站设置，声音允许。

![image](https://user-images.githubusercontent.com/53400642/231704397-ac503427-6e04-4480-b503-06cf6f795380.png)
![image](https://user-images.githubusercontent.com/53400642/231704441-bbb68068-a0d3-41e8-8856-5dfaf94b6efb.png)

# 成果展示
![image](https://user-images.githubusercontent.com/53400642/231704482-e8537d2f-3727-4f7b-8ce8-ad41f66f2edb.png)
# 遇到的问题
## 打包失败
1、SDK缺失，可以上官网下载window的sdk。
![image](https://user-images.githubusercontent.com/53400642/231704518-35e1acf8-f56a-4f71-95eb-15962406ef88.png)

2、does not look like uproject file but no targets have been found。
CompilationResultException: Error: OtherCompilationError  

at UnrealBuildTool.ActionGraph.ExecuteActions(BuildConfiguration BuildConfiguration, List`1 ActionsToExecute, List`1 TargetDescriptors) in d:\build\++UE5\Sync\Engine\Source\Programs\UnrealBuildTool\System\ActionGraph.cs.........................
这两个问题我忘了，一个修改了项目名字解决，就是不能有test或者中文。另一个是安装vs2022解决，安装的时候一定要选那个c++游戏什么的。
## Cesium For Unreal
1、首先是刚提及的网络问题，多认证几次直到出现成功界面。
2、地图瓦片加载问题，目前除了作为地形图的底图加载好像并没有另外的方法加载瓦片，每个地形限制为三，而且叠加多个地形来加载瓦片时候，会出现闪烁问题。可能需要修改c++源码实现，同一个地形多个瓦片。如果还有别的方法希望有大佬能提一下。
## 端口
1、修改端口：目前我仅是使用80端口去操作，在研究后应该是可以实现端口的修改。
2、同步问题：exe和网页上操作是同步进行的，这个可能需要后续看看如何解决多个用户使用间的不冲突问题。
## 前端
1、element缺失：因为Pixel Streaming的代码不单单是展示场景，还有一些参数可调，还有一些参数获取输出展示等，他都有写在js中，如果报错需要一一注释一下。
2、video自动加载问题：因为我想进入页面就直接加载场景，不想通过点击后来加载，所以省去了点击交互一步，但报错了，查了一下网上说是浏览器问题。
DOMException: play() failed because the user didn‘t interact with the document first
先尝试了调整video的muted参数，发现调了没反应（或许是没调对）。后直接修改网站声音为允许实现了谷歌浏览器上网页的加载。
# 结尾
这只是一小步，在UE5的使用上基本不会，后面也会尝试学习，但毫无疑问UE5功能是十分强大的，仿真程度一流，后面随着软硬件技术发展应该在GIS行业会越来越流行。
参考：
vue3集成ue5像素流自定义网页: [https://blog.csdn.net/qq_33377547/article/details/125477517](https://blog.csdn.net/qq_33377547/article/details/125477517)
ue5像素流web自动打开画面: [https://blog.csdn.net/qq_41579327/article/details/128393432](https://blog.csdn.net/qq_41579327/article/details/128393432)
完结撒花，若需转载请注明本文链接，感谢观看。


















