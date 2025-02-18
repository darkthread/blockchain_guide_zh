## Fabric v0.6 安裝部署

如果是初次接觸 Hyperledger Fabric 專案，推薦採用如下的步驟，基於 Docker-Compose 的一鍵部署。

*官方文檔現在也完善了安裝部署的步驟，具體可以參考代碼 `doc` 目錄下內容。*

*動手前，建議適當瞭解一些 [Docker 相關知識](https://github.com/yeasy/docker_practice)。*

### 安裝 Docker

Docker 支持 Linux 常見的發行版，如 Redhat/Centos/Ubuntu 等。

```sh
$ curl -fsSL https://get.docker.com/ | sh
```

以 Ubuntu 14.04 為例，安裝成功後，修改 Docker 服務配置（`/etc/default/docker` 文件）。

```sh
DOCKER_OPTS="$DOCKER_OPTS -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --api-cors-header='*'"
```

重啟 Docker 服務。

```sh
$ sudo service docker restart
```
Ubuntu 16.04 中默認採用了 systemd 管理啟動服務，Docker 配置文件在 `/etc/systemd/system/docker.service.d/override.conf`。

修改後，需要通過如下命令重啟 Docker 服務。

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker.service
```

### 安裝 docker-compose

首先，安裝 python-pip 軟件包。

```sh
$ sudo aptitude install python-pip
```

安裝 docker-compose（推薦為 1.7.0 及以上版本）。

```sh
$ sudo pip install docker-compose>=1.7.0
```

### 下載鏡像

目前 1.0 代碼還沒有正式發佈，推薦使用 v0.6 分支代碼進行測試。

下載相關鏡像，並進行配置。

```sh
$ docker pull yeasy/hyperledger-fabric:0.6-dp \
  && docker pull yeasy/hyperledger-fabric-peer:0.6-dp \
  && docker pull yeasy/hyperledger-fabric-base:0.6-dp \
  && docker pull yeasy/blockchain-explorer:latest \
  && docker tag yeasy/hyperledger-fabric-peer:0.6-dp hyperledger/fabric-peer \
  && docker tag yeasy/hyperledger-fabric-base:0.6-dp hyperledger/fabric-baseimage \
  && docker tag yeasy/hyperledger-fabric:0.6-dp hyperledger/fabric-membersrvc
```

也可以使用 [官方倉庫](https://hub.docker.com/u/hyperledger/) 中的鏡像。

```sh
$ docker pull hyperledger/fabric-peer:x86_64-0.6.1-preview \
  && docker pull hyperledger/fabric-membersrvc:x86_64-0.6.1-preview \
  && docker pull yeasy/blockchain-explorer:latest \
  && docker tag hyperledger/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-peer \
  && docker tag hyperledger/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-baseimage \
  && docker tag hyperledger/fabric-membersrvc:x86_64-0.6.1-preview hyperledger/fabric-membersrvc
```

之後，用戶可以選擇採用不同的一致性機制，包括 noops、pbft 兩類。

### 使用 noops 模式
noops 默認沒有采用 consensus 機制，1 個節點即可，可以用來進行快速測試。

```sh
$ docker run --name=vp0 \
    --restart=unless-stopped \
    -it \
    -p 7050:7050 \
    -p 7051:7051 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e CORE_PEER_ID=vp0 \
    -e CORE_PEER_ADDRESSAUTODETECT=true \
    -e CORE_NOOPS_BLOCK_WAIT=10 \
    hyperledger/fabric-peer:latest peer node start
```

### 使用 PBFT 模式

PBFT 是經典的分散式一致性算法，也是 hyperledger 目前最推薦的算法，該算法至少需要 4 個節點。

首先，下載 Compose 模板文件。

```sh
$ git clone https://github.com/yeasy/docker-compose-files
```

進入 `hyperledger/0.6/pbft` 目錄，查看包括若干模板文件，功能如下。

* `4-peers.yml`: 啟動 4 個 PBFT peer 節點。
* `4-peers-with-membersrvc.yml`: 啟動 4 個 PBFT peer 節點 + 1 個 CA 節點，並啟用 CA 功能。
* `4-peers-with-explorer.yml`: 啟動 4 個 PBFT peer 節點 + 1 個 Blockchain-explorer，可以通過 Web 界面監控集群狀態。
* `4-peers-with-membersrvc-explorer.yml`: 啟動 4 個 PBFT peer 節點 + 1 個 CA 節點 + 1 個 Blockchain-explorer，並啟用 CA 功能。

例如，快速啟動一個 4 個 PBFT 節點的集群。

```sh
$ docker-compose -f 4-peers.yml up
```

### 多物理節點部署

上述方案的典型場景是單物理節點上部署多個 Peer 節點。如果要擴展到多物理節點，需要容器雲平臺的支持，如 Swarm 等。

當然，用戶也可以分別在各個物理節點上通過手動啟動容器的方案來實現跨主機組網，每個物理節點作為一個 peer 節點。

首先，以 4 節點下的 PBFT 模式為例，配置 4 臺互相連通的物理機，分別按照上述步驟配置 Docker，下載鏡像。

4 臺物理機分別命名為 vp0 ~ vp3。

#### vp0

vp0 作為初始的探測節點。

```sh
$ docker run --name=vp0 \
    --net="host" \
    --restart=unless-stopped \
    -it --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e CORE_PEER_ID=vp0 \
    -e CORE_PBFT_GENERAL_N=4 \
    -e CORE_LOGGING_LEVEL=debug \
    -e CORE_PEER_ADDRESSAUTODETECT=true \
    -e CORE_PEER_NETWORKID=dev \
    -e CORE_PEER_VALIDATOR_CONSENSUS_PLUGIN=pbft \
    -e CORE_PBFT_GENERAL_MODE=batch \
    -e CORE_PBFT_GENERAL_TIMEOUT_REQUEST=10s \
    hyperledger/fabric-peer:latest peer node start
```

#### vp1 ~ vp3

以 vp1 為例，假如 vp0 的地址為 10.0.0.1。

```sh
$ NAME=vp1
$ ROOT_NODE=10.0.0.1
$ docker run --name=${NAME} \
    --net="host" \
    --restart=unless-stopped \
    -it --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e CORE_PEER_ID=${NAME} \
    -e CORE_PBFT_GENERAL_N=4 \
    -e CORE_LOGGING_LEVEL=debug \
    -e CORE_PEER_ADDRESSAUTODETECT=true \
    -e CORE_PEER_NETWORKID=dev \
    -e CORE_PEER_VALIDATOR_CONSENSUS_PLUGIN=pbft \
    -e CORE_PBFT_GENERAL_MODE=batch \
    -e CORE_PBFT_GENERAL_TIMEOUT_REQUEST=10s \
    -e CORE_PEER_DISCOVERY_ROOTNODE=${ROOT_NODE}:7051 \
    hyperledger/fabric-peer:latest peer node start
```

### 服務端口
Hyperledger 默認監聽的服務端口包括：

* 7050: REST 服務端口，推薦 NVP 節點開放，0.6 之前版本中為 5000；
* 7051：peer gRPC 服務監聽端口，0.6 之前版本中為 30303；
* 7052：peer CLI 端口，0.6 之前版本中為 30304；
* 7053：peer 事件服務端口，0.6 之前版本中為 31315；
* 7054：eCAP
* 7055：eCAA
* 7056：tCAP
* 7057：tCAA
* 7058：tlsCAP
* 7059：tlsCAA
