### 學歷認證
#### 功能描述
該 [智能合約](chaincode_example04.go) 實現了一個簡單的徵信管理的案例。針對於學歷認證領域，由於條約公開，在條約外無法隨意篡改的特性，天然具備穩定性和中立性。

該智能合約中三種角色如下：
- 學校
- 個人
- 需要學歷認證的機構或公司

學校可以根據相關信息在區塊鏈上為某位個人授予學歷，相關機構可以查詢某人的學歷信息，由於使用私鑰簽名，確保了信息的真實有效。
為了簡單，儘量簡化相關的業務，另未完成學業的學生因違紀或外出創業退學，學校可以修改其相應的學歷信息。

帳戶私鑰應該由安裝在本地的客戶端生成，本例中為了簡便，使用模擬私鑰和公鑰。

#### 數據結構設計
- 學校
    - 名稱
    - 所在位置
    - 帳號地址
    - 帳號公鑰
    - 帳戶私鑰
    - 學校學生
- 個人
    - 姓名
    - 帳號地址
    - 過往學歷
- 學歷信息
    - 學歷信息編號
    - 就讀學校
    - 就讀年份
    - 完成就讀年份
    - 就讀狀態 // 0:畢業 1：退學
- 修改記錄（入學也相當於一種修改記錄）
    - 編號
    - 學校帳戶地址（一般根據帳戶地址可以算出公鑰地址，然後可以進行校驗）
    - 學校簽名
    - 個人帳戶地址
    - 個人公鑰地址（個人不需要公鑰地址）
    - 修改時間 
    - 修改操作// 0:正常畢業 1：退學 2:入學

對學歷操作信息所有的操作都歸為記錄。    
#### function及各自實現的功能
- `init` 初始化函數
- `invoke` 調用合約內部的函數

- `updateDiploma` 由學校更新學生學歷信息，並簽名（返回記錄信息）
- `enrollStudent` 學校招生（返回學校信息）
- `createSchool` 添加一名新學校
- `createStudent` 添加一名新學生
- `getStudentByAddress` 通過學生的帳號地址訪問學生的學歷信息
- `getRecordById` 通過Id獲取記錄
- `getRecords` 獲取全部記錄（如果記錄數大於 10，返回前 10 個）
- `getSchoolByAddress` 通過學校帳號地址獲取學校的信息
- `getBackgroundById` 通過學歷 Id 獲取所存儲的學歷信息

- `writeRecord` 寫入記錄
- `writeSchool` 寫入新創建的學校
- `writeStudent` 寫入新創建的學生

#### 接口設計
 `createSchool`

request參數:
```
args[0] 學校名稱
args[1] 學校所在位置
```
response參數:
```
學校信息的字節數組，當創建一所新學校時，該學校學生帳戶地址列表為空
```

`createStudent`

request參數：
```
args[0] 學生的姓名
```

response參數：
```
學生信息的字節數組表示，剛創建過往學歷信息列表為空
```

`updateDiploma` 

request參數
```
args[0] 學校帳戶地址
args[1] 學校簽名
args[2] 待修改學生的帳戶地址
args[3] //對該學生的學歷進行怎樣的修改，0：正常畢業  1：退學  
```

response參數
```
返回修改記錄的字節數組表示
```

`enrollStudent`

request參數:
```
args[0] 學校帳戶地址
args[1] 學校簽名
args[2] 學生帳戶地址
```

response參數
```
返回修改記錄的字節數組表示
```

`getStudentByAddress`

request參數
```
args[0] address
```
response參數
```
學生信息的字節數組表示
```

`getRecordById`

request參數
```
args[0] 修改記錄的ID
```
response參數
```
修改記錄的字節數組表示
```

`getRecords`

response參數
```
獲取修改記錄數組（如果個數大於10，返回前10個）
```
`getSchoolByAddress`

request參數
```
args[0] address
```
response參數
```
學校信息的字節數組表示
```

`getBackgroundById`

request參數
```
args[0] ID
```

response參數
```
學歷信息的字節數組表示
```

#### 測試
