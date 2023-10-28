## 頂級項目介紹

Hyperledger 所有項目代碼託管在 [Gerrit](https://gerrit.hyperledger.org) 和 [Github](https://github.com/hyperledger/)上。

目前，主要包括如下頂級項目。

* [Fabric](https://github.com/hyperledger/fabric)：包括 [Fabric](https://github.com/hyperledger/fabric)、[Fabric CA](https://github.com/hyperledger/fabric-ca)、Fabric SDK（包括 Node.Js、Python 和 Java 等語言）等，目標是區塊鏈的基礎核心平臺，支持 PBFT 等新的共識機制，支持權限管理，最早由 IBM 和 DAH 於 2015 年底發起；
* [Sawtooth](https://github.com/hyperledger/sawtooth-core)：包括 arcade、[core](https://github.com/hyperledger/sawtooth-core)、dev-tools、[validator](https://github.com/hyperledger/sawtooth-validator)、mktplace 等。是 Intel 主要發起和貢獻的區塊鏈平臺，支持全新的基於硬件芯片的共識機制 Proof of Elapsed Time（PoET）。 2016 年 4 月貢獻到社區。
* [Blockchain Explorer](https://github.com/hyperledger/blockchain-explorer)：提供 Web 操作界面，通過界面快速查看查詢綁定區塊鏈的狀態（區塊個數、交易歷史）信息等，由 DTCC、IBM、Intel 等開發支持。2016 年 8 月貢獻到社區。
* [Iroha](https://github.com/hyperledger/Iroha)：帳本平臺項目，基於 C++ 實現，帶有不少面向 Web 和 Mobile 的特性，主要由 Soramitsu 於 2016 年 10 月發起和貢獻。
* [Cello](https://github.com/hyperledger/cello)：提供區塊鏈平臺的部署和運行時管理功能。使用 Cello，管理員可以輕鬆部署和管理多條區塊鏈；應用開發者可以無需關心如何搭建和維護區塊鏈，由 IBM 團隊於 2017 年 1 月貢獻到社區。
* [Indy](https://github.com/hyperledger/indy)：提供基於分佈式帳本技術的數位身份管理機制，由 Sovrin 基金會發起，2017 年 3 月底正式貢獻到社區。
* [Composer](https://github.com/hyperledger/composer)：提供面向鏈碼開發的高級語言支持，自動生成鏈碼代碼等，由 IBM 團隊發起並維護，2017 年 3 月底貢獻到社區。
* [Burrow](https://github.com/hyperledger/burrow)：提供以太坊虛擬機的支持，實現支持高效交易的帶權限的區塊鏈平臺，由 Monax 公司發起支持，2017 年 4 月貢獻到社區。
* [Quilt](https://github.com/hyperledger/quilt)：對 W3C 支持的跨帳本協議 Interledger 的 Java 實現。2017 年 10 月正式貢獻到社區。
* [Caliper](https://github.com/hyperledger/burrow)：提供對區塊鏈平臺性能的測試工具，由華為 公司發起支持。2018 年 3 月正式貢獻到社區。

這些頂級項目相互協作，構成了完善的生態系統，如下圖所示。

![Hyperledger 頂級項目](_images/top_projects.png)

所有項目一般都需要經歷提案（Proposal）、孵化（Incubation）、活躍（Active）、退出（Deprecated）、終結（End of Life）等 5 個生命週期。

任何希望加入到 Hyperledger 社區中的項目，必須首先由發起人編寫提案。描述項目的目的、範圍和開發計劃等重要信息，並由技術委員會來進行評審投票，評審通過則可以進入到社區內進行孵化。項目成熟後可以申請進入到活躍狀態，發佈正式的版本，最後從社區中退出結束。

### Fabric 項目

作為最早加入到超級帳本項目中的頂級項目，Fabric 由 IBM、DAH 等企業於 2015 年底提交到社區。項目在 Github 上地址為 https://github.com/hyperledger/fabric。

該項目的定位是面向企業的分佈式帳本平臺，創新的引入了權限管理支持，設計上支持可插拔、可擴展，是首個面向聯盟鏈場景的開源項目。

Fabric 項目基於 Go 語言實現，目前提交次數已經超過 15000 次，核心代碼數超過 15 萬行。

Fabric 項目目前處於活躍狀態，已發佈 1.3.0 版本，同時包括 Fabric CA、Fabric SDK 等多個相關的子項目。

### Sawtooth 項目

![Hyperledger Sawtooth 項目](_images/stl.png)

Sawtooth 項目由 Intel 等企業於 2016 年 4 月提交到社區。核心代碼在 Github 上地址為 https://github.com/hyperledger/sawtooth-core。

該項目的定位也是分佈式帳本平臺，基於 Python 語言實現，目前提交次數已經超過 4000 次。

Sawtooth 項目利用 Intel 芯片的專屬功能，實現了低功耗的 Proof of Elapsed Time（PoET）共識機制，並支持交易族（Transaction Family），方便用戶使用它來快速開發應用。

### Iroha 項目

![Hyperledger Iroha 項目](_images/iroha.png)

Iroha 項目由 Soramitsu 等企業於 2016 年 10 月提交到社區。核心代碼在 Github 上地址為 https://github.com/hyperledger/iroha。

該項目的定位是分佈式帳本平臺框架，基於 C++ 語言實現，目前提交次數已經超過 3000 次。

Iroha 項目在設計上類似 Fabric，同時提供了基於 C++ 的區塊鏈開發環境，並考慮了移動端和 Web 端的一些需求。


### Blockchain Explorer 項目

![Hyperledger Blockchain Explorer 項目](_images/be.png)

Blockchain Explorer 項目由 Intel、DTCC、IBM 等企業於 2016 年 8 月提交到社區。核心代碼在 Github 上地址為 https://github.com/hyperledger/blockchain-explorer。後來更名為 Hyperledger Explorer。

該項目的定位是區塊鏈平臺的瀏覽器，基於 Node.js 語言實現，提供 Web 操作界面。用戶可以使用它來快速查看底層區塊鏈平臺的運行信息，如區塊個數、交易情況、網絡狀況等。

### Cello 項目

![Hyperledger Cello 項目](_images/cello.png)

Cello 項目由筆者領導的技術團隊於 2017 年 1 月貢獻到社區。Github 上倉庫地址為 https://github.com/hyperledger/cello（核心代碼）和 https://github.com/hyperledger/cello-analytics（側重數據分析）。

該項目的定位為區塊鏈管理平臺，同時提供區塊鏈即服務（Blockchain-as-a-Service），實現區塊鏈環境的快速部署，以及對區塊鏈平臺的運行時管理。使用 Cello，可以讓區塊鏈應用人員專注到應用開發，而無需關心底層平臺的管理和維護。

Cello 的主要開發語言為 Python 和 JavaScript 等，底層支持包括裸機、虛擬機、容器雲（包括 Swarm、Kubernetes）等多種基礎架構。

### Indy 項目

Indy 項目由 Sovrin 基金會牽頭進行開發，致力於打造一個基於區塊鏈和分佈式帳本技術的數位中心管理平臺。該平臺支持去中心化，支持跨區塊鏈和跨應用的操作，實現全球化的身份管理。Indy 項目於 2017 年 3 月底正式加入到超級帳本項目。

該項目主要由 Python 語言開發，包括服務節點、客戶端和通用庫等，目前已有超過 1000 次提交。

### Composer 項目

![Hyperledger Composer 項目](_images/composer.png)

Composer 項目由 IBM 團隊於 2017 年 3 月底貢獻到社區，試圖提供一個 Hyperledger Fabric 的開發輔助框架。使用 Composer，開發人員可以使用 Javascript 語言定義應用邏輯，再加上資源、參與者、交易等模型和訪問規則，生成 Hyperledger Fabric 支持的鏈碼。

該項目主要由 NodeJs 語言開發，目前已有超過 4000 次提交。

### Burrow 項目

Burrow 項目由 Monax、Intel 等企業於 2017 年 4 月提交到社區。核心代碼在 Github 上地址為 https://github.com/hyperledger/burrow。

該項目的前身為 eris-db，基於 Go 語言實現，目前提交次數已經超過 2000 次。

Burrow 項目提供了支持以太坊虛擬機的智能合約區塊鏈平臺，並支持 Proof-of-Stake 共識機制（Tendermint）和權限管理，可以提供快速的區塊鏈交易。

### Quilt 項目

Quilt 項目由 NTT、Ripple 等企業於 2017 年 10 月提交到社區。核心代碼在 Github 上地址為 https://github.com/hyperledger/quilt。

Quilt 項目前身為 W3C 支持的 Interledger 協議的 Java 實現，主要試圖為轉帳服務提供跨多個區塊鏈平臺的支持。

### Caliper 項目

Caliper 項目由華為於 2018 年 3 月提交到社區。核心代碼在 Github 上地址為 https://github.com/hyperledger/caliper。

Caliper 項目希望能為評測區塊鏈的性能（包括吞吐、延遲、資源使用率等）提供統一的工具套裝，主要基於 Node.js 語言實現，目前提交次數超過 200 次。
