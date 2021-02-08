
# prometheus 和 grafana 监控

node面板可以使用：https://grafana.com/grafana/dashboards/1860


## 注意
- 需要提前创建docker网络：
  - 查看：docker network create --driver bridge monitor-net
  - 创建：docker network inspect monitor-net

## Reference

- http://cizixs.com/2018/01/24/use-prometheus-and-grafana-to-monitor-linux-machine/
- https://www.bogotobogo.com/DevOps/Docker/Docker_Prometheus_Grafana.php
- https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana
