## 數字貨幣發行與管理
### 簡介
該智能合約實現一個簡單的商業應用案例，即數字貨幣的發行與轉帳。在這之中一共分為三種角色：中央銀行，商業銀行，企業。其中中央銀行可以發行一定數量的貨幣，企業之間可以進行相互的轉帳。主要實現如下的功能：

* 初始化中央銀行及其發行的貨幣數量
* 新增商業銀行，同時央行並向其發行一定數量的貨幣
* 新增企業
* 商業銀行向企業轉給一定數量的數字貨幣
* 企業之間進行相互的轉帳
* 查詢企業、銀行、交易信息

### 主要函數
* `init`：初始化中央銀行，併發行一定數量的貨幣；
* `invoke`：調用合約內部的函數；
* `query`：查詢相關的信息；
* `createBank`：新增商業銀行，同時央行向其發行一定數量的貨幣；
* `createCompany`：新增企業；
* `issueCoin`：央行再次發行一定數量的貨幣（歸於交易）；
* `issueCoinToBank`：央行向商業銀行轉一定數量的數字貨幣（歸於交易）；
* `issueCoinToCp`：商業銀行向企業轉一定數量的數字貨幣（歸於交易行為）；
* `transfer`：企業之間進行相互轉帳（歸於交易行為）；
* `getCompanys`：獲取所有的公司信息，如果企業個數大於10，先訪問前10個；
* `getBanks`：獲取所有的商業銀行信息，如果商業銀行個數大於10，先訪問前 10 個
* `getTransactions`：獲取所有的交易記錄 如果交易個數大於10，先訪問前 10 個；
* `getCompanyById`：獲取某家公司信息；
* `getBankById`：獲取某家銀行信息；
* `getTransactionBy`：獲取某筆交易記錄；
* `writeCenterBank`：修改央行信息；
* `writeBank`：修改商業銀行信息；
* `writeCompany`：修改企業信息；
* `writeTransaction`：寫入交易信息。

### 數據結構設計
* centerBank 中央銀行
  * Name：名稱
  * TotalNumber：發行貨幣總數額
  * RestNumber：帳戶餘額
  * ID：ID固定為 0
* bank  商業銀行
  * Name：名稱
  * TotalNumber：收到貨幣總數額
  * RestNumber：帳戶餘額
  * ID：銀行 ID
* company 企業
  * Name：名稱
  * Number：帳戶餘額
  * ID：企業 ID
* transaction 交易內容
  * FromType：發送方角色 //centerBank:0,Bank:1,Company:2
  * FromID：發送方 ID
  * ToType：接收方角色 //Bank:1,Company:2
  * ToID：接收方 ID
  * Time：交易時間
  * Number：交易數額
  * ID：交易 ID
 
### 接口設計
#### `init`
request 參數:

```
args[0] 銀行名稱
args[1] 初始化發佈金額
```

response 參數:

```json
{"Name":"XXX","TotalNumber":"0","RestNumber":"0","ID":"XX"}
```

#### `createBank`

request 參數:
```
args[0] 銀行名稱
```

response 參數:

```json
{"Name":"XXX","TotalNumber":"0","RestNumber":"0","ID":"XX"}
```

#### `createCompany`

request 參數:

```
args[0] 公司名稱
```

response 參數:

```json
{"Name":"XXX","Number":"0","ID":"XX"}
```

#### `issueCoin`

request 參數:

```
args[0] 再次發行貨幣數額

```
response 參數:

```json
{"FromType":"0","FromID":"0","ToType":"0","ToID":"0","Time":"XX","Number":"XX","ID":"XX"}
```

#### `issueCoinToBank`

request 參數:

```
args[0] 商業銀行ID
args[1] 轉帳數額
```

response 參數:

```json
{"FromType":"0","FromID":"0","ToType":"1","ToID":"XX","Time":"XX","Number":"XX","ID":"XX"}
```

#### `issueCoinToCp`

request 參數:

```
args[0] 商業銀行ID
args[1] 企業ID
args[2] 轉帳數額

```
response 參數:

```json
{"FromType":"1","FromID":"XX","ToType":"2","ToID":"XX","Time":"XX","Number":"XX","ID":"XX"}
```

#### `transfer`

request 參數:
```
args[0] 轉帳用戶ID
args[1] 被轉帳用戶ID
args[2] 轉帳餘額
```

response 參數:

```json
{"FromType":"2","FromID":"XX","ToType":"2","ToID":"XX","Time":"XX","Number":"XX","ID":"XX"}
```

#### `getBanks`

response 參數

```json
[{"Name":"XXX","Number":"XX","ID":"XX"},{"Name":"XXX","Number":"XX","ID":"XX"},...]
```

#### `getCompanys`

response 參數

```json
[{"Name":"XXX","TotalNumber":"XX","RestNumber":"XX","ID":"XX"},{"Name":"XXX","TotalNumber":"XX","RestNumber":"XX","ID":"XX"},...]
```

#### `getTransactions`

response 參數

```json
[{"FromType":"XX","FromID":"XX","ToType":"XX","ToID":"XX","Time":"XX","Number":"XX","ID":"XX"},{"FromType":"XX","FromID":"XX","ToType":"XX","ToID":"XX","Time":"XX","Number":"XX","ID":"XX"},...]
```

#### `getCenterBank`

response 參數

```json
[{"Name":"XX","TotalNumber":"XX","RestNumber":"XX","ID":"XX"}]
```

#### `getBankById`

request 參數

```
args[0] 商業銀行ID
```

response 參數

```json
[{"Name":"XX","TotalNumber":"XX","RestNumber":"XX","ID":"XX"}]
```

#### `getCompanyById`

request 參數

```
args[0] 企業ID
```

response 參數

```json
[{"Name":"XXX","Number":"XX","ID":"XX"}]
```

#### `getTransactionById`

request 參數
```
args[0] 交易ID
```

response 參數

```json
{"FromType":"XX","FromID":"XX","ToType":"XX","ToID":"XX","Time":"XX","Number":"XX","ID":"XX"}
```

#### `writeCenterBank`

request 參數

```
CenterBank
```

response 參數

```
err  nil 為成功
```

#### `writeBank`

request 參數

```
Bank
```

response 參數

```
err  nil 為成功
```

#### `writeCompany`

request 參數
```
Company
```

response 參數

```
err  nil 為成功
```

#### `writeTransaction`

request 參數
```
Transaction
```

response 參數

```
err  nil 為成功
···

#### 其它
查詢時為了兼顧讀速率，將一些信息備份存放在非區塊鏈數據庫上也是一個較好的選擇。
