
# sonarqube 使用说明

## 安装sonarqube


### docker快速启动

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
```

TIPS: 以上是使用自带的H2内存数据库，重启后数据会丢失。
更多docker镜像地址：[https://hub.docker.com/_/sonarqube/](https://hub.docker.com/_/sonarqube/)

## docker-compose 安装(推荐)

使用 Postgres 数据库存储数据

```bash
docker-compose up
```

## 创建项目

这里以go项目为例进行说明

打开 `http://localhost:9000/` sonarqube管理地址，项目菜单下新建 go项目 `go-demo`

## 安装SonarScanner扫描器

下载地址：[https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)

可以选择对应系统版本，或者docker安装。

### docker安装

```bash
docker run -e SONAR_HOST_URL=http://localhost:9000 -it -v "$(pwd):/usr/src" sonarsource/sonar-scanner-cli
```

更多可以参考：[https://github.com/SonarSource/sonar-scanner-cli-docker](https://github.com/SonarSource/sonar-scanner-cli-docker)

### 手动下载安装

下载地址列表：[https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/](https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/)

选择适合自己的平台下载即可。

## 配置

### 配置sonarqube地址

进入到下载目录,可以看到下面四个目录:
```bash
.
├── bin
├── conf
├── jre
└── lib
```

修改配置：`vim conf/sonar-scanner.properties`

```bash

#----- Default SonarQube server
# 必须强制开启，不然找不到sonarqube无法同步运行结果
sonar.host.url=http://localhost:9000

#----- Default source code encoding
sonar.sourceEncoding=UTF-8
```

### 加入到path

```bash
# vim ~/.bashrc 或 vim ~/.zshrc
export PATH=$PATH:/path/to/sonar-scanner/bin
```

使配置生效: `source ~/.zshrc` 或 `source ~/.bashrc`
然后就可以使用 `sonar-scanner` 了。

```bash
➜  ~ sonar-scanner -h
INFO:
INFO: usage: sonar-scanner [options]
INFO:
INFO: Options:
INFO:  -D,--define <arg>     Define property
INFO:  -h,--help             Display help information
INFO:  -v,--version          Display version information
INFO:  -X,--debug            Produce execution debug output
```


### 在项目中进行配置

#### 方式一

创建一个文件: `sonar-project.properties`

写入以下内容

```bash
sonar-project.properties
# must be unique in a given SonarQube instance
sonar.projectKey=my:go-demo

# --- optional properties ---

# defaults to project key
sonar.projectName=go-demo
# defaults to 'not provided'
#sonar.projectVersion=1.0

#go
sonar.sources=.
sonar.exclusions=**/*_test.go,**/vendor/**

sonar.tests=.
sonar.test.inclusions=**/*_test.go
sonar.test.exclusions=**/vendor/**
```

更多参数可以参考：[https://docs.sonarqube.org/latest/analysis/analysis-parameters/](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)

#### 方式二

如果不想在想项目中新建文件 `sonar-project.properties`，也可以运行如下命令：

```bash
sonar-scanner \
  -Dsonar.projectKey=my:go-demo \
  -Dsonar.sources=PROJECT_DIR. \
  -Dsonar.host.url=http://xxx.xxx.xxx.xxx:9100 \
  -Dsonar.login=xxxxxxxxxxxxxxxxxxxxxxxxxxxxabc
```

说明

- `PROJECT_DIR` 为项目地址 
- login 是在 sonarqube 上生成的token

## 自动化集成

自动化主要是依赖与第三方的一些ci/cd任务来执行 sonarqube-runner, 目前主要包括

- gitlab的ci/cd, 主要通过配置 `gitlab-ci.yml` 来执行
- jenkins里配置脚本，但是多个项目的话每个都需要配置
- jenkins里使用插件配置(SonarQube Scanner for Jenkins)，好处是一次配置多个项目可以共用

下面我们主要是在gitlab做处理，配置如下

```yaml
image:
  name: sonarsource/sonar-scanner-cli:latest
  entrypoint: [""]
variables:
  SONAR_TOKEN: "your-sonarqube-token"
  SONAR_HOST_URL: "http://your-sonarqube-instance.org"
  GIT_DEPTH: 0

stages:
  - analyze
  - build
  - test
  - deploy

job_sonar_analyze:
  stage: analyze
  script:
    - ci/sonar_analyze.sh
  only:
    - test
  tags:
    - demo

job_build:
  stage: build
  script:
    - echo "build over..."
  only:
    - test
  tags:
    - demo

job_test:
  stage: test
  script:
    - echo "test over..."
  tags:
    - demo

job_deploy:
  stage: deploy
  script:
    - echo "deploy over..."
  tags:
    - demo
```

sonar_analyze.sh 脚本

```bash
#!/bin/bash

sonar-scanner \
    -Dsonar.analysis.mode=preview \
    -Dsonar.projectKey=gitlab:$CI_COMMIT_REF_NAME:$CI_PROJECT_NAME \
    -Dsonar.projectName=gitlab:$CI_COMMIT_REF_NAME:$CI_PROJECT_NAME \
    -Dsonar.projectVersion=1.0.$CI_PIPELINE_ID \
    -Dsonar.issuesReport.html.enable=true \
    -Dsonar.gitlab.project_id=$CI_PROJECT_ID \
    -Dsonar.gitlab.commit_sha=$CI_COMMIT_SHA \
    -Dsonar.gitlab.ref_name=$CI_COMMIT_REF_NAME

if [ $? -eq 0 ]; then
    echo "sonarqube code-analyze over."
fi
```

参考
- [SonarQube 之 gitlab-plugin 配合 gitlab-ci 完成每次 commit 代码检测](https://blog.csdn.net/aixiaoyang168/article/details/78115646)
- [Docker搭建自己的Gitlab CI Runner](https://blog.csdn.net/aixiaoyang168/article/details/72168834)
- [Running Analysis with GitLab CI/CD](https://docs.sonarqube.org/latest/analysis/gitlab-cicd/)
- [Jenkins+SonarQube+Gitlab搭建自动化持续代码扫描质量平台](https://blog.csdn.net/zuozewei/article/details/84539396)
- [有赞 GO 项目单测、集成、增量覆盖率统计与分析](https://mp.weixin.qq.com/s/mNGkMggkkuRSuflHw3vIyA)

