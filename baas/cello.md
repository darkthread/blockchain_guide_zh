﻿## 使用超級帳本 Cello 搭建區塊鏈服務

從前面的講解中可以看到，區塊鏈服務平臺能夠有效加速對區塊鏈技術的應用，解決企業和開發者進行手動運營管理的負擔。但是這些方案都是商業用途，並且只能在線使用。

![Cello 典型應用場景](_images/cello.png)

超級帳本的 Cello 項目為本地搭建區塊鏈服務管理平臺提供了開源的解決方案，可以實現在多種類型的物理資源上實現區塊鏈網路的生命週期管理。

正如 Cello 的名字所蘊意，它就像一把精巧的大提琴，以區塊鏈為琴絃，可以奏出更加動人的樂章。

### 基本架構和特性

Cello 項目由筆者領導的 IBM 技術團隊於 2017 年 1 月貢獻到超級帳本社區，主要基於 Python 和 Javascript 語言編寫。該項目的定位為區塊鏈管理平臺，支持部署、運行時管理和數據分析等功能，可以實現一套完整的 BaaS 系統的快速搭建。其基本架構如下圖所示。

![Cello 基本架構](_images/cello_arch.png)

在實現區塊鏈環境快速部署的同時，Cello 也提供了不少對區塊鏈平臺進行運行時管理的特性，這些特性總結如下。

* 管理區塊鏈的全生命週期，包括創建、配置、使用、健康檢查、刪除等。
* 支持多種基礎架構作為底層資源池，包括裸機、虛擬機、容器雲（Docker、Swarm、Kubernetes）等。
* 支持多種區塊鏈平臺及自定義配置（目前以支持超級帳本 Fabric 為主）。
* 支持監控和分析功能，實現對區塊鏈網路和智能合約的運行狀況分析。
* 提供可插拔的框架設計，包括區塊鏈平臺、資源調度、監控、驅動代理等都很容易引入第三方實現。

下面具體介紹如何以 Docker 主機為資源池，用 Cello 快速搭建一個區塊鏈服務平臺。

### 環境準備

Cello 採用了典型的主從（Master-Worker）架構。用戶可以自行準備一個 Master 物理節點和若干個 Worker 節點。

其中，Master 節點負責管理（例如，創建和刪除）Worker 節點中的區塊鏈集群，其通過 8080 端口對外提供網頁 Dashboard，通過 80 端口對外提供 RESTful API。Worker 節點負責提供區塊鏈集群的物理資源，例如基於 Docker 主機或 Swarm 的方式啟動多個集群，作為提供給用戶可選的多個區塊鏈網路環境。

下圖中展示了一個典型的 Master-Worker 部署拓撲。每個節點默認為 Linux（如 Ubuntu 16.04）服務器或虛擬機。

![Cello 部署拓撲示例](_images/cello_deployment_topo.png)

為了支持區塊鏈網路，Worker 和 Master 節點需要配備足夠的物理資源。例如，如果希望在一個 Worker 節點上能夠啟動至少 10 個區塊鏈集群，則建議節點配置至少為 8 CPU、16G 內存、100G 硬盤容量。

### 下載 Cello 源碼

Cello 代碼的官方倉庫在社區的 gerrit 上，並實時同步到 Github 倉庫中，讀者可以從任一倉庫中獲取代碼。例如通過如下命令從官方倉庫下載 Cello 源碼。

```sh
$ git clone http://gerrit.hyperledger.org/r/cello && cd cello
```


### 配置 Worker 節點

#### 安裝和配置 Docker 服務

首先安裝 Docker，推薦使用 1.12 或者更新的版本。可通過如下命令快速安裝 Docker。

```sh
$ curl -fsSL https://get.docker.com/ | sh
```
安裝成功後，修改 Docker 服務配置。對於 Ubuntu 16.04，更新 `/lib/systemd/system/docker.service` 文件如下。

```sh
[Service]
DOCKER_OPTS="$DOCKER_OPTS -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --api-cors-header='*' --default-ulimit=nofile=8192:16384 --default-ulimit=nproc=8192:16384"
EnvironmentFile=-/etc/default/docker
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// $DOCKER_OPTS
```

修改後，需要通過如下命令重啟 Docker 服務。

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker.service
```


#### 下載 Docker 鏡像

對於超級帳本 Fabric v1.0 集群所需的鏡像，可以使用如下命令進行自動下載。

```sh
$ cd scripts/worker_node_setup && bash download_images.sh
```

#### 防火牆配置

為了確保 Worker 上的容器可以正常訪問，通過如下命令確保主機開啟 IP 轉發。

```sh
$ sysctl -w net.ipv4.ip_forward=1
```

同時檢查主機的 iptables 設置，確保必要的端口被打開（如 2375、7050~10000 等）。

### 配置 Master 節點

#### 下載 Docker 鏡像

使用如下命令下載運行服務所必要的 Docker 鏡像。

其中，python:3.5 鏡像是運行 Cello 核心組件的基礎鏡像；mongo:3.2 提供了數據庫服務；yeasy/nginx:latest 提供了 Nginx 轉發功能；mongo-express:0.30 鏡像是為了調試數據庫，可以選擇性安裝。

```sh
$ docker pull python:3.5 \
	&& docker pull mongo:3.2 \
	&& docker pull yeasy/nginx:latest \
	&& docker pull mongo-express:0.30
```

#### 安裝 Cello 服務

首次運行時，可以通過如下命令對 Master 節點進行快速配置，包括安裝 Docker 環境、創建本地數據庫目錄、安裝依賴軟件包等。

```sh
$ make setup
```

如果安裝過程沒有提示出現問題，則說明當前環境滿足了運行條件。如果出現問題，可通過查看日誌信息進行定位。

#### 管理 Cello 服務

可以通過運行如下命令來快速啟動 Cello 相關的組件服務（包括 dashboard、restserver、watchdog、mongo、nginx 等）。

```sh
$ make start
```

類似地，運行 `make stop` 或 `make restart` 可以停止或重啟全部服務。

若希望重新部署某個特定服務（如 dashboard），可運行如下命令。

```sh
$ make redeploy service=dashboard
```

運行如下命令可以實時查看所有服務的日誌信息。

```sh
$ make logs
```

若希望查看某個特定服務的日誌，可運行如下命令進行過濾，如只查看 watchdog 組件的日誌。

```sh
$ make log service=watchdog
```

### 使用 Cello 管理區塊鏈

Cello 服務啟動後，管理員可以通過 Cello 的 Dashboard 頁面管理區塊鏈。

默認情況下，可通過 Master 節點的 8080 端口訪問 Dashboard。默認的登錄用戶名和密碼為 `admin:pass`。

![Cello Dashboard](_images/cello_dashboard.png)

如圖，Dashboard 有多個頁面，各頁面的功能如下。

| 頁面 |  功能 |
| --- |  --- |
| Overview |  展示系統整體狀態 |
| System Status |  展示一些統計信息 |
| Hosts |  管理所有主機（Worker 節點） |
| Active Chains |  管理資源池中的所有鏈 |
| Inused Chains |  管理正在被用戶佔用的鏈 |
| Released History  | 查看鏈的釋放歷史 |

#### Hosts 頁面

在 Hosts 頁面，管理員可以管理所有資源池中已存在的主機，或添加新主機。表格中會顯示主機的類型、狀態、正在運行的區塊鏈數量、區塊鏈數量上限等。所有設定為 non-schedulable (不會自動分配給用戶）的主機會用灰色背景標識，如下圖所示。

![Hosts 頁面](_images/cello_dashboard_hosts.png)

點擊一個主機的 Action 下拉菜單，有如下選項可供操作該主機。

* Fillup：將主機運行的區塊鏈數添加至上限。
* Clean：清理主機中所有未被用戶佔用的鏈。
* Config：更改主機配置，如名稱和鏈數量上限。
* Reset：重置該主機，只有當該主機沒有用戶佔用的鏈時可以使用。
* Delete：從資源池中刪除該主機。

點擊 Hosts 頁面的 Add Host 按鈕，可以向資源池中添加主機。需要設定該主機的名稱、Daemon URL 地址（例如，Worker 節點的 docker daemon 監聽地址和端口）、鏈數量上限、日誌配置、是否啟動區塊鏈至數量上限、是否可向用戶自動分配，如下圖所示。

![添加主機](_images/cello_dashboard_addhost.png)

#### Active Chains 頁面

Active Chains 頁面會顯示所有正在運行的鏈，包括鏈的名稱、類型、狀態、健康狀況、規模、所屬主機等信息。正在被用戶佔用的鏈會用灰色背景標識，如下圖所示。

![Active Chains 頁面](_images/cello_dashboard_activechains.png)

點擊一條鏈的 Actions 下拉菜單，有如下選項可供操作該鏈。

* Start：如果這條鏈處於停止狀態，則啟動。
* Stop：停止運行中的鏈。
* Restart：重新啟動這條鏈。
* Delete：刪除這條鏈。
* Release：將佔用的鏈釋放，隨後會被刪除。

點擊 Active Chains 頁面的 Add Chain 按鈕，可以向資源池中添加更多鏈（如果還有未被佔滿的主機），如下圖所示。

![添加鏈](_images/cello_dashboard_addcluster.png)

### 用戶控制檯，申請使用Chain

用戶可以登錄User Dashboard來申請和使用Chain

![登錄頁面](_images/cello_user_dashboard_login.png)

#### Chain列表頁面

Chain列表頁面顯示所有用戶已經申請的鏈。

![Chain列表頁面](_images/cello_user_dashboard_chain_list.png)

#### Chain詳情頁面

Chain詳情頁面可以查看鏈的基本信息（鏈高度，channel數，鏈碼安裝/實例化個數，最近的block/transaction），操作歷史記錄。

![Chain詳情頁面](_images/cello_user_dashboard_chain_info.png)

#### 智能合約模板列表頁面

這個頁面列取用戶自己上傳的智能合約代碼模板，支持多個版本管理。

![智能合約模板列取頁面](_images/cello_user_dashboard_chain_code_template.png)

#### 智能合約模板詳情頁面

在合約模板詳情頁面可以查看智能合約模板的詳情，包括合約多版本列表，部署列表，部署合約。

![智能合約詳情頁面](_images/cello_user_dashboard_chain_code_template_info.png)

#### 智能合約操作頁面

在這個頁面可以invoke/query已經部署好的智能合約。

![智能合約操作頁面](_images/cello_user_dashboard_chain_code_operate.png)

#### 智能合約運行列表頁面

這個頁面可以查看所有已經部署，包括成功和失敗的智能合約的列表。

![智能合約運行列表頁面](_images/cello_user_dashboard_chain_code_running.png)

### 基於 Cello 進行功能擴展
Cello 已經提供了完整的區塊鏈管理功能，並提供了圖形界面和 API。

用戶可以通過向 Cello 的 Master 節點（默認為 80 端口）發送 RESTful API 來申請、釋放區塊鏈，或查看區塊鏈相關信息，如其對外開放的接口，可供用戶進行遠程交互。RESTful API 的說明可在 Cello 的文檔中查閱。

對於區塊鏈服務提供者，可以利用這些 API 為用戶呈現友好的區塊鏈申請和操作界面，在 Cello 的基礎之上構建和實現更多功能。