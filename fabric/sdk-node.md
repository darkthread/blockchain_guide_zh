
## 使用 Hyperledger Fabric SDK Node 進行測試

[Hyperledger Fabric Client SDK](https://github.com/hyperledger/fabric-sdk-node) 能夠非常簡單的使用API和 Hyperledger Fabric Blockchain 網路進行交互。其`v1.1`及其以上的版本添加了一個重要的功能[Conection-Profile](https://fabric-sdk-node.github.io/tutorial-network-config.html)來保存整個network中必要的配置信息，方便client讀取和配置。
該Demo基於`Connection-Profile`測試了整個網路的如下功能：
* Fabric CA 相關
  * Enroll用戶
  * Register用戶
* Channel 相關
  * 創建Channel
  * 將指定Peer join Channel
  * 查詢Channel相關信息
  * 動態更新Channel配置信息
* Chaincode 相關
  * Install Chaincode
  * Instantiate Chaincode
  * Invoke Chaincode
  * Query Chaincode
  * 查詢Chaincode相關信息

### 主要依賴

* Node v8.9.0 或更高 （注意目前v9.0+還不支持）
* npm v5.5.1 或更高
* gulp命令。 必須要進行全局安裝 `npm install -g gulp`
* docker運行環境
* docker compose工具

主要fabric環境可參考[Fabric 1.0](https://github.com/yeasy/blockchain_guide/blob/master/fabric/1.0.md)。

### 下載 Demo 工程

```sh
$ git clone https://github.com/Sunnykaby/Hyperledger-fabric-node-sdk-demo
```


進入 `Hyperledger-fabric-node-sdk-demo` 目錄，查看各文件夾和文件，功能如下。

文件/文件夾 | 功能 
-- | --
artifacts-local | 本地準備好構建fabric網路的基礎材料
artifacts-remote | 使用官方fabric-sample動態構建網路
extra | 一些拓展性的材料
node |  基於Fabric SDK Node的demo核心代碼 
src | 測試用chaincode
Init.sh | 構建Demo的初始化腳本

### 構建Demo

該專案提供兩種Demo構建方式：
* 利用本地已經準備好的相關網路資源，啟動fabric network。
* 利用官方fabric-sample專案，動態啟動fabric network。

當然，你也可以使用自己已經創建好的fabric network和其相關的connection-profile來測試Demo。

```sh
##進入專案根目錄

##使用本地資源構建Demo
./Init.sh local

##使用官方資源構建Demo
./Init.sh remote
```

執行之後，會在根目錄中生成一個`demo`文件夾，其就是Demo程序的入口。

清理Demo資源，使用`./Init.sh clean`

### 啟動Fabric網路

首先，我們需要準備一個fabric網路來進行測試。
進入到`demo`文件夾。

#### 本地資源構建網路

進入資源目錄，利用腳本啟動網路即可。
```sh
cd artifacts
##啟動網路
./net.sh up
##關閉網路
./net.sh down
```
用該腳本啟動網路中包含：1個orderer， 2個organisation， 4個peer（每個組織有2個peer）和兩個ca（每個組織一個）。

#### 官方資源構建網路

在demo目錄，利用腳本啟動網路即可。
```sh
##啟動網路,並配置本地資源
./net.sh init
##關閉網路並清理資源
./net.sh clean
```
用該腳本啟動網路中包含：1個orderer， 2個organisation， 4個peer（每個組織有2個peer）和兩個ca（每個組織一個）。

與本地資源啟動不同，該方案主要有以下步驟：
* 將官方[fabric-sample專案](https://github.com/hyperledger/fabric-samples)clone到本地
* 利用`fabric-sample/first-network/bynf.sh up`啟動fabric腳本
* 將一些資源文件連接到指定位置，方便node程序使用
* 通過資源文件構建connection-profile（替換密鑰等）
* 創建一個新的channel的binary

詳細信息可以直接查看`net.sh`腳本。

>`clean`命令會將所有相關的docker 容器和remote的動態資源全部刪除。還原到最初的demo文件狀態。

#### 資源清單

無論是remote還是local模式，最終資源和網路準備完成之後，核心資源列表如下：
```
demo/artifacts/  
├── channel-artifacts                
│   ├── channel2.tx    
│   ├── channel.tx  
│   ├── genesis.block  
│   ├── Org1MSPanchors.tx  
│   └── Org2MSPanchors.tx  
├── connection-profile              
│   ├── network.yaml  
│   ├── org1.yaml  
│   ├── org2.yaml  
├── crypto-config  
│   ├── ordererOrganizations  
│   │   └── example.com  
│   └── peerOrganizations  
│       ├── org1.example.com  
│       └── org2.example.com  
```

### 運行Demo

網路和相關資源準備成功之後，進入`demo/node`目錄。
其主要結構為：
```
├── app                             //核心應用接口
│   ├── api-handler.js              //接口定義文件
│   ├── *.js                        //應用實現模塊
│   ├── tools                       //通用工具類
│   │   ├── ca-tools.js
│   │   ├── config-tool.js
│   │   └── helper.js
├── app-test.js                     //Demo程序啟動文件
├── package.json
└── readme.md
```

使用命令`node app-test.js`即可進行一個完整workflow的測試，包括最開始我們提到的所有功能。
同時可以使用`node app-test.js -m ca|createChannel|joinChannel|install|instantiate|invoke|query|queryChaincodeInfo|queryChannelInfo`來運行單個功能。

程序使用的均為默認參數，其定義在`app-test.js`文件中。可以按照需求修改對應的參數，再運行程序即可。

### 持續更新

如果在使用途中發現任何問題，或者有任何需求可以在該專案的issue中提出改進方案或者建議。
Github地址：[Hyperledger-fabric-node-sdk-demo](https://github.com/Sunnykaby/Hyperledger-fabric-node-sdk-demo)
