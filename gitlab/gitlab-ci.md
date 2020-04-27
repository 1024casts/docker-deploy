# Gitlab-CI

## 目标

我们需要通过 Gitlab-CI 来达到什么目的，解决什么样的问题呢？ 下面是一些需要解决的需求

- 禁止新增代码的测试覆盖度低于80%的PR/MR merge 到 test 分支

那么 Gitlab-CI 是什么以及如何运作的呢？ 下面我们来详细分析下。

## 什么是GitLab-CI

GitLab CI 是 GitLab Continuous Integration （Gitlab 持续集成）的简称。从 GitLab 的 8.0 版本开始，GitLab 就全面集成了 Gitlab-CI,并且对所有项目默认开启。大概需要下面3步，就可以搭建起CI流程

- 在项目仓库的根目录添加 `.gitlab-ci.yml` 文件
- 配置了 Runner（运行器）
- 每此添加提交的动作都会触发 CI pipeline

## 基本概念

在介绍 GitLab CI 之前，我们先看看一些持续集成相关的概念。

![pipeline](https://about.gitlab.com/images/blogimages/cicd_pipeline_infograph.png)

### Pipeline

一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。 任何提交或者 Merge Request 的合并都可以触发 Pipeline，如下图所示：

```bash
+------------------+           +----------------+
|                  |  trigger  |                |
|   Commit / MR    +---------->+    Pipeline    |
|                  |           |                |
+------------------+           +----------------+
```

### Stages

Stages 表示构建阶段，说白了就是上面提到的流程。 我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：  

- 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始
- 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
- 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败

因此，Stages 和 Pipeline 的关系就是：

```bash
+--------------------------------------------------------+
|                                                        |
|  Pipeline                                              |
|                                                        |
|  +-----------+     +------------+      +------------+  |
|  |  Stage 1  |---->|   Stage 2  |----->|   Stage 3  |  |
|  +-----------+     +------------+      +------------+  |
|                                                        |
+--------------------------------------------------------+
```

### Jobs

Jobs 表示构建工作，表示某个 Stage 里面执行的工作。 我们可以在 Stages 里面定义多个 Jobs，这些 Jobs 会有以下特点：  

- 相同 Stage 中的 Jobs 会并行执行
- 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功
- 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 失败

所以，Jobs 和 Stage 的关系图就是：

```bash
+------------------------------------------+
|                                          |
|  Stage 1                                 |
|                                          |
|  +---------+  +---------+  +---------+   |
|  |  Job 1  |  |  Job 2  |  |  Job 3  |   |
|  +---------+  +---------+  +---------+   |
|                                          |
+------------------------------------------+
```

## 部署流程

前面大致介绍了构建自动化构建的用到的一些概念，那么我们就可以开始我们的部署了。

### 定义规则 .gitlab-ci.yml

下面以go项目为例

```bash
before_script:  
  - echo "before script action"

# 定义 stages
stages:
  - build
  - test
  - deploy

# 定义 job
job_build:
  stage: build
  script:
    - echo "test build"
  only:
    - master
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

after_script:  
  - echo "after script action"

```

更多配置可以参考 [官方文档](https://docs.gitlab.com/ce/ci/yaml/README.html)


那么配置了这些任务，具体由谁来执行呢，那就是 `Gitlab Runner`.

### 安装 gitlab-runner

```bash
# For Debian/Ubuntu
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash
$ sudo apt-get install gitlab-ci-multi-runner
​
# For CentOS
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
$ sudo yum install gitlab-ci-multi-runner
```

也可以通过docekr方式来安装，参考 `gitlab-runner.md`。

### 注册 runner

安装好 GitLab Runner 之后，我们只要启动 Runner 然后和 CI 绑定就可以了。

为了能够让 GitLab Runner 能够连接到我们的项目上需要注册操作：

```bash
sudo gitlab-runner register
```

然后根据提示输入配置信息（这些信息可以在项目的gitlab网站的CI/CD 配置里找到， 需要master权限）, 访问 `/admin/runners` 即可。

运行 sudo gitlab-ci-multi-runner register

```bash
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://yourgitlab  //项目gitlab的根域名， 一般公司都会部署自己内部使用的gitlab

Please enter the gitlab-ci token for this runner
xxx   // gitlab token

Please enter the gitlab-ci description for this runner
[hostame] my-runner  // 项目描述， 起个名称

Please enter the gitlab-ci tags for this runner (comma separated):
demo,docker   // 给该 Runner 指派 tags, 稍后也可以在 GitLab's UI 修改, 这里也可以直接回车， 使用默认值

Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
shell  // 选择runner的类型， 这里用shell就好，也可以使用docker

Please enter the default Docker image (e.g. ruby:2.6): // 如果选择了docker会有这一步
golang:latest  // 镜像名称
```

配置完，可以通过 `gitlab-runner list` 查看已注册runner的运行状态

```bash
$ sudo gitlab-runner list
Listing configured runners          ConfigFile=/etc/gitlab-runner/config.toml
my-runner                           Executor=shell Token=cd1cd7cf243afb47094677855aacd3 URL=http://yourgitlab/ci
```

### 更改代码提交触发相关动作来执行 CI


下面通过一个go项目的案例来看下具体流程

## 案例

以部署go项目为例

### 定义 .gitlab-ci.yml

```bash
image: go-tools:1.9.2

stages:
  - build
  - test

before_script:
  - mkdir -p /go/src/gitlab.com/go/go-demo /go/src/_/builds
  - cp -r $CI_PROJECT_DIR /go/src/gitlab.com/go/go-demo
  - ln -s /go/src/gitlab.com/go/go-demo /go/src/_/builds/go-demo  
  - cd /go/src/_/builds/go-demo

unit_tests:
  stage: test
  script:
    - make test
  tags:
    - demo

race_detector:
  stage: test
  script:
    - make race
  tags:
    - demo

coverage:
  stage: test
  script:
    - make coverage
  tags:
    - demo

coverage_report:
  stage: test
  script:
    - make coverhtml
  only:
  - master
  tags: 
    - demo

lint:
  stage: test
  script:
    - make lint

build:
  stage: build
  script:
    - pwd
    - go build .
  tags:
    - demo
```

### 定义 Makefile

如果我们不想在 .gitlab-ci.yml 文件中写的太复杂，那么我们可以把持续集成环境中使用的所有工具，全部打包在Makefile中，并用统一的方式调用它们。
这样的话，.gitlab-ci.yml文件就会更加简洁了。当然了，Makefile同样也可以调用 shell 脚本文件.

```makefile
PROJECT_NAME := "go-demo"
PKG := "http://182.92.238.120:8929/root/$(PROJECT_NAME)"
PKG_LIST := $(shell go list ./... | grep -v /vendor/)
GO_FILES := $(shell find . -name '*.go' | grep -v /vendor/ | grep -v _test.go)

test: ## Run unittests
    @go test -v ${PKG_LIST}

lint: ## Lint the files
    @golint ${PKG_LIST}

race: ## Run data race detector
    @go test -race -short ${PKG_LIST}

coverage: ## Generate global code coverage report
    ./scripts/coverage.sh;

coverhtml: ## Generate global code coverage report in HTML
    ./scripts/coverage.sh html;
```

#### coverage.sh

该脚本主要是统计各个包的测试覆盖率，然后汇总到一个文件

```bash
#!/bin/bash
#
# Code coverage generation

COVERAGE_DIR="${COVERAGE_DIR:-coverage}"
PKG_LIST=$(go list ./... | grep -v /vendor/)

# Create the coverage files directory
mkdir -p "$COVERAGE_DIR";

# Create a coverage file for each package
for package in ${PKG_LIST}; do
    go test -covermode=count -coverprofile "${COVERAGE_DIR}/${package##*/}.cov" "$package" ;
done ;

# Merge the coverage profile files
echo 'mode: count' > "${COVERAGE_DIR}"/coverage.cov ;
tail -q -n +2 "${COVERAGE_DIR}"/*.cov >> "${COVERAGE_DIR}"/coverage.cov ;

# Display the global code coverage
go tool cover -func="${COVERAGE_DIR}"/coverage.cov ;

# If needed, generate HTML report
if [ "$1" == "html" ]; then
    go tool cover -html="${COVERAGE_DIR}"/coverage.cov -o coverage.html ;
fi

# Remove the coverage files directory
rm -rf "$COVERAGE_DIR";
```

#### dockerfile

```bash
FROM golang:1.9.2

#定义环境变量 alpine专用
#ENV TIME_ZONE Asia/Shanghai

ADD application /go/src/demo/

WORKDIR /go/src/demo

ADD run_application.sh /root/
RUN chmod 755 /root/run_application.sh
CMD sh /root/run_application.sh

EXPOSE 8080
```

#### run_application.sh

```bash
#!/bin/bash

#映射ip
cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

cd /go/src/demo/

./application
```

### 执行 MR/PR

上面配置好以后，那如何指定什么时候执行对应的stage呢，是MR还是一提交就执行，这些其实也是可以定义的，需要在 .gitlab-ci.yml 加上workflow:rules, 如下：

```yaml
workflow:
  rules:
    - if: $CI_MERGE_REQUEST_ID             # Execute jobs in merge request context
    - if: $CI_COMMIT_BRANCH == 'test'      # Execute jobs when a new commit is pushed to master branch
```

官方配置文档：https://docs.gitlab.com/ee/ci/merge_request_pipelines/index.html

### 点亮badge

进入到对应项目，在 Settings > General > Badges. 添加相关参数后，即可获得徽章，

#### 添加构建徽章

- 名称: 构建徽章
- 链接: https://yourgitlab/root/go-demo/commits/master
- 徽章图片网址: https://yourgitlab/root/go-demo/badges/master/pipeline.svg

#### 添加测试覆盖率徽章

- 名称: 测试覆盖率徽章
- 链接: https://yourgitlab/root/go-demo
- 徽章图片网址: https://yourgitlab/root/go-demo/badges/master/coverage.svg

然后在项目文件的 readme 的首行添加：

```bash
[![coverage report](https://yourgitlab/root/go-demo/badges/master/coverage.svg)](https://yourgitlab/root/go-demo)
[![pipeline status](https://yourgitlab/root/go-demo/badges/master/pipeline.svg)](https://yourgitlab/root/go-demo/gold/commits/master)
```

### 在gitlab 上查看执行结果

在gitlab中项目的pipelines 里可以看到构建的结果：

`http://yourgitlab/root/go-demo/pipelines`

![file](https://s1.phpcasts.org/uploads/images/2020/04/27/_ZIFu6qZR.png)

## Refernce

- https://www.jianshu.com/p/8655f1ef26ee
- https://www.cnblogs.com/gaorong/p/9193485.html
- [gitlab-ci.yml官方文档](https://docs.gitlab.com/ce/ci/yaml/README.html)
- [Pipelines for Merge Requests](https://docs.gitlab.com/ee/ci/merge_request_pipelines/index.html)
- [Go工具& GitLab-让你如何轻松进行持续集成](https://www.ctolib.com/topics-127012.html)
- https://gitlab.com/pantomath-io/demo-tools
