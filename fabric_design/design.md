## 架構設計

整個功能架構如下圖所示。

![](_images/refarch.png)

包括三大組件：區塊鏈服務（Blockchain）、鏈碼服務（Chaincode）、成員權限管理（Membership）。

### 概念術語

* Auditability（審計性）：在一定權限和許可下，可以對鏈上的交易進行審計和檢查。
* Block（區塊）：代表一批得到確認的交易信息的整體，準備被共識加入到區塊鏈中。
* Blockchain（區塊鏈）：由多個區塊鏈接而成的鏈表結構，除了首個區塊，每個區塊都包括前繼區塊內容的 hash 值。
* Certificate Authority（CA）：負責身份權限管理，又叫 Member Service 或 Identity Service。
* Chaincode（鏈上代碼或鏈碼）：區塊鏈上的應用代碼，擴展自“智能合約”概念，支持 golang、nodejs 等，運行在隔離的容器環境中。
* Committer（提交節點）：1.0 架構中一種 peer 節點角色，負責對 orderer 排序後的交易進行檢查，選擇合法的交易執行並寫入存儲。
* Confidentiality（保密）：只有交易相關方可以看到交易內容，其它人未經授權則無法看到。
* Endorser（背書節點）：1.0 架構中一種 peer 節點角色，負責檢驗某個交易是否合法，是否願意為之背書、簽名。
* Enrollment Certificate Authority（ECA，註冊 CA）：負責成員身份相關證書管理的 CA。
* Ledger（帳本）：包括區塊鏈結構（帶有所有的可驗證交易信息，但只有最終成功的交易會改變世界觀）和當前的世界觀（world state）。Ledger 僅存在於 Peer 節點。
* MSP（Member Service Provider，成員服務提供者）：成員服務的抽象訪問接口，實現對不同成員服務的可拔插支持。
* Non-validating Peer（非驗證節點）：不參與帳本維護，僅作為交易代理響應客戶端的 REST 請求，並對交易進行一些基本的有效性檢查，之後轉發給驗證節點。
* Orderer（排序節點）：1.0 架構中的共識服務角色，負責排序看到的交易，提供全局確認的順序。
* Permissioned Ledger（帶權限的帳本）：網路中所有節點必須是經過許可的，非許可過的節點則無法加入網路。
* Privacy（隱私保護）：交易員可以隱藏交易的身份，其它成員在無特殊權限的情況下，只能對交易進行驗證，而無法獲知身份信息。
* Transaction（交易）：執行帳本上的某個函數調用。具體函數在 chaincode 中實現。
* Transactor（交易者）：發起交易調用的客戶端。
* Transaction Certificate Authority（TCA，交易 CA）：負責維護交易相關證書管理的 CA。
* Validating Peer（驗證節點）：維護帳本的核心節點，參與一致性維護、對交易的驗證和執行。
* World State（世界觀）：是一個鍵值數據庫，chaincode 用它來存儲交易相關的狀態。

### 區塊鏈服務

區塊鏈服務提供一個分散式帳本平臺。一般地，多個交易被打包進區塊中，多個區塊構成一條區塊鏈。區塊鏈代表的是帳本狀態機發生變更的歷史過程。

#### 交易

交易意味著圍繞著某個鏈碼進行操作。

交易可以改變世界狀態。

交易中包括的內容主要有：

* 交易類型：目前包括 Deploy、Invoke、Query、Terminate 四種；
* uuid：代表交易的唯一編號；
* 鏈碼編號 chaincodeID：交易針對的鏈碼；
* 負載內容的 hash 值：Deploy 或 Invoke 時候可以指定負載內容；
* 交易的保密等級 ConfidentialityLevel；
* 交易相關的 metadata 信息；
* 臨時生成值 nonce：跟安全機制相關；
* 交易者的證書信息 cert；
* 簽名信息 signature；
* metadata 信息；
* 時間戳 timestamp。

交易的數據結構（Protobuf 格式）定義為

```protobuf
message Transaction {
    enum Type {
        UNDEFINED = 0;
        // deploy a chaincode to the network and call `Init` function
        CHAINCODE_DEPLOY = 1;
        // call a chaincode `Invoke` function as a transaction
        CHAINCODE_INVOKE = 2;
        // call a chaincode `query` function
        CHAINCODE_QUERY = 3;
        // terminate a chaincode; not implemented yet
        CHAINCODE_TERMINATE = 4;
    }
    Type type = 1;
    //store ChaincodeID as bytes so its encrypted value can be stored
    bytes chaincodeID = 2;
    bytes payload = 3;
    bytes metadata = 4;
    string uuid = 5;
    google.protobuf.Timestamp timestamp = 6;

    ConfidentialityLevel confidentialityLevel = 7;
    string confidentialityProtocolVersion = 8;
    bytes nonce = 9;

    bytes toValidators = 10;
    bytes cert = 11;
    bytes signature = 12;
}
```

在 1.0 架構中，一個 transaction 包括如下信息：

[ledger] [channel], **proposal:**[chaincode, <function name, arguments>] **endorsement:**[proposal hash, simulation result, signature]

* endorsements: proposal hash, simulation result, signature
* function-spec: function name, arguments
* proposal: [channel,] chaincode, <function-spec>

#### 區塊

區塊打包交易，確認交易後的世界狀態。

一個區塊中包括的內容主要有：

* 版本號 version：協議的版本信息；
* 時間戳 timestamp：由區塊提議者設定；
* 交易信息的默克爾樹的根 hash 值：由區塊所包括的交易構成；
* 世界觀的默克爾樹的根 hash 值：由交易發生後整個世界的狀態值構成；
* 前一個區塊的 hash 值：構成鏈所必須；
* 共識相關的元數據：可選值；
* 非 hash 數據：不參與 hash 過程，各個 peer 上的值可能不同，例如本地提交時間、交易處理的返回值等；

_注意具體的交易信息並不存放在區塊中。_

區塊的數據結構（Protobuf 格式）定義為

```protobuf
message Block {
    uint32 version = 1;
    google.protobuf.Timestamp timestamp = 2;
    repeated Transaction transactions = 3;
    bytes stateHash = 4;
    bytes previousBlockHash = 5;
    bytes consensusMetadata = 6;
    NonHashData nonHashData = 7;
}
```

一個真實的區塊內容示例：

```json
{
    "nonHashData": {
        "localLedgerCommitTimestamp": {
            "nanos": 975295157,
                "seconds": 1466057539
        },
            "transactionResults": [
            {
                "uuid": "7be1529ee16969baf9f3156247a0ee8e7eee99a6a0a816776acff65e6e1def71249f4cb1cad5e0f0b60b25dd2a6975efb282741c0e1ecc53fa8c10a9aaa31137"
            }
            ]
    },
        "previousBlockHash": "RrndKwuojRMjOz/rdD7rJD/NUupiuBuCtQwnZG7Vdi/XXcTd2MDyAMsFAZ1ntZL2/IIcSUeatIZAKS6ss7fEvg==",
        "stateHash": "TiIwROg48Z4xXFFIPEunNpavMxnvmZKg+yFxKK3VBY0zqiK3L0QQ5ILIV85iy7U+EiVhwEbkBb1Kb7w1ddqU5g==",
        "transactions": [
        {
            "chaincodeID": "CkdnaXRodWIuY29tL2h5cGVybGVkZ2VyL2ZhYnJpYy9leGFtcGxlcy9jaGFpbmNvZGUvZ28vY2hhaW5jb2RlX2V4YW1wbGUwMhKAATdiZTE1MjllZTE2OTY5YmFmOWYzMTU2MjQ3YTBlZThlN2VlZTk5YTZhMGE4MTY3NzZhY2ZmNjVlNmUxZGVmNzEyNDlmNGNiMWNhZDVlMGYwYjYwYjI1ZGQyYTY5NzVlZmIyODI3NDFjMGUxZWNjNTNmYThjMTBhOWFhYTMxMTM3",
            "payload": "Cu0BCAESzAEKR2dpdGh1Yi5jb20vaHlwZXJsZWRnZXIvZmFicmljL2V4YW1wbGVzL2NoYWluY29kZS9nby9jaGFpbmNvZGVfZXhhbXBsZTAyEoABN2JlMTUyOWVlMTY5NjliYWY5ZjMxNTYyNDdhMGVlOGU3ZWVlOTlhNmEwYTgxNjc3NmFjZmY2NWU2ZTFkZWY3MTI0OWY0Y2IxY2FkNWUwZjBiNjBiMjVkZDJhNjk3NWVmYjI4Mjc0MWMwZTFlY2M1M2ZhOGMxMGE5YWFhMzExMzcaGgoEaW5pdBIBYRIFMTAwMDASAWISBTIwMDAw",
            "timestamp": {
                "nanos": 298275779,
                "seconds": 1466057529
            },
            "type": 1,
            "uuid": "7be1529ee16969baf9f3156247a0ee8e7eee99a6a0a816776acff65e6e1def71249f4cb1cad5e0f0b60b25dd2a6975efb282741c0e1ecc53fa8c10a9aaa31137"
        }
    ]
}
```

#### 世界觀
世界觀用於存放鏈碼執行過程中涉及到的狀態變量，是一個鍵值數據庫。典型的元素為 `[chaincodeID, ckey]: value` 結構。

為了方便計算變更後的 hash 值，一般採用默克爾樹數據結構進行存儲。樹的結構由兩個參數（`numBuckets` 和 `maxGroupingAtEachLevel`）來進行初始配置，並由 `hashFunction` 配置決定存放鍵值到葉子節點的方式。顯然，各個節點必須保持相同的配置，並且啟動後一般不建議變動。

* `numBuckets`：葉子節點的個數，每個葉子節點是一個桶（bucket），所有的鍵值被 `hashFunction` 散列分散到各個桶，決定樹的寬度；
* `maxGroupingAtEachLevel`：決定每個節點由多少個子節點的 hash 值構成，決定樹的深度。

其中，桶的內容由它所保存到鍵值先按照 chaincodeID 聚合，再按照升序方式組成。

一般地，假設某桶中包括 $$ M $$ 個 chaincodeID，對於 $$ chaincodeID_i $$，假設其包括 $$ N $$ 個鍵值對，則聚合 $$G_i$$ 內容可以計算為：

$$ G_i = Len(chaincodeID_i) + chaincodeID_i + N + \sum_{1}^{N} {len(key_j) + key_j + len(value_j) + value_j} $$

該桶的內容則為

$$ bucket = \sum_{1}^{M} G_i $$

*注：這裡的 `+` 代表字符串拼接，並非數學運算。*

### 鏈碼服務

鏈碼包含所有的處理邏輯，並對外提供接口，外部通過調用鏈碼接口來改變世界觀。

#### 接口和操作
鏈碼需要實現 Chaincode 接口，以被 VP 節點調用。

```go
type Chaincode interface { Init(stub *ChaincodeStub, function string, args []string) ([]byte, error) Invoke(stub *ChaincodeStub, function string, args []string) ([]byte, error) Query(stub *ChaincodeStub, function string, args []string) ([]byte, error)}
```

鏈碼目前支持的交易類型包括：部署（Deploy）、調用（Invoke）和查詢（Query）。

* 部署：VP 節點利用鏈碼創建沙盒，沙盒啟動後，處理 protobuf 協議的 shim 層一次性發送包含 ChaincodeID 信息的 REGISTER 消息給 VP 節點，進行註冊，註冊完成後，VP 節點通過 gRPC 傳遞參數並調用鏈碼 Init 函數完成初始化；
* 調用：VP 節點發送 TRANSACTION 消息給鏈碼沙盒的 shim 層，shim 層用傳過來的參數調用鏈碼的 Invoke 函數完成調用；
* 查詢：VP 節點發送 QUERY 消息給鏈碼沙盒的 shim 層，shim 層用傳過來的參數調用鏈碼的 Query 函數完成查詢。

不同鏈碼之間可能互相調用和查詢。

#### 容器

在實現上，鏈碼需要運行在隔離的容器中，超級帳本採用了 Docker 作為默認容器。

對容器的操作支持三種方法：build、start、stop，對應的接口為 VM。

```go
type VM interface { 
  build(ctxt context.Context, id string, args []string, env []string, attachstdin bool, attachstdout bool, reader io.Reader) error 
  start(ctxt context.Context, id string, args []string, env []string, attachstdin bool, attachstdout bool) error 
  stop(ctxt context.Context, id string, timeout uint, dontkill bool, dontremove bool) error 
}
```
鏈碼部署成功後，會創建連接到部署它的 VP 節點的 gRPC 通道，以接受後續 Invoke 或 Query 指令。


#### gRPC 消息
VP 節點和容器之間通過 gRPC 消息來交互。消息基本結構為

```protobuf
message ChaincodeMessage {

 enum Type { UNDEFINED = 0; REGISTER = 1; REGISTERED = 2; INIT = 3; READY = 4; TRANSACTION = 5; COMPLETED = 6; ERROR = 7; GET_STATE = 8; PUT_STATE = 9; DEL_STATE = 10; INVOKE_CHAINCODE = 11; INVOKE_QUERY = 12; RESPONSE = 13; QUERY = 14; QUERY_COMPLETED = 15; QUERY_ERROR = 16; RANGE_QUERY_STATE = 17; }

 Type type = 1; google.protobuf.Timestamp timestamp = 2; bytes payload = 3; string uuid = 4;}
```

當發生鏈碼部署時，容器啟動後發送 `REGISTER` 消息到 VP 節點。如果成功，VP 節點返回 `REGISTERED` 消息，併發送 `INIT` 消息到容器，調用鏈碼中的 Init 方法。

當發生鏈碼調用時，VP 節點發送 `TRANSACTION` 消息到容器，調用其 Invoke 方法。如果成功，容器會返回 `RESPONSE` 消息。

類似的，當發生鏈碼查詢時，VP 節點發送 `QUERY` 消息到容器，調用其 Query 方法。如果成功，容器會返回 `RESPONSE` 消息。

### 成員權限管理

通過基於 PKI 的成員權限管理，平臺可以對接入的節點和客戶端的能力進行限制。

證書有三種，Enrollment，Transaction，以及確保安全通信的 TLS 證書。

* 註冊證書 ECert：頒發給提供了註冊憑證的用戶或節點，一般長期有效；
* 交易證書 TCert：頒發給用戶，控制每個交易的權限，一般針對某個交易，短期有效。
* 通信證書 TLSCert：控制對網路的訪問，並且防止竊聽。

![](_images/memserv-components.png)

### 新的架構設計
目前，VP 節點執行了所有的操作，包括接收交易，進行交易驗證，進行一致性達成，進行帳本維護等。這些功能的耦合導致節點性能很難進行擴展。

新的思路就是對這些功能進行解耦，讓每個功能都相對單一，容易進行擴展。社區內已經有了一些討論。

Fabric 1.0 的設計採用了適當的解耦，根據功能將節點角色解耦開，讓不同節點處理不同類型的工作負載。

![示例工作過程](_images/dataflow.png)

* 客戶端：客戶端應用使用 SDK 來跟 Fabric 打交道，構造合法的交易提案提交給 endorser；收集到足夠多 endorser 支持後可以構造合法的交易請求，發給 orderer 或代理節點。
* Endorser peer：負責對來自客戶端的交易進行合法性和 ACL 權限檢查（模擬交易），通過則簽名並返回結果給客戶端。
* Committer peer：負責維護帳本，將達成一致順序的批量交易結果進行狀態檢查，生成區塊，執行合法的交易，並寫入帳本，同一個物理節點可以同時擔任 endorser 和 committer 的 角色。
* Orderer：僅負責排序，給交易們一個全局的排序，一般不需要跟帳本和交易內容打交道。
* CA：負責所有證書的維護，遵循 PKI。

![示例交易過程](_images/transaction_flow.png)
