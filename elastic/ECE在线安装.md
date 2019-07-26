# Elastic Cloud Enterprise For Have Network

> 基础资源

```txt
1. 系统: CentOS Linux release 7.6.1810 (Core)
2. 内核: Linux vm172-31-0-41.ksc.com 3.10.0-957.5.1.el7.x86_64 #1 SMP Fri Feb 1 14:54:57 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
3. 基础硬件: 8c 16G 500G ssd
```

> 前提

```text
1. 挂载盘路径

mount /dev/vdb /mnt

2. 安装docker

    - sudo yum install -y yum-utils device-mapper-persistent-data lvm2

    - sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

    - sudo yum install docker-ce docker-ce-cli containerd.io

    - 修改docker默认配置存储指向大的磁盘

3. 安装脚本必须是docker组成员, 不能用root用户身份安装

    - 添加新用户: useradd elastic

    - 设置新用户密码: passwd elastic

    - 查找授权文件: whereis sudoers

    - 给sudoers添加可写权限: chmod -v u+w /etc/sudoers

    - 给新用户授权: vim /etc/sudoers  在root下一行添加 elastic  ALL=(ALL)  ALL

    - 收回可写权限: chmod -v u-w /etc/sudoers

    - 新增docker用户组: sudo groupadd docker

    - 登录新用户: su elastic

    - 将新用户添加到docker用户组: sudo gpasswd -a $USER docker

4. 设置系统参数

    - net.ipv4.ip_forward = 1

    - vm.max_map_count = 262145

    - net.ipv4.ip_local_port_range = 20001 65535

    warn:(应该根据建议进行, 但是出于条件没法都进行调整)

        - The installation with extfs can proceed; however, we recommend XFS

        - The installation with overlay2 can proceed; however, we recommend using overlay

        - OS setting 'cgroup.memory' should be set to cgroup.memory=nokmem

5. 升级centos内核到 4.4

    - yum --enablerepo=elrepo-kernel  list  |grep kernel*

    - yum --enablerepo=elrepo-kernel  install  kernel-lt kernel-lt-devel -y

```

> 安装

```shell
bash <(curl -fsSL https://download.elastic.co/cloud/elastic-cloud-enterprise.sh) install
```
