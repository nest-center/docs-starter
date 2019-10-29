# 配置清单

```yaml
consul:
  host: localhost
  port: 8500
  # 多网卡或者容器场景下需要手动指定服务IP，否则可能会自动选择内网IP
  discoveryHost: localhost
  healthCheck:
    timeout: 1s
    interval: 10s
  maxRetry: 5
  retryInterval: 5000
  service:
    id: null
    name: service
    port: 3000
  config:
    # 表达式支持获取yaml以及系统变量中的数据
    key: config__${{ consul.service.name }}__${{ NODE_ENV }}
    retry: 5

# 支持通过 consul-config 加载
gateway:
  routes:
    - id: user
      uri: lb://multi-center-user-service
    - id: pay
      uri: http://pay.example.com

# 支持通过 consul-config 加载
feign:
  axios:
    timeout: 1000

# 支持通过 consul-config 加载
loadbalance:
  ruleCls: RandomRule

logger:
  level: info
  transports:
    - transport: console
      colorize: true
      datePattern: YYYY-MM-DD h:mm:ss
      label: nestcenter
    - transport: file
      name: info
      filename: info.log
      datePattern: YYYY-MM-DD h:mm:ss
      label: nestcenter
      maxSize: 104857600
      json: false
      maxFiles: 10
    - transport: dailyRotateFile
      label: nestcenter
      filename: info.log
      datePattern: YYYY-MM-DD-HH
      zippedArchive: true
      maxSize: 20m
      maxFiles: 14d
```
