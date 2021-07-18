

### 安装 ES

> https://www.elastic.co/cn/downloads/elasticsearch

Starting a single node cluster with Docker

```bash
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.9.0
```

docker install: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

### 安装 Kibana

> https://www.elastic.co/guide/en/kibana/current/install.html

docker 安装：https://www.elastic.co/guide/en/kibana/current/docker.html

`docker pull docker.elastic.co/kibana/kibana:7.9.0`

### 安装 filebeat

> https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html

运行 filebeat

```bash
sudo ./filebeat -e -c filebeat.yml
```


## Reference

- https://juejin.im/post/6844903810536587277