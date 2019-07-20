# 使用cordova和angular开发跨平台APP

时间 2019.5～2019.8

## 技术选择

### 外壳

* [cordova](https://cordova.apache.org/) 9.0.1

### 前端

* 基础框架 [angular](https://angular.io/) 7.2.2
* UI部分 [ant design](http://ng.mobile.ant.design) for 移动端和angular的版本 0.12.5

### 其他
* ionic的问题, ionic4.0以后使用了ios wkwebview的一些特性, 导致不兼容ios11以下版本。详见ionic-webview的requirements列表,  https://github.com/ionic-team/cordova-plugin-ionic-webview#plugin-requirements

## 集成思路

1. 通过angular的ng build将ts代码发布到www目录
2. 在index.html中引入cordova.js
3. 通过cordova prepare ios / cordova prepare android 打包为ios / android应用项目 

## 问题和处理 

### 将angular build的目标目录设置为www

在angular.json中找到outputPath, 改为www
```
  "app": {
      ...
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "www",
            "index": "src/index.html",
            "main": "src/main.ts",
```

### 在src/index.html中引入cordova.js

```
<hear>
  ...
  <script type="text/javascript" src="cordova.js"></script>
</head>
```

 > 注: 用ng serve在浏览器中运行的时候会报找不到cordova.js, 但开发界面时没有重要影响。 cordova在浏览器中的标准做法是 `$ cordova platform add browser` 及 `$ cordova run browser`, 但不能监测文件热更新


### ios / android 编译后运行时白屏

angular内容加载路径不对导致的。 解法, index.html内的 `<base href="/" />` 地址改为  `.` 或者 `./`

不要直接修改index.html中的`<base href>`, 可以通过ng build后增加 `--base-href='.'` 参数在编译时进行修改(或在angular.json中修改)

package.json
```
  "scripts": {
    "start": "ng serve --host 0.0.0.0",
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "npm i && ng build",
    "pre-ios": "ng build --prod --base-href='.' && cordova prepare ios",
    "pre-android": "ng build --prod --base-href='.' && cordova prepare android"
  },
```

 > 影响: css文件中用到的background图片路径可能会受到影响。另外原页面中`/assets`绝对路径开头的图片地址可以改为`assets`相对路径

### cordova deviceready事件的处理

cordova中的底层交互和大部分插件都需要`cordova deviceready`事件后才可以运行, 因此需要将angular内容延迟加载

* 在index.html内添加 <script type="text/javascript" src="cordova.js"></script>

* 在src/main.ts内将bootstrapModule(AppModule)延迟到deviceready后加载
```
document.addEventListener('deviceready', () => 
   platformBrowserDynamic().bootstrapModule(AppModule), false);
```

之后在angular代码中可以使用全局变量`device`

### device变量识别问题
在用到的componet内添加 `declare var device;` 否则ts编译校验不通过

```
ngOnInit() {
  console.log(device.platform);
}
```

### http请求出现Content-Security-Policy错误
index.html内查看以下如果存在 `<meta http-equiv="Content-Security-Policy" content=...>` 屏蔽即可

### 为APP设置icon和开屏页面

`$ cordova plugin add cordova-plugin-splashscreen`

在config.xml中进行设置`<icon>`和`<splash>`

```
<widget id="...>
  <platform name="android">
  
        <icon density="ldpi" src="resources/android/icon/drawable-ldpi-icon.png" />
        <icon density="mdpi" src="resources/android/icon/drawable-mdpi-icon.png" />
        <icon density="hdpi" src="resources/android/icon/drawable-hdpi-icon.png" />
        <icon density="xhdpi" src="resources/android/icon/drawable-xhdpi-icon.png" />
        <icon density="xxhdpi" src="resources/android/icon/drawable-xxhdpi-icon.png" />
        <icon density="xxxhdpi" src="resources/android/icon/drawable-xxxhdpi-icon.png" />
        
        <splash density="land-ldpi" src="resources/android/splash/drawable-land-ldpi-screen.png" />
        <splash density="land-mdpi" src="resources/android/splash/drawable-land-mdpi-screen.png" />
        <splash density="land-hdpi" src="resources/android/splash/drawable-land-hdpi-screen.png" />
        <splash density="land-xhdpi" src="resources/android/splash/drawable-land-xhdpi-screen.png" />
        <splash density="land-xxhdpi" src="resources/android/splash/drawable-land-xxhdpi-screen.png" />
        <splash density="land-xxxhdpi" src="resources/android/splash/drawable-land-xxxhdpi-screen.png" />
        <splash density="port-ldpi" src="resources/android/splash/drawable-port-ldpi-screen.png" />
        <splash density="port-mdpi" src="resources/android/splash/drawable-port-mdpi-screen.png" />
        <splash density="port-hdpi" src="resources/android/splash/drawable-port-hdpi-screen.png" />
        <splash density="port-xhdpi" src="resources/android/splash/drawable-port-xhdpi-screen.png" />
        <splash density="port-xxhdpi" src="resources/android/splash/drawable-port-xxhdpi-screen.png" />
        <splash density="port-xxxhdpi" src="resources/android/splash/drawable-port-xxxhdpi-screen.png" />
        
    </platform>
    <platform name="ios">
    
        <icon height="57" src="resources/ios/icon/icon.png" width="57" />
        <icon height="114" src="resources/ios/icon/icon@2x.png" width="114" />
        <icon height="40" src="resources/ios/icon/icon-40.png" width="40" />
        <icon height="80" src="resources/ios/icon/icon-40@2x.png" width="80" />
        <icon height="120" src="resources/ios/icon/icon-40@3x.png" width="120" />
        <icon height="50" src="resources/ios/icon/icon-50.png" width="50" />
        <icon height="100" src="resources/ios/icon/icon-50@2x.png" width="100" />
        <icon height="60" src="resources/ios/icon/icon-60.png" width="60" />
        <icon height="120" src="resources/ios/icon/icon-60@2x.png" width="120" />
        <icon height="180" src="resources/ios/icon/icon-60@3x.png" width="180" />
        <icon height="72" src="resources/ios/icon/icon-72.png" width="72" />
        <icon height="144" src="resources/ios/icon/icon-72@2x.png" width="144" />
        <icon height="76" src="resources/ios/icon/icon-76.png" width="76" />
        <icon height="152" src="resources/ios/icon/icon-76@2x.png" width="152" />
        <icon height="167" src="resources/ios/icon/icon-83.5@2x.png" width="167" />
        <icon height="29" src="resources/ios/icon/icon-small.png" width="29" />
        <icon height="58" src="resources/ios/icon/icon-small@2x.png" width="58" />
        <icon height="87" src="resources/ios/icon/icon-small@3x.png" width="87" />
        <icon height="1024" src="resources/ios/icon/icon-1024.png" width="1024" />
        
        <splash height="1136" src="resources/ios/splash/Default-568h@2x~iphone.png" width="640" />
        <splash height="1334" src="resources/ios/splash/Default-667h.png" width="750" />
        <splash height="2208" src="resources/ios/splash/Default-736h.png" width="1242" />
        <splash height="1242" src="resources/ios/splash/Default-Landscape-736h.png" width="2208" />
        <splash height="1536" src="resources/ios/splash/Default-Landscape@2x~ipad.png" width="2048" />
        <splash height="2048" src="resources/ios/splash/Default-Landscape@~ipadpro.png" width="2732" />
        <splash height="768" src="resources/ios/splash/Default-Landscape~ipad.png" width="1024" />
        <splash height="2048" src="resources/ios/splash/Default-Portrait@2x~ipad.png" width="1536" />
        <splash height="2732" src="resources/ios/splash/Default-Portrait@~ipadpro.png" width="2048" />
        <splash height="1024" src="resources/ios/splash/Default-Portrait~ipad.png" width="768" />
        <splash height="960" src="resources/ios/splash/Default@2x~iphone.png" width="640" />
        <splash height="480" src="resources/ios/splash/Default~iphone.png" width="320" />
        <splash height="2732" src="resources/ios/splash/Default@2x~universal~anyany.png" width="2732" />
    </platform>
</widget>       
```

 > 其中`resources`目录内存放不同大小分辨率下的icon和开屏页图片
 
### 使用websql进行数据存储

### 使用localstorage进行数据存储

### 拨打电话

`$ npm install call-number --save`

### 返回到上一页及白屏卡顿处理

### iOS应用的检查更新

### 安卓应用的检查更新和自动更新

### 禁用iOS沉浸式状态栏

### 响应安卓系统的返回按钮事件






