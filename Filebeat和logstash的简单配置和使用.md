## Filebeat和Logstash的简单配置和使用

### 1. 介绍

Filebeat是一个轻量级的转发和集中日志数据的托运工具, Filebeat监控指定的日志文件或目录, 收集日志事件, 并将其转发到Elasticsearch或Logstash进行索引. 

Filebeat的工作方式如下: 当启动Filebeat时, 它会开启一个或多个输入, 这些输入将在你所指定的位置查找日志数据. 对于Filebeat所找到的每个日志, Filebeat都会启动收集器. 每个harvester都读取一个日志以获取新内容, 并将新日志数据发送到libbeat, libbeat会汇总事件并将汇总的数据发送到您为Filebeat配置的输出. 
   
![HowFilebeatWork](https://github.com/unknown-admin/document/blob/master/images/filebeat.png)
   
如果想要了解更多关于Filebeat, [查看官方文档](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)
   
Logstash是一个具有实时流水线功能的开源的数据收集引擎. 它可以动态规划来自不同数据来源的数据, 并将数据规范为为你选择的目标. 虽然Logstash最初被用作收集日志，但是它的功能远远超出了最初的规划. 任何类型的事件都可以通过输入(input), 过滤器(filter)和输出(output)插件来丰富和转换. 
   
如果想要了解 更多关于logstash, [查看官方文档](https://www.elastic.co/guide/en/logstash/current/index.html)

### 2. 安装
- #### Filebeat:

mac
````shell script
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-darwin-x86_64.tar.gz
tar xzvf filebeat-7.5.1-darwin-x86_64.tar.gz
````
linux:
```shell script
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-linux-x86_64.tar.gz
tar -zxvf filebeat-7.5.1-linux-x86_64.tar.gz
```
Docker:
```shell script
docker pull docker.elastic.co/beats/filebeat:7.5.1
```

- #### Logstash:

mac & linux
```shell script
curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.5.1.tar.gz
tar -zxvf logstash-7.5.1.tar.gz
```
Docker:
```shell script
docker pull docker.elastic.co/logstash/logstash:7.5.1
```

更多下载方式, [参考官方文档](https://www.elastic.co/cn/downloads/logstash)

### 3. 配置

本文中主要使用Filebeat收集日志, 然后将数据转发到Logstash进行处理, 最后输出到ActiveMQ, 所以本文中也将围绕这个方向进行配置.
- #### Logstash
Logstash有两个必要的基本组件(`input`和`output`)和一个可选组件(`filter`). `input`组件定义了数据的来源, `filter`组件按照你所制定的规则对数据进行修改, `output`指定组件向哪里写入数据.

![HowLogstashWork](https://github.com/unknown-admin/document/blob/master/images/basic_logstash_pipeline.png)

运行下面的命令，来测试你的Logstash是否正常
```shell script
cd logstash-7.5.1
bin/logstash -e 'input { stdin {} } output { stdout {} }'
```
`-e`表示从命令行中读取配置. 在这个例子中可以看到，我们定义个两个组件`input`和`output`, 分别从控制台接收数据，并输出到控制台

如果成功运行，可以看到如下结果
```shell script
Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.headius.backport9.modules.Modules (file:/usr/local/logstash-7.5.0/logstash-core/lib/jars/jruby-complete-9.2.8.0.jar) to field java.io.FileDescriptor.fd
WARNING: Please consider reporting this to the maintainers of com.headius.backport9.modules.Modules
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
Thread.exclusive is deprecated, use Thread::Mutex
Sending Logstash logs to /usr/local/logstash-7.5.0/logs which is now configured via log4j2.properties
[2020-01-15T11:42:36,556][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2020-01-15T11:42:36,651][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"7.5.0"}
[2020-01-15T11:42:37,608][INFO ][org.reflections.Reflections] Reflections took 29 ms to scan 1 urls, producing 20 keys and 40 values 
[2020-01-15T11:42:37,975][WARN ][org.logstash.instrument.metrics.gauge.LazyDelegatingGauge][main] A gauge metric of an unknown type (org.jruby.RubyArray) has been create for key: cluster_uuids. This may result in invalid serialization.  It is recommended to log an issue to the responsible developer/development team.
[2020-01-15T11:42:37,989][INFO ][logstash.javapipeline    ][main] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>2, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>250, "pipeline.sources"=>["config string"], :thread=>"#<Thread:0x626b1ea6 run>"}
[2020-01-15T11:42:38,025][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
The stdin plugin is now waiting for input:
[2020-01-15T11:42:38,064][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2020-01-15T11:42:38,214][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
```
运行成功后，我们可以直接在控制台输入`hello world`
```shell script
hello world
{
       "message" => "hello world",
          "host" => "exampleMBP.lan",
    "@timestamp" => 2020-01-15T03:49:44.853Z,
      "@version" => "1"
}
```
我们可以看到如上所示的输出, 使用`Ctrl+d`来退出Logstash.

- #### Filebeat

在下载的Filebeat目录下, 打开`filebeat.yml`
```shell script
cd filebeat-7.5.1-darwin-x86_64
sudo vim filebeat.yml
```

这是filebeat.yml的部分配置，大部分情况下使用默认配置即可.
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    # Filebeat处理文件的绝对路径
    - /path/to/file/logstash-tutorial.log
output.logstash:
  hosts: ["localhost": 5044]
```

保存配置, 并使用以下命令运行Filebeat
```shell script
sudo ./filebeat -e -c ./filebeat.yml -d "publish"
```

- #### 使用Logstash解析日志
Logstash默认已经包含了`Beat input`插件, 下面这个配置将会同时启用`beat`和`stdint`input插件
```shell script
beats {
    port => "5044"
    client_inactivity_timeout => 3000
}
stdin {}
```

更多`input`插件详见[官方文档](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)

下面的配置会将接收到的数据打印到标准输出
```shell script
stdout {}
```

更多`output`插件详见[官方文档](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)

完成上边两步之后, logstash.conf看起来应该是这样的
```shell script
input {
    beats {
        port => "5044"
        client_inactivity_timeout => 3000
    }
    stdin {}
}
output {
    stdout {}
}
```

检查配置文件语法是否正确
```shell script
bin/logstash -f logstash.conf --config.test_and_exit
```

如果通过了文件检查, 我们就可以执行下面这条命令指定配置文件来运行Logstash
```shell script
bin/logstash -f logstash.conf --config.reload.automatic
```
`--config.reload.automatic`可以在Logstash不重启的情况下自动加载配置文件

这时候, 我们就可以往Filebeat监控的文件中写入数据了
```shell script
echo "hello world" >> /path/to/file/logstash-tutorial.log
```

- #### 为Logstash增加过滤规则
在这里主要给大家介绍简单`grok`这个过滤插件, 更多内容查看[官方文档](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html).


