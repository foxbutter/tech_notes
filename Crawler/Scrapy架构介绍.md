# 概览

本文描述了Scrapy的架构图、数据流动、以及个组件的相互作用

### 一、架构图与数据流

<img src="https://cdn.jsdelivr.net/gh/foxbutter/pics_cdn/cut/scrapy_arc.png" alt="scrapy_arc"  />

上图中各个数字与箭头代表数据的流动方向和流动顺序，具体执行流程如下：

##### 1. Scrapy将会实例化一个Crawler对象，在Crawler中：

　　　　创建spider对象----_create_spider 

　　　　创建engine对象----_create_engine

　　　　通过engine对象打开spider并生成第一个request---- yield self.engine.open_spider(self.spider, start_requests) 

　　　　实例化调度器对象----Scheduler

　　　　启动引擎---- yield defer.maybeDeferred(self.engine.start) 

##### 2. 引擎从Spider获取初始请求

　　　　----_next_request

　　　　----_next_request_from_scheduler

##### 3. 引擎把初始请求给调度器，并向调度器询问下一次请求

　　　　----scheduler.next_request

##### 4. 调度器会对url进行指纹去重，如果是未爬取过的url，就把它放到队列中等待，并把下一个request返回给引擎

　　　　把url放入到队列中----enqueue_request

　　　　返回下一个request----next_request

##### 5. 引擎把从调度器返回的request途径下载中间件交给下载器

　　　　----download

##### 6. 一旦页面完成下载，下载器将会生成一个响应，途径下载中间件，再把它交给引擎

　　　　----download

##### 7. 引擎接收到响应，并把它途径爬虫中间件，再交给spider

　　　　----_handle_downloader_output

##### 8. spdier接收到响应，并对它进行解析，解析出Items或者新的Request，再把它们途径爬虫中间件，提交给引擎

　　　　----parse

##### 9. 引擎把接收到的items提交给Item Pipeline，把接收到的Request提交给调度器

##### 10. 从步骤1开始重复该过程，直到不在有request

### 二、各组件介绍

##### ENGINE

引擎（engine）控制所有部件间的数据流，并在某些事件发生时触发事件

##### Scheduler
调度器（scheduler）接收来自引擎的request，并对它去重，放入到请队列中；并根据队列的取出规则，把请求按顺序返回给引擎

##### Downloader

下载器（Downloader）获取网页数据并返回给引擎

##### Spiders

爬虫（Spiders）用来解析response，提取出Items和新的Requests

##### Item Pipeline

对Items进行进一步的清洗，并持久化

##### Downloader middlewares

下载中间件可以获得下载器和引擎之间的数据流，并对它们做一些处理，比如：

- 在request送到下载器之前对它做一些处理，可以添加User_Agent，修改IP等
- 对response做一些处理

##### Spider middlewares
爬虫中间件可以获取爬虫和引擎之间的数据流，并对它们做一些处理