---
title: "CentOS8 安装部署Docker"
date: 2022-05-16T17:39:12+08:00
tags:
   - cloudnative
   - docker
   - centos
categories:
   - cloudnative
   - docker
   - centos
toc: true
---

`CentOS8`的`YUM`源默认不支持`docker`安装，默认是`redhat`的`Podman`容器，再次记录一下`CentOS8`通过`YUM`安装部署`docker`的步骤。

## 更新YUM源

国内用户推荐使用阿里云的`YUM`源

```shell
# 1. 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 2. 更新YUM源
## CentOS 8 (centos8官方源已下线，建议切换centos-vault源）
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo

## CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

## CentOS 6 （centos6官方源已下线，建议切换centos-vault源）
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-6.10.repo

# 3. 更新YUM缓存
yum clean all && yum makecache


```

## 安装步骤docker

```shell
# 1. 更新YUM
yum update 

## 更新依赖库
yum install -y yum-utils device-mapper-persistent-data lvm2

## 设置YUM源，选择国内阿里云，中央仓库（http://download.docker.com/linux/centos/docker-ce.repo）
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

## 查看YUM支持版本
yum list docker-ce --showduplicates | sort -r

# 2. 选择版本安装
yum install docker-ce-20.10.7

# 3. 启动Docker，然后加入开机启动
systemctl start docker
systemctl enable docker

# 4. docker-compose安装
curl -L https://get.daocloud.io/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m) > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

```
