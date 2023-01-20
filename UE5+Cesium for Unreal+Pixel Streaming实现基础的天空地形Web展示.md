# UE5+Cesium for Unreal+Pixel Streaming实现基础的天空地形Web展示
近期有在研究UE5游戏引擎在GIS上的应用，Cesium刚好有相关的插件能生成一个场景，但又想到不知如何应用到Web，于是参考网上一些大神的参考流程，结合Pixel Streaming提供的两个重要代码，实现了在Vue前端展示嵌入，借此分享记录。
# 软件和插件
下载安装UE5.1，VS2022
安装插件 Cesium for Unreal，Pixel Streaming
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c91a515f1184f8a83d7e8a58fbe75ea.png)
# 处理流程
## UE处理
### 加载插件
先打开UE，新建一个项目（最好不要有中文，尽量不包含test字样），勾选插件，选择确定后，会让你重启一下软件，选择重启。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d122e92b455146b78d9b6052ed0b757a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e713d0a4f8e41a39c19acaf6aaee288.png)
### Cesium认证
先加载天空试试效果，然后点击Connect to Cesium ion去认证信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3abd91092737428d9b55d0f0d1cf21d1.png)
没有账号的需要先注册一个，点击Allow，网络问题可能会加载有点慢，可以多试几次。
![在这里插入图片描述](https://img-blog.csdnimg.cn/15fd4be9cc244439a1b41bbea2d78c4a.png)
出现以下提示代表认证可以了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3fd3b88358eb4d7ea9510b2f94b31cbe.png)
添加一下世界地形和底图，第一次加载会提醒你使用密钥，申请一个就好，可以在实例中添加自己的底图，不过好像只能叠加3个，分别对应Marterial Layer Key属性Overlay0，Overlay1，Overlay2（本来想用作地图服务加载，发现只有3个不够用），这里就不作修改了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c60e0ff8053c4a0b9ba1c6dcad3e340e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/39210e7709b249d8a595b76d047cc3ff.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/166bbc69f62f48029dc981c7e7125953.png)
### 保存关卡，打包
保存一下关卡，编辑打开项目设置，改一下编辑器开始地图和默认地图，改成刚才保存那个。
![在这里插入图片描述](https://img-blog.csdnimg.cn/de0c9a3bbade4458922cb73692aea28e.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/e3d27eddfb554390a23e1763a563eca8.png)
同样是编辑打开编辑器偏好设置，修改一下额外启动参数。
-AudioMixer -PixelStreamingIP=localhost -PixelStreamingPort=8888。
![在这里插入图片描述](https://img-blog.csdnimg.cn/314a7386b93047bd8eba8aa1c280f343.png)
然后打包项目。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae67afbd96a64cd4a2c052697776ecf3.png)
## 文件处理
### 创建快捷方式
找到打包的目录首页，给原本exe文件拉个快捷方式，属性目标后写上
-AudioMixer -PixelStreamingIP=localhost -PixelStreamingPort=8888
![在这里插入图片描述](https://img-blog.csdnimg.cn/2730fd5a5ed34345aa01b3f93f8bc5d8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fbcccc7a77ad4b2890de4817176c2afa.png)
### 生成代码并运行
然后找到这个路径，项目名称\Samples\PixelStreaming\WebServers，运行一下get_ps_server.sh文件，就会生成几个文件夹和md文件（bat也点过，没反应，如果有人点bat成功的，那请自行操作）。然后点击，项目名称\Samples\PixelStreaming\WebServers\SignallingWebServer\platform_scripts\cmd 文件目录中的run_local.bat。
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ff8c65d397442d3acfb6eb077828c2e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/88f9beff3603481685de6ff576275dcf.png)
### 运行exe
运行一下刚创建的快捷方式，然后在80端口看看结果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/da582db5e1c44934b46ec5110fb5750e.png)
## 前端处理
### 两个文件
首先我们找到打包目录下，我的项目\Samples\PixelStreaming\WebServers\SignallingWebServer\scripts中这两个文件复制一下。
![在这里插入图片描述](https://img-blog.csdnimg.cn/23f811ad940e49d799beeb693c99481f.png)
然后复制一下到Vue脚手架，app.js然后改个名字（别问为什么，问就是别人也改）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/22ca9930206f4a12be36b3e3963fc1ba.png)
webRtcPlayer.js
最后加上
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ee6615f9adb479789ff20e4463bcda6.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f482b35972fd448888669707449206e4.png)
# 成果展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/55ee9a283b4645bb874d0fa3222a5134.png#pic_center)
# 遇到的问题
## 打包失败
1、SDK缺失，可以上官网下载window的sdk。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f9a3086c05e43c1aca75395b9c79ed7.png)

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


















