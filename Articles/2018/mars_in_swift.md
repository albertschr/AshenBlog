![](https://user-gold-cdn.xitu.io/2017/12/8/16033e55aca73b9e?imageView2/2/w/480/h/480/q/85/interlace/1)
# 在 swift 中接入微信开源库 Mars

## 1 介绍
### 1.1 mars
[mars](https://github.com/Tencent/mars)是微信官方的跨平台跨业务的终端基础组件。
![](https://user-gold-cdn.xitu.io/2017/12/7/1602f5e627031bc8?w=885&h=580&f=png&s=64116)
- comm：可以独立使用的公共库，包括 socket、线程、消息队列、协程等；
- xlog：高可靠性高性能的运行期日志组件；
- SDT： 网络诊断组件；
- STN： 信令分发网络模块，也是 Mars 最主要的部分。

### 1.2 protobuf
[protobuf](https://github.com/google/protobuf): Google's data interchange format.
protobuf是google提供的一个开源序列化框架，类似于XML，JSON这样的数据表示语言，其最大的特点是基于二进制，因此比传统的XML表示高效短小得多。虽然是二进制数据格式，但并没有因此变得复杂，开发人员通过按照一定的语法定义结构化的消息格式，然后送给命令行工具，工具将自动生成相关的类，可以支持php、java、c++、python等语言环境。通过将这些类包含在项目中，可以很轻松的调用相关方法来完成业务消息的序列化与反序列化工作。


## 2 创建framework
### 2.1 生成framework
```shell
// 下载 mars 仓库: 
git clone https://github.com/Tencent/mars.git
// 切换到 libraries路径
cd mars-master/mars/libraries
// 执行python脚本
python build_apple.py
	提示输入保存文件夹的前缀:(随意)
	input prefix for save directory. like trunk,br,tag:
	选择build的类型: 1代表iphone版mars(其中包括xlog)
	Enter menu:
	build mars for iphone.
```
![success...](https://user-gold-cdn.xitu.io/2017/12/7/1602f5e5c25aeef5?w=680&h=415&f=png&s=309178)

### 2.2 引入framework

// log_crypt.cc.rewriteme, log_crypt.h 这两个文件已被废弃,可直接删除

```
// 删除.rewriteme的后缀
共三个文件log_crypt.cc.rewriteme(废弃),longlink_packer.cc.rewriteme, shortlink_packer.cc.rewriteme
// 拖入工程,并添加库
SystemConfiguration, CoreTelephony, Foundation, libz.tbd, libresolv9.tbd

```
![](https://user-gold-cdn.xitu.io/2017/12/7/1602f5e5c27f08fa?w=701&h=217&f=png&s=34907)

### 2.3 xlog
```
/// 添加文件(demo中有)
LogHelper.h(.mm)、LogUtil.h(.mm), appender-swift-bridge.h(.mm)
/// 桥接头文件添加
#import "appender-swift-bridge.h"

// 配置xlog, 参数分别代表:debug时记录的级别,release时记录的级别,保存文件的相对路径,文件前缀
JinkeyMarsBridge().initXlogger(...)
// 自定义log
JinkeyMarsBridge().log(...)
```

在didFinishLaunchingWithOptions配置xlog
在applicationWillTerminate中destory

### 2.4 引入protobuf

#### 2.4.1 生成.pb.swift
```shell
// 安装protobuf
brew install protobuf
// 安装swift-protobuf-plugin
	git clone https://github.com/apple/swift-protobuf.git
	cd swift-protobuf
	swift build -c release -Xswiftc -static-stdlib

// 将.proto文件转换成.swift
protoc --swift_out=. name.proto
```

写了个shell,可以批量将当前文件夹下的所有.proto转换成对应的.pb.swift
地址为[install.sh](https://github.com/515783034/FileResource/blob/master/swift-join-mars/proto/install.sh),可直接双击使用
内容如下:
```shell
cd $(cd `dirname $0`; pwd)

for i in find *.proto
do
    if [ $i == "find" ]; then
        continue
    fi
    protoc --swift_out=. $i
    echo "生成文件:$i.pb.swift"
done
```

#### 2.4.2 引入.pb.swift

[swift-protobuf](https://github.com/apple/swift-protobuf): Plugin and runtime library for using protobuf with Swift
```
// pod引用
pod 'SwiftProtobuf'
// 将.pb.swift添加到工程中(不用添加.proto)
// 使用
var con = Conversation()
con.name = "ashen"
con.notice = "for test"
// 传输时,直接传递data数据
let data = try? con.serializedData()
```

### 2.5 添加桥接文件(可选)
由于该framework是C++写的,对于不会C++的人(比如我)来说, 调用是个很大的问题, 可以通过demo中的相关文件 + swift-oc桥接头文件直接转换成swift可以直接调用的类
![相关文件](https://user-gold-cdn.xitu.io/2017/12/7/1602f5e5c21b74ee?w=614&h=325&f=png&s=295127)

## 3 xlog文件解析

### 3.1 安装python依赖库.

__首先需要查看自己的python版本, decode_mars_crypt_log_file.py只在python2.+下支持,3.+不能正常运行__

安装依赖库pyelliptic, 如果同时存在2.+ 和3.+ 版本,请使用pip2
pyelliptic的最新版本1.5.8,存在bug, 需要指定版本为1.5.7
```shell
pip install pyelliptic==1.5.7

```

### 3.2 解密logger日志

```shell
// 切换到crypt文件夹
cd mars/mars/log/crypt
// 解密logger
python2 decode_mars_crypt_log_file.py logger_20171205.xlog
// logger_20171205.xlog.log 即为解密出的log日志
```

***


## fix bugs:

- ImportError: No module named pyelliptic

  原因:没有正确安装pyelliptic的依赖库

  解决:  pip2 install pyelliptic==1.5.7

  如果安装后依然报错,可能是由于安装了多个版本的python,pyelliptic并没有安装在2.+的python下.
  在MacOS上, 如果同时安装了2.+ 和3.+的python,  pip2 和pip3 分别代表安装到2.+ 和3.+的库中. 所以可以通过pip2 install pyelliptic==1.5.7 安装


- AttributeError: dlsym(0x7fc443f02f50, EVP_CIPHER_CTX_reset): symbol not found
  原因: pyelliptic的最新版本1.5.8的bug
  解决: 使用1.5.7版本
  ```shell
  pip uninstall pyelliptic
  pip install pyelliptic==1.5.7
  ```

## 资源
- [示例代码](https://github.com/515783034/FileResource/tree/master/swift-join-mars)
- [批量转proto](https://github.com/515783034/FileResource/blob/master/swift-join-mars/proto/install.sh)

参考: [Swift 接入微信 Mars_Xlogger 填坑指南——Jinkey 原创](http://www.jianshu.com/p/fe8c5f3f6389)

