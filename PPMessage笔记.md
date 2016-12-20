---
date: 2016-09-20 11:59
status: public
tags: Python,PPMessage
title: PPMessage代码阅读笔记
---

## 框架
- 后端 Tornado
    - 不在后端render，由前端负责模版渲染（看起来好像是这样，待确认）
- 前端 Angular


## 配置
- 第一次启动后进入配置界面，配置后的项会保存在 /ppmessage/bootstrap/config.json
    - 删掉可以重新安装配置过程
- core/constant.py 有一堆枚举常量
- core/utils/db2cache.py 有需要缓存的对象的列表
- 页面路由总表可以通过在 backend/main.py 的 `tornado.web.Application.__init__` 方法打印 handlers 获得
> 有了总路由表，以后想看某个模块的代码只需要根据链接找到对应的 handler，就可以大概知道在哪里查看/修改代码了（当然Angular的知识也要懂）。

## 启动
- backend/main.py 是整个项目启动后首先加载的
    1. 实例化 MainApplication(tornado.web.Application)
        - MainApplication 
            - 负责初始化 redis、handlers 以及一些配置
            - 通过 core/main/ppwebserivice.py 注册所有的服务、 handlers 和 delegates
    1. copy_default_icon 没研究是干嘛的，应该不重要
    1. load_db_to_cache 将数据库内容加载入缓存        
    1. 启动 tornado http 服务
        
- EntryHandler.get 决定首页路由
    - 初次启动就路由到 ppconfig
    - 配置后路由到后台 ppconsole

## 服务注册机制
- 直接看如下简化后的代码 （core/main/ppwebservice.py）
```python
_r = {}
class Meta(type):
    def __init__(cls, name, bases, dict_):
        _r[name] = cls
        type.__init__(cls, name, bases, dict_)
        return

MetaWebService = Meta("MetaWebService", (object,), {})

class AbstractService(MetaWebService):
    def somefunction(): #随便写个函数避免类里面什么都没有 
        return
```
- 用了上面的代码，只要任何类继承 AbstractService，在 import 这个类的时候就会注册进 _r 里面。
- 也正因为这样，backend/main.py 里面有很多 import 的内容是代码里面没有用到的，如果IDE提示 是否优化import，千万别“优化”掉了（除非二次开发的过程发现某个 Service 确定真的不会用到）。

## ppconsole
- 还没看😄

## Bugs
- 输入中文保存不进MySql
    - 修改：db/sqlmysql.py
         
    ```python
    db_string = "mysql+pymysql://%s:%s@%s:%s/%s" % \
    #改为
    db_string = "mysql+pymysql://%s:%s@%s:%s/%s?charset=utf8" % \
    ```
    
## 可以改进的地方
- 去掉连接 ppmessage.com 的部分以避免官网连不上的时候卡
- `settings['debug']` 写在 main 方法不好，考虑配置单独写在一个文件
- `== none` 改成 `is None`
- 以下代码是不推荐的 
```python
    import sys
    reload(sys)
    sys.setdefaultencoding('utf8')
```
> 参考：[这里](http://stackoverflow.com/questions/28657010/dangers-of-sys-setdefaultencodingutf-8) 和 [这里](http://stackoverflow.com/questions/3828723/why-should-we-not-use-sys-setdefaultencodingutf-8-in-a-py-script)
- Handler 里面打开html文件、读取、返回内容到前端这些代码大量重复，可以考虑封装一下
- 改变IP的话需要重新去掉安装 /ppmessage/bootstrap/config.json ？部署或者迁移是个问题。