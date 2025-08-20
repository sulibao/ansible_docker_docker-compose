# ansible+docker+docker-compose
This document is intended to illustrate the process of quickly deploying multiple Docker + Docker-Compose environments using Ansible containers.

## Before install

### Modify the variables file "./group_vars/all.yml"

```bash
vim group_vars/all.yml
# Docker's data directory
docker_data_dir: /app/docker_data
# The passwords of each host machine
ssh_pass: sulibao
```

### Modify ansible host list

```bash
[ansible]  
# This node runs an Ansible container, and in Ansible, it will deploy Docker and Docker-Compose for other nodes.
192.168.2.190           
[other_nodes]  
192.168.2.191 
192.168.2.192

[ansible_cluster:children]
ansible
other_nodes
```

## Install

```sh
bash setup.sh
```