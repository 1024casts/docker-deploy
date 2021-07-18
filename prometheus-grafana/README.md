
# prometheus 和 grafana 监控

## docker 部署 Prometheus

```
sudo docker run -d -p 9090:9090 -v /data/etc/prometheus.yml:/etc/prometheus/prometheus.yml -v /data/etc/alert.rules:/etc/prometheus/alert.rules --name prometheus prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.enable-lifecycle
```

> 注意： -v 参数必须制定到绝对路径，否则会报错
> 启动时加上“--web.enable-lifecycle”会启用远程热加载配置文件功能，调用指令是 curl -X POST http://localhost:9090/-/reload

## docker 部署 grafana

## grafana 插件推荐

- 监控node 使用 8919



## 参考

http://cizixs.com/2018/01/24/use-prometheus-and-grafana-to-monitor-linux-machine/
https://songjiayang.gitbooks.io/prometheus/content/
https://github.com/songjiayang/prometheus_practice
=======
node面板可以使用：https://grafana.com/grafana/dashboards/1860


## 监控面板
- 1860 监控节点机器

## Reference

- http://cizixs.com/2018/01/24/use-prometheus-and-grafana-to-monitor-linux-machine/
- https://www.bogotobogo.com/DevOps/Docker/Docker_Prometheus_Grafana.php
- https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana
