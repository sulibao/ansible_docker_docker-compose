# ansible+docker+docker-compose
本文档用于说明通过ansible容器快速部署多个docker+docker-compose环境。遵循此文档可完成以下服务的安装:

- 单台/多台docker+docker-compose环境

- 单台ansible容器

## 安装前

### 修改变量文件"./group_vars/all.yml"

```bash
vim group_vars/all.yml
# docker的数据目录
docker_data_dir: /app/docker_data
# 各台主机的密码
ssh_pass: sulibao
```

### 修改ansible主机清单

```bash
[ansible]  
# 该节点运行ansible容器，将在ansible里为其他节点部署docker+docker-compose 
192.168.2.190           
[other_nodes]  
192.168.2.191 
192.168.2.192

[ansible_cluster:children]
ansible
other_nodes
```

## 安装

```sh
bash setup.sh
```