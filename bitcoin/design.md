## 原理和設計

比特幣網路是一個分散式的點對點網路，網路中的礦工通過“挖礦”來完成對交易記錄的記帳過程，維護網路的正常運行。

區塊鏈網路提供一個公共可見的記帳本，該記帳本並非記錄每個帳戶的餘額，而是用來記錄發生過的交易的歷史信息。該設計可以避免重放攻擊，即某個合法交易被多次重新發送造成攻擊。 

### 基本交易過程

比特幣中沒有帳戶的概念。因此，每次發生交易，用戶需要將交易記錄寫到比特幣網路帳本中，等網路確認後即可認為交易完成。

除了挖礦獲得獎勵的 coinbase 交易只有輸出，正常情況下每個交易需要包括若干輸入和輸出，未經使用（引用）的交易的輸出（Unspent Transaction Outputs，UTXO）可以被新的交易引用作為其合法的輸入。被使用過的交易的輸出（Spent Transaction Outputs，STXO），則無法被引用作為合法輸入。

因此，比特幣網路中一筆合法的交易，必須是引用某些已存在交易的 UTXO（必須是屬於付款方才能合法引用）作為新交易的輸入，並生成新的 UTXO（將屬於收款方）。

那麼，在交易過程中，付款方如何證明自己所引用的 UTXO 合法？比特幣中通過“簽名腳本”來實現，並且指定“輸出腳本”來限制將來能使用新 UTXO 者只能為指定收款方。對每筆交易，付款方需要進行簽名確認。並且，對每一筆交易來說，總輸入不能小於總輸出。總輸入相比總輸出多餘的部分稱為交易費用（Transaction Fee），為生成包含該交易區塊的礦工所獲得。目前規定每筆交易的交易費用不能小於 0.0001 BTC，交易費用越高，越多礦工願意包含該交易，也就越早被放到網路中。交易費用在獎勵礦工的同時，也避免了網路受到大量攻擊。

交易中金額的最小單位是“聰”，即一億分之一（10^-8）比特幣。

下圖展示了一些簡單的示例交易。更一般情況下，交易的輸入、輸出可以為多方。

| 交易 | 目的 | 輸入 | 輸出 | 簽名 | 差額 |
| --- | --- | --- | --- | --- | --- |
| T0 | A 轉給 B | 他人向 A 交易的輸出 | B 帳戶可以使用該交易 | A 簽名確認 | 輸入減輸出，為交易服務費 |
| T1 | B 轉給 C | T0 的輸出 | C 帳戶可以使用該交易 | B 簽名確認 | 輸入減輸出，為交易服務費 |
| ... | X 轉給 Y | 他人向 X 交易的輸出 | Y 帳戶可以使用該交易 | X 簽名確認 | 輸入減輸出，為交易服務費 |

需要注意，剛放進網路中的交易（深度為 0）並非是實時得到確認的。進入網路中的交易存在被推翻的可能性，一般要再生成幾個新的區塊後（深度大於 0）才認為該交易被確認。

下面分別介紹比特幣網路中的重要概念和主要設計思路。

### 重要概念

#### 帳戶/地址

比特幣採用了非對稱的加密算法，用戶自己保留私鑰，對自己發出的交易進行簽名確認，並公開公鑰。

比特幣的帳戶地址其實就是用戶公鑰經過一系列 Hash（HASH160，或先進行 SHA256，然後進行 RIPEMD160）及編碼運算後生成的 160 位（20 字節）的字符串。

一般地，也常常對帳戶地址串進行 Base58Check 編碼，並添加前導字節（表明支持哪種腳本）和 4 字節校驗字節，以提高可讀性和準確性。

*注：帳戶並非直接是公鑰內容，而是 Hash 後的值，避免公鑰過早公開後導致被破解出私鑰。*

#### 交易

交易是完成比特幣功能的核心概念，一條交易可能包括如下信息：

* 付款人地址：合法的地址，公鑰經過 SHA256 和 RIPEMD160 兩次 Hash，得到 160 位 Hash 串；
* 付款人對交易的簽字確認：確保交易內容不被篡改；
* 付款人資金的來源交易 ID：從哪個交易的輸出作為本次交易的輸入；
* 交易的金額：多少錢，跟輸入的差額為交易的服務費；
* 收款人地址：合法的地址；
* 時間戳：交易何時能生效。

網路中節點收到交易信息後，將進行如下檢查：

* 交易是否已經處理過；
* 交易是否合法。包括地址是否合法、發起交易者是否是輸入地址的合法擁有者、是否是 UTXO；
* 交易的輸入之和是否大於輸出之和。

檢查都通過，則將交易標記為合法的未確認交易，並在網路內進行廣播。

用戶可以從 blockchain.info 網站查看實時的交易信息，一個示例交易的內容如下圖所示。

![比特幣交易的例子](_images/transaction_example.png)

#### 交易腳本

[腳本（Script）](https://en.bitcoin.it/wiki/Script) 是保障交易完成（主要用於檢驗交易是否合法）的核心機制，當所依附的交易發生時被觸發。通過腳本機制而非寫死交易過程，比特幣網路實現了一定的可擴展性。比特幣腳本語言是一種非圖靈完備的語言，類似 [Forth](https://en.wikipedia.org/wiki/Forth_programming_language) 語言。

一般每個交易都會包括兩個腳本：負責輸入的解鎖腳本（scriptSig）和負責輸出的鎖定腳本（scriptPubKey）。

輸出腳本一般由付款方對交易設置鎖定，用來對能動用這筆交易的輸出（例如，要花費該交易的輸出）的對象（收款方）進行權限控制，例如限制必須是某個公鑰的擁有者才能花費這筆交易。

認領腳本則用來證明自己可以滿足交易輸出腳本的鎖定條件，即對某個交易的輸出（比特幣）的擁有權。

輸出腳本目前支持兩種類型：

* [P2PKH](https://en.bitcoin.it/wiki/Script#Standard_Transaction_to_Bitcoin_address_.28pay-to-pubkey-hash.29)：Pay-To-Public-Key-Hash，允許用戶將比特幣發送到一個或多個典型的比特幣地址上（證明擁有該公鑰），前導字節一般為 0x00； 
* P2SH：Pay-To-Script-Hash，支付者創建一個輸出腳本，裡邊包含另一個腳本（認領腳本）的雜湊，一般用於需要多人簽名的場景，前導字節一般為 0x05； 

以 P2PKH 為例，輸出腳本的格式為

```
scriptPubKey: OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

其中，OP_DUP 是複製棧頂元素；OP_HASH160 是計算 hash 值；OP_EQUALVERIFY 判斷棧頂兩元素是否相等；OP_CHECKSIG 判斷簽名是否合法。這條指令實際上保證了只有 pubKey 的擁有者才能合法引用這個輸出。

另外一個交易如果要花費這個輸出，在引用這個輸出的時候，需要提供認領腳本格式為

```
scriptSig: <sig> <pubKey>
```

其中，<sig> 是拿 pubKey 對應的私鑰對交易（全部交易的輸出、輸入和腳本）Hash 值進行簽名，pubKey 的 Hash 值需要等於 pubKeyHash。

進行交易驗證時，會按照先 scriptSig 後 scriptPubKey 的順序進行依次入棧處理，即完整指令為：

```
<sig> <pubKey> OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

讀者可以按照棧的過程來進行推算，理解整個腳本的驗證過程。

引入腳本機制帶來了靈活性，但也引入了更多的安全風險。比特幣腳本支持的指令集十分簡單，基於棧的處理方式，並且非圖靈完備，此外還添加了額外的一些限制（大小限制等）。

#### 區塊

比特幣區塊鏈的一個區塊不能超過 1 MB，將主要包括如下內容：

* 區塊大小：4 字節；
* 區塊頭：80 字節：
* 交易個數計數器：1~9 字節；
* 所有交易的具體內容，可變長，匹配 Merkle 樹葉子節點順序。

其中，區塊頭信息十分重要，包括：

* 版本號：4 字節；
* 上一個區塊頭的 Hash 值：鏈接到上一個合法的塊上，對其區塊頭進行兩次 SHA256 操作，32 字節；
* 本區塊所包含的所有交易的 Merkle 樹根的雜湊值：兩次 SHA256 操作，32 字節；
* 時間戳：4 字節；
* 難度指標：4 字節；
* Nonce：4 字節，PoW 問題的答案。

可見，要對區塊鏈的完整性進行檢查，只需要檢驗各個區塊頭部信息即可，無需獲取到具體的交易內容，這也是簡單交易驗證（Simple Payment Verification，SPV）的基本原理。另外，通過頭部的鏈接，提供時序關係的同時加大了對區塊中數據進行篡改的難度。


一個示例區塊如下圖所示。

![比特幣區塊的例子](_images/block_example.png)


### 創新設計

比特幣在設計上提出了很多創新點，主要考慮了避免作惡、採用負反饋調節和基於概率的共識機制等三個方面。

#### 如何避免作惡

基於經濟博弈原理。在一個開放的網路中，無法通過技術手段保證每個人都是合作的。但可以通過經濟博弈來讓合作者得到利益，讓非合作者遭受損失和風險。

實際上，博弈論早已被廣泛應用到眾多領域。

一個經典的例子是兩個人來分一個蛋糕，如果都想拿到較大的一塊，在沒有第三方的前提下，該怎麼制定規則才公平？

最簡單的一個方案是任意一個人負責分配蛋糕，並且這個人後挑選。

*注：如果推廣到 N 個人呢？*

比特幣網路中所有試圖參與者（礦工）都首先要付出挖礦的代價，進行算力消耗，越想拿到新區塊的決定權，意味著抵押的算力越多。一旦失敗，這些算力都會被沒收掉，成為沉沒成本。當網路中存在眾多參與者時，個體試圖拿到新區塊決定權要付出的算力成本是巨大的，意味著進行一次作惡付出的代價已經超過可能帶來的好處。

#### 負反饋調節

比特幣網路在設計上，很好的體現了負反饋的控制論基本原理。

比特幣網路中礦工越多，系統就越穩定，比特幣價值就越高，但挖到礦的概率會降低。

反之，網路中礦工減少，會讓系統更容易導致被攻擊，比特幣價值越低，但挖到礦的概率會提高。

因此，比特幣的價格理論上應該穩定在一個合適的值（網路穩定性也會穩定在相應的值），這個價格乘以挖到礦的概率，恰好達到礦工的收益預期。

從長遠角度看，硬件成本是下降的，但每個區塊的比特幣獎勵每隔 4 年減半，最終將在 2140 年達到 2100 萬枚，之後將完全依靠交易的服務費來鼓勵礦工對網路的維護。

*注：比特幣最小單位是“聰”，即 10^(-8) 比特幣，總“聰”數為 2.1E15。對於 64 位處理器來說，高精度浮點計數的限制導致單個數值不能超過 2^53 約等於 9E15。*

#### 共識機制

傳統共識問題往往是考慮在一個相對封閉的分散式系統中，允許同時存在正常節點、故障節點，如何快速達成一致。

對於比特幣網路來說，它是完全開放的，可能面向各種攻擊情況，同時基於 Internet 的網路質量只能保證“盡力而為”，導致問題更加複雜，傳統的一致性算法在這種場景下難以實用。

因此，比特幣網路不得不對共識的目標和過程都進行了一系列限制，提出了基於 Proof of Work（PoW）的共識機制。

首先是不實現面向最終確認的共識，而是基於概率、隨時間逐步增強確認的共識。現有達成的結果在理論上都可能被推翻，只是攻擊者要付出的代價隨時間而指數級上升，被推翻的可能性隨之指數級的下降。

此外，考慮到 Internet 的尺度，達成共識的時間相對比較長。按照區塊（一組交易）來進行階段性的確認（快照），提高網路整體的可用性。

最後，限制網路中共識的噪音。通過進行大量的 Hash 計算和少數的合法結果來限制合法提案的個數，進一步提高網路中共識的穩定性。
