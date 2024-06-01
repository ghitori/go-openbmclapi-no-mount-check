
# Go OpenBmclAPI-No Mount Check

[![License](https://img.shields.io/github/license/ghitori/go-openbmclapi-no-mount-check)](https://github.com/ghitori/go-openbmclapi-no-mount-check/blob/master/LICENSE)
[![Build Status](https://img.shields.io/github/actions/workflow/status/ghitori/go-openbmclapi-no-mount-check/build-latest.yml)](https://github.com/ghitori/go-openbmclapi-no-mount-check/actions)
[![Downloads](https://img.shields.io/github/downloads/ghitori/go-openbmclapi-no-mount-check/total)](https://github.com/ghitori/go-openbmclapi-no-mount-check/releases)
[![Docker Image Version](https://img.shields.io/docker/v/ghitori/go-openbmclapi-no-mount-check?label=docker)](https://hub.docker.com/repository/docker/ghitori/go-openbmclapi-no-mount-check)
==============

一个去除了 `mount` 存储文件检测的修改版 [Go-OpenBmclAPI](https://github.com/LiterMC/go-openbmclapi)

> 警告！本项目针对 `mount` 存储去除了文件列表完整性检测，这意味着该存储无法同步新文件，**不符合原版及openbmclapi设计准则**，该项目仅为满足个人所需的特殊设计，请勿直接使用！
> 
> #### 无特殊需求请使用原项目！
> 
> ### 无特殊需求请使用原项目！
> 
> ## 无特殊需求请使用原项目！

## 项目目的&使用方法

本项目为 `Alist-Webdav` 的延伸，利用多存储，
将 `Alist(签名关闭)-直链(mount)` 作为数据的下载，
将 `Alist-Webdav(webdav)` 作为文件存储的检查与同步，
并可在中间添加 `nginx` 进行反代与 302 缓存，
使 `go-openbmclapi` 可以将收到的请求直接转发至 `Alist` ，
以此降低在低性能设备 ~~(阿里云轻量)~~ 上较高的 `cpu time` ，
并追求可能存在的性能提升 (未经验证，也可能是性能下降？)

示例存储配置：
```yaml
storages:
  - type: mount   # 添加mount存储
    id: alist-mount
    weight: 100   # 指定权重大于0，作为资源提供服务器
    data:
      path: /tmp   # 指向任意文件夹，因不会触发文件检测与同步，故该条目实际并不生效
      redirect-base: http://example.com:5244/d/openbmclapi  # Alist/nginx反代后的公网直连下载地址，注意关闭Alist全局及存储的的签名验证
      pre-gen-measures: false
  - type: webdav   # 添加webdav存储
    id: alist-webdav
    weight: 0   # 指定权重等于0，作为备选服务器，只提供文件检查与同步
    data:
      max-conn: 32
      max-upload-rate: 0
      max-download-rate: 0
      pre-gen-measures: false
      follow-redirect: false
      redirect-link-cache: 2h0m0s
      alias: alist-user
      endpoint: openbmclapi   # 注意对应mount存储中redirect-base的路径
webdav-users:
  alist-user:
    endpoint: http://localhost:5244/dav/
    username: alist-username
    password: alist-password
```

>**本项目使用方法可能有违 [@bangbang93](https://github.com/bangbang93) 的存储设计原则与原项目的定期文件检查规范，且错误的设置可能导致节点信用下降，并扰乱 `openbmclapi` 整个项目**
>
>**因此除非你真的知道我在说什么，否则请使用[原项目](https://github.com/LiterMC/go-openbmclapi)！**


## 安装方法&配置

与原版几乎一致，详情请参阅 [Go-OpenBmclAPI](https://github.com/LiterMC/go-openbmclapi?tab=readme-ov-file#go-openbmclapi)

修改后的源代码位于 [Release](https://github.com/ghitori/go-openbmclapi-no-mount-check/releases) 下的 
[source.tar.gz](https://github.com/ghitori/go-openbmclapi-no-mount-check/releases/download/latest/source.tar.gz) / 
[source.zip](https://github.com/ghitori/go-openbmclapi-no-mount-check/releases/download/latest/source.zip)

支持使用 [Docker](https://hub.docker.com/repository/docker/ghitori/go-openbmclapi-no-mount-check) 运行 

```bash
docker run -d --name my-go-openbmclapi \
	-e CLUSTER_ID=${CLUSTER_ID} \
	-e CLUSTER_SECRET=${CLUSTER_SECRET} \
	-e CLUSTER_PUBLIC_PORT=${CLUSTER_PUBLIC_PORT} \
	-e CLUSTER_IP=${CLUSTER_IP} \
	-v "${PWD}/cache":/opt/openbmclapi/cache \
	-v "${PWD}/data":/opt/openbmclapi/data \
	-v "${PWD}/logs":/opt/openbmclapi/logs \
	-v "${PWD}/config.yaml":/opt/openbmclapi/config.yaml \
	-p ${CLUSTER_PORT}:4000 \
	ghitori/go-openbmclapi-no-mount-check:latest
```

## 致谢（排名不分先后）

[OpenBMCLAPI-TypeScript](https://github.com/bangbang93/openbmclapi)（[@bangbang93](https://github.com/bangbang93)）

[Go-OpenBMCLAPI](https://github.com/LiterMC/go-openbmclapi)（[@zyxkad](https://github.com/zyxkad)）
