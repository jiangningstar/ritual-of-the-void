# esrally 测试elasticsearch测试集群(持续更新测试用例, 以及测试场景)

> 集群配置

```txt
1. 两台centos7.6linux主机
2. 都是8c 16G
3. 一台500Gssd 一台200ssd

```

> 测试命令

```shell
esrally --offline --pipeline=benchmark-only --target-hosts=120.92.110.204:9200,120.92.110.93:9200 --track=http_logs
```

> 索引设置

```txt
1. 索引分片为5
2. 副本为0
```

> bluk设置

```txt
1. 批量每次请求10000条
2. 并发数20
```

> 写入时系统参数

cpu & mem
es1
![es1](/Users/jiangning/Downloads/1564654420587.jpg)
es2
![es2](/Users/jiangning/Downloads/1564654442620.jpg)

io

es1
![es1](/Users/jiangning/Downloads/1564654801000.jpg)
es2
![es2](/Users/jiangning/Downloads/1564654753479.jpg)

> 结果

```txt
1. 测试数据为31.1G的日志数据
2. 数据单条格式为{"@timestamp": 894225606, "clientip":"88.7.0.0", "request": "GET /english/images/nav_venue_off.gif HTTP/1.1", "status": 304, "size": 0}
3. 31.1G 数据写入时间为 35min
```

两台集群测试结果
![result](/Users/jiangning/Downloads/1564655560055.jpg)

测试结果显示, 并发读取的效率和写入的效率都相比单机来说都有所下降, 会继续调优测试配置参数

> kibana监测结果

indexing rate为写入每30秒的写入量的曲线
![kibana](/Users/jiangning/Downloads/1564656069681.jpg)