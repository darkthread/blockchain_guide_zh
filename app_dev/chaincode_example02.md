## 鏈碼示例二：交易資產

### 簡介

[chaincode_example02.go](chaincode_example02.go) 主要實現如下的功能：

* 初始化 A、B 兩個帳戶，併為兩個帳戶賦初始資產值；
* 在 A、B 兩個帳戶之間進行資產交易；
* 分別查詢 A、B 兩個帳戶上的餘額，確認交易成功；
* 刪除帳戶。

### 主要函數

* `init`：初始化 A、B 兩個帳戶；
* `invoke`：實現 A、B 帳戶間的轉帳；
* `query`：查詢 A、B 帳戶上的餘額；
* `delete`：刪除帳戶。

### 依賴的包
```go
import (
	"errors"
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric/core/chaincode/shim"
)
```
`strconv` 實現 int 與 string 類型之間的轉換。

在invoke 函數中，存在：
```go
X, err = strconv.Atoi(args[2])
	Aval = Aval - X
	Bval = Bval + X
```

當 `args[2]<0` 時，A 帳戶餘額增加，否則 B 帳戶餘額減少。

### 可擴展功能
實例中未包含新增帳戶並初始化的功能。開發者可以根據自己的業務模型進行添加。
