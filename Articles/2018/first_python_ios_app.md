![](https://diycode.b0.upaiyun.com/photo/2017/b3605b89e89f5cf951cbc3ee1721a10a.png)
# 我用 Xcode+python 写的第一个 OSX app
>前两天刚订阅了 [bestswifter](https://xiaozhuanlan.com/u/4809976499) 大大的小专栏，其中《2018 年将至，ios 工程师如何自我提高》这篇文章使我感触颇深，最近刚好开始看 python，就萌生了写一个 python 脚本练练手的想法。


## 1. 为什么要写这个 app？
  原因之一当然是学了点东西总想练练手。
  更为重要的原因则是，在写 iOS app 时，每增加一个网络请求，就要写一个 json 对应的 model 类，而构造这些 model 类的代码毫无快感可言。so，人生苦短，我用 python。

![](https://diycode.b0.upaiyun.com/photo/2017/f2d8b0c83668b3839cf7234f9f3d38b2.gif)

***
## 2. 技术栈
### python 最最最基础知识
  1. json 反序列化
将输入的json字符串转成对应的字典(dict) + 数组(list)组合的形式

  ```python
  res = json.loads("输入的json字符串")
  ```
  2. 字符串操作
  解析字典和数组内容，生成 swift 对应的字符串，拼接起来即可
  
  ```python
    # 遍历字典
    for (key, value) in dic.items():

    # 转换成swift格式
    if isinstance(value, str):
        return "String"
    elif isinstance(value, float):
        return "Float"

    # 字符串数组拼接
    result= ''.join([line+'\n' for line in res])
  ```

推荐两个不错学习资源：[Python 简单入门指北（试读）](https://xiaozhuanlan.com/topic/1053427869)，[Python教程- 廖雪峰](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

### Cocoa APP 生成
  - 1.界面搭建
Mac OS 的界面搭建和 iOS 超级相似，这里不做赘述。
![](https://diycode.b0.upaiyun.com/photo/2017/330ec904f4e7d951651ea579a287f3ee.png)
  - 2.图标生成
制作一个 1024*1024 的图片，我习惯使用 [pixelmator](http://www.pixelmator.com/mac/)，个人感觉比较简单上手，然后生成若干张对应尺寸的图标，这种 app 有很多，我用的是 [Prepo](https://itunes.apple.com/cn/app/id476533227?mt=12)。
![](https://diycode.b0.upaiyun.com/photo/2017/3708cfb1f18bb0cca026156cebd0e406.png)

  - 3.Cocoa 运行 python 脚本
NSTask 已经被废弃，应该使用`Process()`执行脚本文件

```
let buildTask = Process()
let outPip = Pipe()
let errorPipe = Pipe()

buildTask.launchPath = "/usr/bin/python"
// arguments是[String]类型，第一个元素应该为xx.py的路径，后面元素为该py接受的参数
buildTask.arguments = args
buildTask.standardInput = Pipe()
buildTask.standardOutput = outPip
buildTask.standardError = errorPipe
// 脚本执行完毕后的回调
buildTask.terminationHandler = { p in
      self.taskFinish()
}
buildTask.launch()
buildTask.waitUntilExit()

// 脚本的输出结果，即脚本文件的print()方法打印的内容
let data = outPip.fileHandleForReading.readDataToEndOfFile()
let output = String(data: data, encoding: String.Encoding.utf8)
        
// 错误处理, python系统错误
let errorData = errorPipe.fileHandleForReading.readDataToEndOfFile()
let errorStr = String(data: errorData, encoding: String.Encoding.utf8)
if let aError = errorStr, aError != "" {
    sendError("解析错误\r\n" + aError)
}
```

 - arguments：脚本参数
 
```
let scripyPath = Bundle.main.path(forResource: "Parse", ofType: "py")! // 脚本路径
let para1 = "input info" // 传入参数，通过sys.argv[I]获取
let args = [scripyPath, para1]
buildTask.arguments = args
```
- standardOutput：脚本输出
> 执行脚本后可通过 output 获取结果
注意：脚本的返回结果不是通过函数的 return，而是通过调用 print(infos) 函数，即 infos 作为结果返回

```
let data = outPip.fileHandleForReading.readDataToEndOfFile()
let output = String(data: data, encoding: String.Encoding.utf8)
```
- standardError: 错误输出
>解决 python 的 exception 信息
task.terminationStatus 不等于 0 也能判断为失败
***

## 3. 封装
  Cocoa  运行 python 脚本的代码，写起来虽然不算特别麻烦，但总也说不是简洁，于是重新封装了一个类方便调用：[CocoaPython](https://github.com/515783034/json2swift/blob/master/json2Swift/CocoaPython.swift)
使用如下：

```swift
let script = CocoaPython(scrPath: pyPath)
script.runAsync()
```
详细说明：

```swift
// python 脚本文件路径
guard let aPath = Bundle.main.path(forResource: "Parse", ofType: "py") else { return }

// args: py 文件接受的参数列表，通过 sys.argv[i] 访问
// block: 完成后的回调，包括返回值和错误内容
let script = CocoaPython(scrPath: aPath, args: [""]) { [weak self] in
    print($0) // 返回值，所有的 py 中 print() 的内容
    print($1) // py 中的错误信息
}

script.spliPara = "$" // 如果有多个结果，每个结果之间的分隔符，不设置则将所有的结果当成一个结果返回，即 result == result[0]
script.runAsync() // 异步执行，回调在异步主线程中调用
// or script.runAsync(asyncComlete: false) // 异步执行，回调在 global 中执行
// or script.runSync() // 同步执行
```
***
## 4. 资源
github地址:  [json2swift](https://github.com/515783034/json2swift)
OS X app:  [json2Swift.app](https://pan.baidu.com/s/1skW4Jxj)
python 脚本:  [json2Swift.py](https://pan.baidu.com/s/1kV1jhbh)


