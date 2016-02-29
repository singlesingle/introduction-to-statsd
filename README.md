# statsd-example
> StatsD 快速入门。 阅读时间 10 分钟

# StatsD 简介

自己搞一套监控系统可以从这三个部分分别着手：数据采集，数据存储和数据展示。

数据采集已经不是问题，例如自己动手统计一个方法的执行时间只需要记录方法进入和退出的时间差。
而对于后台架构中常见的各种组件，它们往往各自提供了数量可观的指标接口，可以快速的获取组件的运行状况。

数据存储部分需要选择一款针对时间序列的数据库。

数据展示也有一些开源方案例如 Grafana 。

**StatsD 的使用场景**

有的时候，没有必要把所有性能数据都存起来。典型的场景是 Web 服务器，当请求达到每分钟上万条时，没有必要把每条请求的响应时间都存入数据库。

这个时候可以自己实现一个缓存模块，来实现数据的累加，计算平均值，中间值等。
StatsD 正是这样一个简单实现数据中转的工具。
此外，由于往 StatsD 写入数据的时候使用的是 UDP 协议，所以对被检测的系统负载的影响可以控制的很小。

## 安装 Statsd 服务

安装指南

https://github.com/etsy/statsd#installation-and-configuration

另外一种方法是使用 CloudInsight 监控系统。

http://docs-ci.oneapm.com/quick-start/

# StatsD 协议

StatsD 的协议其实非常简单，一行就是一条数据。

```
<metric_name>:<metric_value>|<metric_type>
```

以监控系统负载为例，假如某一时刻系统 `1分钟的负载` 是 0.5，通过以下命令就可以写入 StatsD。

```sh
echo "system.load.1:0.5|g" | nc 127.0.0.1 8251
```

结合其中的 `system.load.1:0.5|g`，各个部分分别为：

- 指标名称 metric_name  = system.load.1
- 指标的值 metric_value = 0.5
- 指标类型 metric_type  = g(gauge)

**指标名称**

指标命名没有特别的限制，但是一般的约定是使用点号分割的命名空间。

**指标的值**

指标的值是大于等于 0 的浮点数。

**指标类型**

- [gauge](gauge-标量)
- [counter](#counter-计数器)
- [ms](#ms-时间)
- [set](#set-集合)

StatsD 封装了一些最常用的操作，例如周期内指标值的累加、统计执行时间等，因此对指标的值分成以下几种：

#### gauge 标量

标量是任何可以测量的一维变量。例如人的身高，体重、某一时刻北京市 PM2.5 的值等。

#### counter 计数器

如果某个变量没有办法一下子测量出来，这时就可以使用计数器，分步累加。

例如要统计某个接口的流量

```
ent0.traffic:100|c
```

在每个周期里 StatsD 会自动累加流量的值，并把平均值传给后端。

#### ms 时间

记录执行时间。请求响应时间是 12 ms，写入到 StatsD 服务器的方法是：

```
echo "request.time:12|ms" | nc 127.0.0.1 8251
```

StatsD 会自动计算以下指标：

```
request.time.avg
request.time.count
request.time.max
request.time.mean
...
```

#### set 集合

统计变量不重复的个数。例如：当前在线的用户

'echo "online.user:9587|s" | nc 127.0.0.1 8251'

'echo "online.user:9588|s" | nc 127.0.0.1 8251'


## 参考

https://github.com/etsy/statsd/blob/v0.7.2/docs/metric_types.md

https://github.com/etsy/statsd/wiki#client-implementations
