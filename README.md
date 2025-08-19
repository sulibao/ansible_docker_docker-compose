# prom_alert_node-exporter

This document is intended to illustrate the rapid deployment of Prometheus (via docker-compose) along with Alertmanager, Node-Exporter, SNMP-Exporter and Grafana.

## Pre-installation inspection items

### ./group_vars/all.yml

```bash
docker_data_dir: /app/docker_data
# This is the directory where the docker-compose file for exporting when multiple nodes are installed should be stored.
install_dir: /root/deploy
# Does this control have any nodes other than the monitoring system control nodes that need to be installed? If yes, fill in "true"; if no, fill in "false". When it is "false", the docker_nodes in the hosts file should also be left empty or commented out.
installExporters: false
# This is the password for the node server when installing docker_nodes.
ssh_pass: sulibao
```

### .env

```bash
registry_url="registry.cn-chengdu.aliyuncs.com/su03"   # image registry
prometheus_image="$registry_url/prometheus:2.46.0-debian-11-r5"  # proemtheus image version
alertmanager_image="$registry_url/alertmanager:0.25.0-debian-11-r171"  # alertmanager image version
grafana_image="$registry_url/grafana:9.3.6"   # grafana image version
grafana_user="xxx"     # grafana username
grafana_password="xxx"   # grafana password
pushgateway_image="$registry_url/pushgateway:v1.6.2"   # pushgateway image version
nodeexporter_image="$registry_url/node-exporter:1.6.1-debian-11-r8"   # node_exporter image version
monitor_host="192.168.2.193"   # The IP address of the host where the monitoring system is deployed
```

### hosts

```bash
[docker_main]   
# This field must be filled in. It represents the main control node for installing the monitoring system. When "installExporters" in setup.sh is set to true, this node will also install Ansible, and Docker, Docker Compose, and Node-Exporter will be installed for the nodes in docker_nodes.
192.168.2.190
[docker_nodes]   
# This indicates the nodes other than the main control nodes of the monitoring system. When this requirement is not met, please leave this field blank or add a comment. Additionally, in the ./group_vars/all.yml file, the installExporters field should be set to false.
;192.168.2.193

[docker_cluster:children]
docker_main
docker_nodes
```

### setup.sh

```bash
# This is the password for the node server when installing docker_nodes.
export ssh_pass="sulibao" 
.....
function main() {
  ......
  const installExporters = true;  
  # When there is only one node that needs to be monitored by node-exporter (that is, the IP of the docker_node in the hosts file is not defined), this option should be set to false.
  if (installExporters) {
    install_other_exporter();
  }
  ......
}
```

### alertmanager.yml

```yaml
global:
  resolve_timeout: 5m
  # The email server address requires specifying the connection port.
  smtp_smarthost: 'smtp.qq.com:465'  
  # email address
  smtp_from: 'xxx'     
  # Email authentication user address, usually the same as smtp_from      
  smtp_auth_username: 'xxx'   
  # Email authorization code  
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
  # The email address for receiving warning messages
  - to: 'xxx'               
```

### Network connectivity

1.This project mainly involves enabling online installation, relying on the material resources stored on aliyun, to ensure that the network can reach normally.

2.In the case where the public network cannot be connected, please download the resource package in a networked environment in advance and then transfer it to the server for installation. The resources required for the installation are as follows:

```bash
docker installation package：
X86: https://sulibao.oss-cn-chengdu.aliyuncs.com/docker/amd/docker-27.2.0.tgz
Arm: https://sulibao.oss-cn-chengdu.aliyuncs.com/docker/arm/docker-27.2.0.tgz

docker Installation package storage path (must be confirmed to be correct)：
X86: ./packages/docker/x86/docker-27.2.0.tgz
Arm: ./packages/docker/arm/docker-27.2.0.tgz

The images involved in the monitor:
Refer to the.env variable file
Can be packaged in a networked environment by using `docker/nerdctl pull image` afterwards
In the form of `docker/nerdctl save -o xxx.tar image1 image2 ......`
Transferred to an offline environment and imported using `docker/nerdctl load -i xxx.tar`
```

## Install & Use

```bash
bash setup.sh / bash -x setup.sh          
```

## Dashboard JSON file

```bash
The dashboard file configures a JSON dashboard based on node-exporter.
Node Exporter Full.json
```

## Access point

```bash
prometheus：http://IP:9090

grafana：http://IP:3000, Log in using the user and password defined in the `.env` file
```