# gitlab-runner

## docker 安装gitlab-runner


### 拉取gitlab-runner镜像

```bash
sudo docker pull gitlab/gitlab-runner:latest
```

### 添加gitlab-runner container

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

## docker-compose 安装

```bash
docker-compose up
```

## 注册runner

在gitlab的路由 /admin/runners 下获取到URL和token,然后开始注册

```bash
sudo docker exec -it gitlab-runner gitlab-ci-multi-runner register
```

## Reference

- [gitlab-ci详细的配置文档](https://docs.gitlab.com/ee/ci/yaml/README.html)
- [Runner和GitLab CE / EE兼容性列表](https://gitlab.com/gitlab-org/gitlab-runner)
- [Docker搭建自己的Gitlab CI Runner](https://blog.csdn.net/aixiaoyang168/article/details/72168834)
- [GitLab CI/CD environment variables](https://docs.gitlab.com/ce/ci/variables/)