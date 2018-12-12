## 鏈上代碼

### 什麼是 chaincode
chaincode（鏈碼）是部署在 Hyperledger fabric 網絡節點上，可被調用與分佈式賬本進行交互的一段程序代碼，也即狹義範疇上的“智能合約”。鏈碼在 VP 節點上的隔離沙盒（目前為 Docker 容器）中執行，並通過 gRPC 協議來被相應的 VP 節點調用和查詢。

Hyperledger 支持多種計算機語言實現的 chaincode，包括 Golang、JavaScript、Java 等。

### 實現 chaincode 接口
下面以 golang 為例來實現 chaincode 的 shim 接口。在這之中三個核心的函數是 **Init**, **Invoke**， 和 **Query**。三個函數都以函數名和字符串結構作為輸入，主要的區別在於三個函數被調用的時機。

#### 依賴包

chaincode 需要引入如下的軟件包。

* `fmt`：包含了 `Println` 等標準函數.
* `errors`：標準 errors 類型包；
* `github.com/hyperledger/fabric/core/chaincode/shim`：與 chaincode 節點交互的接口代碼。shim 包 提供了 `stub.PutState` 與 `stub.GetState` 等方法來寫入和查詢鏈上鍵值對的狀態。

比較重要的 shim 包，通過封裝 gRPC 消息到 VP 節點來完成操作，如：

* PUT_STATE：修改某個狀態（鍵值）的值；
* GET_STATE：獲取某個狀態的值；
* DEL_STATE：刪除某個鍵值；
* RANGE_QUERY_STATE：獲取某個範圍內的鍵值，需要鍵的命名可構成規則的範圍；
* INVOKE_CHAINCODE：調用其它鏈碼方法；
* QUERY_CHAINCODE：查詢同一上下文下的其它鏈碼。

#### Init()函數
當首次部署 chaincode 代碼時，init 函數被調用。如同名字所描述的，該函數用來做一些初始化的工作。

#### Invoke()函數
當通過調用 chaincode 代碼來做一些實際性的工作時，可以使用 invoke 函數。發起的交易將會被鏈上的區塊獲取並記錄。

它以被調用的函數名作為參數，並基於該參數去調用 chaincode 中匹配的的 go 函數。

#### Query()函數
顧名思義，當需要查詢 chaincode 的狀態時，可以調用 `Quer()` 函數。

#### Main() 函數
最後，需要創建一個 `main` 函數，當每個節點部署 chaincode 的實例時，該函數會被調用。

它僅僅在 chaincode 在某節點上註冊時會被調用。


### 與 chaincode 代碼進行交互
與 chaincode 交互的主要方法有 cli 命令行與 rest api，關於 rest api 的使用請查看該目錄下的例子。
