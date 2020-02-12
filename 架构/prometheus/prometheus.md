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

##### 1.1.1.prometheus常用参数

> prometheus命令

- --config.file=prometheus.yml #配置文件路径
- --web.listen-address="0.0.0.0:9090" #监听的地址
- --storage.tsdb.path="data/" #本地存储数据的目录
- --storage.tsdb.retention.tim #数据存储多长时间，如果需要长时间保留，可以使用其他时序数据库

> promtool命令

- check config prometheus.yml #检查prometheus配置文件的格式
- check rules rules #检查rule文件

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

> 首先配置docker的加速地址

```bash
[root@node02 opt] cat > /etc/docker/daemon.json <<EOF
{
        "registry-mirrors": ["https://zmoboljg.mirror.aliyuncs.com","https://registry.docker-cn.com"]
}                                                                        
EOF                                                                                    
[root@node02 opt] systemctl daemon-reload
[root@node02 opt] systemctl restart docker
[root@node02 opt] systemctl enable docker
```

> 如有需要，可以创建自定义的prometheus.yml文件和数据存储目录 https://prometheus.io/docs/prometheus/latest/installation/

```bash
# 官方镜像似乎不能把本地data目录挂载到镜像中，可以基于其他镜像写一个dockerfile
docker run -p 9090:9090 -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml  prom/prometheus
```



### 2.配置文件（prometheus.yml）

> https://prometheus.io/docs/prometheus/latest/configuration/configuration/ 后续把每个部分配置细致化

```bash
global: #总局配置
rule_files：#触发报警的规则
scrape_configs：#配置监控目标
alerting：#触发报警后由alertmanager组件接管
remote_write remote_read：#远程存储和读取
```

#### 2.1.案例:node_exporter

##### 2.1.1.node_exporter

> prometheus的架构中，prometheus不是直接去获取监控目标的监控数据，而是周期性的从一个http接口获取监控的数据，然后存储，接着提供一个UI通过promQL查询监控数据。通常一个exporter暴露一个http接口，exporter可以是:
>
> 1.独立于监控目标之外的程序
>
> 2.集成到监控目标的程序
>
> node_exporter是用来监控节点的基础信息：cpu、网络、磁盘等信息。通过/metrics（默认）接口暴露监控信息。https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz

```bash
[root@node01 opt]# cd node_exporter/
[root@node01 node_exporter]# ll
total 16500
-rw-r--r--. 1 3434 3434    11357 Jun  5  2019 LICENSE
-rwxr-xr-x. 1 3434 3434 16878582 Jun  5  2019 node_exporter
-rw-r--r--. 1 3434 3434      463 Jun  5  2019 NOTICE
[root@node01 node_exporter]# cat /usr/lib/systemd/system/node_exporter.service 
[Unit]
Description=node_exporter

[Service]
Restart=on-failure
ExecStart=/opt/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
[root@node01 node_exporter]# 
```

node_exporter的目录只提供了一个node_exporter的二进制文件，只需执行该程序即可。

- --web.listen-address=":9100" # 暴露的端口
- --web.telemetry-path="/metrics" #暴露的接口地址

查看暴露的监控信息

![](image/node_exporter访问.png)

每一个监控指标类似于下面信息

> 第一行表示注释
>
> 第二行表示指标的类型，本次指标的类型为**gauge**
>
> 第三标表示指标的具体信息，指标名：go_goroutines，值为8

```bash
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 8
```

##### 2.2.2.和prometheus集成

```bash
[root@node01 prometheus]# vim prometheus.yml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'node_exporter'
    static_configs:
    - targets: ['192.168.111.111:9100','192.168.111.112:9100']
```

由于修改了配置文件，需要重新加载，systemctl reload prometheus

![](image/prometheus-target访问.png)

使用promQL简单查询

![](image/promQL查询.png)

### 3.promQL

### 4.报警

