## 消息認證碼與數位簽名

消息認證碼和數位簽名技術通過對消息的摘要進行加密，可以防止消息被篡改和認證身份。

### 消息認證碼
消息認證碼（Hash-based Message Authentication Code，HMAC），利用對稱加密，對消息完整性（Integrity）進行保護。

基本過程為對某個消息，利用提前共享的對稱密鑰和 Hash 算法進行處理，得到 HMAC 值。該 HMAC 值持有方可以向對方證明自己擁有某個對稱密鑰，並且確保所傳輸消息內容未被篡改。

典型的 HMAC 生成算法包括 K，H，M 三個參數。K 為提前共享的對稱密鑰，H 為提前商定的 Hash 算法（如 SHA-256），M 為要傳輸的消息內容。三個參數缺失了任何一個，都無法得到正確的 HMAC 值。

消息認證碼可以用於簡單證明身份的場景。如 Alice、Bob 提前共享了 K 和 H。Alice 需要知曉對方是否為 Bob，可發送一段消息 M 給 Bob。Bob 收到 M 後計算其 HMAC 值並返回給 Alice，Alice 檢驗收到 HMAC 值的正確性可以驗證對方是否真是 Bob。

*注：例子中並沒有考慮中間人攻擊的情況，並假定信道是安全的。*

消息認證碼的主要問題是需要提前共享密鑰，並且當密鑰可能被多方同時擁有（甚至洩露）的場景下，無法追蹤消息的真實來源。如果採用非對稱加密算法，則能有效的解決這個問題，即數位簽名。


### 數位簽名
類似在紙質合同上進行簽名以確認合同內容和證明身份，數位簽名既可以證實某數位內容的完整性，又可以確認其來源（即不可抵賴，Non-Repudiation）。

一個典型的場景是，Alice 通過信道發給 Bob 一個文件（一份信息），Bob 如何獲知所收到的文件即為 Alice 發出的原始版本？Alice 可以先對文件內容進行摘要，然後用自己的私鑰對摘要進行加密（簽名），之後同時將文件和簽名都發給 Bob。Bob 收到文件和簽名後，用 Alice 的公鑰來解密簽名，得到數位摘要，與對文件進行摘要後的結果進行比對。如果一致，說明該文件確實是 Alice 發過來的（因為別人無法擁有 Alice 的私鑰），並且文件內容沒有被修改過（摘要結果一致）。

理論上所有的非對稱加密算法都可以用來實現數位簽名，實踐中常用算法包括 1991 年 8 月 NIST 提出的 DSA（Digital Signature Algorithm，基於 ElGamal 算法）和安全強度更高的 ECSDA（Elliptic Curve Digital Signature Algorithm，基於橢圓曲線算法）等。

除普通的數位簽名應用場景外，針對一些特定的安全需求，產生了一些特殊數位簽名技術，包括盲簽名、多重簽名、群簽名、環簽名等。

#### 盲簽名

盲簽名（Blind Signature），1982 年由 David Chaum 在論文《Blind Signatures for Untraceable Payment》中[提出](http://www.hit.bme.hu/~buttyan/courses/BMEVIHIM219/2009/Chaum.BlindSigForPayment.1982.PDF)。簽名者需要在無法看到原始內容的前提下對信息進行簽名。

盲簽名可以實現對所簽名內容的保護，防止簽名者看到原始內容；另一方面，盲簽名還可以實現防止追蹤（Unlinkability），簽名者無法將簽名內容和簽名結果進行對應。典型的實現包括 RSA 盲簽名算法等。

#### 多重簽名
多重簽名（Multiple Signature），即 n 個簽名者中，收集到至少 m 個（n >= m >= 1）的簽名，即認為合法。

其中，n 是提供的公鑰個數，m 是需要匹配公鑰的最少的簽名個數。

多重簽名可以有效地被應用在多人投票共同決策的場景中。例如雙方進行協商，第三方作為審核方。三方中任何兩方達成一致即可完成協商。

比特幣交易中就支持多重簽名，可以實現多個人共同管理某個帳戶的比特幣交易。

#### 群簽名

群簽名（Group Signature），即某個群組內一個成員可以代表群組進行匿名簽名。簽名可以驗證來自於該群組，卻無法準確追蹤到簽名的是哪個成員。

群簽名需要存在一個群管理員來添加新的群成員，因此存在群管理員可能追蹤到簽名成員身份的風險。

群簽名最早在 1991 年由 David Chaum 和 Eugene van Heyst 提出。

#### 環簽名

環簽名（Ring Signature），由 Rivest，Shamir 和 Tauman 三位密碼學家在 2001 年首次提出。環簽名屬於一種簡化的群簽名。

簽名者首先選定一個臨時的簽名者集合，集合中包括簽名者自身。然後簽名者利用自己的私鑰和簽名集合中其他人的公鑰就可以獨立的產生簽名，而無需他人的幫助。簽名者集合中的其他成員可能並不知道自己被包含在最終的簽名中。

環簽名在保護匿名性方面也具有很多用途。

### 安全性

數位簽名算法自身的安全性由數學問題進行保護。但在實踐中，各個環節的安全性都十分重要，一定要嚴格遵循標準流程。

例如，目前常見的數位簽名算法需要選取合適的隨機數作為配置參數，配置參數不合理的使用或洩露都會造成安全漏洞和風險。

2010 年 8 月，SONY 公司因為其 PS3 產品上採用十分安全的 ECDSA 進行簽名時，不慎採用了重複的隨機參數，導致私鑰被最終破解，造成重大經濟損失。
