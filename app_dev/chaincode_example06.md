### 物流供應鏈簡單案例
#### 功能描述
該 [智能合約](chaincode_example06.go) 實現了一個簡單的供應鏈應用案例，針對物流行業的應用場景。由於將合約的協議公開，並且簽收快遞時需要簽名，可以在很大程度上保證不被冒領，實現了一手交錢，一手交貨，同時提高了效率，確保了透明。

該智能合約中三種角色如下：
- 物流公司（本案例中只有1位）
- 寄貨方（本案例中有多位）
- 收貨方（本案例中有多位）

業務流程如下：

1、寄貨方填寫寄貨單，物流公司根據寄貨單寄快遞。

2、寄快遞過程中物流公司各個快遞點對快遞進行掃描，描述目前快遞進度，並更新貨單狀態。寄貨方和收貨方可以根據單號進行查詢。

3、快遞到達後，收貨方檢查商品，確認無誤後，掃碼並使用私鑰簽名，支付相關費用，更新訂單狀態。

在實際中，物流費的支付分為兩類：
- 1、寄貨方支付。收貨方簽收快遞後先預付給物流公司。
- 2、收貨方支付。收貨方簽收快遞後支付給物流公司。

在本案例中暫不考慮貨物損壞、收貨方失聯、貨物保值等的相關問題。具體實現邏輯如下：

- 創建賬戶。為每個用戶生成唯一的私鑰與地址。
- 生成寄貨單。寄貨方填寫紙質寄貨單，物流公司根據此生成電子單。
- 更新寄貨單。物流公司旗下快遞點根據配送信息更新電子寄貨單。
- 收貨方簽收確認。收貨方收到貨物後，使用自己的私鑰進行簽收，完成相應的付款。

賬戶私鑰應該由安裝在本地的客戶端生成，本例中為了簡便，使用模擬私鑰和公鑰。每位用戶的私鑰為guid+“1”，公鑰為guid+“2”。用戶簽名為私鑰+“1”

#### 數據結構設計
- 寄貨單
    - 寄貨單編號
    - 寄貨方地址
    - 收貨方地址
    - 寄貨方聯繫方式
    - 收貨方聯繫方式
    - 物流費用
    - 物流費用支付類型  //0：寄貨方支付 1：收貨方支付
    - 寄貨方預支付費用  //模擬實際預支付，寄貨方支付物流費下值為物流費，否則為0
    - 快遞配送信息    // 快遞運送狀態，所經過快遞分撥中心與快遞點的數組
    - 收貨方簽名 

- 寄貨方
    - 姓名
    - 所在地址
    - 賬戶地址
    - 賬戶公鑰
    - 聯繫方式
    - 賬戶餘額
- 收貨方
    - 姓名
    - 所在地址
    - 賬戶地址
    - 賬戶公鑰
    - 賬戶私鑰
    - 聯繫方式
    - 賬戶餘額
- 物流公司
    - 賬戶公鑰
    - 賬戶私鑰
    - 名稱
    - 地址
    - 聯繫方式
    - 賬戶餘額
    - 物流公司旗下分撥中心與快遞點
- 快遞點
    - 名稱
    - 所在地址
    - 聯繫方式
    - 快遞點公鑰
    - 快遞點私鑰
    - 快遞點賬戶地址

#### function及各自實現的功能
- `init`  初始化物流公司及其下相應快遞點
- `invoke`   調用合約內部的函數
- `query`   查詢相關的信息
- `createUser` 創建用戶 init
- `createExpress` 創建物流公司 init
- `createExpressPoint` 創建快遞點 init
- `createExpressOrder` 寄貨方創建寄貨單  init
- `finishExpressOrder` 收貨方簽收寄貨單 invoke
- `addExpressPointer` 物流公司添加新的快遞點  invoke
- `updateExpressOrder` 更新物流公司訂單,添加快遞點的信息 invoke  


- `getExpressOrderById` 查詢訂單狀態  query
- `getExpress`  獲取物流公司信息      query
- `getUserByAddress` 獲取用戶信息   query
- `getExpressPointByAddress`  獲取快遞點信息  query   

- `writeExpress` 存儲物流公司信息 （以物流公司賬戶地址進行存儲）
- `writeExpressOrder` 存儲寄貨單  （以“express”+id 進行存儲）
- `writeUser` 存儲用戶信息   （以地址進行存儲）
- `writeExpressPoint` 存儲物流點信息  （以快遞點賬戶地址進行存儲）

#### 接口設計
 `createUser`

request參數
```
args[0] 姓名 
args[1] 所在地址
args[2] 聯繫方式
args[3] 賬戶餘額
```

response參數
```
user信息的json表示
```

 `createExpressPointer` 

request參數
```
args[0] 姓名
args[1] 所在地址
args[2] 聯繫方式
```

response參數
```
物流點的信息的json表示
```

 `createExpress`

request 參數
```
args[0]  名稱
args[1]  地址
args[2]  聯繫方式
args[3]  賬戶餘額
```
response 參數
```
物流公司信息的json表示
```

 `addExpressPointer`

request參數
```
args[0] 添加快遞點
```

response參數
```
物流公司信息的json表示
```

 `createExpressOrder`

request參數
```
args[0] 寄貨方地址
args[1] 收貨方地址
args[2] 寄貨方賬戶地址
args[3] 收貨方賬戶地址
args[4] 寄貨方聯繫方式
args[5] 收貨方聯繫方式
args[6] 物流費用支付類型
args[7] 寄貨方預支付費用 （收貨方支付的話值為0）
args[8] 物流費用
```

response 參數
```
訂單信息的json表示
```

 `updateExpressOrder`

request參數
```
args[0]  訂單id
args[1]  快遞點地址
```

response參數
```
訂單信息的json表示
```

  `finishExpressOrder`

request參數
```
args[0] 收貨方賬戶地址
args[1] 賬戶訂單編號
args[2] 收貨方簽名
```

response參數
```
訂單信息的json表示
```

`getExpressOrderById`

request參數：
```
args[0] id
```

response參數：
```
快遞訂單的json表示
```

`getExpress`

response參數：
```
快遞信息的json表示
```

`getUserByAddress`

request參數
```
args[0] address
```

response參數
```
用戶信息的json表示
```

`getExpressPointerByAddress`

request參數
```
args[0] address
```

response參數
```
快遞點的json信息表示
```

#### 測試
