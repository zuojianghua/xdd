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

可以直接使用 `window.openDatabase('database.db', '1.0', 'APP database', 10*1024*1024)`

```
### WebSqLObject ##############################
// 将回调改为promise

class WebSqLObject {
    db: any;

    constructor(db: any) {
        this.db = db;
    }

    public executeSql(queryStatement: string, params?: any[]): Promise<any> {
        return new Promise((resolve, reject) => {
            this.db.transaction((tx) => {
                tx.executeSql(queryStatement, params,
                    (ctx, result) => {
                        resolve(result);
                    },
                    (error) => reject(error));
            });
        });
    }
}

### SqliteService ##############################

@Injectable({
    providedIn: 'root'
})
export class SqliteService {

    database: WebSqLObject;

    constructor() {}
    
    // 建库语句示例
    createDatabase() {
        const db = window.openDatabase('message.db', '1.0', 'my app database', 10 * 1024 * 1024);
        this.database = new WebSqLObject(db);

        return Promise.all([
            this.database.executeSql(`建表语句A`,[])
                .then(() => console.log('表创建成功'))
                .catch(e => console.log(e)),
            this.database.executeSql(`建表语句B`,[])
                .then(() => console.log('表创建成功'))
                .catch(e => console.log(e)),
            this.database.executeSql(`建表语句C`,[])
                .then(() => console.log('表创建成功'))
                .catch(e => console.log(e)),
        ]);
    }
    
    // 查询语句示例
    getInfo(id): Promise<any> {
        return this.database.executeSql('SELECT count(*) FROM tb1 WHERE status=0 and id = ?', [id]);
    }

```

### 使用localstorage进行数据存储

可以直接使用window.localStorage
```
// 存json数据
window.localStorage.setItem('key', JSON.stringify(data));

// 取json数据
const data = JSON.parse(window.localStorage.getItem(key));
```

### 拨打电话

安装cordova插件, [插件说明](https://github.com/Rohfosho/CordovaCallNumberPlugin)

`$ npm install call-number --save`

```
  openCallPhoneNumber(tel) {
    if (window.plugins === undefined) {
        console.log('web模式不支持拨打电话');
        return;
    }

    window.plugins.CallNumber.callNumber((result) => {
        console.log('拨打电话成功:', result);
    }, (error) => {
        console.log('拨打电话失败:', error);
    }, tel, true);
  }
```


### 返回到上一页及白屏卡顿处理

返回上一页可以使用: 
* angular的Location对象back()方法
* window.history.go(-1);
* angular的router.navigateByUrl(url); 需要提前记录下上一页面的路由地址

```
import { Location } from '@angular/common';

@Injectable({
    providedIn: 'root'
})
export class TitleService {

  constructor(
      private location: Location
  ) {}
  
  goBack() {
        // window.history.go(-1);
        this.location.back();
    }
}
```

 > 注：***返回时可能会造成页面卡顿/白屏，解决方法在`src/index.html`的`<header>`中加入***

```
<script>
    // 修复页面返回在真机上白屏卡顿
    window.addEventListener = function () {
        (window.EventTarget || Window).prototype.addEventListener.apply(this, arguments);
    };
    window.removeEventListener = function () {
        (window.EventTarget || Window).prototype.removeEventListener.apply(this, arguments);
    };
</script>

```

### iOS应用的检查更新

已经发布到苹果市场的应用直接使用 `window.open('itms-apps://itunes.apple.com/app/idxxxxxxxxx', '_system', 'location=yes');` 打开APPStore的相应页面。 此处idxxxxxxxxx需要发布后才有

```
// 检查版本自动更新
async checkUpdate() {
    let pf;
    try {
        pf = device.platform.toLowerCase();
    } catch (error) {
        // 简单兼容一下通过浏览器调试时的报错
        // device.platform 更多返回查看 https://cordova.apache.org/docs/en/latest/reference/cordova-plugin-device/index.html#deviceplatform
        pf = 'web';
    }

    if (pf === 'android') {
        // 安卓自动检查下载apk
        await this.androidUpdateService.autoUpdate(false);
        
    } else if (pf === 'ios') {
        // iOS跳转到APP Store页面
        window.open('itms-apps://itunes.apple.com/app/idxxxxxxxxx', '_system', 'location=yes');
        
    }
}
```

### 安卓应用的检查更新和自动更新

安卓的自动更新相比iOS更麻烦, 如将打包好的apk放到自有的web服务器上, 可以通过比较版本号和文件下载来实现。

1. 将apk文件放到web服务器上, 同时还有含有版本号信息的json文件
2. 通过`cordova-plugin-app-version`插件获取当前app的版本号并与线上版本进行比较
3. 通过`cordova-plugin-file-transfer`插件进行文件下载。其中需要用到`cordova-plugin-file`插件找到保存路径
4. 通过`cordova-plugin-file-opener2`插件打开下载的文件, 即请求安装
5. 下载的apk与当前运行的apk需要保证签名一致

***说明：***

environment.apk_url 为线上的版本信息地址。例如 https://127.0.0.1/ver.json 返回的内容为:
```
{
    "ver": "0.3.2",
    "apk": "app-release-0.3.2.apk",
    "url": "https://127.0.0.1/app-release-0.3.2.apk",
    "desc": "版本更新内容描述"
}
```

***安装相应的插件：***

```
// https://github.com/whiteoctober/cordova-plugin-app-version
$ cordova plugin add cordova-plugin-app-version

// https://github.com/apache/cordova-plugin-file-transfer
$ cordova plugin add cordova-plugin-file-transfer

// https://github.com/apache/cordova-plugin-file
$ cordova plugin add cordova-plugin-file

// https://github.com/pwlin/cordova-plugin-file-opener2
$ cordova plugin add cordova-plugin-file-opener2
```

***代码实现：***

```
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams, HttpHeaders } from '@angular/common/http';
import { environment } from '../../../environments/environment';
import { throwError, Observable } from 'rxjs';
import { catchError, map } from 'rxjs/operators';
import { ModalService, ToastService } from 'ng-zorro-antd-mobile';


declare const cordova;
declare const device;
declare const FileTransfer;

@Injectable({
    providedIn: 'root'
})
export class AndroidUpdateService {
    constructor(
        private http: HttpClient,
        private modalService: ModalService,
    ) {
    }
    
    /**
     * 检查自动更新
     * @param quiet: boolean true-安静模式, 不提示  false-提示当前是否为最新版本
     */
    async autoUpdate(quiet = true) {
        // 仅适用于android -------------------------------------------------------------
        let pf;
        try {
            pf = device.platform.toLowerCase();
        } catch (error) {
            pf = 'web';
        }

        if (pf !== 'android') {
            return;
        }
        // TODO 安卓权限申请 --------------------------------------------------------

        // 比较当前版本是否为最新版本 -------------------------------------------------
        console.log(`检查地址: ${environment.apk_url}`);

        this.http.get(`${environment.apk_url}`).subscribe(
            async (res: any) => {
                const nowVersion = await cordova.getAppVersion.getVersionNumber();

                // versionCompare参考 https://gist.github.com/TheDistantSea/8021359 实现
                // 负数代表 v1 小于 v2
                const newVersion = this.versionCompare(res.ver, nowVersion, {
                    lexicographical: true,
                    zeroExtend: true,
                });

                // 发现存在船信版本后的操作 -------------------------------------------
                if (newVersion > 0) {
                    // res.ver     版本号
                    // res.desc    更新内容描述
                    // res.url     apk文件下载地址
                    ModalService.alert(`发现新版本${res.ver}`, '如选择立即更新将在后台进行下载, 下载后自动安装更新。是否立即更新? ', [
                        { text: '下次再说', onPress: () => console.log('自动更新下次再说')},
                        { text: '立即更新', onPress: () => this.update(res.url) }
                    ]);
                } else if (!quiet) {
                    ModalService.alert(`当前已是最新版本`, `版本号为${nowVersion}`, [
                        { text: 'Ok', onPress: () => console.log('已是最新版本') }
                    ]);
                }
            },
            error => console.log('自动更新请求出错', error)
        );
        
    }
    
    
    /**
     * 下载及更新
     * @param url: string 文件下载地址
     */
    async update(url) {
        console.log(cordova.file.dataDirectory);

        const fileTransfer = new FileTransfer();
        const uri = encodeURI(url);
        const localDir = cordova.file.dataDirectory;
        const fileURL = `${localDir}myApp.apk`;

        fileTransfer.download(
            uri,
            fileURL,
            (entry) => {
                console.log('APK下载完成: ' + entry.toURL());

                // 下载完成后进行APK安装
                cordova.plugins.fileOpener2.open(
                    entry.toURL(),
                    'application/vnd.android.package-archive'
                );
            },
            (error) => {
                console.log('下载出错 source ' + error.source);
                console.log('下载出错 target ' + error.target);
                console.log('下载出错 code '   + error.code);
            },
            true
        );

    }
    
    
    /**
     * 版本号比对方法
     * @link https://gist.github.com/TheDistantSea/8021359
     * @param v1 版本号 '1.0.0'
     * @param v2 版本号 '1.2.3'
     * @param options [options] Optional flags that affect comparison behavior:
     * <ul>
     *     <li>
     *         <tt>lexicographical: true</tt> compares each part of the version strings lexicographically instead of
     *         naturally; this allows suffixes such as "b" or "dev" but will cause "1.10" to be considered smaller than
     *         "1.2".
     *     </li>
     *     <li>
     *         <tt>zeroExtend: true</tt> changes the result if one version string has less parts than the other. In
     *         this case the shorter string will be padded with "zero" parts instead of being considered smaller.
     *     </li>
     * </ul>
     * @returns number | NaN
     * <ul>
     *    <li>0 if the versions are equal</li>
     *    <li>a negative integer iff v1 < v2</li>
     *    <li>a positive integer iff v1 > v2</li>
     *    <li>NaN if either version string is in the wrong format</li>
     * </ul>
     */
    private versionCompare(v1, v2, options) {
        const lexicographical = options && options.lexicographical,
            zeroExtend = options && options.zeroExtend;
        let v1parts = v1.split('.'),
            v2parts = v2.split('.');

        function isValidPart(x) {
            return (lexicographical ? /^\d+[A-Za-z]*$/ : /^\d+$/).test(x);
        }

        if (!v1parts.every(isValidPart) || !v2parts.every(isValidPart)) {
            return NaN;
        }

        if (zeroExtend) {
            while (v1parts.length < v2parts.length) {
                v1parts.push('0');
            }
            while (v2parts.length < v1parts.length) {
                v2parts.push('0');
            }
        }

        if (!lexicographical) {
            v1parts = v1parts.map(Number);
            v2parts = v2parts.map(Number);
        }

        for (let i = 0; i < v1parts.length; ++i) {
            if (v2parts.length === i) {
                return 1;
            }

            if (v1parts[i] === v2parts[i]) {
                continue;
            } else if (v1parts[i] > v2parts[i]) {
                return 1;
            } else {
                return -1;
            }
        }

        if (v1parts.length !== v2parts.length) {
            return -1;
        }

        return 0;
    }
    
}
```

### 禁用iOS沉浸式状态栏

沉浸式状态栏会遮盖住APP顶部Title的一部分位置，可以在config.xml中加入以下配置禁用

更多StatusBar设置查看 https://cordova.apache.org/docs/en/latest/reference/cordova-plugin-statusbar/index.html

```
<widget ... >
  ...
  <!-- 状态栏 -->
  <preference name="StatusBarOverlaysWebView"    value="false" />
  <preference name="StatusBarDefaultScrollToTop" value="false" />
  <preference name="StatusBarBackgroundColor"    value="#108ee9" />
  ...
</widget>
```

### 禁用设备旋转

设备旋转时, 画面不会跟随设备自动进行旋转。 在config.xml中加入 

```
  <!-- 禁用画面旋转 -->
  <preference name="orientation" value="portrait" />
```

### 禁用页面弹动

config.xml中加入

```
  <!-- 禁用页面上拉下拉 -->
  <preference name="WebViewBounce" value="false" />
  <preference name="DisallowOverscroll" value="true" />
  
```

### 页面上禁用双击选定

global.css中加入全局设置, 禁用双击选定, 可针对input/textarea或者允许选中的元素进行例外
```
// 禁用双击选定
*{
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

input, textarea{
    -webkit-user-select: auto !important;
    -khtml-user-select: auto !important;
    -moz-user-select: auto !important;
    -ms-user-select: auto !important;
    user-select: auto !important;
}
```

### 让web页面滚动效果更顺滑

global.css中加入scrolling类, 在页面使用到滚动的container元素上应用

```
.scrolling{
    -webkit-overflow-scrolling: touch;
}
```

### 响应安卓系统的返回按钮事件

可以通过监听页面的backbutton事件进行干预, 如不处理默认是返回上一页。详细事件查看 https://cordova.apache.org/docs/en/latest/cordova/events/events.html#backbutton

```
import { Location } from '@angular/common';

@Component({
    selector: 'app-root',
    templateUrl: 'app.component.html'
})
export class AppComponent {

  constructor(
    private location: Location,
  ){
    this.backButtonEvent();
  }

  backButtonEvent() {
      document.addEventListener('backbutton', (evt) => {
          console.log('返回按钮点击');
          
          // 需要尝试关闭掉 模态框Modal, 动作面板ActionSheet, 侧面菜单Drawer, 打开的菜单Menu
          if (this.opened.openedModal) {
              // 示例: 取出当前已经打开的模态框示例，调用close方法关闭, 这样页面返回到上一页后不至于弹框还停留在画面上
              // 此处 ng-zorro-antd-mobile 文档与实际实例返回预期不符, 所以直接阻止掉返回
              // this.opened.openedModal.close();
              
              console.log('阻止返回');
              evt.preventDefault();
              return;
          } 

          // TODO 如果在/login页面或者已经不能再返回到/login页面时，点击显示再次点击将退出APP

          this.location.back();
      }, false);
  }
}
```

## 编译和发布

### iOS编译

### iOS发布

### 安卓编译

#### 允许android9以上访问非https网址

找到 platforms/android/app/src/main/AndroidManifest.xml 文件, 在`<application>`中增加 `android:usesCleartextTraffic="true"`

效果如: `<application ...  android:usesCleartextTraffic="true" >`

 > 注意：该文件在`cordova platform rm android` / `cordova prepare android` 后可能会重置, 需要再次添加以上选项

### 安卓签名

***生成签名文件***

```keytool -genkey -v -keystore myApp.keystore -alias myApp.keystore -keyalg RSA -validity 36500```

***用签名文件为APP签名***

正常通过```cordova build android --release```编译出来的apk为未签名的`app-release-unsigned.apk`, 以下命令为apk签名

```jarsigner -verbose -keystore myApp.keystore -signedjar app-release.apk app-release-unsigned.apk myApp.keystore```

### 安卓发布


