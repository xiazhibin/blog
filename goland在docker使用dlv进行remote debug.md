### 代码结构

跟一般的go项目目录结构一样

├── Dockerfile
├── bin
├── docker-compose.yaml
├── pkg
└── src
    ├── debug
    ├── main.go
    └── utils
        └── utils.go

### Dockerfile

```yaml
FROM golang:1.10

RUN go get github.com/derekparker/delve/cmd/dlv
CMD ["dlv", "debug","--listen=:2345", "--headless=true", "--api-version=2"]
```

### docker-compose
```yaml
version: "3.4"

x-defaults: &default
  restart: unless-stopped
  build: .
  volumes:
    - ./src:/go/src
  networks:
    - default

services:
  app:
    <<: *default
    container_name:app
    hostname: "app"
    working_dir: /go/src
    ports:
        - "2345:2345"
    security_opt:
      - "seccomp:unconfined"


networks:
  default:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.129.2.0/24
```

### 设置goland
编辑`Run configurations`，增加一个`go remote`

然后就开始debug啦！

### 注意事项
- 如果打的断点没有效果，请注意GOPATH的设置,默认`GOPATH`是`/go`
