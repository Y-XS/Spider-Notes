# Spider Notes

# 1.环境配置

``` python
1.环境配置
请求库：request、Selenium、ChromeDriver、PhantomJS、aiohttp
解析库：lxml、Beautiful Soup、pyquery、tesseract
数据库：MySQL、MongoDB、Redis
web库：Flask(主要用作API服务)、Tornado(支持异步的web框架)
网络抓包工具：Charles、mitmproxy、mitmdump
爬虫框架：pyspider、Scrapy、Scrapy-Splash、Scrapy-Redis
部署：Docker/Scrapyd/Scrapyd-Client

pyspider:(注意版本问题 werkzeug=0.0.16.0, 3.2=<tornado<=4.5.3)
Scrapy:需先安装lxml/pyOpenSSL/Twisted/PyWin32/Scrapy  pip即可
```

# 2.爬虫基础

## 2.1 http基本原理

请求：分为4部分：请求方法(POST&GET)、请求网址、请求头、请求体

请求头：Accept/Accept-Language/Accept-Encoding  Host
                Cookie:维持当前访问会话，为了辨别用户会话跟踪而存储在本地的数据
                Referer:标记此请求从哪个页面发过来
                User-Agent:简称UA，可以使服务器识别客户使用的操作系统及版本、浏览器及版本等信息
                Content-Type:也叫互联网媒体类型或MIME类型，在HTTP消息头中表示具体请求中的媒体类型信息：										如：text/html代表HTML，image/gif

请求体：承载POST请求中的表单数据，对于GET请求体为空

响应：分为3部分：响应状态码、响应头、响应体

## 2.2 网页基础

网页组成：HTML、CSS、JavaScript.   对应骨架、皮肤、肌肉

HTML DOM树

选择器：#id   .class    *    element

## 2.3 爬虫原理

获取网页并保存信息的自动化程序

​		1.获取网页：使用urllib、requests等构造请求，获取响应

​		2.提取信息：分析网页源码，提取数据             ps: 正则表达式(万能)/Beautiful Soup、pyquery、lxml等

​		3.保存数据

> JS渲染页面(JS改变html中的节点)，html是空壳：可分析其后台Ajax接口，也可使用Selenium、Splash模拟JS渲染

## 2.4会话和Cookies

会话：在服务端，保存用户的会话信息(属性及配置信息)

Cookies：在客户端，下次访问网页时会自动附带，服务器通过识别Cookies鉴定用户及判断状态

​				会话Cookie和持久Cookie(硬盘)

## 2.5 代理的基本原理

### 原理:

代理(proxy server)：代理网络用户取得网络信息      本机-->代理服务器-->web服务器

### 作用: 

​		 1.突破自身IP访问限制，访问平时不能访问的站点

​		 2.访问一些单位或团体内部资源

​		 3.提高访问速度.     ps:代理服务器有一个较大的硬盘缓冲区，直接由缓冲区取信息，提高速度

​		 4.隐藏真实IP. 

### 代理分类:

1.根据协议区分：

​		FTP代理服务器：主要用于访问FTP服务器.     21、2121

​		HTTP代理服务器：主要用于访问网页			  8080、3128

​		SSL/TLS代理：主要用于访问加密网站			 443

​		RTSP代理：主要用于访问Real流媒体服务器   554

​		Telnet代理：主要用于telnet远程控制(入侵时隐藏身份)    23

​		POP3/SMTP代理：主要用于POP3/SMTP方式收发右键   110、25

​		SOCKS代理：单纯传递数据包，不关心协议和用法，速度快    1080

​								SOCKS4: TCP            SOCKS5: TCP/UDP, 身份验证、服务器端域名解析等

2.根据匿名程度：

​		高度匿名代理：原封不动转发数据包，像普通客户端访问，记录代理服务器的IP

​		普通匿名代理：数据包一些改动，服务端可能发现是代理服务器，有几率追查客户端真实IP

​		透明代理：改动数据包，告诉服务器客户端真实IP，利用缓存技术提高访问速度，过滤内容提								高安全性

​		间谍代理：

### 常见代理:

1.使用网上的免费代理：需筛选，最好使用高匿代理

2.使用付费代理：质量比免费代理好很多

3.ADSL拨号：拨一次号换一次IP，稳定性高


# 3.基本库的使用

## 3.1 使用urllib

Python内置HTTP请求库

​		request：基本HTTP请求模块，可用来模拟发送请求

​		error：异常处理模块

​		parse：工具模块，提高URL处理方法，如拆分、解析、合并等

​		robotparser：主要用来识别网站的robot.txt文件

### 3.1 发送请求

#### 1. urlopen()

``` python
# 1.urlopen()
url = 'https://news.163.com/'
response = urllib.request.urlopen(url)
print(response.read().decode('gbk'))

# -----data参数
#如果传递此参数，则请求方式为POST，模拟表单提交
url = 'http://httpbin.org/post'
data = bytes(urllib.parse.urlencode({'word': 'hello'}), encoding='utf-8')
response = urllib.request.urlopen(url, data=data)
print(response.read().decode('utf-8'))
#运行结果
"form": {
    "word": "hello"
  }
    
# -----timeout参数，超时处理
try:
    response = urllib.request.urlopen('http://httpbin.org/get', timeout=0.1)
except urllib.error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
# -----其他参数：context、cafile、capath
```

#### 2. Request

``` python
# 2.Request
class urllib.request.Request(url, data=None, headers={}, origin_req_host=None,unverifiable=False,methods=None)
#通过构造request将请求独立成一个对象
request = urllib.request.Request(url_wynews)
response = urllib.request.urlopen(request)
print(response.read().decode('gbk'))
#request示例
url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host': 'httpbin.org'
}
dict = {
    'name':'Germey'
}
data = bytes(parse.urlencode(dict), encoding='utf-8')
req = request.Request(url=url, data=data, headers=headers, method='POST')
req.add_header('key', 'value')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

#### 3. 高级用法

两个类：Handler: 各种处理器，有处理登录验证的，有处理Cookies的，有处理代理设置的

​			  OpenerDirector: 可以使用open()方法，用Handler构建Opener

```python
urllib.request.BaseHandler    ps: 所有Handler的父类，提供最基本方法default_open()、													protoclo_request()等
#BaseHandler子类：
HTTPDefaultErrorHandler: 用于处理HTTP响应错误，错误抛HTTPError类型异常
HTTPRedirectHandler    : 用于处理重定向
HTTPCookieProcessor	   : 用于处理Cookies
ProxyHandler		  : 用于设置代理，默认代理为空
HTTPPasswordMgr		   :用于管理密码，维护用户名和密码的表
HTTPBasicAuthHandler   :用于管理认证
```

* 验证

``` python
from urllib.request import HTTPPasswordMgrWithDefaultRealm, HTTPBasicAuthHandler, build_opener
from urllib.error import URLError

username = 'username'
password = 'password'
url = 'http://localhost:5000/'
p = HTTPPasswordMgrWithDefaultRealm()
p.add_password(None, url, username, password)
auth_handler = HTTPBasicAuthHandler(p)
opener = build_opener(auth_handler)
try:
    result = opener.open(url)
    html = result.read().decode('utf-8')
    print(html)
except URLError as e:
    print(e.reason)
```

* 代理

``` python
 from urllib.error import URLError
 from urllib.request import ProxyHandler, build_opener

proxy_handler = ProxyHandler({
     'http': 'http://121.12.249.207:3128',
     'https': 'https://121.12.249.207:3128'
 })
 opener = build_opener(proxy_handler)
 try:
     response = opener.open('https://www.baidu.com')
     print(response.read().decode('utf-8'))
 except URLError as e:
     print(e.reason)
```

* Cookie

``` python
import http.cookiejar, urllib.request
# ----------1.
 cookie = http.cookiejar.CookieJar()
 handler = urllib.request.HTTPCookieProcessor(cookie)
 opener = urllib.request.build_opener(handler)
 response = opener.open('http://www.baidu.com')
 for item in cookie:
     print(item.name+"="+item.value)
# ----------2.输出cookie为文件格式  MozillaCookieJar()
filename = 'cookies.txt'
cookie = http.cookiejar.MozillaCookieJar(filename)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
cookie.save(ignore_discard=True, ignore_expires=True)
# ----------3.LWPCookieJar()  -->  生成libwww-perl(LWP)格式的Cookie文件

```

### 3.2 异常处理

urllib的error模块定义了由request模块产生的异常

#### 1. URLError

URLError来自urllib库的error模块，继承自OSError，是error的基类，由request模块产生的异常都可通过捕获这个类来处理

``` python
from urllib import request, error
try:
    response = request.urlopen('https://cuiqingcai.com/index.htm')
except error.URLError as e:
    print(e.reason)
#OUT: Not Found
```

#### 2. HTTPError

是URLError的子类，专用处理HTTP请求错误，有三个属性

* code：返回HTTP状态码，如404/500
* reason：返回错误原因(不一定是字符串，有可能是对象)
* headers：返回请求头

``` python
from urllib import request, error
try:
    response = request.urlopen('https://cuiqingcai.com/index.htm')
except error.HTTPError as e:
    print(e.reason, e.code, e.headers, sep='\n')
except error.URLError as e:
    print(e.reason)
else:
    print('Request Successfully')
```

### 3.3 解析链接

urllib提供parse模块，定义了处理URL的标准接口

URL规则：scheme://netloc/path;params?query#fragment

#### 1.urlparse()

该方法可实现URL的识别和分段

``` python
from urllib.parse import urlparse
result = urlparse('http://www.baidu.com/index.html;user?id=5#comment')
print(type(result), '\n', result)
```

urllib.parse.urlparse(urlstring, scheme=' ', allow_fragment=True)

* urlstring: 待解析的URL
* scheme: 默认的协议
* allow_fragments: 是否忽略fragment

#### 2.urlunparse()

完成URL的构造，参数列表长度必须为6

``` python
data = ['http', 'www.baiud.com', 'index.html', 'user', 'a=6', 'comment']
print(urlunparse(data))
```

#### 3.urlsplit()

与urlparse()类似，不单独解析params, 只返回5个结果

``` python
result = urlsplit('http://www.baidu.com/index.html;user?id=5#comment')
print(result)

```

#### 4.urlunsplit()

与urlunparse()类似，链接各部分组合成完整链接，传入参数为可迭代对象，长度为5

``` python
data = ['http', 'www.baiud.com', 'index.html', 'a=6', 'comment']
print(urlunsplit(data))
```

#### 5.urljoin()

一个base_url作为第一参数，新链接作为第二参数，分析base_url的scheme、netloc和path，并对新链接确实的部分进行补充，最后返回结果

#### 6.urlencode()

将字典序列化为GET请求参数

``` python
params = {
    'name': 'germey',
    'age': 22
}
base_url = 'http://www.baidu.com?'
url = base_url + urlencode(params)
print(url)
```

#### 7.parse_qs()

反序列化GET参数-->字典

``` python
query = 'name=germey&age=22'
print(parse_qs(query))
```

#### 8.parse_qsl()

将参数转化为元组组成的列表

``` python
query = 'name=germey&age=22'
print(parse_qsl(query))
```

#### 9.quote()

将内容转化为URL编码格式。URL中带中文参数时可能会乱码，此时用此方法将中文字符转化为URL编码。

```python
keyword = '壁纸'
url = 'https://www.baidu.com/s?wd=' + quote(keyword)
print(url)
```

#### 10.unquote()

URL解码

``` python
url = 'https://www.baidu.com/s?wd=%E5%A3%81%E7%BA%B8'
print(unquote(url))
```

### 3.4 分析Robots协议

#### 1. Robots协议

#### 2. 爬虫名称

#### 3. robotparser

urllib.robotparser.RobotFileParser(url=' ')

```python

```



## 3.2 使用requests

### 基本用法

#### 1. 实例

```python
import requests

url = 'https://www.baidu.com/'
r = requests.get(url)
print(type(r))
print(r.status_code)
print(type(r.text))
print(r.text)
print(r.cookies)
```

#### 2.GET请求

* 基本实例

``` python
# 1.先构建一个最简单的GET请求，网站返回相应信息
r = requests.get('http://httpbin.org/get')
print(r.text)
# 2.添加参数
data = {
    'name': 'germey',
    'age': 22
}
r = requests.get('http://httpbin.org/get', params=data)
print(r.text)
print(type(r.text))		#str
#返回类型为str,可调用json()方法转为dict
print(r.json())
print(type(r.json()))	#dict
```

* 抓取网页

```python
import re, requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36'
}
r = requests.get("https://www.zhihu.com/explore", headers=headers)
pattern = re.compile('explore-feed.*?question_link.*?>(.*?)</a>', re.S)
titles = re.findall(pattern, r.text)
print(r.text)
print(titles)
```

* 抓取二进制数据

想抓取图片、音频、视频等文件，需要拿到它们的二进制码

```python
import requests
r = requests.get("https://github.com/favicon.ico")
print(r.text)
print(r.content)
with open('favicon.ico', 'wb') as f:
    f.write(r.content)
```

* 添加headers

``` python
headers = {
     'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36'
 }
r = requests.get("https://www.zhihu.com/explore", headers=headers)
```

#### 3. POST请求

```python
data = {
    'name': 'germey',
    'age': '22'
}
r = requests.post('http://httpbin.org/post', data=data)
print(r.text)
```

#### 4. 响应

```python
# 响应相关信息
r = requests.get('http://www.jianshu.com')
print(type(r.status_code), r.status_code)
print(type(r.headers), r.headers)
print(type(r.cookies), r.cookies)
print(type(r.url), r.url)
print(type(r.history), r.history)
# requests.codes :内置返回码
r = requests.get('http://www.jianshu.com')
exit() if not r.status_code == requests.codes.ok else print('Request Successfully')
```



### 高级用法

#### 1. 文件上传

```python
files = {
    'file': open('favicon.ico', 'rb')
}
r = requests.post('http://httpbin.org/post', files=files)
print(r.text)
# 文件上传部分会有单独的files字段来标识
```

#### 2. Cookies

```python
headers = {
    'Cookie': '_zap=3bf257b6-5897-491a-b2e7-618635456700; d_c0="APDav48cBRGPTp6P-R7lPtH0IBVQ7abnAAs=|1585180807"; _ga=GA1.2.280528651.1585180808; q_c1=1f6a4f6a76d34b94a6056f9e2fcdaae6|1606873134000|1589165790000; _xsrf=e0XNEiXKttSa7O2RybI11UiruN52HyVf; _9755xjdesxxd_=32; YD00517437729195%3AWM_TID=ty2WwMmRG5RBUQUBBBcrLwy73DI6xTIP; Hm_lvt_98beee57fd2ef70ccdd5ca52b9740c49=1626314231,1626402039,1626510266,1626523776; captcha_session_v2="2|1:0|10:1626523775|18:captcha_session_v2|88:WFFvUGozSlQvVEJqNVE1MlV3dGlwczBTZEpEWGlZbXYvQk51bitvOTdYMGw5Tk9MaE9sOXFiaGFidVE3b1JYRw==|8fafd0279b518a5d39f96d973543810c65e46682424e53a8e3192740f3786d67"; SESSIONID=bJgPRCsFVDHPFNrLMgQWJeQsfWivLzI41wlz6Eu95FG; JOID=VV8dAE9x9HdHICwxHXWXJxTQE70DE7EbEHFvTVAksDAcYWFDXv44iSMjIzURZXYZZI7whv6f-_mHzLxxaSKghKg=; osd=VVoXBUlx8X1CJiw0F3CRJxHaFrsDFrseFnFqR1UisDUWZGdDW_Q9jyMmKTAXZXMTYYjwg_Sa_fmCxrl3aSeqga4=; __snaker__id=ORboUMRDkJJDsEjc; gdxidpyhxdE=vs3d9n5WMkEjAbUwBVjliqa%5CT%2BBdXudUQYtatdcc%2FO2t90sKhTpNTDcUWnYeD2LXjWdHYEd3lR0gGSbP%5Czbi%5CqWVf%2BCNG9jCTtoNwaVwWUUA200IbhBDsmTpsqJMLSVU%5C8H1krRwg4JY5iODPMxXg07Ri9sUje%5CM7ViQ49DiI2WQk5QD%3A1626524678016; YD00517437729195%3AWM_NI=WJnM%2B57zFiMZ4Ih8YkRyd4n6KTVfXcNOnI4tic8WyChyrcuWojcOYwJA%2F4jGpUeyiz3OTXGW0BsYQUDujX0zN%2FAqq24UhFdfXuRWs24rwRSNH7DdKooPA0AzbDHcDhu6Q3U%3D; YD00517437729195%3AWM_NIKE=9ca17ae2e6ffcda170e2e6ee86d039f588aca2f75c96b88ab3d84a939a9eabf563a8eca885b534a3ae87b8f32af0fea7c3b92a9596a2aeaa3ff5a68a8ad052f897a8d9dc6192a7acd5ed3e898c9a87db698e91a2aab780abbdf8b8fc49aca7be84f85dadb6adb0dc7d8a9fe1d5e94db48bab9aec72f1919cb9c65df49ebdbad45f88aba6b9b3728ef18f92f43af88ba8a7e76e9cab9eb8f23af8aeb9b3d2468ce98c85ca65ad908fd1c472a29c8884c764b7ea9fd4d837e2a3; captcha_ticket_v2="2|1:0|10:1626523791|17:captcha_ticket_v2|704:eyJ2YWxpZGF0ZSI6IkNOMzFfMFpKbVA5QllMWlFaZ1JGVVpDOUdJQUx3dlZPNThwNWk1eHdVeHFkbHR5Qm82LS1ZR3RfNmdYT09zY0xzU0tZUjJFQlBTNUY3cVdHNy1jcmJpeFRvaWdRUTRoR25wU0VTYVh3X1pPNXdaRWZiWmVJZC5WNWpHVEthaWpOcGtFTG1LUWNVa3pzYXF0N1pJNlE4ZFIxbFMyRFVrZ0pURWM2OUVDUVNvSUVOY2xuRFV3OWZFZVFJdjlYd0thWGlDN1ZkNW1lLXYybnhZR1VGTmEtSzBELU1tRlpvXzk1bkMucGR0RTB0dXFsbl9rUXVsWHVMeDlpWS1RdkZOcTRyWlRTa3dVWU5HTEtGa2d2Z0lYbmljNmZTbkM5Ni0xMi5OdGhvZXo4c3J1cWtyLmhJSGRkUE05VUc3Rk9iOV9aTkpUdnN3cUMxMHgtYXF0QTRFaWllc1RNMHJqTU93OTVVVm12dnFlNm1jTnJ1VXhVUkh6WTBfVnZ1TlNfODlYT3N1NUl1NGx5VFhHc25HMnQ1cGw5QlZSXzZXSXVYbl9zczQxU0hPWld0NUhJUVhIeWtoS1c2VkZIOG9ULkFnMUN2T05OS1RsbmZQcS5mRjFsZ0l3LWdNaC5zaU5oaVZHLjV4NXNnSE1TaHVsMlRxT3BDQUxoYTJNODU1alFUbDFjMyJ9|2b9dea49c203d805b516b98c792ed5d3c993a0a5d2406c78e563c1eb258665b9"; z_c0="2|1:0|10:1626523809|4:z_c0|92:Mi4xNHcyWENnQUFBQUFBOE5xX2p4d0ZFU1lBQUFCZ0FsVk5vUmJnWVFDeGVjY2gyU19iZDZWNjYtM19CVHltMXBDeHJ3|1489ab5b3fa09bfacbc101fba5e482c5cf4b04fec7f94c3673ba082c0165a3ba"; unlock_ticket="AfBlxZ7Rzw0mAAAAYAJVTanP8mBHcN_hCVg1jhz9QAgaW14FZlQcGQ=="; tst=r; Hm_lpvt_98beee57fd2ef70ccdd5ca52b9740c49=1626523811; KLBRSID=b33d76655747159914ef8c32323d16fd|1626523818|1626523775',
    'Host': 'www.zhihu.com',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36'
}
r = requests.get('https://www.zhihu.com', headers=headers)
print(r.text)
```

#### 3. 会话维持

Session可以模拟同一个会话而不用担心Cookies问题，常用与模拟登陆后下一步操作

```python
requests.get('http://httpbin.org/cookies/set/number/123456789')
r = requests.get('http://httpbin.org/cookies')
print(r.text)
# 上面的方法不是同一个会话，获取不到cookies
# 下面的方法维持同一个会话，登录后能获取到cookies
s = requests.session()
s.get("http://httpbin.org/cookies/set/number/123456789")
r = s.get('http://httpbin.org/cookies')
print(r.text)
```

#### 4. SSL证书验证

```python
# 默认verify为True, 设为False则可忽略证书验证
response = requests.get('https://www.12306.cn', verify=False)
print(response.status_code)
```

#### 5. 代理设置

```python
# 需换成自己的有效代理
proxies = {
    'http': 'http://10.10.1.10:3128',
    'https': 'http://10.10.1.10:1080',
}
requests.get("https://www.taobao.com", proxies=proxies)
# 若代理需要使用HTTP Basic Auth, 则可使用类似
# http://user:password@host:port这样的语法设置代理
proxies = {
    'http': 'http://user:password@10.10.1.10:3128',
}
requests.get("https://www.taobao.com", proxies=proxies)
# 除基本的HTTP代理外，requests还支持SOCKS协议代理
# 首先需安装socks库	pip3 install 'requests[socks]'
proxies = {
    'http': 'socks5://user:password@host:port',
    'https': 'socks5://user:password@host:port',
}
requests.get("https://www.taobao.com", proxies=proxies)
```

#### 6. 超时设置

```python
r = requests.get("https://www.taobao.com", timeout=1)
print(r.status_code)
# 请求分为两个阶段：连接(connect)和读取(read), timeout作为两种总和，也可单独设置
r = requests.get("https://www.taobao.com", timeout=(5,11,30))
#如想永久等待，timeout=None或不写(默认None)
r = requests.get("https://www.taobao.com", timeout=None)
```

#### 7. 身份验证

1.自带的身份认证

```python
from requests.auth import HTTPBasicAuth
auth = HTTPBasicAuth('username', 'password')
r = requests.get('http://localhost:5000', auth=auth)
print(r.status_code)
# 简洁写法 auth=('username', 'password')
```

2.OAuth认证

> 需安装oauth包    pip install requests_oauthlib

```python
url = ''
auth = OAuth1()
requests.get(url, auth=auth)
```

#### 8. Prepared Request

## 3.3 使用正则表达式

在线正则表达式测试：https://tool.oschina.net/regex/

```python
# \w 匹配字母、数字及下划线
# \s 匹配任意空白字符
# \d 匹配任意数字
# \n \t 匹配一个换行符 制表符

# ^ 一行字符串的开头
# $ 一行字符串的结尾
# . 匹配任意字符
# * 匹配0个或多个表达式
# + 匹配1个或多个表达式
# ? 匹配0/1个前面正则表达式定义的片段，非贪婪

# [...] 表示一组字符	[^...]不在[...]内的字符
# {n} 精确匹配n个前面的表达式
# {n,m} 匹配n~m次前面的表达式判断，贪婪
# () 匹配括号内表达式，也表示一个组
```



### 1. 实例

匹配URL ：[a-zA-Z]+://\[^\s]*

### 2. match()

从字符串**起始位置**匹配正则表达式

用法：re.match(rex, str)

```python
content = 'Hello 123 4567 World_This is a Regex Demo'
print(len(content))
result = re.match(r'^Hello\s\d\d\d\s\d{4}\s\w{10}', content)
print(result)
print(result.group())   #匹配的内容
print(result.span())    #匹配的范围
```

* 匹配目标

()标记了一个子表达式的开始和结束位置，可以用()将想提取的子字符串括起来，被标记的子表达式会一次对应每个分组，调用group()方法传入分组索引可获取

```python
content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^Hello\s(\d+)\sWorld', content)
print(result)
print(result.group())
print(result.group(1))
print(result.span())
# group(0) == group()
# group(1)输出第一个被()包围的匹配结果
# 如果后面还有被()包括的内容,可用group(2)/group(3)
```

* 通用匹配

.(点)可以匹配任意字符(除换行符), *代表匹配前面的字符无限次

.*匹配任意字符

```python
content = 'Hello 123 4567 World_This is a Regex Demo'
result = re.match('^Hello.*Demo$', content)
print(result)
print(result.group())
print(result.span())
```

* 贪婪与非贪婪

```python
# 贪婪尽可能匹配更多的字符，非贪婪尽可能匹配少的字符
content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^He.*(\d+).*Demo$', content)
print(result)
print(result.group(1))
#.*贪婪匹配，匹配123456，剩下\d+至少一个数字，就只剩7
# .*?转为非贪婪匹配，如果在字符串结尾，可能匹配不到任何内容
conten = 'http://weibo.com/comment/kEraCN'
r1 = re.mathck('http.*?comment/(.*?)', content)
r2 = re.mathck('http.*?comment/(.*)', content)
# OUT:result1     result2 kEraCN
```

* 修饰符

> re.S和re.I 在网页匹配中常用，因为HTML节点会有换行

```python
# re.I : 使匹配对大小写不敏感
# re.L : 做本地化识别匹配
# re.M : 多行匹配，影响^和$
# re.S : 使.匹配包括换行符在内的所有字符
# re.U : 根据Unicode字符集解析字符。此标志影响\w、\b
# re.X : 


# 字符串中加入换行符后匹配失败，.匹配的是除换行符之外的任意字符，当遇到换行符时.*?就不能匹配了，需加修饰符re.S
content = '''Hello 1234567 World_This
is a Regex Demo
'''
result = re.match('^He.*?(\d+).*?Demo$', content, re.S)
print(result.group(1))
```

* 转义匹配

正则匹配特殊字符时，在前面加反斜线转义

```python
content = '(百度)www.baidu.com'
result = re.match('\(百度\)www\.baidu\.com', content)
print(result)
```

### 3. search()

扫描整个字符串，返回第一个成功匹配的结果

```python
html = '''<div id="songs-list">
<h2 class="title">经典老歌</h2>
<p class="introduction">
经典老歌列表
</p>
<ul id="list" class="list-group">
<li data-view="2">一路上有你</li>
<li data-view="7">
<a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
</li>
<li data-view="4" class="active">
<a href="/3.mp3" singer="齐秦">往事随风</a>
</li>
<li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a> </li>
<li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a> </li>
<li data-view="5">
<a href="/6.mp3" singer="邓丽君">但愿人长久</a>
</li>
</ul>
</div>'''

reg = '<li.*?active.*?singer="(.*?)">(.*?)</a>'
result = re.search(reg, html, re.S)
if result:
    print(result.group(1), result.group(2))
```

### 4. findall()

搜索整个字符串，返回匹配regex的所有内容

```python
reg = '<li.*?href="(.*?)".*?singer="(.*?)">(.*?)</a>'
results = re.findall(reg, html, re.S)
print(results)
print(type(results))
for result in results:
    print(result)
    print(result[0], result[1], result[2])
```

### 5.sub()

用regex修改文本

```python
# 去掉文本中的所有数字
content = '54aK54yr5oiR54ix5L2g'
content = re.sub('\d+', '', content)
print(content)
# 要提取所有li节点的歌名，直接用正则麻烦，先去掉a节点，再findall
# 直接提,regex: '<li.*?>\s*?(<a.*?>)?(\w+)(</a>)?\s*?</li>'
html = re.sub('<a.*?>|</a>', '', html)
print(html)
results = re.findall('<li.*?>(.*?)</li>', html, re.S)
for result in results:
    print(result.strip())
```

### 6. compile()

将正则字符串编译成正则表达式对象，方便后面匹配中复用

```python
content1 = '2016-12-15 12:00'
content2 = '2016-12-17 12:55'
content3 = '2016-12-22 13:21'
reg = re.compile('\d{2}:\d{2}')
result1 = re.sub(reg, '', content1)
result2 = re.sub(reg, '', content2)
result3 = re.sub(reg, '', content3)
print(result1, result2, result3)
```



## 3.4 抓取猫眼电影排行

```python
import requests
import re
import json
import time
from requests.exceptions import RequestException


def get_one_page(url):
    try:
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36'
        }
        response = requests.get(url, headers=headers, auth=('17708314402', 'qaz.13579'))
        if response.status_code == 200:
            return response.text
        return None
    except RequestException:
        return None


def parse_one_page(html):
    pattern = re.compile(
        '<dd>.*?board-index.*?>(.*?)</i>.*?data-src="(.*?)".*?name.*?a.*?>(.*?)</a>.*?star.*?>(.*?)</p>.*?releaset' \
                 'ime.*?>(.*?)</p>.*?integer.*?>(.*?)</i>.*?fraction.*?>(.*?)</i>.*?</dd>', re.S)
    regex = '<dd>.*?board-index.*?>(.*?)</i>.*?data-src="(.*?)".*?name.*?a.*?>(.*?)</a>.*?star.*?>(.*?)</p>.*?releaset' \
                 'ime.*?>(.*?)</p>.*?integer.*?>(.*?)</i>.*?fraction.*?>(.*?)</i>.*?</dd>'
    items = re.findall(pattern, html)
    for item in items:
        yield {
            'index': item[0],
            'image': item[1],
            'title': item[2].strip(),
            'actor': item[3].strip()[3:] if len(item[3]) > 3 else '',
            'time': item[4].strip()[5:] if len(item[4]) > 5 else '',
            'score': item[5].strip()+item[6].strip(),
        }


def write_to_file(content):
    with open('res/result.txt', 'a', encoding='utf-8') as f:
        print(type(json.dumps(content)))
        f.write(json.dumps(content, ensure_ascii=False)+'\n')


def main(offset):
    url = 'http://maoyan.com/board/4?offset=' + str(offset)
    html = get_one_page(url)
    for item in parse_one_page(html):
        print(item)
        write_to_file(item)


if __name__ == '__main__':

    for i in range(10):
        main(offset=i*10)
        time.sleep(1)


    regex = '<dd>.*?board-index.*?>(.*?)</i>'
    regex_img = '<dd>.*?board-index.*?>(.*?)</i>.*?data-src="(.*?)"'
    regex_filename = '<dd>.*?board-index.*?>(.*?)</i>.*?data-src="(.*?)".*?name.*?a.*?>(.*?)</a>'
    regex_star = '<dd>.*?board-index.*?>(.*?)</i>.*?data-src="(.*?)".*?name.*?a.*?>(.*?)</a>.*?star.*?>(.*?)</p>.*?releaset' \
                 'ime.*?>(.*?)</p>.*?integer.*?>(.*?)</i>.*?fraction.*?>(.*?)</i>.*?</dd>'

```

# 4. 解析库的使用

### 4.1 使用 XPath

XPath: XML Path Language, 即XML路径语言，在XML文档中查找信息的语言，同样适用HTML

原理：利用XPath或CSS选择器来提取某个节点，然后再调用相应方法获取它的正文内容或属性

```python
# XPath 常用规则
#	nodename	:	选取此节点的所有子节点
#	/		    :	从当前节点选取直接子节点
#	//		    :   从当前节点选取子孙节点
#   .  			：   选取当前节点
#   ..			:   选取当前节点的父节点
#   @			:   选取属性
#   *			：  匹配所有节点
```

#### XPath 示例

```python
text = '''
<div>
<ul>
<li class="item-0"><a href="link1.html">first item</a></li>
<li class="item-0"><a href="link2.html">second item</a></li>
<li class="item-inactive"><a href="link3.html">third item</a></li>
<li class="item-1"><a href="link4.html">fourth item</a></li>
<li class="item-0"><a href="link5.html">fifth item</a>
</ul>
</div>
'''
html = etree.HTML(text)			#构造XPath解析对象，自动补全最后一个</li>
result = etree.tostring(html)	#bytes类型，需解码
print(result.decode("utf-8"))
```

#### 用法--节点

```python
# 匹配所有节点
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//*')  #*表示匹配所有节点
print(result)
# 匹配指定节点
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//li')
print(result)
print(result[0])
#=============子节点===============
# 查找li节点的所有直接a子节点
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//li/a')
print(result)
# 获取ul节点下所有子孙a节点
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//ul//a')
print(result)
#===============父节点==============
# 先选中href属性为link4.html的a节点，然后再获取其父节点，再获取其class属性
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//a[@href="link4.html"]/../@class')
print(result)
# 也可通过parent::来获取父节点
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//a[@href="link4.html"]/parent::*/@class')
print(result)
#===============属性匹配=============
# 加入[@class="item-0"]，限制节点的class属性为item-0
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]')
print(result)
#===============文本获取=============
# 此时获取结果为：['\n  ']，因为/为选取li直接子节点，而li直接子节点都是a节点，仅有自动补全的li节点尾标签换行了，所以为此结果
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]/text()')
print(result)
# 想获取li节点内部文本，两个方法：
# 1.先选取a节点再获取文本
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]/a/text()')
print(result)
# 2.使用//
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]//text()')
print(result)
# 如果想获取某些特定子孙节点下的所有文本，可先选取到特定的子孙节点，再调用text()方法，可保证获取到的结果是整洁的。(//text()获取子孙节点内部的所有文本，可保证信息最全面，但是会夹杂一些换行符等特殊字符)
#=================属性获取===============
# 通过@href即可获取节点的href属性
# 注：此次和属性匹配方法不同，属性匹配是中括号加属性名和值来限定某个属性，如[@href="link1.html"]，而@href指的是获取节点的某个属性
html = etree.parse('res/test.html', etree.HTMLParser())
result = html.xpath('//li/a/@href')
print(result)
#=================属性多值匹配=============
text = '''
<li class="li li-first"><a href="link.html">first item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[@class="li"]/a/text()')
print(result)
# 此时li节点的class属性有两个值li和li-list，此时需要使用contains()函数
result = html.xpath('//li[contains(@class, "li")]/a/text()')
print(result)
# 通过contains()方法，参数1传属性名称，参数2传属性值
#==================多属性匹配================
# 这里li节点两个属性，用and连接，连接后置于中括号内进行条件筛选
text = '''
<li class="li li-first" name="item"><a href="link.html">first item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[contains(@class, "li") and @name="item"]/a/text()')
print(result)
# and是XPath中的运算符，还有or、mod等
#===================按序选择=================
html = etree.HTML(text)
result = html.xpath('//li[1]/a/text()')			#选取第一个li节点
print(result)
result = html.xpath('//li[last()]/a/text()')	#选取最后一个li节点
print(result)
result = html.xpath('//li[position()<3]/a/text()')	#选取位置<3的li节点
print(result)
result = html.xpath('//li[last()-2]/a/text()')    #选取倒数第三个li节点
print(result)
#====================节点轴选择==============
text='''
text =
<div>
<ul>
<li class="item-0"><a href="link1.html"><span>first item</span></a></li>
<li class="item-1"><a href="link2.html">second item</a></li>
<li class="iten-inactive"><a href="link3.html">third item</a></li>
<li class="item-1"><a href="link4.html">fourth item</a></li>
<li class="item-0"><a href="link5.html">fifth item</a>
</ul>
</div>
'''
html = etree.HTML(text)
result = html.xpath('//li[1]/ancestor::*')	#调用ancestor轴，可获取所有祖先节点
print(result)
result = html.xpath('//li[1]/ancestor::div') #仅div这个祖先节点
print(result)
result = html.xpath('//li[1]/attribute::*') #调用attribute轴，可获取所有属性
print(result)
#选取href为link1.html的a节点
result = html.xpath('//li[1]/child::a[@href="link1.html"]')
print(result)
#调用descendant轴，可获取所有子孙节点，加入限定条件获取span节点
result = html.xpath('//li[1]/descendant::span')
print(result)
#调用following轴，可获取当前节点后所有节点，虽然用*匹配，但是加索引只获取第二个后续点
result = html.xpath('//li[1]/following::*[2]')
print(result)
#调用following-sibling轴，可获取当前节点之后的所有同级节点
result = html.xpath('//li[1]/following-sibling::*')
print(result)
```



### 4.2 使用 Beautiful Soup

原理：借助网页的结构和属性等特性来解析网页

#### 解析器

Beautiful Soup在解析时依赖解析器， 支持python标准库的HTML解析器、lxml HTML解析器、lxml XML解析器、html5lib

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup('<p>Hello</o>', 'lxml')
print(soup.p.string)
```

#### 基本用法

```python
html = '''
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title” name="dromouse"><b>The Dormouse’s story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie” class: link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class: link2">Lacie</a> and
<a href="http://example.com/tillie” class="sister" id="Link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
'''
# 初始化BeautifulSoup对象，采用lxml解析器，自动更正格式
soup = BeautifulSoup(html, 'lxml')
# 调用prettigy()方法，把要解析的字符串以标准缩进格式输出
print(soup.prettify())
# 调用soup.title.string，输出HTML中title节点内的文本内容
print(soup.title.string)
```

#### 节点选择器

直接调用节点的名称就可选择节点元素，再调用string属性就可得到节点内文本

```python
#=====选择元素=====
# 这种选择方式只会选择到第一个匹配的节点
soup = BeautifulSoup(html, 'lxml')
print(soup.title)
print(type(soup.title))	#bs4.element.Tag类型
print(soup.title.string)
print(soup.head)
print(soup.p)
#=====提取信息=====
# 1.获取节点名称
print(soup.title.name)
# 2.获取属性
print(soup.p.attrs)
print(soup.p.attrs['name'])
#attrs返回字典形式，简写↓
print(soup.p['name'])  #name唯一，返回字符串
print(soup.p['class']) #class可能有多个，返回列表
# 3.获取内容
print(soup.p.string) #p为第一个节点
#=====嵌套选择=====

#=====关联选择=====
```

#### 方法选择器

```python
#find_all()
#find()
```

#### CSS选择器

```python
#嵌套选择
#获取属性
#获取文本
```



### 4.3 使用 pyquery

#### 初始化

#### 基本CSS选择器

#### 查找节点

```python
#子节点
#父节点
#兄弟节点

```

#### 遍历

#### 获取信息

```python
#获取属性
#获取文本

```

#### 节点操作

```python
#addClass/removeClass
#attr/text/html
#remove()
```

#### 伪类选择器

# 5. 数据存储

