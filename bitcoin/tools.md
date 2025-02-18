## 相關工具

比特幣相關工具包括客戶端、錢包和礦機等。

### 客戶端

比特幣客戶端用於和比特幣網路進行交互，同時可以參與到網路的維護。

客戶端分為三種：完整客戶端、輕量級客戶端和在線客戶端。

* 完整客戶端：存儲所有的交易歷史記錄，功能完備；
* 輕量級客戶端：不保存交易副本，交易需要向別人查詢；
* 在線客戶端：通過網頁模式來瀏覽第三方服務器提供的服務。

比特幣客戶端可以從 https://bitcoin.org/en/download 下載到。

基於比特幣客戶端，可以很容易實現用戶錢包功能。

### 錢包

比特幣錢包存儲和保護用戶的私鑰，並提供查詢比特幣餘額、收發比特幣等功能。根據私鑰存儲方式不同，錢包主要分為以下幾種：

* 離線錢包：離線存儲私鑰，也稱為“冷錢包”。安全性相對最強，但無法直接發送交易，便利性差。
* 本地錢包：用本地設備存儲私鑰。可直接向比特幣網路發送交易，易用性強，但本地設備存在被攻擊風險。
* 在線錢包：用錢包服務器存儲經用戶口令加密過的私鑰。易用性強，但錢包服務器同樣可能被攻擊。
* 多重簽名錢包：由多方共同管理一個錢包地址，比如 2 of 3 模式下，集合三位管理者中的兩位的私鑰便可以發送交易。

比特幣錢包可以從 https://bitcoin.org/en/choose-your-wallet 獲取到。

### 礦機

比特幣礦機是專門為“挖礦”設計的硬件設備，目前主要包括基於 GPU 和 ASIC 芯片的專用礦機。這些礦機往往採用特殊的設計來加速挖礦過程中的計算處理。

礦機最重要的屬性是可提供的算力（通常以每秒可進行 Hash 計算的次數來表示）和所需要的功耗。當算力足夠大，可以在概率意義上挖到足夠多的新的區塊，來彌補電力費用時，該礦機是可以盈利的；當單位電力產生的算力不足以支付電力費用時，該礦機無法盈利，意味著只能被淘汰。

目前，比特幣網路中的全網算力仍然在快速增長中，礦工需要綜合考慮算力變化、比特幣價格、功耗帶來的電費等許多問題，需要算好“經濟帳”。

