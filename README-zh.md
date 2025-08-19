# prom_alert_node-exporter

本文档用于说明通过docker-compose快速部署prometheus(email)+alertmanager+node-exporter+grafana。

## 安装前检查项

### setup.sh

```bash
# 此项为存储docker数据的目录，请根据实际需求进行更改
export docker_data="/data/docker_data"
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