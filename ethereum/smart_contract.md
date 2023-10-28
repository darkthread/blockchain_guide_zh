## 使用智能合約

以太坊社區有不少提供智能合約編寫、編譯、發佈、調用等功能的工具，用戶和開發者可以根據需求或開發環境自行選擇。

本節將向開發者介紹使用 Geth 客戶端搭建測試用的本地區塊鏈，以及如何在鏈上部署和調用智能合約。

### 搭建測試用區塊鏈

由於在以太坊公鏈上測試智能合約需要消耗以太幣，所以對於開發者開發測試場景，可以選擇本地自行搭建一條測試鏈。開發好的智能合約可以很容易的切換接口部署到公有鏈上。注意測試鏈不同於以太坊公鏈，需要給出一些非默認的手動配置。

#### 配置初始狀態

首先配置私有區塊鏈網路的初始狀態。新建文件 `genesis.json`，內容如下。

```json
{
  "config": {
        "chainId": 22,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
  },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x400",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000038",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

其中，`chainId` 指定了獨立的區塊鏈網路 ID，不同 ID 網路的節點無法互相連接。配置文件還對當前挖礦難度 `difficulty`、區塊 Gas 消耗限制 `gasLimit` 等參數進行了設置。

#### 啟動區塊鏈

用以下命令初始化區塊鏈，生成創世區塊和初始狀態。

```bash
$ geth --datadir /path/to/datadir init /path/to/genesis.json
```

其中，`--datadir` 指定區塊鏈數據的存儲位置，可自行選擇一個目錄地址。

接下來用以下命令啟動節點，並進入 Geth 命令行界面。

```bash
$ geth --identity "TestNode" --rpc --rpcport "8545" --datadir /path/to/datadir --port "30303" --nodiscover console
```

各選項的含義如下。

* `--identity`：指定節點 ID；
* `--rpc`： 表示開啟 HTTP-RPC 服務；
* `--rpcport`： 指定 HTTP-RPC 服務監聽端口號（默認為 8545）；
* `--datadir`： 指定區塊鏈數據的存儲位置；
* `--port`： 指定和其他節點連接所用的端口號（默認為 30303）；
* `--nodiscover`： 關閉節點發現機制，防止加入有同樣初始配置的陌生節點；

#### 創建帳號

用上述 `geth console` 命令進入的命令行界面採用 JavaScript 語法。可以用以下命令新建一個帳號。

```
> personal.newAccount()

Passphrase:
Repeat passphrase:
"0x1b6eaa5c016af9a3d7549c8679966311183f129e"
```

輸入兩遍密碼後，會顯示生成的帳號，如`"0x1b6eaa5c016af9a3d7549c8679966311183f129e"`。可以用以下命令查看該帳號餘額。

```
> myAddress = "0x1b6eaa5c016af9a3d7549c8679966311183f129e"
> eth.getBalance(myAddress)
0
```

看到該帳號當前餘額為 0。可用 `miner.start()` 命令進行挖礦，由於初始難度設置的較小，所以很容易就可挖出一些餘額。`miner.stop()` 命令可以停止挖礦。

### 創建和編譯智能合約

以 Solidity 編寫的智能合約為例。為了將合約代碼編譯為 EVM 二進制，需要安裝 Solidity 編譯器 solc。

```bash
$ apt-get install solc
```

新建一個 Solidity 智能合約文件，命名為 `testContract.sol`，內容如下。該合約包含一個方法 multiply，作用是將輸入的整數乘以 7 後輸出。

```
pragma solidity ^0.4.0;
contract testContract {
  function multiply(uint a) returns(uint d) {
    d = a * 7;
  }
}
```

用 solc 獲得合約編譯後的 EVM 二進制碼。

```bash
$ solc --bin testContract.sol

======= testContract.sol:testContract =======
Binary:
6060604052341561000c57fe5b5b60a58061001b6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063c6888fa114603a575bfe5b3415604157fe5b60556004808035906020019091905050606b565b6040518082815260200191505060405180910390f35b60006007820290505b9190505600a165627a7a72305820748467daab52f2f1a63180df2c4926f3431a2aa82dcdfbcbde5e7d036742a94b0029
```

再用 solc 獲得合約的 JSON ABI（Application Binary Interface），其中指定了合約接口，包括可調用的合約方法、變量、事件等。

```bash
$ solc --abi testContract.sol

======= testContract.sol:testContract =======
Contract JSON ABI
[{"constant":false,"inputs":[{"name":"a","type":"uint256"}],"name":"multiply","outputs":[{"name":"d","type":"uint256"}],"payable":false,"type":"function"}]
```

下面回到 Geth 的 JavaScript 環境命令行界面，用變量記錄上述兩個值。注意在 code 前加上 `0x` 前綴。

```
> code = "0x6060604052341561000c57fe5b5b60a58061001b6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063c6888fa114603a575bfe5b3415604157fe5b60556004808035906020019091905050606b565b6040518082815260200191505060405180910390f35b60006007820290505b9190505600a165627a7a72305820748467daab52f2f1a63180df2c4926f3431a2aa82dcdfbcbde5e7d036742a94b0029"
> abi = [{"constant":false,"inputs":[{"name":"a","type":"uint256"}],"name":"multiply","outputs":[{"name":"d","type":"uint256"}],"payable":false,"type":"function"}]
```

### 部署智能合約

在 Geth 的 JavaScript 環境命令行界面，首先用以下命令解鎖自己的帳戶，否則無法發送交易。

```
> personal.unlockAccount(myAddress)

Unlock account 0x1b6eaa5c016af9a3d7549c8679966311183f129e
Passphrase:
true
```

接下來發送部署合約的交易。

```
> myContract = eth.contract(abi)
> contract = myContract.new({from:myAddress,data:code,gas:1000000})
```

如果此時沒有在挖礦，用 `txpool.status` 命令可看到本地交易池中有一個待確認的交易。可用以下命令查看當前待確認的交易。

```
> eth.getBlock("pending",true).transactions

[{
    blockHash: "0xbf0619ca48d9e3cc27cd0ab0b433a49a2b1bed90ab57c0357071b033aca1f2cf",
    blockNumber: 17,
    from: "0x1b6eaa5c016af9a3d7549c8679966311183f129e",
    gas: 90000,
    gasPrice: 20000000000,
    hash: "0xa019c2e5367b3ad2bbfa427b046ab65c81ce2590672a512cc973b84610eee53e",
    input: "0x6060604052341561000c57fe5b5b60a58061001b6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063c6888fa114603a575bfe5b3415604157fe5b60556004808035906020019091905050606b565b6040518082815260200191505060405180910390f35b60006007820290505b9190505600a165627a7a72305820748467daab52f2f1a63180df2c4926f3431a2aa82dcdfbcbde5e7d036742a94b0029",
    nonce: 1,
    r: "0xbcb2ba94f45dfb900a0533be3c2c603c2b358774e5fe89f3344031b202995a41",
    s: "0x5f55fb1f76aa11953e12746bc2d19fbea6aeb1b9f9f1c53a2eefab7058515d99",
    to: null,
    transactionIndex: 0,
    v: "0x4f",
    value: 0
}]
```

可以用 `miner.start()` 命令挖礦，一段時間後，交易會被確認，即隨新區塊進入區塊鏈。

### 調用智能合約

用以下命令可以發送交易，其中 sendTransaction 方法的前幾個參數與合約中 multiply 方法的輸入參數對應。這種方式，交易會通過挖礦記錄到區塊鏈中，如果涉及狀態改變也會獲得全網共識。

```
> contract.multiply.sendTransaction(10, {from:myAddress})
```

如果只是想本地運行該方法查看返回結果，可採用如下方式獲取結果。

```
> contract.multiply.call(10)
70
```
