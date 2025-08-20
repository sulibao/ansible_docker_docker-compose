# prom_alert_node-exporter

本文档用于说明通过docker-compose快速部署监控系统。遵循此文档可完成以下服务的安装:

- 控制节点prometheus+grafana+alertmanager+node-exporter+blackbox-exporter服务

- 单节点ansible容器

- 其他节点node-exporter

## 安装前检查项

### ./group_vars/all.yml

```bash
# 此项为存储docker数据的目录，请根据实际需求进行更改
docker_data_dir: /app/docker_data
# 此项为需要多节点安装exporter时的docker-compose文件存放目录
install_dir: /root/deploy
# 此项控制是否有除监控系统控制节点以外的节点要安装，有则填true，无则填false，为false的同时hosts文件中的docker_nodes也应该置空或注释
installExporters: false
# 此项为需要安装docker_nodes时的节点服务器密码
ssh_pass: sulibao
```

### .env

```bash
registry_url="registry.cn-chengdu.aliyuncs.com/su03"   # 镜像仓库
prometheus_image="$registry_url/prometheus:2.46.0-debian-11-r5"  # proemtheus镜像版本
alertmanager_image="$registry_url/alertmanager:0.25.0-debian-11-r171"  # alertmanager镜像版本
grafana_image="$registry_url/grafana:9.3.6"   # grafana镜像版本
grafana_user="xxx"     # grafana用户名
grafana_password="xxx"   # grafana用户密码
pushgateway_image="$registry_url/pushgateway:v1.6.2"   # pushgateway镜像版本
nodeexporter_image="$registry_url/node-exporter:1.6.1-debian-11-r8"   #node_exporter镜像版本
monitor_host="192.168.2.193"   # 部署监控系统的主机IP
```

### hosts

```bash
[docker_main]   
# 此项必填，表示安装监控系统的主要控制节点，在setup.sh中的installExporters为true时，此节点上还会安装ansible，并且为docker_nodes中的节点安装docker和docker-compose以及node-exporter
192.168.2.190  
[docker_nodes]   
# 此项表示除了监控系统的主要控制节点以外的其他节点，没有此需求时请将此项置空或注释，同时./group_vars/all.yml中的installExporters应填写false
;192.168.2.193

[docker_cluster:children]
docker_main
docker_nodes
```

### alertmanager.yml

```yaml
global:
  resolve_timeout: 5m
  # 邮件服务器地址，需要指定连接端口
  smtp_smarthost: 'smtp.qq.com:465'  
  # 邮箱地址 
  smtp_from: 'xxx'     
  # Email认证用户地址，通常与smtp_from相同           
  smtp_auth_username: 'xxx'   
  # 电子邮件授权码    
  smtp_auth_password: 'xxx'       
  smtp_require_tls: false
 
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'test-alertmanager'
 
receivers:
- name: 'test-alertmanager'
  email_configs:
  # 接收警告信息的电子邮件地址
  - to: 'xxx'               
```

### 网络连通性

1.此项目主要涉及为在线安装，依赖存储于aliyun上的物料资源，确保网络能够正常到达

2.无法联通公网的情况下，请提前在联网环境下下载好资源包再传入服务器安装，依赖下载的资源如下：

```bash
docker安装包：
X86: https://sulibao.oss-cn-chengdu.aliyuncs.com/docker/amd/docker-27.2.0.tgz
Arm: https://sulibao.oss-cn-chengdu.aliyuncs.com/docker/arm/docker-27.2.0.tgz

docker安装包存放路径(必须确认正确)：
X86: ./packages/docker/x86/docker-27.2.0.tgz
Arm: ./packages/docker/arm/docker-27.2.0.tgz

monitor涉及到的镜像：
见.env变量文件
可在联网环境下通过`docker/nerdctl pull image`过后
以`docker/nerdctl save -o xxx.tar image1 image2 ......`的形式打包
传输到离线环境中使用`docker/nerdctl load -i xxx.tar`进行导入
```

### Optional operation

#### prometheus.yml

```bash
......
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://xxxxxx
        - http://xxxxxx
      # 可以填写此处的网络地址也可以保持现状，blackbox将会为您监控这些地址是否健康
      # 后续要新添加时，填写完毕后请重启prometheus容器
```

#### blackbox-config.yml

```bash
modules:
  http_2xx:
    ......
      tls_config:
        insecure_skip_verify: false     
        # 此内容涉及在监测 https 网址时是否可以省略证书验证的问题。若为false，且未配置其他可信证书，则可能无法监测 https 链接及 SSL 证书。
        # 如果你涉及到要检测https网站，1.请修改此处为true后重启blackbox-exporter容器; 2.对blackbox-exporter镜像进行证书拷贝可构建或者手动拷贝证书进blackbox-exporter容器，此处不进行会说明。
      ......
```

## 安装和使用

```bash
bash setup.sh / bash -x setup.sh          
```

## 面板json文件

```bash
# 仪表板文件配置了一个基于node-exporter的json dashboard
Node Exporter Full.json
```

## 访问地址

```bash
prometheus：http://IP:9090

grafana：http://IP:3000, 使用`.env`文件中定义的用户和密码进行登录
```