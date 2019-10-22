pod (源码编译后里面的pod)

```
pod 'ijkplayer' //https://github.com/iOSDevLog/ijkplayer
```

### 编译源码
```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
cd ijkplayer-ios
# git checkout -B latest k0.8.8
./init-ios.sh
```


### 添加 https 支持
这是编译好的[IJKMediaFrameworkWithSSL](https://github.com/zdhgithub/ijkplayer_IJKMediaFrameworkWithSSL)

```
# 获取 openssl 并初始化
./init-ios-openssl.sh
cd ios
# 在模块文件中添加一行配置 以启用 openssl 组件
echo 'export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-openssl"' >> ../config/module.sh
./compile-ffmpeg.sh clean
```

```
# 编译openssl, 如果不需要https可以跳过这一步
./compile-openssl.sh all

# 编译ffmpeg
./compile-ffmpeg.sh all
```
>执行最后一步，如果出现错误如下： 

```
./libavutil/arm/asm.S:50:9: error: unknown directive
        .arch armv7-a
        ^
make: *** [libavcodec/arm/aacpsdsp_neon.o] Error 1
```
是因为最新的 Xcode 已经弱化了对 32 位的支持， 解决方法:

>在 compile-ffmpeg.sh 中删除 armv7 ，也就是修改 `FF_ALL_ARCHS_IOS8_SDKΩ="armv7 arm64 i386 x86_64"` 为 `FF_ALL_ARCHS_IOS8_SDK="arm64 i386 x86_64"`
然后重新执行 ./compile-ffmpeg.sh all

添加 openssl 相关依赖：

>选择 Targets ——> IJKMediaFrameworkWithSSL ——> General ——> Linked Frameworks and Libraries ，选择 Add Other... , 然后在 ijkplayer-ios 目录下（也可能是别的名字）的 ios —— build —— universal —— lib 中，选中 libcrypto.a 和 libssl.a 文件，添加进去

打包 framework
>选择 Xcode 上方导航栏中的 Product ——> Scheme ——> Edit Scheme..., 打开后选择 Run ——> Info ——> Build Configuration，将 Debug 改为 Release。

Cmd + b 直接运行，会报错 avconfig.h 文件找不到
>TARGETS ——> IJKMediaFrameworkWithSSL ——> Build Settings ——>Architecutres 
中，删掉其中的 armv7、armv7s

### 合并Framework
```
lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework

将生成的 IJKMediaFramework  替换掉 Release-iphoneos/IJKMediaFramework .framework 文件下的 IJKMediaFramework ，就 OK 啦。

lipo -create Release-iphoneos/IJKMediaFrameworkWithSSL.framework/IJKMediaFrameworkWithSSL Release-iphonesimulator/IJKMediaFrameworkWithSSL.framework/IJKMediaFrameworkWithSSL -output IJKMediaFrameworkWithSSL
```
直接将 `IJKMediaFramework.framework` 拖入到工程中即可，注意记得勾选 `Copy items if needed` 和 对应的 target
然后添加相关依赖 `libc++.tbd`  `libz.tbd`

在相应文件中导入 `#import <IJKMediaFrameworkWithSSL/IJKMediaPlayer.h>`
即可使用 `IJKMediaPlayer`了

如果运行程序报 `image not found` 错误 ，需在工程对应 `TARGETS —— General —— Embedded Binaries` 中添加 `IJKMediaFrameworkWithSSL.framework` 即可.

[参考](https://www.jianshu.com/p/cc1c0e63f70d)
