## Filebeat和Logstash的简单配置和使用

1. 介绍

Filebeat是一个轻量级的转发和集中日志数据的托运工具, Filebeat监控指定的日志文件或位置, 收集日志事件, 并将其转发到Elasticsearch或Logstash进行索引. 

Filebeat的工作方式如下: 当启动Filebeat时, 它会开启一个或多个输入, 这些输入将在你所指定的位置查找日志数据. 对于Filebeat所找到的每个日志, Filebeat都会启动收集器. 每个harvester都读取一个日志以获取新内容, 并将新日志数据发送到libbeat, libbeat会汇总事件并将汇总的数据发送到您为Filebeat配置的输出. 
   
![HowFilebeatWork](https://github.com/unknown-admin/document/blob/master/images/filebeat.png)
   
如果想要了解更多关于Filebeat, [查看官方文档](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)
   
Logstash是一个具有实时流水线功能的开源的数据收集引擎. 它可以动态规划来自不同数据来源的数据, 并将数据规范为为你选择的目标. 虽然Logstash最初被用作收集日志，但是它的功能远远超出了最初的规划. 任何类型的事件都可以通过输入(input), 过滤器(filter)和输出(output)插件来丰富和转换. 
   
如果想要了解 更多关于logstash, [查看官方文档](https://www.elastic.co/guide/en/logstash/current/index.html)


   