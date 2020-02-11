## prometheus

**架构图**

<img src="image/prometheus架构图.png" style="zoom:80%;" />

### 1.部署

#### 1.1.二级制部署

```bash
[root@node02 opt]# wget https://github.com/prometheus/prometheus/releases/download/v2.16.0-rc.1/prometheus-2.16.0-rc.1.linux-amd64.tar.gz
[root@node02 opt]# tar -xf prometheus-2.16.0-rc.0.linux-amd64.tar.gz
[root@node02 opt]# mv prometheus-2.16.0-rc.0.linux-amd64 prometheus
```

##### 1.1.1.prometheus参数

- --config.file=prometheus.yml #配置文件路径
- --web.listen-address="0.0.0.0:9090" #监听的地址
- --storage.tsdb.path="data/" #本地存储数据的目录
- --storage.tsdb.retention.tim #数据存储多长时间，如果需要长时间保留，可以使用其他时序数据库

##### 1.1.2.编写prometheus.service文件

> 编写下面prometheus.service文件后可以使用systemctl start prometheus启动服务以及systemctl reload prometheus热加载prometheus的配置文件。

```bash
[root@node01 prometheus]# cat /usr/lib/systemd/system/prometheus.service 
[Unit]
Description=prometheus


[Service]
Restart=on-failure
ExecStart=/opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml
ExecReload=/bin/kill -s HUP $MAINPID

[Install]
WantedBy=multi-user.target
[root@node01 prometheus]# 
```



#### 1.2.docker部署

### 2.配置文件

### 3.promQL

