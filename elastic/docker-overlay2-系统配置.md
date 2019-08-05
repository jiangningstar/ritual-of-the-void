# docker overlay2 系统配置

> docker配置

```txt
1. 格式化挂载盘为 xfs
    - mkfs.xfs -f /dev/vdb

2. docker的overlay2需要的是 pquota ，在 /etc/fstab 中设置
    - /dev/vdb /data xfs rw,pquota 0 0

3. 挂载数据盘
    - mount -o pquota,uqnoenforce /dev/vdb /data

4. 在/etc/docker/daemon.json下配置 每个容器可以使用的磁盘空间

    {
    "data-root": "/data/docker",
    "storage-driver": "overlay2",
    "storage-opts": [
      "overlay2.override_kernel_check=true",
      "overlay2.size=200G"
    ]
    }
```
