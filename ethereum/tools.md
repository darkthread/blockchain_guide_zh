## 相關工具

### 客戶端和開發庫

以太坊客戶端可用於接入以太坊網絡，進行帳戶管理、交易、挖礦、智能合約等各方面操作。

以太坊社區現在提供了多種語言實現的客戶端和開發庫，支持標準的 JSON-RPC 協議。用戶可根據自己熟悉的開發語言進行選擇。

* [go-ethereum](https://github.com/ethereum/go-ethereum)：Go 語言實現；
* [Parity](https://github.com/ethcore/parity)：Rust 語言實現；
* [cpp-ethereum](https://github.com/bobsummerwill/cpp-ethereum)：C++ 語言實現；
* [ethereumjs-lib](https://github.com/ethereumjs/ethereumjs-lib)：javascript 語言實現；
* [Ethereum(J)](https://github.com/ethereum/ethereumj)：Java 語言實現；
* [ethereumH](https://github.com/blockapps/ethereumH)：Haskell 語言實現；
* [pyethapp](https://github.com/ethereum/pyethapp)：Python 語言實現；
* [ruby-ethereum](https://github.com/janx/ruby-ethereum)：Ruby 語言實現。

#### Geth

上述實現中，go-ethereum 的獨立客戶端 Geth 是最常用的以太坊客戶端之一。

用戶可通過安裝 Geth 來接入以太坊網絡併成為一個完整節點。Geth 也可作為一個 HTTP-RPC 服務器，對外暴露 JSON-RPC 接口，供用戶與以太坊網絡交互。

Geth 的使用需要基本的命令行基礎，其功能相對完整，源碼託管於 github.com/ethereum/go-ethereum。

### 以太坊錢包

對於只需進行帳戶管理、以太坊轉帳、DApp 使用等基本操作的用戶，則可選擇直觀易用的錢包客戶端。

Mist 是官方提供的一套包含圖形界面的錢包客戶端，除了可用於進行交易，也支持直接編寫和部署智能合約。

![Mist 瀏覽器](_images/mist.png)

所編寫的代碼編譯發佈後，可以部署到區塊鏈上。使用者可通過發送調用相應合約方法的交易，來執行智能合約。

### IDE

對於開發者，以太坊社區湧現出許多服務於編寫智能合約和 DApp 的 IDE，例如：

* [Truffle](http://truffleframework.com/)：一個功能豐富的以太坊應用開發環境。
* [Embark](https://github.com/iurimatias/embark-framework)：一個 DApp 開發框架，支持集成以太坊、IPFS 等。
* [Remix](http://remix.ethereum.org)：一個用於編寫 Solidity 的 IDE，內置調試器和測試環境。

### 網站資源

已有一些網站提供對以太坊網絡的數據、運行在以太坊上的 DApp 等信息進行查看，例如：

* ethstats.net：實時查看網絡的信息，如區塊、價格、交易數等。
* ethernodes.org：顯示整個網絡的歷史統計信息，如客戶端的分佈情況等。
* dapps.ethercasts.com：查看運行在以太坊上的 DApp 的信息，包括簡介、所處階段和狀態等。

![以太坊網絡上的 Dapp 信息](_images/dapps.png)