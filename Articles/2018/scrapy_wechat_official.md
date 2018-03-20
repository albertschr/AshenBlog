![](https://diycode.b0.upaiyun.com/photo/2018/d5c2fae2609a20f63ce8874272126d01.png)
# 用 scrapy 爬微信公众号的内容

## 抓包工具配置
### 1. Fiddler(windows)
- 安装：[https://www.telerik.com/download/fiddler](https://www.telerik.com/download/fiddler)
- 配置：  

```ruby
# 1. 允许远程连接
Tools > Fiddler Options > Connections > Allow remote computers to connect
# 2. 支持 HTTPS
Tools > Fiddler Options > HTTPS > Decrypt HTTPS traffic
# 3. 重启
restart Fiddler

# 4. 手机配置
手机 wifi 设置代理，服务器地址为为电脑 ip 地址 xxx.xxx.xxx.xxx，端口为 8888
# 5. 下载根证书
safari 打开 xxx.xxx.xxx.xxx:8888， 下载安装根证书
# 6. 完全信任证书
设置 > 通用 > 关于本机 > 证书信任设置 > 针对根证书启用完全信任
```
### 2.Charles(MacOS)
- 安装：[Charles Web Debugging Proxy](https://www.charlesproxy.com/)
- 配置：

```ruby
# 1. 手机配置
手机 wifi 设置代理，服务器地址为为电脑 ip 地址 xxx.xxx.xxx.xxx，端口为 8888
# 2. 手机安装 SSL 证书
Help -> SSL Proxying -> Install Charles Root Certificate on a Mobile Device
safari 打开地址 chls.pro/ssl，下载根证书
# 3. 完全信任证书
设置 > 通用 > 关于本机 > 证书信任设置 > 针对根证书启用完全信任
# 4. 设置Proxy
Proxy -> SSL Proxying Settings...，勾选 Enable SSL Proxying
# 5. 添加域名
Host: https://mp.weixin.qq.com  Port: 443
```

## scrapy(python package) 使用
- 官网介绍：

```
An open source and collaborative framework for extracting the data you need from websites.
In a fast, simple, yet extensible way. 
```
- 使用：

```shell
# 1. 创建项目 WCSpider
scrapy startproject WCSpider

# 2. 切换到项目目录
cd WCSpider

# 3. 创建爬虫 wechat
scrapy genspider wechat mp.qq.com

# 4. 设置 Agent
在 settings 中打开 Agent 注释

# 5. 添加 item
在 items.py 中完成 item 创建
    name = scrapy.Field
    age = scrapy.Field
    ...
    
# 6. 解析 response
设置 start_urls
在 wechat.py 中解析 response, 返回item
    res = response.xpath(...)
    item['name'] = res['_name']
    item['age'] = res['_age']
    ...
    yield item
    
# 7. 设置管道文件
在 open_spider 初始化数据库等
    self.f = open("file.json","w")
在 close_spider 关闭数据库等
    self.f.close()
在 deal_spider 处理 item
    self.f.write(json.dumps(dict(item)))
在 settings 中启用该管道文件
```

## 公众号历史消息抓包

本次抓包以 Charles 为例。

### 1.获取首页链接
![抓包](https://diycode.b0.upaiyun.com/photo/2018/2f41e5d1a0a4ad2851619b136b6db5f5.png)

如上图所示，需要的内容在 var msgList = '{...}' 中，由于并不是标准 html，所以这里 xpath 并不好用，可尝试正则表达式快速取出内容。

```python
rex = "msgList = '({.*?})'"
pattern = re.compile(pattern=rex, flags=re.S)
match = pattern.search(response.text)
```

取出并格式化匹配项，本例中公众号中的内容都是以文本的形式发送，所以直接取 comm_msg_info 内容，具体请根据实际情况解析。

```python
if match:
    data = match.group(1)
    data = html.unescape(data)
    data = json.loads(data)
    articles = data.get("list")

    item = WechatItem()
    for article in articles:
        info = article["comm_msg_info"]
        content = html.unescape(info["content"])
        # 将获取的数据传递给管道
        item["content"] = content
        item["we_id"] = info["id"]
        item["datetime"] = info["datetime"]
        item["we_type"] = info["type"]
        yield item
```

### 2. 获取上拉刷新链接

向上滚动历史列表，当到接近最底部时，会自动获取更多内容，此时可以通过抓包获取内容。
![](https://diycode.b0.upaiyun.com/photo/2018/784da6b14584170d45393a0476105e63.png)

如图所示，返回的是 json 格式内容，通过 can_msg_continue 确定是否有后续内容，general_msg_list 是解析的内容。通过分析多个加载历史可知，offset 控制加载数据的位置。
**--本例中使用 python 第三方库 requests 加载数据**

```python
def getMore():
   # header
	header = sj_utils.header2dict(base_header)
	response = requests.get(getUrl(),headers = header, verify = False)
	
	js = json.loads(response.text)
	if js["errmsg"] == "ok":
		alist = js["general_msg_list"] # 内容是 json 字符串，需要先转为 json
		alist = json.loads(alist)["list"]
		for item in alist:
		  # 具体处理
			print(item["comm_msg_info"]["content"])

	if js["can_msg_continue"]:
	   url_offset += 10 # 设置偏移量
		getMore()
```

### 3.处理数据
开头已经介绍了 scrapy 使用，通过管道文件可以轻松处理爬取内容。

```python
# 保存 json 文件
self.f.write(json.dumps(dict(item)))
```



## 参考
- [基于 Python 实现微信公众号爬虫](https://juejin.im/book/5a157c155188254a701eb3c1)


