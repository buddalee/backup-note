# Docker 一日初中階學習工作坊 共筆內容

## 講者資訊

### Michael Kuan

I.X （大宏數創意）/ Sr. Principal Software Engineer
Hiiir Inc. Principal Developer
SITCON、COSCUP Speaker

## 工作坊內容

- docker 基本概念說明（名詞說明、lifecycle的說明、架構說明）
- docker 小歷史及基本安裝
- 怎麼寫自己的 dockerfile
- 寫 dockerfile 常會犯的錯誤
- 常見的 repository 介紹
- 最佳化自己的 docker image
- 如何保護自己的 docker image
- 利用 docker 架一個可擴充的服務

透過這次講完大概會對下面的東西有概念
- docker
  - dockerfile
  - seccomp
  - container
  - image
  - network
  - repository
  - runtime (看情況)
- docker swarm
  - service
  - worker
  - manager
  - routing mesh
  - task
- s6 overlay (看情況)
- docker slim
- gitlab-ci

## 簡報
[第一部分](https://slides.com/michael34435/docker_training_p1)
[第二部分](https://slides.com/michael34435/docker_training_p2)
[第三部分](https://slides.com/michael34435/docker_training_p3)
[第四部分 - compose](https://slides.com/michael34435/docker_training_p4_docker-compose#/)
[第四部分 - swarm](https://slides.com/michael34435/docker_training_p4_swarm#/)

## 共筆（共筆共起來QQ）

這是誤解: 輕量化 Virtual Machine、完全獨立環境


### 前言 docker 基本概念說明（名詞說明、lifecycle的說明、架構說明）

![](https://i.imgur.com/0iiVbqD.png)

Docker 的前身：FreeBSD jail release
Linux Container 用 Group 把 XX 分開

2013: Docker 出現(至今 5 年)

Windows Container 較新的，只建立在 window server, 另一個忘了
主要相容 docker ???
只能在 windowns 上使用

Docker 不是輕量化 VM ，是以 LXC 為基礎
docker共享linux核心
dokcer = OS app
把作業系統包成 App

把 app 包成固定環境，隔離 host OS

![](https://i.imgur.com/OmSKz3S.png)
VM 可以自己接硬體資料

libcontainer 由 golang 打造，擺脫LXC依賴

### 解釋名詞: 

- stateful
- stateless：在未設的前提之下，每次建立時會消除舊的資料

Docker 是 stateless 
沒有任何設定之下，是無法在Docker 儲存資料。

- Repository：儲存許多的 package 的地方。

- Image：
    - image layer：儲存做過的行為，產生特殊的cache，本地有 cache 的話，就不需再從 Respository (如 Docker hub) 抓。

- Container：相等於一個一個程式，可以持續執行。
image 會建成 container
視為一個程式，可以持續執行, 要有一個在 foreground 跑的程式

- Network (Docker 子網路)：
    - none：沒有網路
    - bridge：區網
    - overlay：不同網段
    - container: Docker 預設的網路連線, 無法由外部直接進來
    - host：跟本機一樣和 docker contariner ... ??，與本機共用網路介面

- [Volume](https://docs.docker.com/storage/volumes/) 
    - Docker 可以 stateful 的秘密(空間)
    - 和本機的內容一樣

#### 生命週期

![](https://i.imgur.com/eMTFgS3.png)

docker生命週期
1. client呼叫docker daemon
1. docker daemon與host network / host cgroups溝通
1. 判斷docker container是否存在
    - 存在，從local image將layer解開
    - 不存在，從repository拿
1. 建立docker container


### 第一部分 docker 小歷史及基本安裝

#### Windows 安裝

:::danger
完全不建議用 windows
**敦親睦鄰邊緣人不包含 windows 使用者，請叫他節哀順變**
- Hyper-v 才行 (Linux like 環境) (家用版沒有QQ)
- 不要用 WinXP, Win7, Win10 Home
- 還要主機版支援 Hyper-v
:::

#### Mac 安裝

用 [Docker Store](https://store.docker.com/) 的 stable 版，很多不能玩

下載這個：[Docker for Mac Edge](https://docs.docker.com/docker-for-mac/edge-release-notes/)

如果 mac 版本太過老舊，要找那個年份的 docker 版本。

建議使用：
lmgfy: docker edge mac

#### Linux 安裝：
* 不建議用 apt-get, yum 裝
`curl https://get.docker.com | bash`
* 使用安裝腳本裝。
* 沒有 bash 的人，可以參考
* 需要sudo
* Docker 會建議user group:docker, 一般user無法跑，要加到group
* docker ps可以看
```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sh get-docker.sh
```
CentOS 支援但建議使用debian, 更新較快

- `docker ps`
    - 列出 所有活著的容器

docker 屬於 linux kernel，所以都在 linux 環境下執行。

Mac -> Qemu -> Linux -> Docker
Windows -> Hyper-V -> Linux -> Docker

### 第二部分 怎麼寫自己的 dockerfile

先預設觀念: 任何的image預設都是latest
拉下來都是最新版

#### 玩轉docker指令

- `docker ps`
    - 列出 所有活著的容器
    - -a 列出所有的活得死的容器
- `docker logs [container]`

![](https://i.imgur.com/JLLdUkS.png)

- `docker run`
    - 從本地環境的 image 建立 container
    - 如果不存在從 repository 訪問，private repo 必須要有「登入」行為
    - image 的格式可以為 `[IMAGE NAME]:[TAG]`（ex: `node:8`）
    - docker run -idt nginx (ex:run nginx)
:::info
`-d` detach 背景執行
`-i` interactive 可互動
`-t` 提供 tty
`-p` expose socket(ex -p 8080:80) `-p Host Port:Docker Port`
`--restart` 失敗重啟(always-default, none, on-failure)
`--rm` 關閉後 container 自動消滅, 若不刪可能會不斷累積,即使已經死掉了
:::

- `docker exec`
    - 指定已存在、執行中的 container 執行指令
    - ex: docker exec container echo
    - ex: docker exec -it container bash
- `docker build`
    - 從 Dockerfile -> docker image
    - ex: docker build . -t image-name 在目前的資料夾建立image,來源是Dockerfile
    - ex: docker build . -t image-name -f DockerfileName
    - -f 手動指定 Dockerfile 名稱
- `docker push/pull`
    - 把docker image放到repo/從repo拉下來
    - 要登入 docker login

以上就是基本指令囉!!

### 第三部分 寫 dockerfile 常會犯的錯誤
#### 怎麼寫自己的 dockerfile

**[![](https://i.imgur.com/rgriIbc.png)
](https://hub.docker.com)**

- 註冊

![Alpine](https://i.imgur.com/gr5J9sU.png =400x)
才 5M 為 Docker 量身訂做的 輕量化OS
不推薦 docker 預設 image，推薦指定 [Alpine 版本](https://www.alpinelinux.org/)。

![](https://i.imgur.com/GjrIENZ.png =400x)
View Available Tags 點擊後可以看見支援的 OS

沒有螢幕輸出，沒辦法做畫面處理(PDF export之類的功能無法)

[Nginx in alpine](https://store.docker.com/images/nginx)

幾個常見的Dockerfile命令
- FROM：指定基底 image
- RUN：
    - docker build 時候執行，裝一些相依套件。
    - 執行會結束
- ARG（無法在 container 中被使用）
    - container docker build 時的建置參數
- ENV (在docker內部)
    - printenv  容器內使用的環境變數 

檔案複製

- ADD
    - 將local/remote檔案複製到 container(ex. github)
- COPY
    - 將在本地端的檔案複製到容器內

執行的指令

- ENTRYPOINT
    - Container 起來預設會執行的指令
    - 最後指令，啟動時要保持在前景的指令
- CMD
    - container起來預設會執行的指令（或參數）
    - ENTRYPOINT, CMD 同時存在 CMD 當參數使用
- WORKDIR(`cd` + `mkdir`)
    - 初始容器內工作目錄

[在Alpine安裝 套件](http://www.10tiao.com/html/357/201702/2247484888/1.html)
`apk add nginx`


```shell
$ docker login -u [username] repo.path[default docker hub]
``` 

```Dockerfile
# Dockerfile
FROM node:8-alpine

WORKDIR /app ## mkdir app && cd app
COPY . .     ## 複製當下目錄的全部的資料進來 /app

ENTRYPOINT ["node", "app.js"]
```

[koa hello world](https://koajs.com/#introduction)

```javascript=
//app.js
const Koa = require('koa');
const app = new Koa();

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);
```

**Dockerfile -> images**

`docker build . -t [username/image-name]`
`docker build . -t [image-name]` 等一下要下 tag 才可以上傳

**images -> container**

`docker run -idt [image-name]` 

**上傳**

`docker push [username]/[image-name]` 
`docker push [image-name]` 會預設 push 到 `docker.io/library`

`docker tag [image-name] [username]/[image-name]` 
[image-name]≠[username/image-name] 不包含 username 時，要下這一行

- 清掉所有 docker container

    ```
    $ docker rm -f $(docker ps -aq)
    ```
    - q 只列 id
    - a 列全部

- 刪掉 image

```
$ docker rmi -f $(docker images -aq)
```
- 看 docker top
    - docker top [container id]
- 看 docker 是否正常啟用
```
$ docker ps -a
```

寫dockerfile常犯的錯誤
介紹常見的repository
發佈你的docker image

> ## 日安的 docker 實作
> 
> 1. 安裝 docker
> 2. 新建 repo
> ### 看是否安裝成功（`$ docker ps` ）
> ```
> $ docker login -u [usernamw] # 登入 Docker hub
> $ mkdir project/docker-test # 新建資料夾
> $ cd project/docker-test # 進到資料夾
> $ vi Dockerfile # 新增進入 dockerfile （要寫好之後出來，請看上面）
> $ npm init # 建立執行環境
> $ npm install koa -D # 建立執行環境
> $ vi app.js # 新增進入 app.js （要寫好之後出來，請看上面）
> $ docker build . -t [username]/[repoName] #[username]/[repoName] 
>     新建 image 檔案，名字是Docker hub 的 ID
>  
> $ docker run -idt -p [本機 port 號]:[container port 號] [image]  
>    讓 container 跑起來。
>    login 之後 docker hub 會幫你抓相對應環境的 image，如果是 node 環境，imagename 就是 node。
>    可以用本機 port 號看是否有成功跑起來
> $ docker push [username]/[repoName] 
> ```

### 第五部分 常見的 repository 介紹

prive repository

registry.gitlab
hub.docker
goharbor.io


### 第六部分 最佳化自己的 docker image
#### 最佳化你的Docker Image

#### 1. size optimized

用 alpine based image
alpine 為 基底 OS

**用 Docker Slim 的優點**
[docker-slim](http://dockersl.im/)
- 不會改變Docker image的內容
- 壓縮比30

`docker-slim build --http-probe....`
[示範影片](https://asciinema.org/a/czxJahFcUHWCGKVWzT7LDpVdE)
[![asciicast](https://asciinema.org/a/czxJahFcUHWCGKVWzT7LDpVdE.svg)](https://asciinema.org/a/czxJahFcUHWCGKVWzT7LDpVdE)

監聽所有的 syscall 並打包到 image
可以編輯 seccomp.json 去限制 syscall

**不適合使用 Docker Slim**
- stateful volume`-v`
- 使用alpine的前提下

#### 2. resources optimized

**對 image 限制**

預防建立過多 container，導致電腦資源不足，可以下以下參數：
- cpus
- cpu-period (指定容器對CPU的使用要在多長時間內做一次重新分配)
- cpu-quota (限制%)
- memory
- memory-swap

**新的問題**
1. 把 OS 當 App
2. container 一次只執行一個指令

**複習docker的特性**

- 將OS做為Application化
- 所有container只能允許一個指令執行
- 預設權限都會是root

```shell
$ docker -v -idt -v /etc:/test alpine
```

`-v` 是把實體機器的檔案放進 docker
那麼這部份就用 root 執行。危險!!!


#### 3. [s6-overlay](https://github.com/just-containers/s6-overlay)

run as daemon, 負責啟動和監視 service (ex. nginx, mysql)
類似supervisord
一個常駐型daemon thread
擁有一系列unix liked系統的工具

### 第七部分 如何保護自己的 docker image
#### Docker 商業上的好處

- 方便部署
- 可攜性(可移植性)

#### 缺點

code 會被看光(不安全)

- [Falco](https://github.com/falcosecurity/falco/wiki)
    - [Falco Alerts](https://github.com/falcosecurity/falco/wiki/Falco-Alerts)

Falco: 防君子不防小人（有其他人在看的警報器）
- 無法防止直接拆image
- 由sysdig所開發
- 是系統內常駐daemon
- docker檢測工具
    
[更多相關設定](https://juejin.im/entry/5be163b6e51d451c6a14bff5) - 掘金的文章

seccomp
- Linux上的安全元件
- 可以限制呼叫指令

### 保護你的docker image

### 第八部分 利用 docker 架一個可擴充的服務

#### [docker-compose](https://slides.com/michael34435/docker_training_p4_docker-compose#/)

單機

#### 什麼是docker-compose

一個可執行檔

Docker for Mac 預設
python 所寫成的工具
描述檔由 yaml 定義

yaml 有兩個 key
 - version （版本要字串）
 - services 
     - key -> DNS name(hostname)


docker-compose.yml
docker-compose.override.yml
[Exmaple](https://github.com/CapsLock-Studio/Tickets)
目前為止版本為3.7
設定檔預設名稱為 docker-compose.yml
依賴 docker 版本

docker-compose up -d --build --force-recreate
docekr-compose down
docker-compose ps/logs


#### [docker-swarm](https://slides.com/michael34435/docker_training_p4_swarm#/)

- 看成 kubernetes 的分身
- 落地方便（kubernetes 準備工作很多）
- 高可用性[HA](https://zh.wikipedia.org/wiki/%E9%AB%98%E5%8F%AF%E7%94%A8%E6%80%A7)
- 價格便宜

**組成——Node**
- 分兩個節點。
    - Manager：
        - 分派任務給 worker node 
        - 同時也是 worker node
        - 控制整個 Docker Swarm 的部署
        - 主控制節點
        - 如果死掉，整個 docker swarm 就死掉了
        - 可以有多台 manager，但只有一台 &*$%^^#???
    - worker：
        - 被分派任務
        - 不會主動分派任務
        - 不知道其他 worker node
- 可以隨時 rolling update。
- 在所有 node 底下：
    - service：
        - 一個Service包含一種container
        - Service啟動不代表container啟動
        - 可以對container進行health check
        - Service可以啟動複數個container(replicas)
    - Task
        - 主要是由Service帶起來的
        - 代表在docker container內執行的指令
        - task會在node上到執行結束為止
