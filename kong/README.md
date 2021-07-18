
## 官网地址

官网地址：https://konghq.com/
安装地址：https://konghq.com/get-started/#install

## 名词解释

Konga是 Kong 的 Dashboard，它是基于 JavaScript 的客户端管理工具，对外暴露的端口为 8080；
Kong-database是 Kong 的数据库服务，它用于存储配置信息，此处使用的是 Postgres。

## 创建网络

`docker-compose.yml` 会启动三个容器服务：Kong、Konga 和 Kong-database。
这三个容器之间的通信需要增加 network 段，把容器放在同一个网段内，相关链接修改为容器名称来访问。

```bash
docker network create kong-net
```

## 初始化

### 启动数据库

```bash
docker run -d --name kong-database \
            --network=kong-net \
            -p 5432:5432 \
            -e "POSTGRES_USER=kong" \
            -e "POSTGRES_DB=kong" \
            -e "POSTGRES_PASSWORD=kong" \
            postgres:9.6

```

### 初始化数据

```bash
docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     kong:1.1.2 kong migrations bootstrap

```

## 启动

```bash
docker-compose up -d
```