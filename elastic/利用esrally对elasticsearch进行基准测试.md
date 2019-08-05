# esrally工具安装以及入门使用(持续更新)

> 基础环境

```text
* esrally基于centos7.6(8c 16G)
* elaticsearch基于centos7.6(8c 16G)

基础环境依赖

1. python3.5+
2. git 1.9+
3. java JDK 8
```

> 安装python3.7

```shell
1. 利用yum安装python3.7相关的第三方包
    - yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel

2. 下载python3.7安装包
    - wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz

3. 解压
    - tar -zxvf Python-3.7.0.tgz

4. 编译
    - cd Python-3.7.0
    - ./configure prefix=/usr/local/python3 --with-ssl
    - make && make install

5. 添加软链
    - ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3
    - ln -s /usr/local/python3/bin/pip3.7 /usr/bin/pip3

6. 测试是否安装成功
    - python3 -V
    - pip3 -V
```

> 更新git版本

```shell
1. 卸载系统自带git
    - yum remove git

2. 下载git安装包
    - wget https://www.kernel.org/pub/software/scm/git/git-2.15.1.tar.xz

3. 解压
    - tar -vxf git-2.15.1.tar.xz

4. 编译
    - cd git-2.15.1
    - make prefix=/usr/local/git all
    - make prefix=/usr/local/git install

5. 配置环境变量
    - echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
    - source /etc/profile

6. 确认git版本是否更新
    - git --version
```

> 安装esrally(本地安装)

```shell
1. pip安装
    - pip3 install esrally

2. 离线安装
    - 下载离线安装包
        wget https://github.com/elastic/rally/releases/download/1.2.1/esrally-dist-1.2.1.tar.gz

    - 解压
        tar -xzf esrally-dist-1.2.1.tar.gz

    - 安装
        sudo ./esrally-dist-1。2.1/install.sh

3. 测试安装完成
    esrally --version
```

> docker安装esrally

```shell
1. 下载docker镜像
    - docker pull elastic/rally

2. 将docker容器当做esrally工具使用即可
    - docker run elastic/rally list tracks

3. 可以挂载配置
    - docker run -v /home/<myuser>/custom_rally.ini:/rally/.rally/rally.ini elastic/rally list tracks
```

> 配置你的esrally

```text
先决条件:
配置esrally需要配置他的存储方式，一种是内存方式, 一种会存储在你配置的es当中
* 推荐配置存储在es当中, 因为测试完成, esrally会将测试多项重要结果存储在es当中方便分析, 在kibana上可以进行可视化, 能够更好的分析我们的压力测试的结果

1. 简单配置
    - esrally configure

2. 高级配置
    - esrally configure --advanced-config

3. 工具目录
    - /root/.rally

4. 目录结构
    |── ├── benchmarks(测试所需的数据, 以及测试用例模板的定义的存放文件夹)
    │   └── tracks
    │       └── default
    ├── logging.json(log配置文件)
    ├── logs
    │   └── rally.log(log 文件)
    └── rally.ini(esrally的配置文件)

3. 配置选项

    [meta]
    config.version = 17

    [system]
    env.name = local

    [node]
    root.dir = /data/.rally/benchmarks(测试数据, 测试用例存放路径)
    src.root.dir = /data/.rally/benchmarks/src

    [source]
    remote.repo.url = https://github.com/elastic/elasticsearch.git
    elasticsearch.src.subdir = elasticsearch

    [benchmarks]
    local.dataset.cache = /data/.rally/benchmarks/data

    [reporting] (存放esrally生成测试结果的es配置)
    datastore.type = elasticsearch
    datastore.host = 120.92.110.204
    datastore.port = 9200
    datastore.secure = False
    datastore.user =
    datastore.password =

    [tracks]
    default.url = https://github.com/elastic/rally-tracks

    [teams]
    default.url = https://github.com/elastic/rally-teams

    [defaults]
    preserve_benchmark_candidate = False

    [distributions]
    release.cache = true

***
在有网的情况下, 运行esrally list tracks, esrally会将对应的tracks下载到您设置的root.dir路径下的benchmarks下的tracks

上面是基础git的配置, 所以会依赖于git, 如果您在无网的情况下, 需要将官方对应的tracks下载到您刚刚设置的root.dir下, 相对数据也应该下载下来放到root.dir下

benchmarks的目录结构

├── data
│   ├── geopoint
│   └── http_logs
├── races
│   ├── 2019-07-23-09-11-31
│   ├── 2019-07-23-09-21-41
│   ├── 2019-07-23-09-28-49
│   └── 2019-07-23-09-52-46
├── teams
│   └── default
│       ├── cars
│       └── plugins
└── tracks
    └── default
        ├── eventdata
        ├── geonames
        ├── geopoint
        ├── geopointshape
        ├── geoshape
        ├── http_logs
        ├── metricbeat
        ├── nested
        ├── noaa
        ├── nyc_taxis
        ├── percolator
        ├── pmc
        └── so

data: 是相对于tracks当中同名测试用例准备的测试数据
tracks: 测试用例定义存放的地方
races: 当你每次跑一次测试, 测试结果都会在这生成
teams: 测试工具自带的对于测试自行编译es的配置
***

```

> 运行第一个压力测试

```text
第一次压力测试:
    esrally --pipeline=benchmark-only --target-hosts=xxx.xxx.xxx.xxx:9200 --track=http_logs --test-mode

针对以上的命令介绍下基本参数(参数比较多, 我只介绍我常用的)

1. pipelines
    - 运行esrally list pipelines将得到以下结果

        ____        ____
   / __ \____ _/ / /_  __
  / /_/ / __ `/ / / / / /
 / _, _/ /_/ / / / /_/ /
/_/ |_|\__,_/_/_/\__, /
                /____/

Available pipelines:

Name                     Description
-----------------------  ---------------------------------------------------------------------------------------------
from-sources-complete    Builds and provisions Elasticsearch, runs a benchmark and reports results.
from-sources-skip-build  Provisions Elasticsearch (skips the build), runs a benchmark and reports results.
from-distribution        Downloads an Elasticsearch distribution, provisions it, runs a benchmark and reports results.
benchmark-only           Assumes an already running Elasticsearch instance, runs a benchmark and reports results

-------------------------------
[INFO] SUCCESS (took 1 seconds)
-------------------------------

from-sources-complete: 从源码进行编译es, 运行测试用例报告结果。
from-sources-skip-build: 已经编译好的跳过编译, 运行测试用例报告结果。
from-distribution: 从官网下载es可执行文件进行测试, 运行测试用例报告结果。
benchmark-only: 测试自己已经部署搭建好的外部es, 运行测试用例报告结果。

我的目的是测试自己本地的单节点ES, 所以一般选择benchmark-only管道, 大家可以根据自己的目的来选择管道

2. track
    - 运行esrally list tracks将得到以下结果

    ____        ____
   / __ \____ _/ / /_  __
  / /_/ / __ `/ / / / / /
 / _, _/ /_/ / / / /_/ /
/_/ |_|\__,_/_/_/\__, /
                /____/

Available tracks:

Name           Description                                                                                                                                                                        Documents    Compressed Size    Uncompressed Size    Default Challenge        All Challenges
-------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  -----------  -----------------  -------------------  -----------------------  ---------------------------------------------------------------------------------------------------------------------------
metricbeat     Metricbeat data                                                                                                                                                                    1,079,600    87.6 MB            1.2 GB               append-no-conflicts      append-no-conflicts
geopoint       Point coordinates from PlanetOSM                                                                                                                                                   60,844,404   481.9 MB           2.3 GB               append-no-conflicts      append-no-conflicts,append-no-conflicts-index-only,append-fast-with-conflicts
geonames       POIs from Geonames                                                                                                                                                                 11,396,505   252.4 MB           3.3 GB               append-no-conflicts      append-no-conflicts,append-no-conflicts-index-only,append-sorted-no-conflicts,append-fast-with-conflicts
geoshape       Shapes from PlanetOSM                                                                                                                                                              60,523,283   13.4 GB            45.4 GB              append-no-conflicts      append-no-conflicts
eventdata      This benchmark indexes HTTP access logs generated based sample logs from the elastic.co website using the generator available in https://github.com/elastic/rally-eventdata-track  20,000,000   755.1 MB           15.3 GB              append-no-conflicts      append-no-conflicts
noaa           Global daily weather measurements from NOAA                                                                                                                                        33,659,481   947.3 MB           9.0 GB               append-no-conflicts      append-no-conflicts,append-no-conflicts-index-only
geopointshape  Point coordinates from PlanetOSM indexed as geoshapes                                                                                                                              60,844,404   470.5 MB           2.6 GB               append-no-conflicts      append-no-conflicts,append-no-conflicts-index-only,append-fast-with-conflicts
nested         StackOverflow Q&A stored as nested docs                                                                                                                                            11,203,029   663.1 MB           3.4 GB               nested-search-challenge  nested-search-challenge,index-only
http_logs      HTTP server log data                                                                                                                                                               247,249,096  1.2 GB             31.1 GB              append-no-conflicts      append-no-conflicts,append-no-conflicts-index-only,append-sorted-no-conflicts,append-index-only-with-ingest-pipeline,update
percolator     Percolator benchmark based on AOL queries                                                                                                                                          2,000,000    102.7 kB           104.9 MB             append-no-conflicts      append-no-conflicts
nyc_taxis      Taxi rides in New York in 2015                                                                                                                                                     165,346,692  4.5 GB             74.3 GB              append-no-conflicts      append-no-conflicts,append-no-conflicts-index-only,append-sorted-no-conflicts-index-only,update,append-ml
pmc            Full text benchmark with academic papers from PMC                                                                                                                                  574,199      5.5 GB             21.7 GB              append-no-conflicts      append-no-conflicts,append-no-conflicts-index-only,append-sorted-no-conflicts,append-fast-with-conflicts
so             Indexing benchmark using up to questions and answers from StackOverflow                                                                                                            36,062,278   8.9 GB             33.1 GB              append-no-conflicts      append-no-conflicts

-------------------------------
[INFO] SUCCESS (took 3 seconds)
-------------------------------

上面的列表是esrally官方给出的测试用例以及对应的数据, 每一个track下面都有自己的测试用例列表, 下面我拿其中的http_logs进行举例.

例: http_logs

http_logs测试用例的目录结构

├── challenges
│   └── default.json
├── files.txt
├── index.json
├── operations
│   └── default.json
├── README.md
├── _tools
│   └── unparse.rb
└── track.json

- challenges: 这个文件夹下的default.json文件中定义了, 对于http_logs这些数据的测试用例

    下面是我截取的一部分测试用例定义:
    {
        "name": "append-no-conflicts",
        "description": "Indexes the whole document corpus using Elasticsearch default settings. We only adjust the number of replicas as we benchmark a single node cluster and Rally will only start the benchmark if the cluster turns green. Document ids are unique so all index operations are append only. After that a couple of queries are run.",
        "default": true,
        "schedule": [
            {
            "operation": "delete-index"
            },
            {
            "operation": {
                "operation-type": "create-index",
                "settings": {{index_settings | default({}) | tojson}}
            }
            },
            {
            "name": "check-cluster-health",
            "operation": {
                "operation-type": "cluster-health",
                "index": "logs-*",
                "request-params": {
                "wait_for_status": "{{cluster_health | default('green')}}",
                "wait_for_no_relocating_shards": "true"
                }
            }
            },
            {
            "operation": "index-append",
            "warmup-time-period": 240,
            "clients": {{bulk_indexing_clients | default(8)}}
            }
    }

    * name: 测试用例名字
    * description: 测试用例描述
    * schedule: 这个测试用例的执行流程列表, 当中的operation是对应operations文件下定义的方法. 具体的方法用途不解释了, 从名字上应该能看出来.


- files.txt: 是您对应数据的文件名
- index.json: 定义了es当中数据的mapping格式
- operations: 定义了challenges当中测试用例每个测试动作的方法
    {
      "name": "index-append",
      "operation-type": "bulk",
      "bulk-size": {{bulk_size | default(5000)}},
      "ingest-percentage": {{ingest_percentage | default(100)}},
      "corpora": "http_logs"
    }

    * name: 方法名
    * operation-type: 方法的类型(bulk应该对应的是bulk接口)
    * bulk-size: 传入的大小
    * ingest-percentage: 传入百分比
    * corpora: 对应的测试入口名称

- track.json: 这个文件是您对你数据的在es当中的索引配置, 以及确定您测试数据的描述, 让esrally确定数据位置

- README.md: 每一个track下的README都会描述他的测试目的

3. --target-hosts
    指定外部需要测试的es地址

4. --test-mode
    因为压力测试会进行大概一个小时多的时间, 如果您只为了测试一下效果, 加上这个参数, esrally只会截取1000条数据来返回结果
```

> 测试结果分析

```text
1. 在运行第一次压力测试之后, 我们在终端上能得到一个esrally提供的metrics列表
    esrally --pipeline=benchmark-only --target-hosts=xxx.xxx.xxx.xxx:9200 --track=http_logs --test-mode

    结果如下:

        ____        ____
   / __ \____ _/ / /_  __
  / /_/ / __ `/ / / / / /
 / _, _/ /_/ / / / /_/ /
/_/ |_|\__,_/_/_/\__, /
                /____/


************************************************************************
************** WARNING: A dark dungeon lies ahead of you  **************
************************************************************************

Rally does not have control over the configuration of the benchmarked
Elasticsearch cluster.

Be aware that results may be misleading due to problems with the setup.
Rally is also not able to gather lots of metrics at all (like CPU usage
of the benchmarked cluster) or may even produce misleading metrics (like
the index size).

************************************************************************
****** Use this pipeline only if you are aware of the tradeoffs.  ******
*************************** Watch your step! ***************************
************************************************************************

[INFO] Racing on track [http_logs], challenge [append-no-conflicts] and car ['external'] with version [7.2.0].

[WARNING] merges_total_time is 4664 ms indicating that the cluster is not in a defined clean state. Recorded index time metrics may be misleading.
[WARNING] indexing_total_time is 89611 ms indicating that the cluster is not in a defined clean state. Recorded index time metrics may be misleading.
[WARNING] refresh_total_time is 38976 ms indicating that the cluster is not in a defined clean state. Recorded index time metrics may be misleading.
[WARNING] flush_total_time is 7096 ms indicating that the cluster is not in a defined clean state. Recorded index time metrics may be misleading.
Running delete-index                                                           [100% done]
Running create-index                                                           [100% done]
Running check-cluster-health                                                   [100% done]
Running index-append                                                           [100% done]
Running refresh-after-index                                                    [100% done]
Running force-merge                                                            [100% done]
Running refresh-after-force-merge                                              [100% done]
Running default                                                                [100% done]
Running term                                                                   [100% done]
Running range                                                                  [100% done]
Running hourly_agg                                                             [100% done]
Running scroll                                                                 [100% done]
Running desc_sort_timestamp                                                    [100% done]
Running asc_sort_timestamp                                                     [100% done]

------------------------------------------------------
    _______             __   _____
   / ____(_)___  ____ _/ /  / ___/_________  ________
  / /_  / / __ \/ __ `/ /   \__ \/ ___/ __ \/ ___/ _ \
 / __/ / / / / / /_/ / /   ___/ / /__/ /_/ / /  /  __/
/_/   /_/_/ /_/\__,_/_/   /____/\___/\____/_/   \___/
------------------------------------------------------
            
|   Lap |                                                         Metric |                Task |       Value |    Unit |
|------:|---------------------------------------------------------------:|--------------------:|------------:|--------:|
|   All |                     Cumulative indexing time of primary shards |                     |      1.4842 |     min |
|   All |             Min cumulative indexing time across primary shards |                     | 0.000283333 |     min |
|   All |          Median cumulative indexing time across primary shards |                     | 0.000666667 |     min |
|   All |             Max cumulative indexing time across primary shards |                     |    0.827783 |     min |
|   All |            Cumulative indexing throttle time of primary shards |                     |           0 |     min |
|   All |    Min cumulative indexing throttle time across primary shards |                     |           0 |     min |
|   All | Median cumulative indexing throttle time across primary shards |                     |           0 |     min |
|   All |    Max cumulative indexing throttle time across primary shards |                     |           0 |     min |
|   All |                        Cumulative merge time of primary shards |                     |     0.09015 |     min |
|   All |                       Cumulative merge count of primary shards |                     |          59 |         |
|   All |                Min cumulative merge time across primary shards |                     |           0 |     min |
|   All |             Median cumulative merge time across primary shards |                     |           0 |     min |
|   All |                Max cumulative merge time across primary shards |                     |   0.0809167 |     min |
|   All |               Cumulative merge throttle time of primary shards |                     |           0 |     min |
|   All |       Min cumulative merge throttle time across primary shards |                     |           0 |     min |
|   All |    Median cumulative merge throttle time across primary shards |                     |           0 |     min |
|   All |       Max cumulative merge throttle time across primary shards |                     |           0 |     min |
|   All |                      Cumulative refresh time of primary shards |                     |    0.654517 |     min |
|   All |                     Cumulative refresh count of primary shards |                     |        1102 |         |
|   All |              Min cumulative refresh time across primary shards |                     |     0.00015 |     min |
|   All |           Median cumulative refresh time across primary shards |                     | 0.000433333 |     min |
|   All |              Max cumulative refresh time across primary shards |                     |       0.461 |     min |
|   All |                        Cumulative flush time of primary shards |                     |    0.118217 |     min |
|   All |                       Cumulative flush count of primary shards |                     |          29 |         |
|   All |                Min cumulative flush time across primary shards |                     |           0 |     min |
|   All |             Median cumulative flush time across primary shards |                     |           0 |     min |
|   All |                Max cumulative flush time across primary shards |                     |    0.109867 |     min |
|   All |                                             Total Young Gen GC |                     |       0.075 |       s |
|   All |                                               Total Old Gen GC |                     |           0 |       s |
|   All |                                                     Store size |                     |    0.130022 |      GB |
|   All |                                                  Translog size |                     |    0.311347 |      GB |
|   All |                                         Heap used for segments |                     |    0.485718 |      MB |
|   All |                                       Heap used for doc values |                     |    0.142853 |      MB |
|   All |                                            Heap used for terms |                     |    0.240282 |      MB |
|   All |                                            Heap used for norms |                     |  0.00402832 |      MB |
|   All |                                           Heap used for points |                     |   0.0519161 |      MB |
|   All |                                    Heap used for stored fields |                     |   0.0466385 |      MB |
|   All |                                                  Segment count |                     |          96 |         |
|   All |                                                 Min Throughput |        index-append |     18375.7 |  docs/s |
|   All |                                              Median Throughput |        index-append |     18375.7 |  docs/s |
|   All |                                                 Max Throughput |        index-append |     18375.7 |  docs/s |
|   All |                                        50th percentile latency |        index-append |     34.1464 |      ms |
|   All |                                        90th percentile latency |        index-append |     57.7246 |      ms |
|   All |                                       100th percentile latency |        index-append |     82.2463 |      ms |
|   All |                                   50th percentile service time |        index-append |     34.1464 |      ms |
|   All |                                   90th percentile service time |        index-append |     57.7246 |      ms |
|   All |                                  100th percentile service time |        index-append |     82.2463 |      ms |
|   All |                                                     error rate |        index-append |           0 |       % |
|   All |                                                 Min Throughput |             default |      101.29 |   ops/s |
|   All |                                              Median Throughput |             default |      101.29 |   ops/s |
|   All |                                                 Max Throughput |             default |      101.29 |   ops/s |
|   All |                                       100th percentile latency |             default |     11.9992 |      ms |
|   All |                                  100th percentile service time |             default |     11.9992 |      ms |
|   All |                                                     error rate |             default |           0 |       % |
|   All |                                                 Min Throughput |                term |      103.03 |   ops/s |
|   All |                                              Median Throughput |                term |      103.03 |   ops/s |
|   All |                                                 Max Throughput |                term |      103.03 |   ops/s |
|   All |                                       100th percentile latency |                term |     11.4583 |      ms |
|   All |                                  100th percentile service time |                term |     11.4583 |      ms |
|   All |                                                     error rate |                term |           0 |       % |
|   All |                                                 Min Throughput |               range |      149.81 |   ops/s |
|   All |                                              Median Throughput |               range |      149.81 |   ops/s |
|   All |                                                 Max Throughput |               range |      149.81 |   ops/s |
|   All |                                       100th percentile latency |               range |     8.53396 |      ms |
|   All |                                  100th percentile service time |               range |     8.53396 |      ms |
|   All |                                                     error rate |               range |           0 |       % |
|   All |                                                 Min Throughput |          hourly_agg |       55.26 |   ops/s |
|   All |                                              Median Throughput |          hourly_agg |       55.26 |   ops/s |
|   All |                                                 Max Throughput |          hourly_agg |       55.26 |   ops/s |
|   All |                                       100th percentile latency |          hourly_agg |     18.0846 |      ms |
|   All |                                  100th percentile service time |          hourly_agg |     18.0846 |      ms |
|   All |                                                     error rate |          hourly_agg |           0 |       % |
|   All |                                                 Min Throughput |              scroll |       48.49 | pages/s |
|   All |                                              Median Throughput |              scroll |       48.49 | pages/s |
|   All |                                                 Max Throughput |              scroll |       48.49 | pages/s |
|   All |                                       100th percentile latency |              scroll |      116.48 |      ms |
|   All |                                  100th percentile service time |              scroll |      116.48 |      ms |
|   All |                                                     error rate |              scroll |           0 |       % |
|   All |                                                 Min Throughput | desc_sort_timestamp |       89.43 |   ops/s |
|   All |                                              Median Throughput | desc_sort_timestamp |       89.43 |   ops/s |
|   All |                                                 Max Throughput | desc_sort_timestamp |       89.43 |   ops/s |
|   All |                                       100th percentile latency | desc_sort_timestamp |     9.95675 |      ms |
|   All |                                  100th percentile service time | desc_sort_timestamp |     9.95675 |      ms |
|   All |                                                     error rate | desc_sort_timestamp |           0 |       % |
|   All |                                                 Min Throughput |  asc_sort_timestamp |      277.62 |   ops/s |
|   All |                                              Median Throughput |  asc_sort_timestamp |      277.62 |   ops/s |
|   All |                                                 Max Throughput |  asc_sort_timestamp |      277.62 |   ops/s |
|   All |                                       100th percentile latency |  asc_sort_timestamp |     3.22652 |      ms |
|   All |                                  100th percentile service time |  asc_sort_timestamp |     3.22652 |      ms |
|   All |                                                     error rate |  asc_sort_timestamp |           0 |       % |


--------------------------------
[INFO] SUCCESS (took 19 seconds)
--------------------------------

1. primary shards  一些主分片的指标
2. Throughput 关于es的一些吞吐量的关键指标
3. latency 查询的延迟时间(一个请求到最终返回)
4. service time  服务的延迟时间
5. Segment count 分片之后的分段数
6. Error rate 错误率

* 这些指标的深度理解还在研究

```

> 利用kibana进行数据结果分析

** 熟悉后面补上
