---
date: 2016-11-14 15:36
status: public
tags: Python,Flask,nginx
title: Flask部署
---

> 部署一个Flask写的Python后端，碰到几个不大不小的坑。记录一下。

## 系统架构
- 前端部署在nginx，后端部署在tornado，前端通过proxy访问后端api。

## 前端
### nginx
- 开启 gzip 
```nginx
  gzip on;
  gzip_min_length 1k;
  gzip_comp_level 5;
  gzip_types text/json text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;  
```
+ api请求转发到后端服务器
```nginx
server{
  location ^~/api/ {
    proxy_pass http://127.0.0.1:5000;
  }  
}
```
## 后端
### 升级python到2.7
- [参考文档](http://ruter.sundaystart.net/2015/12/03/Update-python/)

### pip
- 依赖
```bash
yum -y install pcre-devel openssl openssl-devel      
```
- ssl还是没装上，所以不用https方式安装其他库
```bash
# 利用豆瓣的源来安装，比较快
 /usr/local/bin/pip2.7 install -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com -r requirements.txt                            
```


### tornado
```python
#coding=utf-8
from tornado.wsgi import WSGIContainer
from tornado.httpserver import HTTPServer
from tornado.ioloop import IOLoop
from app import app # flask主文件为app.py

http_server = HTTPServer(WSGIContainer(app))
http_server.listen(5000) 
IOLoop.instance().start() 
```
然后用python启动上面的文件即可。

## 坑
- 安装MySQL-Python的时候死活装不上
  - 解决：改用 PyMySQL ，python 原生无任何native依赖。而且和 SQLAlchemy 结合良好，SQLAlchemy 在create_engine 的时候连接字符串从 `mysql` 改成 `mysql+pymysql` 即可。另外还有个小问题，PyMySQL在调用`func.IF(XXX.id is null,1,0)`的时候会报错，不知道是啥原因，只好改成`func.IF(XXX.id > 0,1,0)`。
- 用过的 session 记得关闭（手动滑稽）