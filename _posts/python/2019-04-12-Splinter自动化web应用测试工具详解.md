---
layout: post
title: Splinter自动化web应用测试工具详解
date: 2019-04-12
tags: python
---  
### Splinter简介

Splinter是一个开源的工具用来通过python自动化测试web应用，可以使用python启动浏览器并自动操作网页，模拟用户点击、输入。  

[官方文档](https://splinter.readthedocs.io/en/latest/index.html)  
[中文文档](https://splinter-docs-zh-cn.readthedocs.io/zh/latest/)  

python安装Splinter模块：  
```
pip install splinter
```
Splinter基于selenium，所以需要有浏览器驱动：[Firefox驱动](https://github.com/mozilla/geckodriver/releases)或[Chrome驱动](http://chromedriver.chromium.org/downloads)，
也可以去[selenium官网](https://www.seleniumhq.org/download/)下载。  

python程序中指定使用的浏览器以及浏览器驱动的路径，执行`Browser(driver_name=driver_name, executable_path=executable_path)`便可自动打开相应的浏览器。  

下面示例程序具体操作为打开浏览器，访问登录网站界面，输入用户名、密码，随后点击登录：  
``` python
driver_name = 'chrome'
executable_path = 'D:\ML\Code\chromedriver.exe'
driver = Browser(driver_name=driver_name, executable_path=executable_path)
driver.driver.set_window_size(1400, 1000)
driver.visit(login_url)
driver.find_by_id('username').fill(username)
driver.find_by_id('password').fill(passwd)
driver.find_by_id('FormLoginBtn').click()
```
登陆后，`browser.cookies.all()`中保存了本次登录的cookie信息（dict类型），调用`quit()`方法即可关闭浏览器。  

Splinter可以控制浏览器访问，模拟用户操作，则可以用来做一些自动化操作的简单应用。  

### Splinter查找标签的基本函数  

[基本函数](https://splinter-docs-zh-cn.readthedocs.io/zh/latest/finding.html)：  
```python
browser.find_by_css('h1')
browser.find_by_xpath('//h1')
browser.find_by_tag('h1')
browser.find_by_name('name')
browser.find_by_text('Hello World!')
browser.find_by_id('firstheader')
browser.find_by_value('query')
```
通过`id`查找页面标签：  
```
find_by_id()
```
通过`name`查找页面标签：
```
find_by_name()
```
通过`text`查找页面标签：  
```html
<button class="btn btn-item connect">下一步</button>
```
```python
find_by_text(u"下一步")
```
通过`option`的`text`查找特定`option`标签：  
```html
<ul class="select2-results__options" role="tree" id="select2--x3-results" aria-expanded="true" aria-hidden="false">
    <li class="select2-results__option" role="treeitem" aria-disabled="true">请选择数据库连接</li>
    <li class="select2-results__option" id="select2--x3-result-ie1y-35" role="treeitem" aria-selected="true">10.110.181.39_3306_datahub</li>
    <li class="select2-results__option select2-results__option--highlighted" id="select2--x3-result-ebx3-34" role="treeitem" aria-selected="false">10.110.181.39_3306_leapid</li>
    <li class="select2-results__option" id="select2--x3-result-7jr1-33" role="treeitem" aria-selected="false">10.110.181.39_3306_testDB</li>
</ul>
```
```python
find_option_by_text("10.110.181.39_3306_leapid")
```
使用`find_by_xpath`选取`class="keyword"`的标签：  
```html
<input type="text" class="keyword" placeholder="请输入要查找的表名   ">
```
```python
find_by_xpath('//*[@class="keyword"]')
```
使用`find_by_xpath`选取`id`中包含`default`字符串的元素：  
```html
<li class="select2-results__option select2-results__option--highlighted" id="select2-batch-dbName-result-42ba-default" role="treeitem" aria-selected="false">default</li>
```
```python
find_by_xpath('//*[contains(@id, "default")]')
```

### Splinter查找标签的复杂用法  

Splinter中标签查找支持链式查找，及可以使用`driver.find_by_id().first.find_by_xpath()`查找隐藏在特定标签(特征明显)内部的指定标签(特征不明显)。  

需要注意的是：  

* `find_by_*()`返回的是一个`ElementList`。  
* 链式查询只能是在包含关系时，才能查找到隐藏的标签，如2所示。  

```html
<li class="db-item" data-catalog="datahub" data-schema="" data-dbname="datahub">
    <i class="check dbSelected"></i>
    <i class="db-icon"></i>
    <p title="datahub">datahub</p>
    <a class="getTbl got">-</a>
    <ul class="tbl">
        <li class=" tbl_client_task" data-tblcatalog="datahub" data-tblschema="">
            <b>1</b>
            <i class="check tblSelected "></i>
            <i class="tbl-icon"></i>
            <p title="client_task">client_task</p>
        </li> 
        <li class=" tbl_db_info" data-tblcatalog="datahub" data-tblschema="">
            <b>2</b>
            <i class="check tblSelected "></i>
            <i class="tbl-icon"></i>
            <p title="db_info">db_info</p>
        </li>
    </ul>
</li>
```

1.选取数据库所有表：

```python
driver.find_by_xpath('//*[@data-catalog="'+db+'"]').first.find_by_xpath('.//*[@class="check dbSelected"]').click()
```

2.选择特定数据库中的特定表：

```python
driver.find_by_xpath('//*[@data-catalog="'+db+'"]').first.find_by_xpath('.//*[@class="tbl_'+table+'"]').first.find_by_xpath('.//*[contains(@class,"check tblSelected")]').first.click()
# eles = driver.find_by_xpath('//*[@data-catalog="'+db+'"]')
# for ele in eles:
#     ele_ts = ele.find_by_xpath('.//*[@class="tbl_'+table+'"]')
#     for ele_t in ele_ts:
#         print(len(ele_t.find_by_xpath('.//*[contains(@class,"check tblSelected")]')))
#         ele_t.find_by_xpath('.//*[contains(@class,"check tblSelected")]').first.click()
```

### 完整实例

* [12306自动订票](https://github.com/jacky-wangjj/12306/blob/master/splinterTicket/hackTickets.py)
* [datahub数据自动导入](https://github.com/jacky-wangjj/PythonUtils/blob/master/datahub/SplinterUtils.py)