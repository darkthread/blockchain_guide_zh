### 社區能源共享
#### 功能描述
本 [合約](chaincode_example05.go) 以紐約實驗性的能源微電網為例，作為一個簡單的案例進行實現。

>“在總統大道的一邊，五戶家庭通過太陽能板發電；在街道的另一邊的五戶家庭可以購買對面家庭不需要的電力。而連接這項交易的就是區塊鏈網路，幾乎不需要人員參與就可以管理記錄交易。”但是這個想法是非常有潛力的，能夠代表未來社區管理能源系統。”

布魯克林微電網開發商 LO3 創始人 Lawrence Orsini 說：

>“我們正在這條街道上建立一個可再生電力市場，來測試人們對於購買彼此手中的電力是否感興趣。如果你在很遠的地方生產能源，運輸途中會有很多損耗，你也得不到這電力價值。但是如果你就在街對面，你就能高效的利用能源。”

在某一塊區域內存在一個能源微電網，每一戶家庭可能為生產者也可能為消費者。部分家庭擁有太陽能電池板，太陽能電池板的剩餘電量為可以售出的電力的值，為了簡化，單位為1.需要電力的家庭可以向有足夠餘額的電力的家庭購買電力。

帳戶私鑰應該由安裝在本地的客戶端生成，本例中為了簡便，使用模擬私鑰和公鑰。每位用戶的私鑰為guid+“1”，公鑰為guid+“2”。簽名方式簡化為私鑰+"1"

#### 數據結構設計
在該智能合約中暫時只有一種角色，為每一戶家庭用戶。

- 家庭用戶
    - 帳戶地址
    - 剩餘能量 //部分家庭沒有太陽能電池板，值為0
    - 帳戶餘額（電子貨幣）
    - 編號
    - 狀態  //0：不可購買， 1：可以購買
    - 帳戶公鑰
    - 帳戶私鑰
- 交易(一筆交易必須同時具有賣方和買方的公鑰簽名，方能承認這筆交易。公鑰簽名生成規則，公鑰+待創建交易的ID號，在本交易類型中，只要買家有足夠的貨幣，賣家自動會對交易進行簽名)
    - 購買方地址
    - 銷售方地址
    - 電量銷售量
    - 電量交易金額
    - 編號
    - 交易時間

#### function及各自實現的功能
- `init`  初始化操作
- `invoke`   調用合約內部的函數
- `query`   查詢相關的信息
- `createUser` 創建新用戶，並加入到能源微網中   invoke
- `buyByAddress` 向某一位用戶購買一定量的電力   invoke
- `getTransactionById` 通過id獲取交易內容  query
- `getTransactions` 獲取交易（如果交易數大於10，獲取前10個） query
- `getHomes` 獲取用戶（如果用戶數大於10，獲取前10個） query
- `getHomeByAddress` 通過地址獲取用戶 query
- `changeStatus` 某一位用戶修改自身的狀態  invoke

- `writeUser` 將新用戶寫入到鍵值對中  
- `writeTransaction` 記錄交易
#### 接口設計
`createUser`

request參數:
```
args[0] 剩餘能量值 
args[1] 剩餘金額
```
response參數:
```
新建家庭用戶的json表示
```

`buyByAddress`

request參數:
```
args[0] 賣家的帳戶地址
args[1] 買家簽名
args[2] 買家的帳戶地址
args[3] 想要購買的電量數值
```
response參數:
```
購買成功的話返回該transaction的json串。
購買失敗返回error
```

`getTransactionById`

request參數:
```
args[0] 交易編號
```
response參數:
``` 
查詢結果的transaction 交易表示
```

`getTransactions`

request參數:
```
none
 ```
response參數:
```
獲取所有的交易列表（如果交易大於10，則返回前10個）
```

`getHomeByAddress`

request參數
```
args[0] address
```
response參數
```
用戶信息的json表示
```

`getHomes`

response參數
```
獲取所有的用戶列表（如果用戶個數大於10，則返回前10個）
```

`changeStatus`

request參數:
```
args[0] 帳戶地址
args[1]　帳戶簽名
args[2] 對自己的帳戶進行的操作，0：設置為不可購買  1：設置狀態為可購買
```
response參數:
```
修改後的用戶信息json表示
```

#### 測試
