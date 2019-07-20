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

angular内容加载路径不对导致的。 解法, index.html内的 <base href="/" /> 地址改为  . 或者 ./

不要直接修改index.html中的<base href>, 可以通过ng build后增加 --base-href='.' 参数在编译时进行修改

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

### http请求出现Content-Security-Policy错误
index.html内 屏蔽 <meta http-equiv="Content-Security-Policy" content=...>


### cordova deviceready事件的处理
在index.html内添加 <script type="text/javascript" src="cordova.js"></script>

在main.ts内将bootstrapModule(AppModule)延迟到deviceready后加载
```
document.addEventListener('deviceready', () => 
   platformBrowserDynamic().bootstrapModule(AppModule), false);
```


### device变量识别问题
在componet内添加 `declare var device`;

```
ngOnInit() {
    document.addEventListener("deviceready", onDeviceReady, false);
    function onDeviceReady() {
       alert(device.platform);
    }
}
```
