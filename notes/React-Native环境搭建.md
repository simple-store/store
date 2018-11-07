### React-Native环境搭建

> 最近要做一个移动端的app项目，所以在办公电脑上安装了一下rn的环境
>
> 本人用的是mac噢，所以环境都是以mac为标准

- 下载homebrew

  ```shell
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

- 安装node

  ```shell
  brew install node
  ```

- 安装react-native-cli

  > 本人用的是yarn

  ```shell
  yarn add global react-native-cli
  ```

- 去AppStore安装xcode



- 打开xcode初始化

  > 需要同意证书以及安装一些组件



#### 到这里我们就可以启动一个新的react-native项目啦



- 初始化项目

  ```shell
  react-native init myproject
  ```

- 进入项目文件夹

  ```shell
  cd myproject
  ```

- 运行

  > 在项目文件夹下 myproject/

  ```
  react-native run-ios
  ```



本人在运行的时候遇到一个错误，信息如下

```
'React/RCTBundleURLProvider.h' file not found
```

这里你需要用xcode打开项目文件夹下的ios目录

- 双击右边BuildSetting

- 搜索"Header Search Path"

- 在其下面添加

  ```shell
  $(SRCROOT)/../node_modules/react-native/React
  
  ```

- 重新运行react-native run-ios

这个时候你就可以看到react-native的欢迎语了，恭喜你。





#### 安卓端的启动

> 必须使用Android Studio下载安装sdk以后才可以正常运行

- 安装Android Studio

- 下载需要的sdk（建议从4.2开始下载）

- 打开项目下的android目录

- 待gradle自动安装好依赖以及注入sdk

- 连接手机（或者安装官方推荐的generation模拟器）

- 运行react-native run-android

  