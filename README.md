# yuedu
香色闺阁、微信重签名实战

## 背景

博客地址：[iPA重签名 + 香色闺阁、微信重签名实战](https://morganwang.cn/2023/07/14/ipa%E9%87%8D%E7%AD%BE%E5%90%8D/#more)

换手机之后，原来的香色闺阁不能下载，转移的时候就丢失了。又最近看到有人截屏iPhone上装了两个微信，一个是自己重签名的；想到自己有开发者账号，但是还没试过重签名APP，是不是可以通过重签名来安装香色闺阁，以及实现多开微信？

<!--more-->

## 步骤

说干就干，首先从简单的开始，先来尝试重签名香色闺阁，再尝试微信，因为香色闺阁的包内容比微信的少很多，文件容易找，相对简单，步骤如下：

### 重签名香色闺阁

首先从网上搜素下载香色闺阁的 ipa 包，没有的可以从这里[yuedu.ipa](https://github.com/mokong/yuedu/blob/main/original_iPA/yuedu.ipa)下载。

然后将 ipa 改为 zip，然后解压，会出现一个 Payload 文件夹，里面有一个`StandarReader.app`的文件。选中`StandarReader.app`，右键显示包内容，可以看到APP包中的所有内容，如下图：

![ipa解压](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230714-174131.png)


![显示包内容](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230714-174206.png)


![package content](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230714-174305.png)

重签名就是用自己的账号新建一个APP（bundleID自己定义），运行到手机上，找到运行的包(`xxx/DriveData/xxx/Build/Products/Debug-iphoneos/xxx.app`)或者通过打包的方式，使用`AdHoc`方式生成一个iPA（用于获取到描述文件和`Entitlements.plist`），这里需要注意选择的描述文件包含要安装的设备，然后把待签名的iPA中的`bundleID`、证书和描述文件，替换成自己创建的APP的。

所以通常需要修改的有以下内容：
- Info.plist 中的 bundleID
- embedded.mobileprovision 签名文件
- _CodeSignature中的内容
- 因为普通账号不能对插件进行签名，在包内容路径中找到Watch和PlugIns文件夹直接删除
- 重签 framework

而针对香色闺阁的 ipa 来说，没有插件，没有 framework，所以需要修改的文件有：`Info.plist`、`embedded.mobileprovision`、`_CodeSignature`中的内容，下面具体来看如何修改：

#### 修改Info.plist中的bundleID

找到`Payload/StandarReader.app`中的Info.plist，用 Xcode 打开，或者其他编辑器打开，找到`Bundle identifier`，可以看到香色闺阁的bundleID为`com.appbox.StandarReader`，替换`com.appbox.StandarReader`为自己创建APP的bundleID， 如下图：

Ps: 如果不是Xcode打开可以直接搜索替换

![替换bundleID](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230714-181407.png)

#### 替换`embedded.mobileprovision`

从自己新建APP的iPA中获取到新的`embedded.mobileprovision`，步骤同上：改为zip -> 解压缩 -> 查看包内容，找到`embedded.mobileprovision`拷贝出来，可以放到要替换的`Payload`目录的外层，如下图:

![新的embedded.mobileprovision](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230714-180247.png)

然后用这个新的替换原APP中`embedded.mobileprovision`，可以通过命令行，如下：

1. 删除原APP中的描述文件
    ``` sh
    rm -rf Payload/xxx.app/embedded.mobileprovision
    ```
2. 把新的描述文件，放入xxx.app中
    ``` sh
    cp embedded.mobileprovision Payload/xxx.app/
    ```

#### 重新签名

从自己创建的APP的`embedded.mobileprovision`中获取`Entitlements.plist`，*注意是自己创建的APP中获取，而不是香色闺阁的包中获取*，然后删除香色闺阁包中的`_CodeSignature`，再用生成的`Entitlements.plist`给香色闺阁包生成新的签名，具体步骤如下：

1. 生成`Entitlements.plist`
   ``` bash
   /usr/libexec/PlistBuddy -x -c "print:Entitlements " /dev/stdin <<< $(security cms -D -i xxx.app/embedded.mobileprovision) > Entitlements.plist
   ```
2. 删除香色闺阁包中的`_CodeSignature`
   ``` bash
   rm -rf Payload/xxx.app/_CodeSignature/
   ```
3. 用新的`Entitlements.plist`给香色闺阁包生成新的签名
   ``` bash
   // 首先获取证书名称
   security find-identity -v -p codesigning
   // 选择创建APP时使用的证书进行签名
   codesign -f -s "证书名称" --entitlements entitlements.plist Payload/xxx.app
   ```

#### 打包，安装

将`Payload`文件夹在压缩为`ipa`文件，命令如下：

``` bash
zip -r xxx.ipa Payload/
```

最后安装，可以选中Xcode，然后选择 Windows -> Device And Simulators，或者选中Xcode后使用快捷键`Shift+CMD+2`打开，选中设备，点击`+`，选择生成的ipa，即可安装。

![Xcode安装ipa](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230717-092647.png)

### 重签名微信

第一步，获取微信的 ipa 文件，要注意下载 ipa 的可用性，要不然辛苦半天最后发现不能用，误以为是步骤有问题，有可能是包的问题。。。笔者最终可用的是从这里下载的[微信 ipa](https://ipa.store/3073.html)

ipa 下载完成后，和上面步骤类似，改为.zip，解压，获取到Payload/WeChat.app，如下图：

![WeChat_app](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230717-163712.png)

用自己的账号新建一个APP（bundleID自己定义），运行到手机上，找到运行的包(`xxx/DriveData/xxx/Build/Products/Debug-iphoneos/xxx.app`)或者通过打包的方式，使用`AdHoc`方式生成一个iPA（用于获取到描述文件和`Entitlements.plist`），这里需要注意选择的描述文件包含要安装的设备，然后把待签名的iPA中的`bundleID`、证书和描述文件，替换成自己创建的APP的。

#### 修改 info.plist 的 bundleIdentifier
然后查看WeChat.app的包内容，找到`info.plist`，并替换其中的`bundleIdentifier`为自己创建的(WeChat.app中内容很多，可以按修改日期排序，会相对容易找到要修改的文件)，如下图：
![微信bundleID修改](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230717-164527.png)

#### 替换`embedded.mobileprovision`
从自己创建的APP中获取到`embedded.mobileprovision`，然后替换`WeChat.app`中的`embedded.mobileprovision`，可以直接复制替换。


#### 重签名Frameworks

对比上面的香色闺阁，这里多了重签名Frameworks的步骤，需要把`Frameworks`中的所有库，都用自己的证书重新签名一下，`Frameworks`中内容如下图（这里可能有非官方的库，一并签名了，不影响）：

![Frameworks内容](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230717-165700.png)

重签名的命令如下，重复执行，直到`Frameworks`下所有库都签名完成：

``` bash
codesign -fs "你的证书" xxx.framework
```

![替换framework签名](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230717-170142.png)

### 删除Plugins

笔者下载的这个版本的`WeChat.app`的Content中未找到`Plugins`的内容，故而不需要处理

#### 替换签名
从自己创建的APP中获取到的`embedded.mobileprovision`生成`Entitlements.plist`，命令如下：
``` bash
   /usr/libexec/PlistBuddy -x -c "print:Entitlements " /dev/stdin <<< $(security cms -D -i xxx.app/embedded.mobileprovision) > Entitlements.plist
```

然后删除`WeChat.app`中的`_CodeSignature`，再用生成的`Entitlements.plist`，给`WeChat.app`重签名，命令如下，执行时注意`Entitlements.plist`和`WeChat.app`的路径：

``` bash
codesign -fs "你的证书" --no-strict --entitlements=Entitlements.plist Payload/WeChat.app/
```

![重签名WeChat.app](https://raw.githubusercontent.com/mokong/BlogImages/main/img/screenshot-20230717-170443.png)

 然后压缩`/Payload/WeChat.app`，生成`xxx.ipa`，命令如下：

 ```
 zip -r xxx.ipa Payload/
 ```

 最后使用 Xcode 安装 xxx.ipa 到手机，步骤如下：
 选中Xcode，`Shipt+CMD+2`打开窗口，然后选中设备，点击"+"，选择`xxx.ipa`，等待安装
 
 最后安装后效果如下：

<!-- ![最终](https://raw.githubusercontent.com/mokong/BlogImages/main/img/202307171734346.png) -->
<!-- ![最终效果](https://raw.githubusercontent.com/mokong/BlogImages/main/img/RPReplay_Final1689585448.MP4) -->
![最终效果](https://raw.githubusercontent.com/mokong/BlogImages/main/img/RPReplay_Final1689585448%20(1).gif)

## 总结

总结一下，ipa重签名是用自己的证书和描述文件替换对应包中的证书和描述文件的过程，总的步骤如下：
- 找到可用的 ipa
- 新建项目，编译或者打包，获取到对应的`embeded.mobileprovision`
- 修改替换包的`bundleIdentifier`为自己创建项目的`bundleIdentifier`
- 用自己的`embeded.mobileprovision`更新替换包中的
- 重签名`Frameworks`
- 删除`Plugins`
- 用自己的`embeded.mobileprovision`生成`Entitlements.plist`，然后重签名`xxx.app`
- 最后打包`xxx.app`成`xxx.ipa`，安装


## 参考

- [iOS逆向 应用重签名+微信重签名实战](https://juejin.cn/post/6844903998110040077)
- [iOS ipa重签名](https://juejin.cn/post/7031729444914102303)