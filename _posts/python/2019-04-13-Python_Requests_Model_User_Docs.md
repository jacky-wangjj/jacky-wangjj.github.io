---
layout: post
title: Python之Requests模块使用详解
date: 2019-04-13
tags: python
---  

### Requests模块简介

Requests模块是在Python内置模块的基础上进行了高度的封装，主要用来发送HTTP网络请求，可以轻而易举的完成浏览器的任何操作。
Requests模块比urllib2模块更简洁。  
[官方文档](http://docs.python-requests.org/en/master/)  
[中文文档](http://docs.python-requests.org/zh_CN/latest/)  
[Requests模块源码](https://github.com/kennethreitz/requests)  

#### **Requests模块功能特性**  

* Keep-Alive & 连接池
* 国际化域名和 URL
* 带持久 Cookie 的会话
* 浏览器式的 SSL 认证
* 自动内容解码
* 基本/摘要式的身份认证
* 优雅的 key/value Cookie
* 自动解压
* Unicode 响应体
* HTTP(S) 代理支持
* 文件分块上传
* 流下载
* 连接超时
* 分块请求
* 支持 .netrc

#### **[Requests模块安装](http://docs.python-requests.org/zh_CN/latest/user/install.html)**

可以直接使用python pip进行安装：  
```python
pip install requests
```

### [Requests模块快速上手](http://docs.python-requests.org/zh_CN/latest/user/quickstart.html)

#### **导入模块**  

```python
import requests
```
####  **定制请求头headers**

```python
headers = {
    'Host': 'node16.sleap.com:8089',
    'Referer': 'http://node16.sleap.com:8089/leapid-admin/view/login.html?cb=http%3A%2F%2Fnode15.sleap.com%3A2017',
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36'
}
r = requests.get(url, headers=headers)
```
####  **发送请求**

```python
r = requests.get('https://api.github.com/events')
r = requests.post('http://httpbin.org/post', data = {'key':'value'})
r = requests.put('http://httpbin.org/put', data = {'key':'value'})
r = requests.delete('http://httpbin.org/delete')
r = requests.head('http://httpbin.org/get')
r = requests.options('http://httpbin.org/get')
```
其中`r`为`Response`对象，我们可以从这个对象中获取请求返回的信息。  

####  **传递URL参数**  

例如， httpbin.org/get?key=val。 Requests 允许你使用 params 关键字参数，以一个字符串字典来提供这些参数。
举例来说，如果你想传递 key1=value1 和 key2=value2 到 httpbin.org/get ，那么你可以使用如下代码：  
```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.get("http://httpbin.org/get", params=payload)
```
通过打印输出该 URL，你能看到 URL 已被正确编码：
```python
>>> print(r.url)
http://httpbin.org/get?key1=value1&key2=value2&key2=value3
```

####  **响应内容**

Requests 会自动解码来自服务器的内容。大多数 unicode 字符集都能被无缝地解码。
```python
>>> r = requests.get('https://api.github.com/events')
>>> r.text
u'[{"repository":{"open_issues":0,"url":"https://github.com/...
```
你可以找出 Requests 使用了什么编码，并且能够使用 r.encoding 属性来改变它：
```python
>>> r.encoding
'utf-8'
>>> r.encoding = 'ISO-8859-1'
```

####  **JSON响应内容**

Requests 中也有一个内置的 JSON 解码器，助你处理 JSON 数据：
```python
>>> r = requests.get('https://api.github.com/events')
>>> r.json()
[{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...
```
如果 JSON 解码失败， r.json() 就会抛出一个异常。例如，响应内容是 401 (Unauthorized)，尝试访问 r.json() 将会抛出 ValueError: No JSON object could be decoded 异常。  

####  **更加复杂的POST请求**

你想要发送一些编码为表单形式的数据——非常像一个 HTML 表单。要实现这个，只需简单地传递一个字典给 data 参数。你的数据字典在发出请求时会自动编码为表单形式：  
```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```

#### **响应状态码**

我们可以通过`r.status_code`检测响应状态码：
```python
>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
```

#### **响应头**

```python
>>> r.headers
{
    'content-encoding': 'gzip',
    'transfer-encoding': 'chunked',
    'connection': 'close',
    'server': 'nginx/1.0.4',
    'x-runtime': '148ms',
    'etag': '"e1ca502697e5c9317743dc078f67693f"',
    'content-type': 'application/json'
}
```

#### **Cookie**

如果某个响应中包含一些 cookie，你可以快速访问它们：  
```python
>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)
>>> r.cookies['example_cookie_name']
'example_cookie_value'
```
发送得到的cookie到服务器，可以使用`cookie`参数：
```python
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')
>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```
Cookie 的返回对象为 RequestsCookieJar，它的行为和字典类似，但接口更为完整，适合跨域名跨路径使用。你还可以把 Cookie Jar 传到 Requests 中：  
```python
>>> jar = requests.cookies.RequestsCookieJar()
>>> jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
>>> jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
>>> url = 'http://httpbin.org/cookies'
>>> r = requests.get(url, cookies=jar)
>>> r.text
'{"cookies": {"tasty_cookie": "yum"}}'
```

### [高级用法](http://docs.python-requests.org/zh_CN/latest/user/advanced.html)

#### **会话对象**

会话对象让你能够跨请求保持某些参数。它也会在同一个 Session 实例发出的所有请求之间保持 cookie，所以如果你向同一主机发送多个请求，底层的 TCP 连接将会被重用，从而带来显著的性能提升。  
会话对象具有主要的 Requests API 的所有方法。  
跨请求保持一些 cookie:  
```python
s = requests.Session()

s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
r = s.get("http://httpbin.org/cookies")

print(r.text)
# '{"cookies": {"sessioncookie": "123456789"}}'
```

#### **请求与响应对象**

任何时候进行了类似`requests.get()`的调用，你都在做两件主要的事情。其一，你在构建一个`Request`对象， 该对象将被发送到某个服务器请求或查询一些资源。
其二，一旦`requests`得到一个从服务器返回的响应就会产生一个`Response`对象。**该响应对象包含服务器返回的所有信息，也包含你原来创建的`Request`对象。**  
如果想访问服务器返回给我们的响应头部信息，可以这样做：  
```python
>>> r.headers
{'content-length': '56170', 'x-content-type-options': 'nosniff', 'x-cache':
'HIT from cp1006.eqiad.wmnet, MISS from cp1010.eqiad.wmnet', 'content-encoding':
'gzip', 'age': '3080', 'content-language': 'en', 'vary': 'Accept-Encoding,Cookie',
'server': 'Apache', 'last-modified': 'Wed, 13 Jun 2012 01:33:50 GMT',
'connection': 'close', 'cache-control': 'private, s-maxage=0, max-age=0,
must-revalidate', 'date': 'Thu, 14 Jun 2012 12:59:39 GMT', 'content-type':
'text/html; charset=UTF-8', 'x-cache-lookup': 'HIT from cp1006.eqiad.wmnet:3128,
MISS from cp1010.eqiad.wmnet:80'}
```
**然而，如果想得到发送到服务器的请求的头部，我们可以简单地访问该请求，然后是该请求的头部：**
```python
>>> r.request.headers
{'Accept-Encoding': 'identity, deflate, compress, gzip',
'Accept': '*/*', 'User-Agent': 'python-requests/0.13.1'}
```

### 完整实例

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
# -------------------------
# Author:   wangjj17
# Name:     RequestUtils
# Date:     2019/4/4
# -------------------------

import requests

# requests.Session对象可以在多个http请求之间保持变量、公用cookie、保持长连接从而提高性能等。
session = requests.Session()

headers = {
    'Host': 'node16.sleap.com:8089',
    'Referer': 'http://node16.sleap.com:8089/leapid-admin/view/login.html?cb=http%3A%2F%2Fnode15.sleap.com%3A2017',
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36'
}

login_url = 'http://node16.sleap.com:8089/leapid-admin/api/v1/login'

def login(login_url, username, passwd):
    params = {
        'un': username,
        'pw': passwd
    }
    response = session.post(login_url, params=params, headers= headers)
    # requests会自动管理cookies,通过requests get或post网页之后，若是第一次访问，在response headers里会有set-cookies字段，
    print('response headers:',response.headers)
    # requests会识别这些字段，同时在接下来的get\post中，自动添加这些cookies。
    # 登录成功之后服务器返回给客户端一个cookie sid，保存在session.cookies中。
    print('session cookies:',session.cookies)
    print('result:',response.json()['result'])
    print('data:',response.json()['data'])

create_user_url = 'http://node16.sleap.com:8089/leapid-admin/p/api/v1/leapid/'
def create_user(username, password, roleType, realname, email, phone, state):
    if roleType == 'admin':
        role = 'leapid.admin'
    elif roleType == 'pm':
        role = 'leapid.pm,sql,proc,dhub'
    elif roleType == 'member':
        role = 'leapid.member,sql,proc,dhub'
    params = {
        'username': username,
        'password': password,
        'roles': role,
        'realname': realname,
        'email': email,
        'phone': phone,
        'state': state,
        'department': '',
        'address': '',
        'remark': ''
    }
    response = session.post(create_user_url, params=params, headers=headers)
    print('response headers:', response.headers)
    print('session cookies:', session.cookies)
    print('result:',response.json()['result'])
    print('data:',response.json()['data'])

if __name__ == "__main__":
    login(login_url, 'leapadmin', 'leapadmin')
    username = 'test5'
    password = '123456'
    roleType = 'member'
    realname = 'test'
    email = ''
    phone = ''
    state = 0
    create_user(username,password,roleType,realname,email,phone,state)
```

### 常见问题及解决方法  
- 手动设置一个固定的cookie  
  通过`requests.utils.add_dict_to_cookiejar`对session对象设置cookie  
  ```python
  cookie_default = {
    'ADMSystemCookie': 'CurrentRegion=BJ',
    's_fid': '6802050D76143D97-0A725FF637519282',
    's_vi': '[CS]v1|2E5C0691850361BD-40001198A006DD1B[CE]',
    '_ga': 'GA1.2.664304442.1555566091',
    's_nr': '1555568316455'
  }
  requests.utils.add_dict_to_cookiejar(session.cookies, cookie_default)
  print(session.cookie)
  ```
  通过`requests.utils.cookiejar_from_dict`生成一个cookiejar对象，然后赋值给session.cookie
  ```python
  cookie_default = {
    'ADMSystemCookie': 'CurrentRegion=BJ',
    's_fid': '6802050D76143D97-0A725FF637519282',
    's_vi': '[CS]v1|2E5C0691850361BD-40001198A006DD1B[CE]',
    '_ga': 'GA1.2.664304442.1555566091',
    's_nr': '1555568316455'
  }
  session.cookie = requests.utils.cookiejar_from_dict(cookie_default)
  ```
  也可以使用`session.cookies.set()`或者`update()`

- 获取session中的cookie到字典对象  
  ```python
  cookie_dict = requests.utils.dict_from_cookiejar(session.cookies)
  cookie_dict: {"ADMSystemCookie": "CurrentRegion=BJ", "_ga": "GA1.2.664304442.1555566091", "s_fid": "6802050D76143D97-0A725FF637519282", "s_nr": "1555568316455", "s_vi": "[CS]v1|2E5C0691850361BD-40001198A006DD1B[CE]", "HPassportSignIn": "zI8iM/Ir9AD+rGM7bOdCc4ZchQFlzCTKr+DkaDcjA6FUTwsJVD2wNKCZ9itVpmN/Sl+hCZwwLlAbGyH+6+uNdns7BcOHYTNLWdi+TrI7BqlQSWGSmY/g9Ejiot/BvR4vqOIVQWLOZZgkXBde5u4eLMbXA2ZaTfADw5YwiTe/Vn2FfduONGNltRLPFKpwLDU7EtfWyiUSbiFVa4/UoHxtHY0jb/xEDTarKg+9swiwZuVCwFkBS+uOth8VKUxCrMsMxjLj27lDrlnJXf6/GaHt0yp94TLABl+6l7EzJmo5jPTBcqqblnSLo+GjHEjXVqj3FT7j/GbTqJsxbTr7E59aIGKWfJUjcIXTTMkY4Zcc4wK0SIsTzZDc84J6Sy5/z3hB", ".ASPXAUTH": "B4D0BDCE120E24BEB870F50FF1B664B4B893351BD37906EFD5EDA61965C04899054734C32A1C0CA1BEB2DA0A88B30D299F128C0256E75198041821897EC982F622E6EC89851169477B01922CE190D717C2FB0D3C3527737E4D4A090D630BAB4A51003F0C4965E6CF4C6DDA00142B63045CB9B5619DC4D6AF6CBEE34FE05AA45A3064EB7C05B57644A140C3BAE3B3CDEE", "ASP.NET_SessionId": "iusb3m05na1fbg2kqw1zrg0b"}
  ```

- 当请求的`url`为`https`协议时，报错`requests.exceptions.SSLError:("bad handshake:Error([('SSL routines', 'ssl3_get_server_certificate', 'certificate verify failed')],)",)`  
  [高级用法](http://docs.python-requests.org/zh_CN/latest/user/advanced.html) 中**SSL证书验证**一节中有介绍，可以使用`verify`设置为`False`，Requests能忽略对SSL证书的验证。    
  ```python
  url_WISTS = 'https://oacp.lenovo.com/MCSWebApp/ResponsivePassportService/Anonymous/WISTS.ashx'
  session.get(url_WISTS, params=params, headers=headers, cookies=cookie_dict, verify=False)
  ```

- 以form形式或json形式发送POST请求    

  常常会遇到明明写的是`POST请求`却得到了和`GET请求`一样的结果，但是使用`postman工具`进行模拟POST请求时却能成功，这就是发送请求时参数的形式不对造成的。[参考链接](https://blog.csdn.net/weixin_40283480/article/details/79208413)  

  以form形式发送POST请求  
  ```python
  # 获取小米有品的分类信息  
  headers = {
          "Host": "youpin.mi.com",
          "Content-Type": "application/x-www-form-urlencoded",
          "Referer": "https://youpin.mi.com/",  # 必须带这个参数，不然会报错
          "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36",
               }
  url = "https://youpin.mi.com/app/shopv3/pipe"
  form_data = {"data": '{"result": {"model": "Homepage", "action": "BuildClass", "parameters": {"id": -6}}}'}
  results = requests.post(url, data=form_data, headers=headers).text
  print(results)
  ```
  以json形式发送POST请求
  ```python
  # http://jinbao.pinduoduo.com/index?page=1里面的分类，
  import requests
  import json
  headers = {
      "Content-Type": "application/json; charset=UTF-8",
      "Referer": "http://jinbao.pinduoduo.com/index?page=5",
      "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36",
      }
  url = "http://jinbao.pinduoduo.com/network/api/common/goodsList"
  pyload = {"keyword": "", "sortType": 0, "withCoupon": 0, "categoryId": 16, "pageNumber": 1, "pageSize": 60}
  response = requests.post(url, data=json.dumps(pyload), headers=headers).text
  print(response)
  ```
