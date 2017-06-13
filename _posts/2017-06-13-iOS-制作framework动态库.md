---
layout: post
title: iOS 制作framework动态库
date: 2017-06-13 10:35
tags: Framework
---
##指令集
通常会把CPU的扩展指令集称为”CPU的指令集”（因为基本的，类似加减的指令似乎是必须被CPU所支持的指令）。每款CPU在设计时就规定了一系列与其硬件电路相配合的指令集。
##arm处理器
Arm处理器，因为其低功耗和小尺寸而闻名，几乎所有的手机处理器都基于arm，其在嵌入式系统中的应用非常广泛，它的性能在同等功耗产品中也很出色。想了解ARM指令集的请[点击这里](https://zh.wikipedia.org/wiki/ARM架构)。
##iPhone指令集
苹果A7处理器支持两个不同的指令集：32位ARM指令集（armv6｜armv7｜armv7s）和64位ARM指令集（arm64），i386｜x86_64 是Mac处理器的指令集，i386是针对intel通用微处理器32架构的。x86_64是针对x86架构的64位处理器。当使用iOS模拟器的时候会遇到i386｜x86_64，iOS模拟器没有arm指令集。
##xcode中设置指令集
主要有以下三个：Architectures、Valid Architectures、Build Active Architecture Only
#####Architectures
该编译选项指定了工程将被编译成支持哪些指令集，支持指令集是通过编译生成对应的二进制数据包实现的，如果支持的指令集数目有多个，就会编译出包含多个指令集代码的数据包，造成最终编译的包很大。
#####Valid Architectures
该编译项指定可能支持的指令集，该列表和Architectures列表的交集，将是Xcode最终生成二进制包所支持的指令集。

比如，Valid Architectures设置的支持arm指令集版本有：armv7/armv7s/arm64，对应的Architectures设置的支持arm指令集版本有：armv7s，这时Xcode只会生成一个armv7s指令集的二进制包。
#####Build Active Architecture Only
该编译项用于设置是否只编译当前使用的设备对应的arm指令集。

当该选项设置成YES时，你连上一个armv7指令集的设备，就算你的Valid Architectures和Architectures都设置成armv7/armv7s/arm64，还是依然只会生成一个armv7指令集的二进制包。

通常情况下，开发测试都是在 Debug模式下，所以该编译选项在Debug模式都设成YES，这样在测试时只编译一份指令集，有效提高开发效率。

Release模式为发布模式，需要支持各种设备指令集，所以设置为NO。

----
##制作framework
xcode版本: 8.3.2，Mac OS版本:10.12
创建一个新工程，在choose template页面选择Cocoa Touch Framework,如下：

![choose template.png](http://upload-images.jianshu.io/upload_images/6009581-9bc18efa3e51265b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在工程中添加或新建类，然后编译生成framework，下面是选择不同设备或模拟器时所生成framework:
```
1、debug环境下
设备：arm64（测试机型有限:6P、5、7）
模拟器：iPhone7-Plus：x86_64、iPhone4s：i386
2、release环境下
设备：armv7、arm64
模拟器：i386、x86_64
```
以上所生成的framework均不包含armv7s，在 Building Setting 中设置一下 Architectures，在原有基础上添加一行 armv7s ，如下：

![Architectures.png](http://upload-images.jianshu.io/upload_images/6009581-a1a12d032134391b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有一点需要注意的是，如果制作 framework 时采用的是 Objective-C，则需要将希望外部使用的.h文件添加到 public 中，如下：

![public_header.png](http://upload-images.jianshu.io/upload_images/6009581-31b26986a91d8729.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###查看生成的framework
右键 Products 目录下的 framework 文件 “show in finder”，会看到 finder 下存在模拟器和真机生成的 framework，如下：

![framework.png](http://upload-images.jianshu.io/upload_images/6009581-8aea380c395d0e96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###合并framework
制作framework的过程中经常会遇到编译出来的framework只能被真机使用或者只能被模拟器使用的情况。造成这个问题的原因是由于在编译时选择的目标设备不同的情况下编译出来framework体系结构不同，选择真机进行编辑时会编译产生armv7、armv7s、arm64下的库文件，而选择模拟器会产生i386、x86_64下的库文件。但导入过程中时希望支持所有架构，所以需要将 framework 合并
####查看 framework 所支持架构
>lipo -info frame_name.framework/framework_name

#### framework 合并命令：
-create后为要合并在一起的两个framework，-output后面为合并后的framework名称及路径
>lipo –create /Debug-iphoneos/Someframework.framwork/Someframework Debug-iphonesimulator/Someframework.framwork/Someframework –output Someframework

合并命令只是合并 framework 下的可执行文件，因此需要用合并后的 exec 文件替换真机或模拟器 framework 中的 exec 文件。

####framework合并脚本：
在xcode中添加脚本后会自动将生成的模拟器与设备的framework合并，比使用合并命令手动合并更简单，添加步骤如下：

在 Build Phases 中点击左上角 + ，选择 New Run Script Phase 添加一项

![New Run Script Phase.png](http://upload-images.jianshu.io/upload_images/6009581-3214dc75f695e7cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

会在页面底部添加一项 Run Script ，如下：

![Run Script.png](http://upload-images.jianshu.io/upload_images/6009581-a8f78f27e638fd90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将下面的脚本代码粘贴至提示“Type a script or drag…”输入框内
```
if [ "${ACTION}" = "build" ]
then
INSTALL_DIR=${SRCROOT}/Products/${PROJECT_NAME}.framework
DEVICE_DIR=${BUILD_ROOT}/${CONFIGURATION}-iphoneos/${PROJECT_NAME}.framework
SIMULATOR_DIR=${BUILD_ROOT}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}.framework
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}"
cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}/"
#ditto "${DEVICE_DIR}/Headers" "${INSTALL_DIR}/Headers"
lipo -create "${DEVICE_DIR}/${PROJECT_NAME}" "${SIMULATOR_DIR}/${PROJECT_NAME}" -output "${INSTALL_DIR}/${PROJECT_NAME}"
open "${DEVICE_DIR}"
open "${SRCROOT}/Products"
fi
```
测试一下脚本，在release环境下分别选择真机和模拟器运行，运行成功后会自动打开Finder并定位到工程根目录下的Products:

![Products.png](http://upload-images.jianshu.io/upload_images/6009581-5ddb0f9cad9eb6b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时查看Products下的framework所支持的指令集：

![TestFrameWork_Architectures.png](http://upload-images.jianshu.io/upload_images/6009581-ee370da07b83a89a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----

##动态库、静态库与Framework
库(Library)说白了就是一段编译好的二进制代码，加上头文件就可以供别人使用。

什么时候我们会用到库呢？一种情况是某些代码需要给别人使用，但是我们不希望别人看到源码，就需要以库的形式进行封装，只暴露出头文件。另外一种情况是，对于某些不会进行大的改动的代码，我们想减少编译的时间，就可以把它打包成库，因为库是已经编译好的二进制了，编译的时候只需要 Link 一下，不会浪费编译时间。

上面提到库在使用的时候需要 Link，Link 的方式有两种，静态和动态，于是便产生了静态库和动态库。

####静态库
静态库即静态链接库（Windows 下的 .lib，Linux 和 Mac 下的 .a）。之所以叫做静态，是因为静态库在编译的时候会被直接拷贝一份，复制到目标程序里，这段代码在目标程序里就不会再改变了。

静态库的好处很明显，编译完成之后，库文件实际上就没有作用了。目标程序没有外部依赖，直接就可以运行。当然其缺点也很明显，就是会使用目标程序的体积增大。

####动态库
动态库即动态链接库（Windows 下的 .dll，Linux 下的 .so，Mac 下的 .dylib/.tbd）。与静态库相反，动态库在编译时并不会被拷贝到目标程序中，目标程序中只会存储指向动态库的引用。等到程序运行时，动态库才会被真正加载进来。

动态库的优点是，不需要拷贝到目标程序中，不会影响目标程序的体积，而且同一份库可以被多个程序使用（因为这个原因，动态库也被称作共享库）。同时，编译时才载入的特性，也可以让我们随时对库进行替换，而不需要重新编译代码。动态库带来的问题主要是，动态载入会带来一部分性能损失，使用动态库也会使得程序依赖于外部环境。如果环境缺少动态库或者库的版本不正确，就会导致程序无法运行（Linux 下喜闻乐见的 lib not found 错误）。

####iOS Framework
除了上面提到的 .a 和 .dylib/.tbd 之外，Mac OS/iOS 平台还可以使用 Framework。Framework 实际上是一种打包方式，将库的二进制文件，头文件和有关的资源文件打包到一起，方便管理和分发。
在 iOS 8 之前，iOS 平台不支持使用动态 Framework，开发者可以使用的 Framework 只有苹果自家的 UIKit.Framework，Foundation.Framework 等。这种限制可能是出于安全的考虑（见[这里的讨论](https://stackoverflow.com/questions/4733847/can-you-build-dynamic-libraries-for-ios-and-load-them-at-runtime))。换一个角度讲，因为 iOS 应用都是运行在沙盒当中，不同的程序之间不能共享代码，同时动态下载代码又是被苹果明令禁止的，没办法发挥出动态库的优势，实际上动态库也就没有存在的必要了。
由于上面提到的限制，开发者想要在 iOS 平台共享代码，唯一的选择就是打包成静态库 .a 文件，同时附上头文件（例如[微信的SDK](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319164&token=&lang=zh_CN)）。但是这样的打包方式不够方便，使用时也比较麻烦，大家还是希望共享代码都能能像 Framework 一样，直接扔到工程里就可以用。于是人们想出了各种奇技淫巧去让 Xcode Build 出 iOS 可以使用的 Framework，具体做法参考[这里](https://github.com/kstenerud/iOS-Universal-Framework)和[这里](https://github.com/jverkoey/iOS-Framework)，这种方法产生的 Framework 还有 “伪”(Fake) Framework 和 “真”(Real) Framework 的区别。
iOS 8/Xcode 6 推出之后，iOS 平台添加了动态库的支持，同时 Xcode 6 也原生自带了 Framework 支持（动态和静态都可以），上面提到的的奇技淫巧也就没有必要了（新的做法参考[这里](http://www.cocoachina.com/ios/20141126/10322.html)）。为什么 iOS 8 要添加动态库的支持？唯一的理由大概就是 Extension 的出现。Extension 和 App 是两个分开的可执行文件，同时需要共享代码，这种情况下动态库的支持就是必不可少的了。但是这种动态 Framework 和系统的 UIKit.Framework 还是有很大区别。系统的 Framework 不需要拷贝到目标程序中，我们自己做出来的 Framework 哪怕是动态的，最后也还是要拷贝到 App 中（App 和 Extension 的 Bundle 是共享的），因此苹果又把这种 Framework 称为 [Embedded Framework](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html)。
