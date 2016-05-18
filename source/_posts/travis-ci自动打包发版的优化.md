title: travis-ci自动打包发版的优化
date: 2016-05-17 13:18:55
tags: travis, 慢, 打包, 发版, 优化
---


其实之前很早就弄好了travis-ci自动打包和发版的流程，在使用的过程中，因为各种原因，导致travis服务器慢慢的出现error，整个流程的时间也越来越长，慢慢地又变回手工发版了。但如果能健壮自动化的流程，比手工打包还是强很多的。

<!--more-->

#### 导致弃用的原因
原因有好几个，首先是网络访问的问题，因为用到了cocoapods管理第三方库，travis服务器在执行pod install的时候，需要去下载对应的包，因为每个包的源都可以由库作者自己指定，所以有可能会遇到网络访问的问题。

然后是上传指令的更改，因为上传规则和第三方工具库api的修改，上传testflight的脚本，坏过好几次，烦到我直接就手动发版了。

还有其他原因，比如因为travis-ci每次都是启动一个全新模拟器，我每次打包都需要从头编译，这。。。花了我很多时间去调试，后来烦了就手动发版好了。

#### 解决问题
##### 1、网络问题，更改下载源
每一个第三方库其实都对应着一个.podspec.json的文件，文件描述里该库的所有基本信息，这个文件可以在本地安装好cocoapods的~/.cocoapods/repos/master/Specs目录下相应的库里找到，也可以在cocoapods的[github库](https://github.com/CocoaPods/Specs/tree/master/Specs)中找到。

```JSON
{
    "name": "AVOSCloud",
    "version": "3.2.8",
    "platforms": {
        "ios": "6.0"
    },
    "summary": "LeanCloud iOS SDK for mobile backend.",
    "homepage": "https://leancloud.cn",
    "documentation_url": "https://leancloud.cn/api-docs/iOS/index.html",
    "license": {
        "type": "Commercial",
        "text": "Copyright 2015 LeanCloud, Inc. See https://leancloud.cn/terms.html"
    },
    "authors": {
        "LeanCloud": "support@leancloud.cn"
    },
    "source": {
        "http": "https://download.leancloud.cn/sdk/iOS/release-v3.2.8/Static/AVOSCloud.framework.zip"
    },
    "source_files": "*.h",
    "public_header_files": "*.h",
    "preserve_paths": "iOS/release-v3.2.8/Static/AVOSCloud.framework/AVOSCloud",
    "vendored_libraries": "libAVOSCloud.a",
    "frameworks": [
        "CFNetwork",...
    ],
    "libraries": [
        "icucore",
        "sqlite3"
    ],
    "xcconfig": {
        "OTHER_LDFLAGS": "$(inherited) -ObjC -lz"
    },
    "prepare_command": "cp iOS/release-v3.2.8/Static/AVOSCloud.framework/AVOSCloud libAVOSCloud.a\ncp iOS/release-v3.2.8/Static/AVOSCloud.framework/Headers/* ."
}
```
其中的source就是该库的下载地址，文件的其他字段和规则，详细可以查看cocoapods的官方文档，通过改动这个地址，加快travis-ci下载该库的速度，具体步骤就是下载这个地址的东西，然后放在某个文件服务器，再把对应的地址替换掉，我是把包直接托管在github仓库上，为每个库创建一个分支，然后打一个对应版本的tag，通过这个tag就可以拿到对应版本的第三方库了。改完这个文件之后，我们又不是作者，怎么让它生效呢？这时候需要修改项目的podfile文件，
```shell
pod 'AVOSCloud', :podspec => 'BRPodsHelp/AVOSCloud.podspec.json'
pod 'AVOSCloudIM', :podspec => 'BRPodsHelp/AVOSCloudIM.podspec.json'
```
让pod指定到本地podspec.json的地址，而对应的这个本地文件可以跟随仓库一起提交，这样维护起来就方便多了。至此，解决了网络问题。

##### 2、脚本问题
顺便把各个步骤用到的脚本简单说下，具体的话，还是要先了解对应工具和命令的用法
###### compile
使用xctool编译出xcachive格式的包，为啥使用xctool，因为它log很有可读性。
###### sign签名
这个命令有点烦，testflight的包，我是用xcodebuild中，带有-exportOptionsPlist选项的命令去完成打包的
```shell
xcodebuild -exportArchive -archivePath "$OUTPUTDIR_BUILD/$APPNAME.xcarchive" -exportPath "$OUTPUTDIR_BUILD" -exportOptionsPlist "scripts/$CONFIG.plist"
```
而inhouse的包，我是使用
```shell
xcodebuild -exportArchive -archivePath "$OUTPUTDIR_BUILD/$APPNAME.xcarchive" -exportPath "$OUTPUTDIR_BUILD/$APPNAME.ipa" -exportFormat ipa -exportProvisioningProfile "$PROFILE_NAME_INH_REAL_NAME" -configuration "$CONFIG_INHOUSE"
```
为啥inhouse和testflight的sign命令不同？因为坑爹苹果咯，我尝试了很多次，inhouse只有这样才能打包并且安装成功
###### 上传脚本
第三方：选择不同的分发平台，就有不同的上传脚本，这块还是参照你所选用的分发分发平台吧。
testflight：fastlane的credentials、pilot， 都是fastlane家的东西，github一下就有了，前者用来授权后者用来上传

自此，之前弃用travis的两个问题都解决了。

#### 优化
完成上面的东西，整个流程应该就能跑通了，但是，我总不能打个tag就编译和签名app-store版本和inhouse版本，并且还分发到第三方平台和testflight吧，整个过程起码1个多小时，而且不能按需打包总是很怪。所以我就把按需的需求，放在了tag上面
##### tag支持按需打包
这是什么鬼，其实就是判断tag是要打inhouse包，还是打testflight包咯，因为travis-ci可以拿到当前的tag，所以还是有可行性的。最后我选用-pgy和-testflight，分别表示上传到蒲公英和testflight，打tag的时候，比如1.0.1build1-pgy，这样就表示要上传蒲公英了。脚本如下：
```shell
STRING_A="$TRAVIS_TAG"
STRING_B="$PGY_TAG"
STRING_C="$TEST_FLIGHT_TAG"
if [[ ${STRING_A/${STRING_B}//} == $STRING_A ]]
then
    if [[ ${STRING_A/${STRING_C}//} == $STRING_A ]]
    then
        exit 0
    else
        /#这里是testflight
    fi
else
    /#这里是pgy
fi
```
if语句的原理就是stringA去掉stringB之后，是否等于stringA，等于的话，说明不包含这个命令，不等于的话，就说明包含咯。以此实现按需编译打包上传发版的需求。

##### 自动打tag
启动一个定时任务去自动打包并发版，这样就能有效率的迭代和测试。定时任务我用的是leancloud的后台云函数，支持Cron表达式的定时任务，语言用的是js，具体就是node.js。用node.js的[require](https://github.com/request/request)模块去调用[github release api](https://developer.github.com/v3/repos/releases/#create-a-release)，具体用法：
```javascript
var request = require("request");
var options = {
    method: 'POST',
    url: 'https://api.github.com/repos/beautifulreading/rio-ios/releases',
    headers: { 
        'cache-control': 'no-cache',
        'content-type': 'application/json',
        'user-agent': 'node.js'
    },
    body: {   
        tag_name: "name",
        target_commitish: 'develop',
        name: "name",
        body: "name",
        draft: false,
        prerelease: false 
    },
    json: true 
};

request(options, function (error, res, body) {
if (error) throw new Error(error);
    console.log(body);
});
```
记得在headers加上认证信息， 这样每天早上6点发一个inhouse，每周发两个testflight，有点爽，不要问为啥要早上6点发，因为5点的时候程序猿还没睡觉。
##### 上传dsym文件
之前做的上传流程，只管上传包，却没有管dsym文件，这样要分析错误还要回头去找个tag，然后重新打包再拿出来，烦到。。。崩溃监控我们现在用的是腾讯的bugly，好处就是他只需要上传几m的dsym包，当然这个包是经过bugly的脚本处理过的，比起平时几十m，还是很有优势的。而他的处理脚本，也都被我放进仓库里，直接打包后处理。  本来是想travis直接上传到bugly对应版本的，但是bugly的api有点烦，需要的参数太多，以至于很难自动化，所以我就直接把这个处理过的dsym文件上传到github对应的release tag上，在tarvis的yml文件加上：
```shell
deploy:
  provider: releases
  api-key: "key"
  file:
    - "$PWD/buglydsym.zip"
    - "$OUTPUTDIR_BUILD/$APPNAME.ipa"
  skip_cleanup: true
  on:
    tags: true
    all_branches: true
```
注意yml文件是区分缩进的，这样要分析bug的时候，就从对应tag下载这个文件，然后手动传到bugly。在这里我顺便把ipa包也传上github，也好做个备份吧。
##### travis命令支持
除了以上这些之外，我们还可以用travis-ci支持的指令去使保证整个过程更加顺利，比如retry，在yml的命令后面加上 -retry=3，这样在命令失败的情况下，travis会重新执行，3代表次数。具体还是看文档吧，还有其他可选项来保证整个流程的执行。 不过有时，自动化还是会失败的😂，毕竟是机器，x因素又比较多。

好吧，travis的优化大概就结束了，接下来要继续完善持续集成，让项目的迭代和质量更好咯。






