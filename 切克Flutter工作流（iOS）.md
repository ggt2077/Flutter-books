# 切克Flutter工作流

## 一、Flutter侧怎么玩

### 1.1 Flutter侧项目结构

我们目前使用module的形式开发flutter侧的功能，项目结构如下：

> .android: 安卓工程，可独立打开运行；
> 
> .ios: iOS工程，可独立打开运行。如直接在Android Stdio中运行至iOS设备，需打开此项目配置证书；
> 
> build: 编译产物
> 
> lib: 代码文件
> 
> * main.dart: 程序入口
> * CKGoodDetail: 切克商品详情页
> * ZZLib: Flutter侧基础库
> 
> images: 资源文件
> 
> pubspec.yaml: 项目配置文件，主要包含资源指定、三方库依赖设置
> 

### 1.2 Flutter侧基础库

目前已有部分基础库如下：

> ZZBridge: [重要]Flutter与Native通信基础库 
> 
> ZZRouter: 路由，基于ZZBridge
> 
> ZZWidgets: 依赖了其它多个基础UI库，如ZZColors、ZZAlert、ZZNavBar、ZZTabBar、ZZLoding、ZZToast
> 
> ZZLoginUtil: 登陆，基于ZZBridge
> 
> ZZReport: 日志上报，基于ZZBridge
> 
> ZZNetwork：网络请求
> 
> ZZShare: 分享，基于ZZBridge
> 
> ZZImageBrowser: 选图，基于ZZBridge
> 

### 1.3 Flutter侧JSON转Model

Flutter侧实现Json和Model的相互转换，引入了一个第三方库```json_annotation```，其原理是通过一个自动生成的中间类进行转换，这个中间类实现了手动转换的过程。具体步骤如下：

1.3.1 在Model类（如People类）中先行引入中间类(People.g.dart)，增加```@JsonSerializable()```注解；

1.3.2 执行指令生成中间类，已封装成项目根目录下的```create_model.sh```脚本方便执行；

1.3.3 增加```toJson``````formJson```方法调用中间类的转换方法；

### 1.4 Flutter侧的异步

Flutter侧的异步与iOS侧不同，dart有await、async关键词，用写同步的方法写异步，用起来更加简单；

```
  void showPageIfNeed() async {
    if (!await ZZLoginUtils.haveLogged()) {
      // 没有登录，需要去登录
      ZZLoginUtils.gotoLogin((bool success) {
        if (success) {
          showPage();
        }
      });
    }
    else {
    	showPage();
    }
  }
```

## 二、iOS侧如何接入Flutter

Flutter侧需求开发完成后，直接执行项目根目录下的```build_ios.sh```，done~

```build_ios.sh```脚本做了什么，大致说就是将Flutter项目编译出iOS Release产物，然后连同其所有依赖一同集成进iOS项目。

为做到无感知接入，我们会将Flutter产物更新至ZZFlutterLib_iOS组件库，Navtive项目引入该组件库。

```build_ios.sh```详细工作流程如下：

### 2.1 编译Flutter项目

主要指令为：

```
flutter build ios --release
```

编译产物存放在```.ios\Flutter```文件夹下，依赖库描述在```.flutter-plugins```中。

##### 关于三方库：

* Flutter侧可以在```pubspec.yaml```中添加依赖的三方库，此文件类似iOS的```Podfile```。
* Flutter侧依赖的部分库，会存在Native侧对应的Pod库，用于的通信。如相册库，需要native侧Pod库获取相册信息，Flutter侧库处理和展示。
* Native侧需要引入库的信息，存放在```.flutter-plugins```文件中，需要手动解析，添加到主项目。该操作实际由```build_ios.sh```自动完成，见下文。

### 2.2 将编译产物更新至ZZFlutterLib_iOS组件

此过程分为以下4步：

2.2.1 判断zzfutter项目的上级目录下，是否存在ZZFlutterLib_iOS组件，如不存在将自动克隆远端仓库；

2.2.2 更新Flutter引擎库（Flutter）；

2.2.3 更新Flutter业务产物库（App）；

2.2.4 更新插件管理组件（FlutterPluginRegistrant）；

2.2.5 解析并拷贝所有```.flutter-plugins```依赖库；

至此，所有Flutter相关文件，均在ZZFlutterLib_iOS库中。

### 2.3 修改Native插件库代码，解决依赖冲突

转转有将FMDB自行封装为Common_IMDB库，显然二者无法同时存在。

此时```build_ios.sh```会调用```build_ios_modify.py```，对Native插件库中所有类似的信息进行检索和替换。

### 2.4 展示Podfile接入代码

根据1.2中信息生成，直接复制此段至Podfile即可，如有必要可修改git分支名。

### 2.5 打开ZZFlutterLib_iOS Demo项目

会更新此组件Demo项目pod库，然后自动打开Demo项目用于验证Release效果。该步骤非必须。

另，实际重新build后，无需更新pod，直接重新运行项目即可看到效果。

## 三、iOS侧如何调用和交互

### 3.1 ZZFlutterLib_iOS

Flutter产物库

### 3.2 ZZFlutterAdapter

> ZZFlutterAdapter: Flutter适配器、插件注册
>
> ZZFlutterRouter: Flutter页面路由控制
> 
> ZZFlutterViewController：Flutter页面控制器，可以直接使用
> 
> ZZFlutterPluginBridge：通信控制中心

### 3.3 CKFlutterPlugin

所有插件，插件内包含Flutter通信节点事件的注册，主要结构：

> ZZFlutterPluginManager：插件注册及管理
>
> ZZFlutterCommonPlugin: 通用