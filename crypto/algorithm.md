## 加解密算法

加解密算法是現代密碼學核心技術，從設計理念和應用場景上可以分為兩大基本類型，如下表所示。

算法類型 | 特點 | 優勢 | 缺陷 | 代表算法
------  | --- | --- | --- | -------
對稱加密 | 加解密的密鑰相同 | 計算效率高，加密強度高 | 需提前共享密鑰，易洩露 | DES、3DES、AES、IDEA
非對稱加密 | 加解密的密鑰不相同 | 無需提前共享密鑰 | 計算效率低，存在中間人攻擊可能 | RSA、ElGamal、橢圓曲線算法

### 加解密系統基本組成

現代加解密系統的典型組件包括算法和密鑰（包括加密密鑰、解密密鑰）。

其中，加解密算法自身是固定不變的，並且一般是公開可見的；密鑰則是最關鍵的信息，需要安全地保存起來，甚至通過特殊硬件進行保護。一般來說，密鑰需要在加密前按照特定算法隨機生成，長度越長，則加密強度越大。

加解密的典型過程如下圖所示。加密過程中，通過加密算法和加密密鑰，對明文進行加密，獲得密文；解密過程中，通過解密算法和解密密鑰，對密文進行解密，獲得明文。

![加解密的基本過程](_images/basic_flow.png)

根據加解密過程中所使用的密鑰是否相同，算法可以分為對稱加密（Symmetric Cryptography，又稱共有密鑰加密，Common-key cryptography）和非對稱加密（Asymmetric Cryptography，又稱公鑰加密，Public-key Cryptography）。兩種模式適用於不同的需求，形成互補。某些場景下可以組合使用，形成混合加密機制。

### 對稱加密算法
對稱加密算法，顧名思義，加密和解密過程的密鑰是相同的。

該類算法優點是加解密效率（速度快，空間佔用小）和加密強度都很高。

缺點是參與方需要提前持有密鑰，一旦有人洩露則系統安全性被破壞；另外如何在不安全通道中提前分發密鑰也是個問題，需要藉助額外的 Diffie–Hellman 協商協議或非對稱加密算法來實現。

對稱密碼從實現原理上可以分為兩種：分組密碼和序列密碼。前者將明文切分為定長數據塊作為基本加密單位，應用最為廣泛。後者則每次只對一個字節或字符進行加密處理，且密碼不斷變化，只用在一些特定領域（如數位媒介的加密）。

**分組對稱加密**代表算法包括 DES、3DES、AES、IDEA 等。

* DES（Data Encryption Standard）：經典的分組加密算法，最早是 1977 年美國聯邦信息處理標準（FIPS）採用 FIPS-46-3，將 64 位明文加密為 64 位的密文。其密鑰長度為 64 位（包括 8 位校驗碼），現在已經很容易被暴力破解；
* 3DES：三重 DES 操作：加密 --> 解密 --> 加密，處理過程和加密強度優於 DES，但現在也被認為不夠安全；
* AES（Advanced Encryption Standard）：由美國國家標準研究所（NIST）採用，取代 DES 成為對稱加密實現的標準，1997~2000 年 NIST 從 15 個候選算法中評選 Rijndael 算法（由比利時密碼學家 Joan Daemon 和 Vincent Rijmen 發明）作為 AES，標準為 FIPS-197。AES 也是分組算法，分組長度為 128、192、256 位三種。AES 的優勢在於處理速度快，整個過程可以數學化描述，目前尚未有有效的破解手段；
* IDEA（International Data Encryption Algorithm）：1991 年由密碼學家 James Massey 與來學嘉共同提出。設計類似於 3DES，密鑰長度增加到 128 位，具有更好的加密強度。

**序列加密**，又稱流加密。1949 年，Claude Elwood Shannon（信息論創始人）首次證明，要實現絕對安全的完善保密性（Perfect Secrecy），可以通過“一次性密碼本”的對稱加密處理。即通信雙方每次使用跟明文等長的隨機密鑰串對明文進行加密處理。序列密碼採用了類似的思想，每次通過偽隨機數生成器來生成偽隨機密鑰串。代表算法包括 RC4 等。

總結下，對稱加密算法適用於大量數據的加解密過程；不能用於簽名場景；並且需要提前安全地分發密鑰。

*注：分組加密每次只能處理固定長度的明文，因此對於過長的內容需要採用一定模式進行分割，《實用密碼學》一書中推薦使用密文分組鏈（Cipher Block Chain，CBC）、計數器（Counter，CTR）等模式。*

### 非對稱加密算法
非對稱加密是現代密碼學的偉大發明，它有效解決了對稱加密需要安全分發密鑰的問題。

顧名思義，非對稱加密中，加密密鑰和解密密鑰是不同的，分別被稱為公鑰（Public Key）和私鑰（Private Key）。私鑰一般通過隨機數算法生成，公鑰可以根據私鑰生成。

其中，公鑰一般是公開的，他人可獲取的；私鑰則是個人持有並且要嚴密保護，不能被他人獲取。

非對稱加密算法優點是公私鑰分開，無需安全通道來分發密鑰。缺點是處理速度（特別是生成密鑰和解密過程）往往比較慢，一般比對稱加解密算法慢 2~3 個數量級；同時加密強度也往往不如對稱加密。

非對稱加密算法的安全性往往基於數學問題，包括大數質因子分解、離散對數、橢圓曲線等經典數學難題。

代表算法包括：RSA、ElGamal、橢圓曲線（Elliptic Curve Crytosystems，ECC）、SM2 等系列算法。

* RSA：經典的公鑰算法，1978 年由 Ron Rivest、Adi Shamir、Leonard Adleman 共同提出，三人於 2002 年因此獲得圖靈獎。算法利用了對大數進行質因子分解困難的特性，但目前還沒有數學證明兩者難度等價，或許存在未知算法可以繞過大數分解而進行解密。
* ElGamal：由 Taher ElGamal 設計，利用了模運算下求離散對數困難的特性，比 RSA 產生密鑰更快。被應用在 PGP 等安全工具中。
* 橢圓曲線算法（Elliptic Curve Cryptography，ECC）：應用最廣也是強度最早的系列算法，基於對橢圓曲線上特定點進行特殊乘法逆運算（求離散對數）難以計算的特性。最早在 1985 年由 Neal Koblitz 和 Victor Miller 分別獨立提出。ECC 系列算法具有多種國際標準（包括 ANSI X9.63、NIST FIPS 186-2、IEEE 1363-2000、ISO/IEC 14888-3 等），一般被認為具備較高的安全性，但加解密過程比較費時。其中，密碼學家 Daniel J.Bernstein 於 2006 年提出的 Curve25519/Ed25519/X25519 等算法（分別解決加密、簽名和密鑰交換），由於其設計完全公開、性能突出等特點，近些年引起了廣泛關注和應用。
* SM2（ShangMi 2）：中國國家商用密碼系列算法標準，由中國密碼管理局於 2010 年 12 月 17 日發佈，同樣基於橢圓曲線算法，一般認為其安全強度優於 RSA 系列算法。

非對稱加密算法適用於簽名場景或密鑰協商過程，但不適於大量數據的加解密。除了 SM2 之外，大部分算法的簽名速度要比驗籤速度慢（1~2個數量級）。

RSA 類算法被認為已經很難抵禦現代計算設備的破解，一般推薦商用場景下密鑰至少為 2048 位。如果採用安全強度更高的橢圓曲線算法，256 位密鑰即可滿足絕大部分安全需求。

#### 選擇明文攻擊
細心的讀者可能會想到，非對稱加密中公鑰是公開的，因此任何人都可以利用它加密給定明文，獲取對應的密文，這就帶來選擇明文攻擊的風險。

為了規避這種風險，現代的非對稱加密算法（如 RSA、ECC）都引入了一定的保護機制：對同樣的明文使用同樣密鑰進行多次加密，得到的結果完全不同，這就避免了選擇明文攻擊的破壞。

在實現上可以有多種思路。一種是對明文先進行變形，添加隨機的字符串或標記，再對添加後結果進行處理。另外一種是先用隨機生成的臨時密鑰對明文進行對稱加密，然後再將對稱密鑰進行加密，即利用多層加密機制。

### 混合加密機制

混合加密機制同時結合了對稱加密和非對稱加密的優點。

該機制的主要過程為：先用非對稱加密（計算複雜度較高）協商出一個臨時的對稱加密密鑰（或稱會話密鑰），然後雙方再通過對稱加密算法（計算複雜度較低）對所傳遞的大量數據進行快速的加密處理。

典型的應用案例是網站中使用越來越普遍的通信協議 -- 安全超文本傳輸協議（Hyper Text Transfer Protocol Secure，HTTPS）。

與以明文方式傳輸數據的 HTTP 協議不同，HTTPS 在傳統的 HTTP 層和 TCP 層之間引入 Transport Layer Security/Secure Socket Layer（TLS/SSL）加密層來實現安全傳輸。

SSL 協議是HTTPS 初期採用的標準協議，最早由 Netscape 於 1994 年設計實現，其兩個主要版本（包括 v2.0 和 v3.0）曾得到大量應用。SSL 存在安全缺陷易受攻擊（如 POODLE 和 DROWN 攻擊），無法滿足現代安全需求，已於 2011 和 2015 年被 IETF 宣佈廢棄。基於 SSL 協議（v3.1），IETF 提出了改善的安全標準協議 TLS，成為目前廣泛採用的方案。2008 年 8 月，TLS 1.2 版本（RFC 5246）發佈，修正了之前版本的不少漏洞，極大增強了安全性；2018 年 8 月，TLS 1.3 版本（RFC 8446）發佈，提高了握手性能同時增強了安全性。商用場景推薦使用這兩個版本。除了 Web 服務外，TLS 協議也被廣泛應用到 FTP、Email、實時消息、音視頻通話等場景中。

採用 HTTPS 建立安全連接（TLS 握手協商過程）的基本步驟如下：

![TLS 握手協商過程](_images/tls_handshake.png)

* 客戶端瀏覽器發送握手信息到服務器，包括隨機數 R1、支持的加密算法套件（Cipher Suite）類型、協議版本、壓縮算法等。注意該過程傳輸為明文。
* 服務端返回信息，包括隨機數 R2、選定加密算法套件、協議版本，以及服務器證書。注意該過程為明文。
* 瀏覽器檢查帶有該網站公鑰的證書。該證書需要由第三方 CA 來簽發，瀏覽器和操作系統會預置權威 CA 的根證書。如果證書被篡改作假（中間人攻擊），很容易通過 CA 的證書驗證出來。
* 如果證書沒問題，則客戶端用服務端證書中公鑰加密隨機數 R3（又叫 Pre-MasterSecret），發送給服務器。此時，只有客戶端和服務器都擁有 R1、R2 和 R3 信息，基於隨機數 R1、R2 和 R3，雙方通過偽隨機數函數來生成共同的對稱會話密鑰 MasterSecret。
* 後續客戶端和服務端的通信都通過協商後的對稱加密（如 AES）進行保護。

可以看出，該過程是實現防止中間人竊聽和篡改的前提下完成會話密鑰的交換。為了保障前向安全性（Perfect Forward Secrecy），TLS 對每個會話連接都可以生成不同的密鑰，避免某個會話密鑰洩露後對其它會話連接產生安全威脅。需要注意，選用合適的加密算法套件對於 TLS 的安全性十分重要。要合理選擇安全強度高的算法組合，如 ECDHE-RSA 和 ECDHE-ECDSA 等，而不要使用安全性較差的 DES/3DES 等。

示例中對稱密鑰的協商過程採用了 RSA 非對稱加密算法，實踐中也可以通過 Diffie–Hellman（DH）協議來完成。

加密算法套件包括一組算法，包括交換、認證、加密、校驗等。

* 密鑰交換算法：負責協商對稱密鑰，常見類型包括 RSA、DH、ECDH、ECDHE 等；
* 證書籤名算法：負責驗證身份，常見類型包括 RSA、DSA、ECDSA 等；
* 加密數據算法：對建立連接的通信內容進行對稱加密，常見類型包括 AES 等;
* 消息認證信息碼（MAC）算法：創建報文摘要，驗證消息的完整性，常見類型包括 SHA 等。

一個典型的 TLS 密碼算法套件可能為 “TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384”，意味著：

* 協商過程算法是 ECDHE（Elliptic Curve Diffie–Hellman Ephemeral），基於橢圓曲線的短期 EH 交換，每次交換都用新的密鑰，保障前向安全性；
* 證書籤名算法是 ECDSA（Elliptic Curve Digital Signature Algorithm），基於橢圓曲線的簽名；
* 加密數據算法是 AES，密鑰的長度和初始向量的長度都是 256，模式是 CBC；
* 消息認證信息碼算法是 SHA，結果是 384 位。

目前，推薦選用如下的加密算法套件：

* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384
* TLS_DHE_RSA_WITH_AES_256_GCM_SHA384

*注：TLS 1.0 版本已被發現存在安全漏洞，NIST、HIPAA 於 2014 年公開建議停用該版本的 TLS 協議。*

### 離散對數與 Diffie–Hellman 密鑰交換協議

Diffie–Hellman（DH）密鑰交換協議是一個應用十分廣泛的協議，最早由惠特菲爾德·迪菲（Bailey Whitfield Diffie）和馬丁·赫爾曼（Martin Edward Hellman）於 1976 年提出。該協議可以實現在不安全信道中協商對稱密鑰。

DH 協議的設計基於著名的離散對數問題（Discrete Logarithm Problem，DLP）。離散對數問題是指對於一個很大的素數 p，已知 g 為 p 的模循環群的原根，給定任意 x，求解 X=g^x mod p 是可以很快獲取的。但在已知 p，g 和 X 的前提下，逆向求解 x 很難（目前沒有找到多項式時間實現的算法）。該問題同時也是 ECC 類加密算法的基礎。

DH 協議的基本交換過程如下，以 Alice 和 Bob 兩人協商為例：

* Alice 和 Bob 兩個人協商密鑰，先公開商定 p，g；
* Alice 自行選取私密的整數 x，計算 X=g^x mod p，發送 X 給 Bob；
* Bob 自行選取私密的整數 y，計算 Y=g^y mod p，發送 Y 給 A；
* Alice 根據 x 和 Y，求解共同密鑰 Z_A=Y^x mod p；
* Bob 根據 X 和 y，求解共同密鑰 Z_B=X^y mod p。

實際上，Alice 和 Bob 計算出來的結果將完全相同，因為在 mod p 的前提下，Y^x =(g^y)^x =g^(xy) = (g^x)^y=X^y。而信道監聽者在已知 p，g，X，Y 的前提下，無法求得 Z。


### 安全性

雖然很多加密算法的安全性建立在數學難題基礎之上，但並非所有算法的安全性都可以從數學上得到證明。

公認高安全的加密算法和實現往往是經過長時間充分實踐論證後，才被大家所認可，但不代表其絕對不存在漏洞。使用方式和參數不當，也會造成安全強度的下降。

另一方面，自行設計和發明未經過大規模驗證的加密算法是一種不太明智的行為。即便不公開算法加密過程，也很容易被分析和攻擊，無法在安全性上得到足夠保障。

實際上，現代密碼學算法的安全性是通過數學難題來提供的，並非通過對算法自身的實現過程進行保密。
