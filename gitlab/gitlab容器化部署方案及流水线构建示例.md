# gitlab 容器化部署方案&流水线构建

## 一、gitlab部署方案

官方参考文档： [https://docs.gitlab.com/install/docker/installation/#install-gitlab-by-using-docker-compose](https://docs.gitlab.com/install/docker/installation/#install-gitlab-by-using-docker-compose)

容器化部署方案：

```yaml
services:
  gitlab:
    image: gitlab/gitlab-ce:18.1.5-ce.0
    container_name: gitlab
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.example.com:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2424
    ports:
      - '8929:8929'
      - '443:443'
      - '2424:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
```

自动生成root用户密码

```yaml
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

Password: LVJQlzXi6A1t1aaEnw3ogwdb1tMmXwCjeI6/0dnW0Wc=

2.  配置CI/CD
    

这里的runner 安装分为宿主机安装和docker安装，我这里还是docker方式安装

[https://docs.gitlab.com/runner/install/docker/](https://docs.gitlab.com/runner/install/docker/)

## 二、gitlab runner使用

### 使用 GitLab CI Runner 的完整流程

整个过程可以分为三个主要步骤：

1.  **配置 Runner**：在 GitLab 项目中注册一个 Runner。
    
2.  **编写配置文件**：在项目中创建 `.gitlab-ci.yml` 文件，告诉 Runner 要做什么。
    
3.  **提交并观察**：将 `.gitlab-ci.yml` 文件推送到 GitLab，Runner 会自动开始工作。
    

---

### 第 1 步：配置 Runner

#### 1.1 获取 Runner 注册令牌

首先，你需要一个“通行证”来让 Runner 加入你的项目。

1.  打开你的 GitLab 项目页面。
    
2.  进入 **Settings** -> **CI/CD**。
    
3.  展开 **Runners** 设置区域。
    
4.  你会看到一个 **"Set up a specific Runner manually"** 的部分，这里有一个 **"Registration token"**（注册令牌）。**复制这个令牌**，它将在接下来的步骤中使用。
    

#### 1.2 安装并注册 Runner

现在，你需要在一台机器上安装 Runner 程序并使用刚才的令牌进行注册。 **前提条件**：这台机器需要安装了 Git 和 Go（Runner 是用 Go 语言编写的）。 **安装 Runner (以 Linux 为例)**

```bash
# 下载并安装官方二进制文件
curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
# 添加执行权限
sudo chmod +x /usr/local/bin/gitlab-runner

```

**注册 Runner**

```bash
sudo gitlab-runner register

```

接下来，它会问你一系列问题：

1.  \*\*Enter the GitLab instance URL (e.g. https://gitlab.com/)😗\*
    
    *   输入你的 GitLab 服务器地址。如果是 `gitlab.com`，就直接回车。如果是自建的，就输入你的服务器地址，如 `https://gitlab.yourcompany.com\`。
        
2.  **Enter the registration token:**
    
    *   **粘贴**你刚才在 GitLab 项目设置页面复制的 **Registration token**。
        
3.  **Enter a description for the runner:**
    
    *   给这个 Runner 起个名字，方便你识别。例如：`my-project-runner` 或 `docker-runner`。
        
4.  **Enter tags for the runner:**
    
    *   输入标签。**标签非常重要**，它用来让 Runner 匹配 `.gitlab-ci.yml` 文件中的特定作业。例如：`docker`, `build`, `deploy`。你可以用逗号分隔多个标签。
        
5.  **Enter optional maintenance note for the runner:**
    
    *   可以留空，直接回车。
        
6.  **Enter an executor:**
    
    *   选择 Runner 的执行器。这是最关键的一步，它决定了 Runner 在什么样的环境中运行你的任务。
        
    *   `**docker**` **(最常用)**：在 Docker 容器中运行任务。可以确保环境隔离和一致性。
        
    *   `**shell**`：直接在 Runner 所在服务器的命令行中运行任务。权限较高，但可能影响服务器本身。
        
    *   `**kubernetes**`：在 Kubernetes 集群中动态创建 Pod 来运行任务。
        
    *   `**ssh**`：通过 SSH 连接到另一台远程服务器来运行任务。
        
    *   `**virtualbox**`：在 VirtualBox 虚拟机中运行任务。 **对于初学者，强烈推荐** `**docker**`。 如果你选择了 `docker`，它还会进一步询问：
        
    *   **Default Docker image (e.g. ruby:2.7):**
        
        *   默认的 Docker 镜像。当你的作业没有指定镜像时，会使用这个。可以填一个通用的，如 `alpine:latest`。
            

#### 1.3 启动 Runner 

注册完成后，Runner 不会自动运行，你需要手动启动它：

```bash
sudo gitlab-runner start

```

## 现在，回到 GitLab 的 Settings -> CI/CD -> Runners 页面，你应该能看到你刚刚注册的 Runner 状态变成了 "Online"。

### 第 2 步：编写 `.gitlab-ci.yml` 文件

这是 Runner 的工作手册。在你的项目根目录下创建这个文件。 **示例：一个简单的 Node.js 项目**

```yaml
# 定义流水线的阶段，按顺序执行
stages:
  - build
  - test
  - deploy
# 定义作业
build_job:
  stage: build
  script:
    - echo "正在安装依赖..."
    - npm install
  tags:
    - docker # 关键：只有带有 "docker" 标签的 Runner 才能执行这个作业
  artifacts:
    paths:
      - node_modules/ # 将 node_modules 目录作为产物保存，供后续作业使用
test_job:
  stage: test
  script:
    - echo "正在运行测试..."
    - npm test
  tags:
    - docker
deploy_job:
  stage: deploy
  script:
    - echo "部署到测试服务器..."
    # 这里可以写部署脚本，比如 scp、ssh 等
    # 例如: ssh user@server "cd /path/to/app && git pull"
  tags:
    - docker
  when: manual # 将部署设置为手动触发，避免自动部署到生产环境

```

**文件解析**：

*   `**stages**`: 定义了流水线的三个阶段：`build`、`test`、`deploy`。作业会按这个顺序执行。
    
*   `**build_job**` **/** `**test_job**` **/** `**deploy_job**`: 定义了三个具体的作业。
    
*   `**stage**`: 指定作业属于哪个阶段。
    
*   `**script**`: **核心部分**，定义了要执行的 shell 命令列表。
    
*   `**tags**`: **核心部分**，指定了哪些 Runner 可以运行这个作业。只有标签匹配的 Runner（我们注册时设置的 `docker`）才会被调用。
    
*   `**artifacts**`: 定义了作业成功后需要保留的文件或目录。这些产物会被传递给后续的作业，或者下载到 GitLab。
    
*   `**when: manual**`: 让这个作业需要手动点击才能运行，而不是自动运行。
    

---

### 第 3 步：提交并观察

1.  将 `.gitlab-ci.yml` 文件添加到 Git 并推送到你的仓库。
    
2.  现在，回到 GitLab 项目页面，点击左侧菜单栏的 **CI/CD -> Pipelines**。
    
3.  你会看到一个流水线正在运行或刚刚运行。点击它可以查看每个阶段的执行日志，就像看机器人工作一样！
    

### 常见问题

*   **Runner 状态是 "Offline"**：
    
    *   检查 Runner 服务是否正在运行：`sudo gitlab-runner status`。
        
    *   检查 Runner 服务器是否能正常访问你的 GitLab URL。
        
    *   检查 Runner 的配置文件 `/etc/gitlab-runner/config.toml` 中的 URL 和 Token 是否正确。
        
*   **作业一直处于 "Pending" 状态**：
    
    *   最常见的原因是 `**tags**` **不匹配**。检查 `.gitlab-ci.yml` 中的 `tags` 和 Runner 注册时设置的标签是否完全一致（包括大小写）。
        
    *   检查 Runner 是否有足够的资源（CPU, 内存）来运行作业。
        
*   **作业执行失败**：
    
    *   点击作业，查看 **"Job log"**（作业日志），里面会显示 `script` 中每条命令的执行结果和错误信息，这是排查问题的最佳途径。 通过以上步骤，你就可以成功配置并使用 GitLab CI Runner 来自动化你的开发流程了。
        

---

1.配置docker 运行时环境变量  vim /opt/gitlab-runner/runner\_vars

```plaintext
CI_SERVER_URL=http://192.168.2.201:8888
REGISTRATION_TOKEN=glrt-zytrWq83MNoL7U1l1-Q1FG86MQpwOjEKdDozCnU6MQ8.01.1717436o7
REGISTER_NON_INTERACTIVE=true
RUNNER_EXECUTOR=docker
DOCKER_IMAGE=alpine:latest
# 根据需要添加更多变量，如 RUNNER_TAG_LIST="docker,linux"
```

2.  运行容器
    

```shell
sudo docker run --rm -it -v /opt/gitlab-runner/config:/etc/gitlab-runner --env-file /opt/gitlab-runner/runner_vars gitlab/gitlab-runner:latest register
```

注册完之后会在宿主机 /opt/gitlab-runner/config/  文件夹下生产一系列配置文件

启动runner （必须要要启动一个常驻的runner来运行CI）

```shell
sudo docker run -d --name gitlab-runner --restart always \
  -v /opt/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \ 
 gitlab/gitlab-runner:latest
```

测试CI  在一个测试项目中 vim  .gitlab-ci.yml

```yaml
hello-world-job:
  stage: test
  tags:
    - docker # 必须和Runner的标签之一匹配
  script:
    - echo "Hello World"
```

再docker-runner中远程登录机器进行操作

1.  等于docker-gitlab-runner 并生成ssh 密钥对,然后将生成的公钥copy到远程机器上面
    
    ```yaml
    sudo docker exec -it  gitlab-runner  bash
    
    ssh-keygen -t rsa -b 4096 
    
    ssh-copy-id username@B机器的IP地址
    ```
    
2.  在项目所属的CI/CD 设置中添加 环境变量 SSH\_PRIVATE\_KEY   将上一步生成的私钥填入
    
3.  编写.gitlab-ci.yml 
    

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

variables:
  BINARY_NAME: "demo"
  REMOTE_SERVER: "1.13.246.180"
  REMOTE_USER: "ubuntu"
  REMOTE_PATH: "/home/ubuntu/app"
# 编译
compile_go_binary:
  stage: build
  tags:
    - docker
  image: golang:1.24.5-alpine
  script:
    - echo "编译Go应用..."
    - CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o $BINARY_NAME .
    - ls -la $BINARY_NAME
  artifacts:
    paths:
      - $BINARY_NAME
# 部署
deploy_to_production:
  stage: deploy
  tags:
    - docker
  needs: ["compile_go_binary"]
  image: alpine:latest  # 镜像更小
  before_script:
    - sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
    - apk update && apk add --no-cache openssh-client
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa
    - ssh-keyscan -H $REMOTE_SERVER >> ~/.ssh/known_hosts
  script:
    - scp -i ~/.ssh/id_rsa $BINARY_NAME $REMOTE_USER@$REMOTE_SERVER:$REMOTE_PATH/
    - ssh -i ~/.ssh/id_rsa $REMOTE_USER@$REMOTE_SERVER <<EOF
      cd $REMOTE_PATH
      chmod +x $BINARY_NAME
      pkill -f $BINARY_NAME || true
      nohup ./$BINARY_NAME > app.log 2>&1 &
      echo '应用已部署并启动'
      EOF


```

## 配置镜像仓库功能

TODO: 功能开启成功，但是推送镜像失败。解决了，因为docker中配置了代理的原因。

## ✅ GitLab Container Registry 简介

GitLab Container Registry 是一个内置的 Docker 镜像仓库，与 GitLab 项目紧密集成，支持：

*   私有/公开镜像存储
    
*   基于权限的访问控制
    
*   与 CI/CD 流程无缝集成
    
*   自动清理镜像策略
    

---

## 📦 开启 Container Registry

### 1. 确认 GitLab 版本支持

Container Registry 自 GitLab 8.8 开始支持，建议使用 GitLab 11.0+ 以获得最佳体验。

### 2. 启用 Container Registry

在 GitLab 管理员后台中：

*   进入 **Admin Area > Settings > CI/CD**
    
*   找到 **Container Registry**，勾选 **Enable Container Registry**
    

---

## 🔐 配置访问权限

### 权限说明

| 角色 | 权限 |
| --- | --- |
| Guest | 可拉取公开镜像 |
| Reporter | 可拉取私有镜像 |
| Developer | 可拉取和推送镜像 |
| Maintainer/Owner | 可管理镜像 |

---

## 🚀 使用 GitLab Container Registry

### 1. 登录 Registry

使用 GitLab 账号登录：

```bash
docker login registry.gitlab.com

```
> 若是自建 GitLab，则替换为你的域名，如 `docker login gitlab.example.com:5050`

### 2. 构建 Docker 镜像并推送

#### 示例 Dockerfile

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]

```

#### 构建镜像

```bash
docker build -t registry.gitlab.com/your-group/your-project/app:latest .

```

#### 推送镜像

```bash
docker push registry.gitlab.com/your-group/your-project/app:latest

```
---

## 🔄 与 GitLab CI/CD 集成

你可以在 `.gitlab-ci.yml` 中自动构建和推送 Docker 镜像：

```yaml
build-image:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

```

### 变量说明

*   `$CI_REGISTRY`：GitLab Registry 地址
    
*   `$CI_REGISTRY_USER`：部署用户名（通常是 `gitlab-ci-token`）
    
*   `$CI_REGISTRY_PASSWORD`：临时 token
    
*   `$CI_REGISTRY_IMAGE`：项目镜像地址
    

---

## 🧹 清理镜像（可选）

GitLab 支持设置自动清理策略：

*   进入 **项目 > Settings > CI/CD > Container Registry**
    
*   开启 **Cleanup tags**
    
*   可配置保留数量、时间等规则
    

---

docker添加私有仓库地址（不安全方式）

vim  /etc/docker/daemon.json

```json
  {
    "registry-mirrors": [
	    "https://reg-mirror.qiniu.com",
	    "https://hub-mirror.c.163.com"
    ],
    "insecure-registries": ["192.168.2.201:5050"]
  }
```

重启docke服务

```shell
sudo systemctl restart docker
```

---

### 第一部分：确保 GitLab Runner 配置正确（Docker 执行器）

首先，您的 Runner 必须使用 `docker` 执行器（Executor），并且配置正确。注册时应该类似这样（参考您之前的 token）：

```bash
sudo docker run --rm -it \
  -v /opt/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest \
  register \
  --url "http://192.168.2.201:8888" \
  --token "glrt-zytrWq83MNoL7U1l1-Q1FG86MQpwOjEKdDozCnU6MQ8.01.1717436o7" \
  --executor "docker" \          # 最重要：指定执行器为 docker
  --docker-image "alpine:latest" # 指定默认镜像

```

注册成功后，查看 `/opt/gitlab-runner/config/config.toml`，核心配置应该类似这样：

```toml
[[runners]]
  name = "my-docker-runner"
  url = "http://192.168.2.201:8888"
  token = "YOUR_PROJECT_OR_SHARED_TOKEN"
  executor = "docker" # 必须是 docker
  [runners.docker]
    image = "alpine:latest" # 默认镜像
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"] # 常见挂载

```

**确保 Runner 容器本身在运行**：

```bash
sudo docker run -d --name gitlab-runner --restart always \
  -v /opt/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \ # 允许在容器内运行 Docker 命令（用于 DinD）
  gitlab/gitlab-runner:latest

```
---

### 第二部分：编写 `.gitlab-ci.yml` 文件

这个文件必须放在您 Git 仓库的**根目录**下。当您推送代码时，GitLab 会读取这个文件并根据定义执行流水线。

#### 一个最简单的示例：测试一个 Node.js 项目

假设您有一个简单的 Node.js 项目，您想在每个 `push` 时运行测试。

```yaml
# .gitlab-ci.yml

# 定义流水线的阶段（stage），按顺序执行
stages:
  - test
  - build # 您可以后续添加 build, deploy 等阶段

# 1. 定义第一个工作：运行测试
test-job:          # 作业的名称，可以自定义
  stage: test      # 这个作业属于 'test' 阶段
  image: node:18   # 指定这个作业在哪个 Docker 镜像中运行（覆盖默认的 alpine）
  
  # 指定缓存，可以加速后续流水线运行
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

  # 在运行 script 前执行的命令
  before_script:
    - npm install # 安装依赖

  # 该作业要运行的核心脚本命令
  script:
    - npm run test # 运行测试脚本，假设您在 package.json 中定义了 test 命令

  # 定义哪些分支触发这个作业
  rules:
    - if: $CI_COMMIT_BRANCH # 对所有分支的推送都触发

```

#### 一个更复杂的示例：包含构建和测试的 Java Maven 项目

```yaml
# .gitlab-ci.yml

stages:
  - test
  - build

# 使用 Maven 镜像来运行测试
unit-test:
  stage: test
  image: maven:3.8.6-openjdk-11
  variables:
    MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository" # 将 Maven 仓库缓存到当前目录
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .m2/repository/ # 缓存 Maven 依赖
      - target/         # 缓存编译结果
  script:
    - mvn clean test   # 清理并运行测试

# 构建 JAR 包
build-jar:
  stage: build
  image: maven:3.8.6-openjdk-11
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .m2/repository/
      - target/
  script:
    - mvn clean package -DskipTests # 跳过测试（因为上一个作业已经做过了）
  artifacts:
    paths:
      - target/*.jar # 将生成的 JAR 包作为制品保存，可供下载或后续阶段使用
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH # 只有推送到默认分支（如 main）时才构建

```
---

### 第三部分：工作流程和关键概念解释

1.  **触发流程**：
    
    *   您将代码和 `.gitlab-ci.yml` 文件推送到 GitLab。
        
    *   GitLab 检测到推送，读取 `.gitlab-ci.yml`。
        
    *   它根据文件内容创建一条流水线（Pipeline），其中包含多个阶段（Stage）和作业（Job）。
        
    *   空闲的 Runner（您的 Docker Runner）会“领取”这些作业。
        
    *   Runner 根据作业中指定的 `image` 拉取 Docker 镜像，启动一个全新的容器。
        
    *   在这个新容器中，Runner 按顺序执行 `before_script` 和 `script` 中的命令。
        
    *   执行成功或失败后，结果会反馈到 GitLab 界面上。
        
2.  **核心关键字**：
    
    *   `image`：**指定每个作业的运行环境**。这是最重要的部分，您需要根据项目技术栈选择（如 `node:18`, `python:3.11`, `maven:3.8-openjdk-11`, `golang:1.20`）。
        
    *   `script`：**必须存在的部分**，定义要执行的 Shell 命令。
        
    *   `before_script`：在 `script` 之前运行，常用于安装依赖、配置环境。
        
    *   `cache`：**加速流水线的关键**，用于在多次运行之间缓存依赖（如 `node_modules/`, `.m2/repository/`）。
        
    *   `artifacts`：将作业生成的文件（如编译好的包、日志报告）保存起来，可供下载或传递给后续阶段。
        
    *   `rules` / `only` / `except`：控制作业在何时运行（例如：仅针对特定分支或标签）。
        

### 总结

1.  **确保 Runner 注册为** `**executor = "docker"**`。
    
2.  **在项目根目录创建** `**.gitlab-ci.yml**` **文件**。
    
3.  **在文件中定义** `**stages**` **和** `**jobs**`。
    
4.  **每个job必须指定** `**image**`（选择适合您项目的环境镜像）和 `script`（需要运行的命令）。
    
5.  **使用** `**cache**` **来缓存依赖以提升速度**。
    
6.  **将代码推送到 GitLab**，然后在项目的 **CI/CD > Pipelines** 页面查看自动触发的流水线状态。
    

从最简单的 `script: - echo "Hello CI"` 开始测试，然后逐步添加您项目真实的测试和构建命令。
