---
layout: post
title: Kettle Summary
modified:
categories: 
description: "记录最近一段时间使用Kettle做数据迁移踩过的坑"
tags: []
image:
  feature:
  credit:
  creditlink:
date: 2015-09-01T23:30:00-04:00
---


最近公司系统重写接近尾声，接下来重要的一步就是做老数据的迁移，这次数据库表结构变化还是相当多。由于之前就听说过Kettle这个工具，趁此次机会赶紧小玩一把。

# 使用配置
启动GUI程序之后首先设置了资源库，创建资源库时可选基于数据库的和基于文件的，由于不想创建数据库，直接选择了基于文件的资源库创建。

接下来是数据库连接的设置，关于此设置，推荐使用如下配置方法。

- 编辑$HOME/.kettle/kettle.properties，添加以下内容
{% highlight ini %}
mysql.db1.host=192.168.0.11
mysql.db1.database=db1
mysql.db1.port=3306
mysql.db1.user=db_user
mysql.db1.password=db_password
mysql.db2.host=192.168.0.12
mysql.db2.database=db2
mysql.db2.port=3306
mysql.db2.user=db_user
mysql.db2.password=db_password
{% endhighlight %}

- 在资源库的数据连接里面做如下设置，注意密码是 ${mysql.db2.password}
![DB Connection]({{ site.url }}/images/kettle_resource_db_conn.png)

- 注意若有表名列名等与数据库保留关键字冲突，一定要勾选如下选项 Preserve case of reserved words
![Preserve Word]({{ site.url }}/images/kettle_resource_db_preserve.png)

- 若数据库导入导出乱码，请确保加入characterEncoding参数
![Preserve Word]({{ site.url }}/images/kettle_resource_db_messy.png)

# 转换作业

## 转换
Kettle里面一个转换类似于一个函数，有数据输入，数据处理，然后有输出，但这些数据更像是集合。
![Transform]({{ site.url }}/images/kettle_transform.png)
上面是一个kettle转换图,首先从表中取出数据流，然后经过各种处理，连接，转换，最后到数据表的更新。
从设计考虑上不能以单个看待数据，要以二维的数据集合考虑流程的设计。

## 作业
Kettle里面的作业可以调用转换，更可以定时执行，或远程执行。
![Job]({{ site.url }}/images/kettle_job.png)
以上是一个作业的流程图，有开始，有结束，更像是一个事件的执行。

# 命令行执行
以上转换和作业最终需求是能在linux服务器上执行，经过研究pan.sh脚本和kitchen.sh脚本。

在命令行运行转换
{% highlight bash %}
pan.sh -file=/path/to/transform/file.ktr -level=basic -param:id=998
{% endhighlight %}

在命令行执行JOB
{% highlight bash %}
kitchen.sh -file=/path/to/job/file.kjb -level=debug -param:id=998 -param:name=haha -logfile=/tmp/kettle_123.log
{% endhighlight %}

# 总结
Kettle在图形化方面做的挺好的，最近又加入了一些NOSQL的支持，以及大数据的支持，用其做一些数据迁移工作还是挺方便的。
缺点是某些细节不够友好，比如说Internal.Job.Filename.Directory在资源库里面的处理不是很好等，希望能够在细节方面完善下就好了。








