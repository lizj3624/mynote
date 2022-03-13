---
title: "APISIX快速入门"
date: 2022-03-12T20:26:56+08:00
tags:
   - apisix 
   - nginx 
categories:
   - apisix 
   - nginx 
toc: true
---

> 前面介绍过[APISIX](https://lizj3624.github.io/post/apisix-primer/)，github开源社区比较活跃的云原生API网关，
> 自己根据官方的[快速入门指南](https://apisix.apache.org/zh/docs/apisix/getting-started)学习使用APISIX。

## 安装部署
### 安装部署APISIX
APISIX支持`rpm、docker image、helm chart、source release package`安装部署，我开始开始用源码安装但是一直没有
成功，为了先开始学习，用`rpm`包安装的，以后再慢慢研究源码安装过程。再此提醒一下APISIX暂时不支持`CentOS8`，这个我
在社区提交[issue](https://github.com/apache/apisix/issues/6462)，后续可能会改进。
在此我简单写一下步骤，详细安装步骤可以参考[官方步骤](https://github.com/apache/apisix/blob/master/docs/en/latest/how-to-build.md#installation-via-rpm-repository-centos-7)
```shell
## apisix yum源, 会安装依赖的OpenResty(apisix-base-1.19.9.1.3-0.el7.x86_64)
sudo yum-config-manager --add-repo https://repos.apiseven.com/packages/centos/apache-apisix.repo
sudo yum info -y apisix
sudo yum --showduplicates list apisix
sudo yum install apisix
```

### 安装部署etcd
apisix的数据存储在etcd中，启动apisix前需要安装etcd并启动。目前最新版本1.12版本只支持etcdv3版本。
```shell
## 安装
ETCD_VERSION='3.4.13'
wget https://github.com/etcd-io/etcd/releases/download/v${ETCD_VERSION}/etcd-v${ETCD_VERSION}-linux-amd64.tar.gz
tar -xvf etcd-v${ETCD_VERSION}-linux-amd64.tar.gz && \
  cd etcd-v${ETCD_VERSION}-linux-amd64 && \
  sudo cp -a etcd etcdctl /usr/bin/

## 启动, etcd要支持数据持久化
ETCD_HOME=`pwd`
sudo nohup etcd --data-dir $ETCD_HOME/data.etcd --wal-dir $ETCD_HOME/wal --snapshot-count 100000 > $ETCD_HOME/etcd.log 2>&1 &
```

### 启动apisix
```shell
# initialize NGINX config file and etcd
apisix init

# generate `nginx.conf` from `config.yaml` and test it
apisix test

# start Apache APISIX server
apisix start
```

## upstream的增删改查
详细可以查看upstream [admin API](https://apisix.apache.org/zh/docs/apisix/admin-api#upstream)
### 增加upstream
```shell
curl "http://127.0.0.1:9080/apisix/admin/upstreams" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
  "type": "roundrobin",
  "nodes": {
    "127.0.0.1:8080": 1,
    "127.0.0.1:8880": 1,
    "127.0.0.1:8881": 1
  },
  "retries": 2,
  "checks": {
    "active": {
      "timeout": 5,
      "http_path": "/healthcheck",
      "host": "test.my.com",
      "healthy": {
        "interval": 2,
        "successes": 1
      },
      "unhealthy": {
        "interval": 1,
        "http_failures": 2
      },
      "req_headers": ["User-Agent: curl/7.29.0"]
    },
    "passive": {
      "healthy": {
        "http_statuses": [200, 201, 302, 304],
        "successes": 3
      },
      "unhealthy": {
        "http_statuses": [500, 503, 504],
        "http_failures": 3,
        "tcp_failures": 3
      }
    }
  }
}'
```

添加成功后，返回数据中包含upstream_id(00000000000000000122)，以后可以通过这个id去关联`route`或`service`
```json
{
    "action":"create",
    "node":{
        "key":"\/apisix\/upstreams\/00000000000000000122",
        "value":{
            "retries":2,
            "nodes":{
                "127.0.0.1:8881":1,
                "127.0.0.1:8080":1,
                "127.0.0.1:8880":1
            },
            "hash_on":"vars",
            "id":"00000000000000000122",
            "scheme":"http",
            "checks":{
                "passive":{
                    "healthy":{
                        "http_statuses":[
                            200,
                            201,
                            302,
                            304
                        ],
                        "successes":3
                    },
                    "unhealthy":{
                        "http_statuses":[
                            500,
                            503,
                            504
                        ],
                        "tcp_failures":3,
                        "timeouts":7,
                        "http_failures":3
                    },
                    "type":"http"
                },
                "active":{
                    "healthy":{
                        "http_statuses":[
                            200,
                            302
                        ],
                        "interval":2,
                        "successes":1
                    },
                    "timeout":5,
                    "req_headers":[
                        "User-Agent: curl\/7.29.0"
                    ],
                    "unhealthy":{
                        "http_failures":2,
                        "tcp_failures":2,
                        "timeouts":3,
                        "interval":1,
                        "http_statuses":[
                            429,
                            404,
                            500,
                            501,
                            502,
                            503,
                            504,
                            505
                        ]
                    },
                    "host":"test.my.com",
                    "concurrency":10,
                    "http_path":"\/healthcheck",
                    "https_verify_certificate":true,
                    "type":"http"
                }
            },
            "pass_host":"pass",
            "update_time":1647059500,
            "type":"roundrobin",
            "create_time":1647059500
        }
    }
}
```
> 这个upstream还支持主动/被动健康检查，apisix健康检查只有有请求转发到这个upstream时才启动。
> 有admin API请求时，必须每次都得加token('X-API-KEY: edd1c9f034335f136f87ad84b625c8f1')，不然API校验不通过。

### 修改upstream
```shell
# 修改时，可以通过PUT方法
curl "http://127.0.0.1:9080/apisix/admin/upstreams/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "type": "roundrobin",
  "nodes": {
    "httpbin.org:80": 1
  }
}'
```
### 查询
```shell
curl http://127.0.0.1:9080/apisix/admin/upstreams/00000000000000000122  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
```

返回数据
```json
{
    "node":{
        "key":"\/apisix\/upstreams\/00000000000000000122",
        "value":{
            "retries":2,
            "nodes":{
                "127.0.0.1:8881":1,
                "127.0.0.1:8080":1,
                "127.0.0.1:8880":1
            },
            "hash_on":"vars",
            "create_time":1647059500,
            "type":"roundrobin",
            "pass_host":"pass",
            "checks":{
                "passive":{
                    "healthy":{
                        "http_statuses":[
                            200,
                            201,
                            302,
                            304
                        ],
                        "successes":3
                    },
                    "unhealthy":{
                        "http_statuses":[
                            500,
                            503,
                            504
                        ],
                        "tcp_failures":3,
                        "timeouts":7,
                        "http_failures":3
                    },
                    "type":"http"
                },
                "active":{
                    "healthy":{
                        "http_statuses":[
                            200,
                            302
                        ],
                        "successes":1,
                        "interval":2
                    },
                    "timeout":5,
                    "req_headers":[
                        "User-Agent: curl\/7.29.0"
                    ],
                    "unhealthy":{
                        "http_failures":2,
                        "tcp_failures":2,
                        "timeouts":3,
                        "interval":1,
                        "http_statuses":[
                            429,
                            404,
                            500,
                            501,
                            502,
                            503,
                            504,
                            505
                        ]
                    },
                    "host":"test.my.com",
                    "concurrency":10,
                    "http_path":"\/healthcheck",
                    "https_verify_certificate":true,
                    "type":"http"
                }
            },
            "update_time":1647059500,
            "scheme":"http",
            "id":"00000000000000000122"
        }
    },
    "action":"get",
    "count":1
}
```

## route的增删改查
route [admin API](https://apisix.apache.org/zh/docs/apisix/admin-api#route)
### 新增route
```shell
curl "http://127.0.0.1:9080/apisix/admin/routes" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
    "methods": ["GET"],
    "host": "test.my.com",
    "uri": "/",
    "upstream_id": "00000000000000000122"
}'
```

返回数据中也是包含route_id，00000000000000000212
```json
{
    "action":"create",
    "node":{
        "key":"\/apisix\/routes\/00000000000000000212",
        "value":{
            "methods":[
                "GET"
            ],
            "upstream_id":"00000000000000000122",
            "uri":"\/",
            "host":"test.my.com",
            "id":"00000000000000000212",
            "status":1,
            "update_time":1647064681,
            "priority":0,
            "create_time":1647064681
        }
    }
}
```
> route的规则是`GET`方法、`host`，`uri`

> upstream_id就是上面创建upstream的id，这样命中这个route的请求就会转发到这个upstream上，route也是直接添加upstream，
> 但是route跟upstream是1:N得关系，为了重复添加重复的upstream，推荐用upstream_id的方式与route绑定；route也是支持service的，
> 如果service和(upstream或upstream_id)并存时，upstream优先。

### 修改route
```shell
curl "http://127.0.0.1:9080/apisix/admin/routes/00000000000000000212" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '{
    "methods": ["GET"],
    "host": "test.my.com",
    "uri": "/mypath",
    "upstream_id": "00000000000000000122"
}'
```

### 删除route
```shell
curl http://127.0.0.1:9080/apisix/admin/routes/00000000000000000212 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X DELETE
```

## 测试验证
```shell
curl http://127.0.0.1:9080/ -H "Host: test.my.com"
```

## Service增删改查
`Service`是一组`upstream`和`plugin`的组合，`plugin`可选。

### 创建Service
```shell
curl http://127.0.0.1:9080/apisix/admin/services  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X POST -i -d  '{"upstream_id": "00000000000000000122"}'

## 返回数据
{
    "action":"create",
    "node":{
        "key":"\/apisix\/services\/00000000000000000292",
        "value":{
            "id":"00000000000000000292",
            "update_time":1647069365,
            "upstream_id":"00000000000000000122",
            "create_time":1647069365
        }
    }
}
```
> upstream_id就是前面，创建的upstream，这个Service没有关联plugin

### Service绑定Route上
```shell
curl "http://127.0.0.1:9080/apisix/admin/routes" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '
{
  "methods": ["GET"],
  "host": "test.my.com",
  "uri": "/mypath01/*",
  "service_id": "00000000000000000292"
}'

## 返回数据
{
    "action":"create",
    "node":{
        "key":"\/apisix\/routes\/00000000000000000308",
        "value":{
            "service_id":"00000000000000000292",
            "methods":[
                "GET"
            ],
            "uri":"\/mypath01\/*",
            "host":"test.my.com",
            "id":"00000000000000000308",
            "status":1,
            "update_time":1647070197,
            "priority":0,
            "create_time":1647070197
        }
    }
}
```
> 将上面创建的Service绑定一个rout上

### 删除Service
```shell
curl "http://127.0.0.1:9080/apisix/admin/routes/00000000000000000292" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X DELETE
```

### 测试验证route
```shell
curl http://127.0.0.1:9080/mypath01/json -H "Host: test.my.com"
```

## Consumer增删改查
```shell
curl http://127.0.0.1:9080/apisix/admin/consumers -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '{
    "username": "test",
    "plugins": {
        "key-auth": {
            "key": "auth-one-test"
        },
        "limit-count": {
            "count": 2,
            "time_window": 60,
            "rejected_code": 503,
            "key": "remote_addr"
        }
    }
}'

## 返回数据
{
    "action":"set",
    "node":{
        "key":"\/apisix\/consumers\/test",
        "value":{
            "username":"test",
            "update_time":1647070804,
            "plugins":{
                "limit-count":{
                    "count":2,
                    "key":"remote_addr",
                    "time_window":60,
                    "show_limit_quota_header":true,
                    "key_type":"var",
                    "policy":"local",
                    "rejected_code":503,
                    "allow_degradation":false
                },
                "key-auth":{
                    "key":"auth-one-test"
                }
            },
            "create_time":1647070804
        }
    }
}
```

### 创建route支持consumer
```shell
curl "http://127.0.0.1:9080/apisix/admin/routes" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
    "methods": ["GET"],
    "host": "test.my.com",
    "uri": "/mypath04/consumer",
    "plugins": {
        "key-auth": {}
    },
    "upstream_id": "00000000000000000122"
}'
```
> upstream_id也是上面创建的upstream

### 删除consumer
```shell
curl "http://127.0.0.1:9080/apisix/admin/consumers/test" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X DELETE
```
### 测试验证请求
```shell
curl http://127.0.0.1:9080/mypath04/consumer -H "Host: test.my.com" -H 'apikey: auth-one-test'
```

## 配置Plugin

### 在route创建Plugin
plugin可以在`Consumer`、`Service`，`Route`配置，同一个Plugin在一个请求中只执行一次，如果配置相同的Plugin，优先顺序：
`Consumer`、`Route`，`Service`。

```shell
curl "http://127.0.0.1:9080/apisix/admin/routes" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
    "methods": ["GET"],
    "host": "test.my.com",
    "uri": "/mypath03/plugin",
    "plugins": {
        "limit-count": {
            "count": 2,
            "time_window": 60,
            "rejected_code": 503,
            "key": "remote_addr"
        },
        "prometheus": {}
    },
    "upstream_id": "00000000000000000122"
}'
```
> plugin绑定在route上

### 测试验证
```shell
curl http://127.0.0.1:9080/mypath03/plugin -H "Host: test.my.com"
```

## 引用
1. [APISIX架构](https://apisix.apache.org/zh/docs/apisix/architecture-design/route)

2. [APISIX admin API](https://apisix.apache.org/zh/docs/apisix/admin-api)

3. [APISIX快速入门指南](https://apisix.apache.org/zh/docs/apisix/getting-started)
