## 快速上手

鉴于 Docker 的便利性，目前教程只提供此种方法进行安装部署，首先克隆本项目：

```
$ git clone https://github.com/icyleaf/hpr.git
$ cd hpr
```

复制 [config/hpr.json.example](config/hpr.json.example) 并改名 `config/config.json` 后可修改

```json
{
  "schedule": 3600,
  "basic_auth": {
    "enable": false,
    "user": "hpr",
    "password": "p@ssw0rd"
  },
  "gitlab": {
    "ssh_port": 22,
    "endpoint": "http://gitlab.example.com/api/v3",
    "private_token": "abc",

    "group_name": "mirrors",

    "project_public": false,
    "project_issue": false,
    "project_wiki": false,
    "project_merge_request": false,
    "project_snippet": false
  }
}
```

核心需要修改的参数有如下四项：

- `endpoint`: Gitlab API 的地址，**无需修改后面部分**
- `private_token`: 在个人设置的 Account 页面获得
- `group_name`: 项目镜像的项目都会归属到这个组内，**务必确保你的账户拥有创建组的权限** (如果是管理员请忽略加粗字样)
- `ssh_port`: 如果 SSH 不是 22 端口的话需要根据你的实际情况修改

配置文件修改保存后还需要在 `docker-compose.yml` 文件中配置下：

```yaml
version: '2'

services:
  hpr:
    image: icyleafcn/hpr
    ports:
      - 8848:8848
    volumes:
      - ./config:/app/config
      - ./repositories:/app/repositories
    environment:
      REDIS_URL: tcp://redis:6379
      REDIS_PROVIDER: REDIS_URL

      HPR_SSH_HOST: git.example.com
      HPR_SSH_PORT: 22
    depends_on:
      - redis
  redis:
    image: redis:alpine
```

其中 `HPR_SSH_HOST` 和 `HPR_SSH_PORT` 变量用于设置 Docker 实例中的 SSH 配置。如果 SSH 端口是 22 的可忽略设置这俩参数。

编辑完成后运行下面命令快完成了！

```bash
$ docker-compose up
...
hpr_1      | Generating public/private rsa key pair ...
hpr_1      |
hpr_1      | GENERATED SSH PUBLIC KEY:
hpr_1      | ##################################################################
hpr_1      | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDq8O3HbLn9x8Uy8RUotlpOnxdakrmCyDpZrGBeLARmEbd6BOIBQ+UWm8NUKthQ7UOavmlsq4j8lY4kyFW2eFX2qWcbvI+s2gI+05MXax+mAukSszaNSnpAoTyJCRipilSkqiOV99V8JIJhrHPtTO0o/Ui
9WiyyWsUM4M9lEKHpZ486lDGk3IM2XQW+pxAoMKb0TYzqCsrduHUtjzy0M0BqgMPe9EtVQqCbnTMzDLXmRONoTYyTV51NQ12mMwEQcDaLQ28e5gqouQJKS81JaoRpQWa7pHsOCki6Fk9TB+EQFrGz5nOrmYYM+O1MKnFkzmVHv7Fh50Sz7d2nYzzOKAkR hpr@docker
hpr_1      | ##################################################################
hpr_1      |
hpr_1      | Configuring ssh config ...
hpr_1      | Starting hpr server ...
hpr_1      |   _
hpr_1      |  | |__  _ __  _ __
hpr_1      |  | '_ \| '_ \| '__|
hpr_1      |  | | | | |_) | |
hpr_1      |  |_| |_| .__/|_|
hpr_1      |        |_|
```

最后从执行命令的输出找到生成的 SSH PUBLIC KEY（两个井号中间的部分，以 `ssh-rsa` 开头，`hpr@docker` 结尾），
复制添加到 gitlab 的账户 SSH Keys 页面中。

部署的部分介绍完毕，下面是具体使用方法。

## 用法

hpr 提供两者方法来管理 git 仓库:

- [Web API](#web-api) (推荐)
- [命令行工具](#cli-tool)

### Web API

执行 hpr 可在本地允许一个 Web API 服务（端口号 `8848`)：

```bash
$ hpr --server
       _
      | |__  _ __  _ __
      | '_ \| '_ \| '__|
      | | | | |_) | |
      |_| |_| .__/|_|
            |_|
I, [2018-03-21 16:44:50 +08:00 #55483]  INFO -- hpr: API Server now listening at localhost:8848, press Ctrl-C to stop
```

#### 显示已镜像的仓库列表

```
GET /repositores
```

##### 参数

| 名称 | 类型 | 是否必须 | 描述 |
|---|---|---|---|
| page | Integer | false | |
| per_page | Integer | false | |

#### 创建镜像仓库

```
POST /repositores
```

##### 参数

| 名称 | 类型 | 是否必须 | 描述 |
|---|---|---|---|
| url | String | true | |
| name | String | false | |


#### 更新镜像仓库

```
PUT /repositores/:name
```

##### 参数

| 名称 | 类型 | 是否必须 | 描述 |
|---|---|---|---|
| name | String | false | |


#### 删除镜像仓库

```
DELETE /repositores/:name
```

##### 参数

| 名称 | 类型 | 是否必须 | 描述 |
|---|---|---|---|
| name | String | false | |


### Cli tool

#### 显示已镜像的仓库列表

```bash
$ hpr -l
```

#### 创建镜像仓库

```bash
$ hpr -c --name hpr-mirror https://github.com/icyleaf/hpr.git
```

#### 更新镜像仓库

```bash
$ hpr -u --name hpr-mirror
```

#### 删除镜像仓库

```bash
$ hpr -d --name hpr-mirror
```

## 本地开发

首先需要安装 [Crystal](https://crystal-lang.org/docs/installation/index.html) 之后执行：

```bash
$ shards install
```

## 贡献你的力量

可能你对本项目是由 Crystal 编写的很陌生，不需要担心如果你熟悉 Ruby 就没有障碍了。

1. [Fork 本项目](https://github.com/icyleaf/hpr/fork)
2. 创建你的新特性/修复分支 (`git checkout -b my-new-feature`)
3. 提交你的代码 (`git commit -am 'Add some feature'`)
4. 推送分支到服务器 (`git push origin my-new-feature`)
5. 创建一个新的 PR

## 项目维护者

- [icyleaf](https://github.com/icyleaf) - 核心开发维护者