# gitlab-runner

## docker 安装


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

进入交互模式后，需要提供以下内容：

- 输入 CI URL
- 输入 Token
- 输入 Runner 的名字
- 输入 tag 的名字, 注意：这个tag会和gitlab-ci.yml里job里指定的tag相关联
- 选择 Runner 的类型，简单起见还是选 Shell 吧
- 完成

## Reference

- [gitlab-ci详细的配置文档](https://docs.gitlab.com/ee/ci/yaml/README.html)
- [Runner和GitLab CE / EE兼容性列表](https://gitlab.com/gitlab-org/gitlab-runner)
- [Docker搭建自己的Gitlab CI Runner](https://blog.csdn.net/aixiaoyang168/article/details/72168834)
- [GitLab CI/CD environment variables](https://docs.gitlab.com/ce/ci/variables/)