## 微軟 Azure 雲區塊鏈服務

Azure 是微軟推出的雲計算平臺，向用戶提供開放的 IaaS 和 PaaS 服務。

Azure 陸續在其應用市場中提供了若干個與區塊鏈相關的服務，分別面向多種不同的區塊鏈底層平臺，其中包括以太坊和超級帳本 Fabric。

可以在應用市場（https://azuremarketplace.microsoft.com/en-us/marketplace/apps）中搜索 “blockchain” 關鍵字查看這些服務，如下圖所示。

![Azure 上的區塊鏈服務](_images/azure_marketplace.png)

下面具體介紹其中的 Azure Blockchain Service。

### 使用服務

使用 Azure 服務，用戶可以在幾分鐘之內在雲中部署一個區塊鏈網絡。雲平臺會將一些耗時的配置流程自動化，使用戶專注在上層應用方案。

Azure 區塊鏈服務目前支持部署以太坊或超級帳本 Fabric 網絡。

下面以以太坊為例，在 Azure 的儀表盤中，選擇創建 Ethereum Consortium Blockchain 後，輸入一些配置選項，則可以開始部署該模擬網絡，如下圖所示。

![Azure 區塊鏈配置](_images/azure_config.png)

部署過程需要幾分鐘時間。完成後，可進入資源組查看部署結果，如下圖所示，成功部署了一個以太坊網絡。

![Azure 區塊鏈部署結果](_images/azure_deploy.png)

點擊 microsoft-azure-blockchain 開頭的鏈接，可以查看網絡的一些關鍵接口，包括管理網址、RPC 接口地址等。

複製管理網址 ADMIN-SITE 的鏈接，用瀏覽器打開，可以進入區塊鏈管理界面。界面中可查看網絡各節點信息，也可以新建一個帳戶，並從 admin 帳戶向其發送 1000 個以太幣。結果如下圖所示。

![Azure 區塊鏈管理界面](_images/azure_admin.png)

Azure 雲平臺提供了相對簡單的操作界面，更多的是希望用戶通過 RPC 接口地址來訪問所部署的區塊鏈示例。用戶可以自行通過 RPC 接口與以太坊模擬網絡交互，部署和測試智能合約，此處不再贅述。