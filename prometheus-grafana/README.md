

## docker 部署 Prometheus

```
sudo docker run -d -p 9090:9090 -v /data/etc/prometheus.yml:/etc/prometheus/prometheus.yml -v /data/etc/alert.rules:/etc/prometheus/alert.rules --name prometheus prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.enable-lifecycle
```

> 注意： -v 参数必须制定到绝对路径，否则会报错
> 启动时加上“--web.enable-lifecycle”会启用远程热加载配置文件功能，调用指令是 curl -X POST http://localhost:9090/-/reload


## 参考

http://cizixs.com/2018/01/24/use-prometheus-and-grafana-to-monitor-linux-machine/
https://songjiayang.gitbooks.io/prometheus/content/
https://github.com/songjiayang/prometheus_practice
