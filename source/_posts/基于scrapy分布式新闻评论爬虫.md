---
toc: true
title: 基于scrapy分布式新闻评论爬虫
date: 2018-03-28 21:00:04
tags: [爬虫, scrapy, redis, weibo, news163, ifeng, toutiao]
---


我的毕设主题是：时事新闻评论分析软件的设计与实现。要分析评论，就需要评论数据，于是就写起了爬虫。

<!--more-->
项目仓库：[NewsCommentsSpider](https://github.com/cap-ljf/NewsCommentsSpider)
如果感觉对您有帮助的话给个小小的`star`吧，哈哈。

对于爬虫，搜索得到的文章大部分都是python爬虫，“python有天然的爬虫优势”，虽然我也讲不清python爬虫有什么天然的优势，可能在html文件解析、json解析、有大量成熟好用的工具包。于是我开始学习`scrapy`框架，上手很简单，而且官方文档比较详细。

[Scrapy入门教程](http://scrapy-chs.readthedocs.io/zh_CN/0.24/intro/tutorial.html)

#### 开发环境
系统：`Ubuntu16.10`
开发软件：`Pycharm 2017.3.3 Professinal Edition`
工具包：
```
python 3.5
pip 9.0.1
scrapy 
beautifulsoup4 4.5.1
ipython 6.2.1
lxml 4.1.1
PyMySQL3 0.5
redis 2.10.6
requests 2.18.4
Scrapy 1.5.0
scrapy-redis 0.6.8
selenium 2.48.0 
```
### 1. Scrapy入门，搭建NewsCommentsSpider项目
#### 安装scrapy
我的系统是`Ubuntu16.10`，使用下列命令：
`pip install scrapy`
#### 创建项目
`scrapy startproject NewsCommentsSpider`
*下面一段解释来自scrapy中文官网*
该命令将会创建包含下列内容的`NewsCommentsSpider`目录：
```
NewsCommentsSpider/
    scrapy.cfg
    NewsCommentsSpider/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
            ...
```
这些文件分别是:
- `scrapy.cfg`: 项目的配置文件
- `NewsCommentsSpider`/: 该项目的python模块。之后您将在此加入代码。
- `NewsCommentsSpider/items.py`: 项目中的item文件.
- `NewsCommentsSpider/pipelines.py`: 项目中的pipelines文件.
- `NewsCommentsSpider/settings.py`: 项目的设置文件.
- `NewsCommentsSpider/spiders/`: 放置spider代码的目录.

#### 数据库设计
OK，到这儿我们就已经成功搭建好了scrapy框架。
现在来看我毕设的题目：
```
题目：基于Python的时事新闻评论分析软件的设计与实现
主要数据：凤凰网、今日头条、网易新闻、微博四大平台的新闻评论。（一开始我是准备爬知乎和微信公众号的评论的，但是知乎问题的回答一般都很长，而且表达的主观情绪较少，一般都是摆数据讲道理，不太好分析；而微信公众号的评论接口我一直没找到，只好作罢，如果您有关于微信公众号文章评论接口相关的信息或想法，恳求告知：邮箱1579461369@qq.com）
```
针对上述需求，我设计了两个表：
1. 文章表：（**article**）新闻文章或微博信息
2. 评论表：（**comment**）文章评论相关信息

请看：
【**article**】
| Field      |    Type | Null  | Key | Default | Extra |
| :-------- | --------:| :--: |
| id  | int(11) |  NO   | PRI |  | auto_increment |
| keyword  | varchar(20) |  NO   | MUL |  |  |
| url  | varchar(150) |  NO   |  |  |  |
| author  | varchar(50) |  YES   |  |  |  |
| create_time  | int(11) |  NO   |  | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| comment_count  | int(11) |  NO   |  |  |  |

【**comment**】
| Field      |    Type | Null  | Key | Default | Extra |
| :-------- | --------:| :--: |
| id  | int(11) |  NO   | PRI |  | auto_increment |
| url  | varchar(150) |  NO   | MUL |  |  |
| author  | varchar(50) |  YES   |  |  |  |
| create_time  | int(11) |  NO   |  | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| comment  | varchar(1000) |  NO   |  |  |  |
| praise  | smallint(6) |  YES   |  |  |  |

**SQL语句**
第一次设计的时候`author`字段长度设计的比较小，导致在测试网易新闻评论爬取的时候出现异常，于是增加了`author`字段的大小。
```sql
CREATE DATABASE minsheng;
USE minsheng;
CREATE TABLE article(
  id INT NOT NULL AUTO_INCREMENT COMMENT '主键',
  keyword varchar(20) NOT NULL COMMENT '关键词',
  url VARCHAR(150) NOT NULL UNIQUE COMMENT '文章url',
  author VARCHAR(20) COMMENT '作者',
  create_time TIMESTAMP COMMENT '文章创建时间',
  comment_count int NOT NULL COMMENT '评论数',
  PRIMARY KEY (id),
  INDEX idx_keyword (keyword)
)ENGINE = InnoDB CHARSET = 'utf8' COMMENT '文章表';
CREATE TABLE comment(
  id INT NOT NULL AUTO_INCREMENT COMMENT '主键',
  url VARCHAR(150) NOT NULL COMMENT '文章链接',
  comment VARCHAR(1000) NOT NULL COMMENT '评论',
  author VARCHAR(20) COMMENT '评论者',
  praise SMALLINT COMMENT '点赞数',
  create_time TIMESTAMP COMMENT '评论时间',
  PRIMARY KEY (id),
  INDEX idx_url (url)
)ENGINE = InnoDB CHARSET = 'utf8mb4' COMMENT '评论表';
ALTER TABLE comment MODIFY COLUMN author VARCHAR(50);
ALTER TABLE article MODIFY COLUMN author VARCHAR(50);
```
#### 定义Item
既然已经有了数据库表，我们就可以知道我们需要那些数据，就可以编写`Item`类。
```python
import scrapy
from scrapy import Field

class CommentItem(scrapy.Item):
    # 文章链接
    url = Field()
    # 评论内容
    comment = Field()
    # 评论者
    author = Field()
    # 点赞数
    praise = Field()
    # 评论时间
    create_time = Field()
```
#### Item Pipeline，使用Mysql数据库存储数据
编写你自己的item pipeline很简单，每个item pipiline组件是一个独立的Python类，同时必须实现`process_item`方法。
这里我们使用`pymysql`包操作mysql数据库。首先在`pipeline`初始化时创建数据库连接，并获取`cursor`。然后在`process_item`方法中执行增删改查操作。最后**关闭连接**，一定要记得关闭连接，否则会产生大量的连接以及游标，最后导致内存泄漏。
废话少说，上代码：
```python
# -*- coding: utf-8 -*-
import logging

import pymysql
from datetime import datetime

from NewsCommentsSpider.settings import *


class MySQLPipeline(object):
    # 定义数据存储方式
    def __init__(self):
        try:
            self.connect = pymysql.connect(
                host=MYSQL_HOST,
                db=MYSQL_DBNAME,
                user=MYSQL_USER,
                passwd=MYSQL_PASSWD,
                charset='utf8',
                use_unicode=True
            )
            # 通过cursor执行增删改查
            self.cursor = self.connect.cursor()
            logging.debug('mysql conn success!')
        except Exception as error:
            logging.error('mysql conn error!:', error)

    def process_item(self, item, spider):
        try:
            self.cursor.execute(
                """insert into comment(url, comment, author, praise, create_time)
                                  value (%s, %s, %s, %s, %s)""",
                (item['url'],
                 item['comment'],
                 item['author'],
                 item['praise'],
                 # item['create_time']
                 datetime.fromtimestamp(item['create_time'])
                 )
            )
            # 提交sql语句
            self.connect.commit()
        except Exception as error:
            # 异常打印日志
            logging.error("数据库插入异常:", error)

        return item

    def __del__(self):
        try:
            # if self.cursor:
            #     self.cursor.close()
            if self.connect:
                self.connect.close()
        except Exception as error:
            logging.error("conn or cursor 关闭失败:", error)
```
代码中我注释了这样两段代码
1. `item['create_time']`，如果直接使用`timestamp`执行`insert`。这里出现异常：
```bash
Message: '数据库插入异常:'
Arguments: (InternalError(InternalError(1292, "Incorrect datetime value: '1522110959' for column 'create_time' at row 1"),),)
```
我也不知道为什么，python操作mysql插入数据库为什么会出现这种情况，后来我将时间戳改成`datetime`类型就没问题了。
2. `__del__()`，在这个函数我进行了`cursor`和`connect`的关闭，但是不知道为什么游标一直关不掉，debug信息指示：
```
Exception ignored in: <bound method Cursor.__del__ of <pymysql.cursors.Cursor object at 0x7f702d1c93c8>>
Traceback (most recent call last):
  File "/usr/local/lib/python3.5/dist-packages/PyMySQL3-0.5-py3.5.egg/pymysql/cursors.py", line 41, in __del__
  File "/usr/local/lib/python3.5/dist-packages/PyMySQL3-0.5-py3.5.egg/pymysql/cursors.py", line 47, in close
ReferenceError: weakly-referenced object no longer exists
```
在`stackoverflow`上多次解释说在`__def__`中按顺序关闭，但是我尝试之后还是不行，不过这个`Exception`不影响正常运行。

费老大劲终于写完了`item Pipeline`，但是还有最关键的一步，就是在`settings.py`中添加这个`MySQLPipeline`。
在`settings.py`中加上下面内容：
```python
# mysql配置
MYSQL_HOST = 'localhost'
MYSQL_DBNAME = 'minsheng'
MYSQL_USER = 'root'
MYSQL_PASSWD = ''

ITEM_PIPELINES = {
    'NewsCommentsSpider.pipelines.MySQLPipeline': 300,
}
```
#### 下载器中间件（Downloader Middleware）
现在的网站不好爬啊，各个大公司都有丰富的反爬虫手段，我们刚才写的都不是爬虫核心，真正的爬虫核心是解析数据和伪装手段。而伪装手段有：
1. request header
2. User-Agent
3. proxy
4. cookies

`scrapy`框架提供了`Downloader Middleware`供我们对`request`进行设置。
上代码：
【middlewares.py】
```python
# -*- coding: utf-8 -*-
import random

from scrapy.downloadermiddlewares.useragent import UserAgentMiddleware
# from NewsCommentsSpider.cookies import cookies
from NewsCommentsSpider.user_agent import agents


class UserAgentMiddleware(UserAgentMiddleware):
    """ 换User-Agent """
    pass
    def process_request(self, request, spider):
        agent = random.choice(agents)
        request.headers['User-Agent'] = agent


class CookiesMiddleware(object):
    """ 四个网站，选择相应cookie """
    # def process_request(self, request, spider):
    #     if spider.name == 'weibo':
    #         cookie = random.choice(cookies)
    #         request.cookies = cookie
    

class HeadersMiddleware(object):
    pass
    

class ProxiesMiddleware(object):
    pass
```
【user-agent.py】
```python
# encoding=utf-8

agents = [
	"Mozilla/5.0 (Linux; U; Android 2.3.6; en-us; Nexus S Build/GRK39F) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1",
	...
]
```
这里就不列出所有内容了
同理，`cookies.py`和`proxies.py`也是同样的道理。

到这里，“准备工作”就已经都完成了，现在终于到主角`Spider`登场。

#### Spiders
看一个示例，就拿相对简单的今日头条评论`ToutiaoSpider`来讲解。
先上代码：
```python
# -*- coding: utf-8 -*-

"""
@author: cap_ljf
@time: 18-3-16 上午11:25
"""
import json
import re

from scrapy_redis.spiders import RedisSpider

from NewsCommentsSpider.items import CommentItem


class ToutiaoSpider(RedisSpider):
    name = 'toutiao'
    redis_key = 'toutiao_url'

    '''
        新闻链接：https://www.toutiao.com/a6533396129860551182/
        评论接口：https://www.toutiao.com/api/comment/list/?group_id=6533396129860551182&item_id=6533396129860551182
    '''
    def __init__(self):
        self.allowed_domains = ['www.toutiao.com']
        self.url = 'https://www.toutiao.com/a'

    def parse(self, response):
        content = json.loads(response.body.decode())
        id = re.search(r'([0-9]+)', response.url).group(1)
        url = self.url + str(id)
        comments = content['data']['comments']
        for comment in comments:
            item = CommentItem()
            item['url'] = url
            item['comment'] = comment['text']
            item['author'] = comment['user']['name']
            item['praise'] = comment['digg_count']
            item['create_time'] = comment['create_time']
            yield item
```
- `name`：用于区别Spider。 该名字必须是唯一的，您不可以为不同的Spider设定相同的名字。
- `redis_key`：使用scrapy-redis，必须要有这个字段且唯一，相当于`scrapy`的`start_urls`，包含了Spider在启动时进行爬取的url列表。 因此，第一个被获取到的页面将是其中之一。 后续的URL则从初始的URL获取到的数据中提取。
- `parse()`：是spider的一个方法。 被调用时，每个初始URL完成下载后生成的 Response 对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成item)以及生成需要进一步处理的URL的 Request 对象。

因为我们项目使用了`scrapy-redis`，所以这里的`class`父对象是`RedisSpider`，普通的`xxxSpider`父对象是`Spider`。

得到了`response`对象之后就可使用多种方法解析获取有用的信息了，解析方法有：
1. beautifulSoup
2. XPath 
3. re

如果使用`scrapy`，推荐使用[XPath](http://www.w3school.com.cn/xpath/xpath_syntax.asp)，语法很简单，半个小时基本能掌握。

由于解析数据是一个麻烦的事情，而我**刚好**找到了四个网站中的三个的评论接口，它们返回的数据直接是评论的`JSON`数据。可以直接用Python的`dict`解析数据。
在项目目录下使用命令行执行`scrapy crawl toutiao`，就可以在数据库中看到数据啦。
不信？悟空，你看（观音指）：
![Alt text](https://app.yinxiang.com/shard/s15/res/dcd35021-e4af-42dd-856b-712b5c0061bc/1522163039866.png)
哦哦，因为代码是用的`redis`版本，还有相关`settings.py`没有讲解，所以上面无法执行，可直接去github clone代码下来运行看看效果。如果非要运行起来，看下面`2. Redis，分布式爬虫`

### 2. Redis，分布式爬虫
第一部分介绍了如何使用scrapy爬取评论。现在我们对代码进行重构，增加`Redis`进行分布式爬虫，增加爬虫效率。

参考：[新浪微博分布式爬虫分享](https://blog.csdn.net/bone_ace/article/details/50904718)
> 分布式中有一台机充当Master，安装Redis进行任务调度，其余机子充当Slaver只管从Master那里拿任务去爬。原理是：Slaver运行的时候，scrapy遇到Request并不是交给spider去爬，而是统一交给Master机上的Redis数据库，spider要爬的Request也都是从Redis中取来的，而Redis接收到Request后先去重再存入数据库，哪个Slaver要Request了再给它，由此实现任务协同。

#### Redis
> Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持字符串、哈希表、列表、集合、有序集合，位图，hyperloglogs等数据类型。内置复制、Lua脚本、LRU收回、事务以及不同级别磁盘持久化功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动分区。

不了解redis的去官网学一下，上手很简单。我的导师某日天是Redis中文翻译者之一。
[Redis中文官网](http://www.redis.net.cn/tutorial/3501.html)

#### scrapy-redis
scrapy-redis是一个基于redis的scrapy组件，通过它可以快速实现简单分布式爬虫程序，该组件本质上提供了三大功能：
- scheduler - 调度器
- dupefilter - URL去重规则（被调度器使用）
- pipeline   - 数据持久化

看一下`settings.py`中的`Redis配置`
```python
# redis配置
# 使用scrapy-redis里的调度器组件，不使用默认的调度器
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
# 允许暂停，redis请求记录不丢失
SCHEDULER_PERSIST = True
# 使用scrapy-redis里的去重组件，不使用scrapy默认的去重方式
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 连接Redis配置
REDIS_HOST = 'localhost'
REDIS_PORT = '6379'
REDIS_DB = 0 # 指定db为0
REDIS_PASSWD = ''
```



### 3. 爬虫技巧之接口的获取

当你了解了基础的爬虫知识之后，阻碍你顺利爬取数据的还有数据源的获取，每一个网站都有不同的架构，有些网站可能想让自己的内容更容易的让搜索引擎引用，同时又不想被恶意的爬虫访问，所以各个网站的后台数据送到前台展示方式都不同。

经过我最近频繁的爬虫，我了解到一个非常重要的规律，那就是**对于文本类型的数据一般都是使用JSON数据格式传输**。但是这个`JSON数据接口`一般都非常隐蔽，比如`凤凰网`的评论展示接口，它使用了回调使用`js`进行处理之后再给到前台展示，所以你第一眼看到的不是整齐的`JSON`数据，而是一对乱糟糟不规律的类html文本。而`今日头条`则非常开放。


好了，这一次的分享就到这儿了，分享一个不太哲学的网站[哲学](http://www.mmjpg.com)
爬取太频繁会封ip哦～