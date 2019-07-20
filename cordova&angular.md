### ios编译后白屏
index.html内的 <base href="/" /> 地址改为  . 或者 ./

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
