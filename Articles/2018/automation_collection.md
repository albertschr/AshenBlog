![](https://diycode.b0.upaiyun.com/photo/2017/1eceab5933faae2b6b06b39877fa6847.png)
## 记一次半自动化的文稿搜集

![两步搜集.gif](https://diycode.b0.upaiyun.com/photo/2017/8934e4f7b3f146cae8ae5a24e1d2b626.gif)
--

### 1. 我想做什么
```
1. 每天搜集若干篇文章
2. 在微信公众号上分享
3. 将每次分享内容归档存储
```
看似很简单的需求，每天写起来还是要花不少时间，一是人力搜集自己觉得不错的文章(当然用爬虫另算)；二是整理，尤其是拷贝链接+标题，以 xml 形式组织，统一拷贝再归档。这个整理的过程既索然无味又容易出错，有没有一个好的办法来放弃 cmd+c 和 cmd+V 呢？

### 2. 我是这么做的
- 工具：workflow + 印象笔记
- 效果：如开头 gif 所示，一键搜集，一键归档。
![](https://diycode.b0.upaiyun.com/photo/2017/e95fd1a212f99bc77e1c8bf16284f5f4.jpg)

- workflow帮助我做了哪些工作？
  - 获取剪贴板的 URL
  - 获取 URL 的标题，并和内容以 \<a\> 标签组合
  - 获取本次期数，并将上一步的内容追加到印象笔记中存储
  - 如果归档，将本期内容拷贝到剪贴板并追加到历史中，更新期数

### 3. 有何借鉴意义？
workflow 可以结合一些其他的 app 简化生活中的很多操作，对一些`重复度很高`或者`流程相对固定`的操作尤其有用。
**我们要实现某种功能，需要 A->B->C->D ==> E (根据输入的条件A，我们依次通过 BCD 三种操作最后得到结果E)，如果这个功能使用频率很高，我们便可以尝试通过构建自己的 workflow 将以上所有操作封装起来，每次使用时只需要输入 A，workflow 会自动计算出结果 E。**

```
# 举例说明，请脑洞大开
1. 搜集若干篇文章，以固定格式显示(本文)
2. 将一段文字、链接等转成一个二维码地址(文字->二维码->上传图床->获取地址)
3. 将手机的截屏加壳(如上图所示)
```
---


### 4. 相关资源
APP: [workflow for iOS](https://itunes.apple.com/us/app/workflow-powerful-automation/id915249334?mt=8)
workflow：[添加到集锦](https://workflow.is/workflows/c87107d28419439b930fcd58aa1d8204) 和 [集锦归档](https://workflow.is/workflows/ff4f405dd2774fe788f606ee30a4a2c1)
workflow 系列教程：[Workflow 教程](https://sspai.com/post/27733)
参考文章:  [我是这么制作「coding01 日报」的](https://segmentfault.com/a/1190000011356011)

