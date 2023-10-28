## 貢獻代碼

超級帳本的各個子專案，都提供了十分豐富的開發和提交代碼的指南和文檔，一般可以在代碼的 `docs` 目錄下找到。部分專案（如 Fabric 和 Cello）使用了社區自建的代碼管理和評審方案，其他專案多數直接使用 Github 來管理流程。

這裡以 Fabric 專案為例進行講解。

### 安裝環境

推薦在 Linux（如 Ubuntu 18.04+）或 MacOS 環境中開發 Hyperledger 專案代碼。

不同專案會依賴不同的環境，可以從專案文檔中找到。以 Fabric 專案為例，開發需要安裝如下依賴。

* Git：用來從 Gerrit 倉庫獲取代碼並進行版本管理。
* Golang 1.10+：訪問 golang.org 進行安裝，之後需要配置 `$GOPATH` 環境變量。注意不同專案可能需要不同語言環境。
* Docker 1.18+：用來支持容器環境，MacOS 下推薦使用 [Docker for Mac](https://docs.docker.com/docker-for-mac)。

### 獲取代碼

首先註冊 Linux Foundation ID（LF ID），如果沒有可以到 https://identity.linuxfoundation.org/ 進行免費註冊。

使用 LF ID 登陸 [https://gerrit.hyperledger.org/](https://gerrit.hyperledger.org/)，在配置頁面（https://gerrit.hyperledger.org/r/#/settings/ssh-keys），添加個人 ssh Pub key，否則每次訪問倉庫需要手動輸入用戶名和密碼。

查看專案列表，找到對應專案，採用 `Clone with commit-msg hook` 的方式來獲取源碼。

以 Fabric 專案為例，按照 Go 語言推薦代碼結構，執行如下命令拉取代碼，放到 `$GOPATH/src/github.com/hyperledger/` 路徑下，其中 `LF_ID` 替換為用戶個人的 Linux Foundation ID。

```sh
$ mkdir $GOPATH/src/github.com/hyperledger/
$ cd $GOPATH/src/github.com/hyperledger/
$ git clone ssh://LF_ID@gerrit.hyperledger.org:29418/fabric && scp -p -P 29418 LF_ID@gerrit.hyperledger.org:hooks/commit-msg fabric/.git/hooks/
```

如果沒有添加個人 ssh pubkey，則可以通過 http 方式 clone，此時需要手動輸入用戶名和密碼信息。

```sh
$ git clone http://LF_ID@gerrit.hyperledger.org/r/fabric && (cd fabric && curl -kLo `git rev-parse --git-dir`/hooks/commit-msg http://LF_ID@gerrit.hyperledger.org/r/tools/hooks/commit-msg; chmod +x `git rev-parse --git-dir`/hooks/commit-msg)
```

如果是首次使用 Git，可能還會提示配置默認的用戶名和 Email 地址等信息。通過如下命令進行簡單配置即可。

```bash
$ git config user.name "your name"
$ git config user.email "your email"
```

### 編譯和測試

大部分編譯和安裝過程都可以利用 Makefile 來執行，具體以專案代碼為準。以 Fabric 專案為例，包括如下常見操作。

#### 安裝 go tools
執行如下命令。

```sh
$ make gotools
```

#### 語法格式檢查

執行如下命令。

```sh
$ make linter
```

#### 編譯 peer

執行如下命令。

```sh
$ make peer
```

會自動編譯生成 Docker 鏡像，並生成本地 peer 可執行文件。

*注意：有時候會因為獲取安裝包不穩定而報錯，需要執行 `make clean`，然後再次執行。*

#### 生成 Docker 鏡像
執行如下命令。

```sh
$ make images
```

#### 執行所有的檢查和測試
執行如下命令。

```sh
$ make checks
```

#### 執行單元測試

執行如下命令。 

```sh
$ make unit-test
```

如果要運行某個特定單元測試，則可以通過類似如下格式。

```sh
$ go test -v -run=TestGetFoo
```

#### 執行 BDD 測試
需先生成本地 Docker 鏡像。

執行如下命令。 

```sh
$ make behave
```

### 提交代碼

仍然使用 LF ID 登錄 [jira.hyperledger.org](http://jira.hyperledger.org)，查看有沒有未分配（unassigned）的任務，如果對某個任務感興趣，可以添加自己為任務的 assignee。任何人都可以自行創建新的任務。

初始創建的任務處於 `TODO` 狀態；開始工作後可以標記為 `In Progress` 狀態；提交對應補丁後需要更新為 `In Review` 狀態；任務完成後更新為 `Done` 狀態。

如果希望完成某個任務（如 FAB-XXX），則對於前面 Clone 下來的代碼，本地創建新的分支 FAB-XXX。

```sh
$ git checkout -b FAB-XXX
```

實現任務代碼，完成後，執行語法格式檢查和測試等，確保所有檢查和測試都通過。

提交代碼到本地倉庫。

```sh
$ git commit -a -s
```

會自動打開一個編輯器窗口，需要填寫 commit 信息，格式一般要求為：

```bash
[FAB-XXX] Quick brief on the change

This pathset fixes a duplication msg bug in gossip protocol.

A more detailed description can be here, with several paragraphs and 
sentences, including issue to fix, why to fix, what is done in the 
patchset and potential remaining issues...
```

提交消息中要寫清楚所解決的問題、為何進行修改、主要改動內容、遺留問題等，並且首行寬不超過 50 個字符，詳情段落行寬不要超過 72 個字符。

如果是首次往官方倉庫提交代碼，需要先配置 `git review`，根據提示輸入所需要的用戶名密碼等。

```bash
$ git review -s
```

驗證通過後可以使用 git review 命令推送到遠端倉庫，推送成功後會得到補丁的編號和訪問地址。

```sh
$ git review
```

例如：

```bash
$ git review
remote: Processing changes: new: 1, refs: 1, done
remote:
remote: New Changes:
remote:   http://gerrit.hyperledger.org/r/YYY [FAB-XXX] Fix some problem
remote:
To ssh://gerrit.hyperledger.org:29418/fabric.git
 * [new branch]      HEAD -> refs/publish/master/FAB-XXX
```

### 評審代碼

提交成功後，可以打開 [gerrit.hyperledger.org/r/](https://gerrit.hyperledger.org/r/)，查看自己最新提交的 patchset 信息。新提交的 patchset 會自動觸發 CI 的測試任務，測試都通過後可邀請專案的維護者（maintainer）們進行評審。為了引起關注，可將鏈接添加到對應的 Jira 任務，並在 RocketChat 上的專案頻道貼出。

*注：手動觸發某個 CI 測試任務可以通過 Run <Task> 命令，例如重新運行單元測試可以使用 Run UnitTest。*

如果評審通過，則會被合併到主分支。否則還需要針對審閱意見進一步的修正。修正過程跟提交代碼過程類似，唯一不同是提交的時候添加 `-a --amend` 參數。

```sh
$ git commit -a --amend
```

表示這個提交是對舊提交的一次修訂。

一般情況下，為了方便評審，儘量保證每個 patchset 完成的改動不要太多（最好不要超過 200 行），並且實現功能要明確，集中在對應 Jira 任務定義的範圍內。

### 完整流程

![代碼提交流程](_images/patchset-lifecycle.png)

總結下，完整的流程如上圖所示，開發者用 git 進行代碼的版本管理，用 gerrit 進行代碼的評審合作。

如果需要修復某個提交補丁的問題，則通過 `git commit -a --amend` 進行修復，並作為補丁的新版本再次提交審閱。每次通過 `git review` 提交時，應當通過 `git log` 查看確保本地只有一條提交記錄。

