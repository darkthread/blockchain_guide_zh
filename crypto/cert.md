## 數位證書

對於非對稱加密算法和數位簽名來說，很重要的步驟就是公鑰的分發。理論上任何人都可以獲取到公開的公鑰。然而這個公鑰文件有沒有可能是偽造的呢？傳輸過程中有沒有可能被篡改呢？一旦公鑰自身出了問題，則整個建立在其上的的安全性將不復成立。

數位證書機制正是為了解決這個問題，它就像日常生活中的證書一樣，可以確保所記錄信息的合法性。比如證明某個公鑰是某個實體（個人或組織）擁有，並且確保任何篡改都能被檢測出來，從而實現對用戶公鑰的安全分發。

根據所保護公鑰的用途，數位證書可以分為加密數位證書（Encryption Certificate）和簽名驗證數位證書（Signature Certificate）。前者往往用於保護用於加密用途的公鑰；後者則保護用於簽名用途的公鑰。兩種類型的公鑰也可以同時放在同一證書中。

一般情況下，證書需要由證書認證機構（Certification Authority，CA）來進行簽發和背書。權威的商業證書認證機構包括 DigiCert、GlobalSign、VeriSign 等。用戶也可以自行搭建本地 CA 系統，在私有網絡中進行使用。

### X.509 證書規範
一般的，一個數位證書內容可能包括證書域（證書的版本、序列號、簽名算法類型、簽發者信息、有效期、被簽發主體、**簽發的公開密鑰**）、CA 對證書的簽名算法和簽名值等。

目前使用最廣泛的標準為 ITU 和 ISO 聯合制定的 X.509 的 v3 版本規範（RFC 5280），其中定義瞭如下證書信息域：

* 版本號（Version Number）：規範的版本號，目前為版本 3，值為 0x2；
* 序列號（Serial Number）：由 CA 維護的為它所頒發的每個證書分配的唯一的序列號，用來追蹤和撤銷證書。只要擁有簽發者信息和序列號，就可以唯一標識一個證書。最大不能超過 20 個字節；
* 簽名算法（Signature Algorithm）：數位簽名所採用的算法，如 sha256WithRSAEncryption 或 ecdsa-with-SHA256；
* 頒發者（Issuer）：頒發證書單位的信息，如 “C=CN, ST=Beijing, L=Beijing, O=org.example.com, CN=ca.org.example.com”；
* 有效期（Validity）：證書的有效期限，包括起止時間（如 Not Before 2018-08-08-00-00UTC，Not After 2028-08-08-00-00UTC）；
* 被簽發主體（Subject）：證書擁有者的標識信息（Distinguished Name），如 “C=CN, ST=Beijing, L=Beijing, CN=personA.org.example.com”；
* 主體的公鑰信息（Subject Public Key Info）：所保護的公鑰相關的信息；
    * 公鑰算法（Public Key Algorithm）：公鑰採用的算法；
    * 主體公鑰（Subject Public Key）：公鑰的內容；
* 頒發者唯一號（Issuer Unique Identifier，可選）：代表頒發者的唯一信息，僅 2、3 版本支持，可選；
* 主體唯一號（Subject Unique Identifier，可選）：代表擁有證書實體的唯一信息，僅 2、3 版本支持，可選；
* 擴展（Extensions，可選）：可選的一些擴展。可能包括：
    * Subject Key Identifier：實體的密鑰標識符，區分實體的多對密鑰；
    * Basic Constraints：一般指明該證書是否屬於某個 CA；
    * Authority Key Identifier：頒發這個證書的頒發者的公鑰標識符；
    * Authority Information Access：頒發相關的服務地址，如頒發者證書獲取地址和吊銷證書列表信息查詢地址；
    * CRL Distribution Points：證書註銷列表的發佈地址；
    * Key Usage: 表明證書的用途或功能信息，如 Digital Signature、Key CertSign；
    * Subject Alternative Name：證書身份實體的別名，如該證書可以同樣代表 *.org.example.com，org.example.com，*.example.com，example.com 身份等。

此外，證書的頒發者還需要對證書內容利用自己的私鑰進行簽名，以防止他人篡改證書內容。

### 證書格式

X.509 規範中一般推薦使用 PEM（Privacy Enhanced Mail）格式來存儲證書相關的文件。證書文件的文件名後綴一般為 `.crt` 或 `.cer`，對應私鑰文件的文件名後綴一般為 `.key`，證書請求文件的文件名後綴為 `.csr`。有時候也統一用 `.pem` 作為文件名後綴。

PEM 格式採用文本方式進行存儲，一般包括首尾標記和內容塊，內容塊採用 base64 編碼。

例如，一個示例證書文件的 PEM 格式如下所示。

```
-----BEGIN CERTIFICATE-----
MIICMzCCAdmgAwIBAgIQIhMiRzqkCljq3ZXnsl6EijAKBggqhkjOPQQDAjBmMQsw
CQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy
YW5jaXNjbzEUMBIGA1UEChMLZXhhbXBsZS5jb20xFDASBgNVBAMTC2V4YW1wbGUu
Y29tMB4XDTE3MDQyNTAzMzAzN1oXDTI3MDQyMzAzMzAzN1owZjELMAkGA1UEBhMC
VVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBGcmFuY2lzY28x
FDASBgNVBAoTC2V4YW1wbGUuY29tMRQwEgYDVQQDEwtleGFtcGxlLmNvbTBZMBMG
ByqGSM49AgEGCCqGSM49AwEHA0IABCkIHZ3mJCEPbIbUdh/Kz3zWW1C9wxnZOwfy
yrhr6aHwWREW3ZpMWKUcbsYup5kbouBc2dvMFUgoPBoaFYJ9D0SjaTBnMA4GA1Ud
DwEB/wQEAwIBpjAZBgNVHSUEEjAQBgRVHSUABggrBgEFBQcDATAPBgNVHRMBAf8E
BTADAQH/MCkGA1UdDgQiBCBIA/DmemwTGibbGe8uWjt5hnlE63SUsXuNKO9iGEhV
qDAKBggqhkjOPQQDAgNIADBFAiEAyoMO2BAQ3c9gBJOk1oSyXP70XRk4dTwXMF7q
R72ijLECIFKLANpgWFoMoo3W91uzJeUmnbJJt8Jlr00ByjurfAvv
-----END CERTIFICATE-----
```

可以通過 openssl 工具來查看其內容。

```bash
# openssl x509 -in example.com-cert.pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            22:13:22:47:3a:a4:0a:58:ea:dd:95:e7:b2:5e:84:8a
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=US, ST=California, L=San Francisco, O=example.com, CN=example.com
        Validity
            Not Before: Apr 25 03:30:37 2017 GMT
            Not After : Apr 23 03:30:37 2027 GMT
        Subject: C=US, ST=California, L=San Francisco, O=example.com, CN=example.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:29:08:1d:9d:e6:24:21:0f:6c:86:d4:76:1f:ca:
                    cf:7c:d6:5b:50:bd:c3:19:d9:3b:07:f2:ca:b8:6b:
                    e9:a1:f0:59:11:16:dd:9a:4c:58:a5:1c:6e:c6:2e:
                    a7:99:1b:a2:e0:5c:d9:db:cc:15:48:28:3c:1a:1a:
                    15:82:7d:0f:44
                ASN1 OID: prime256v1
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign, CRL Sign
            X509v3 Extended Key Usage:
                Any Extended Key Usage, TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier:
                48:03:F0:E6:7A:6C:13:1A:26:DB:19:EF:2E:5A:3B:79:86:79:44:EB:74:94:B1:7B:8D:28:EF:62:18:48:55:A8
    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:21:00:ca:83:0e:d8:10:10:dd:cf:60:04:93:a4:d6:
         84:b2:5c:fe:f4:5d:19:38:75:3c:17:30:5e:ea:47:bd:a2:8c:
         b1:02:20:52:8b:00:da:60:58:5a:0c:a2:8d:d6:f7:5b:b3:25:
         e5:26:9d:b2:49:b7:c2:65:af:4d:01:ca:3b:ab:7c:0b:ef
```

此外，還有 DER（Distinguished Encoding Rules）格式，是採用二進制對證書進行保存，可以與 PEM 格式互相轉換。

### 證書信任鏈

證書中記錄了大量信息，其中最重要的包括 `簽發的公開密鑰` 和 `CA 數位簽名` 兩個信息。因此，只要使用 CA 的公鑰再次對這個證書進行簽名比對，就能證明所記錄的公鑰是否合法。

讀者可能會想到，怎麼證明用來驗證對實體證書進行簽名的 CA 公鑰自身是否合法呢？畢竟在獲取 CA 公鑰的過程中，它也可能被篡改掉。

實際上，CA 的公鑰是否合法，一方面可以通過更上層的 CA 頒發的證書來進行認證；另一方面某些根 CA（Root CA）可以通過預先分發證書來實現信任基礎。例如，主流操作系統和瀏覽器裡面，往往會提前預置一些權威 CA 的證書（通過自身的私鑰簽名，系統承認這些是合法的證書）。之後所有基於這些 CA 認證過的中間層 CA（Intermediate CA）和後繼 CA 都會被驗證合法。這樣就從預先信任的根證書，經過中間層證書，到最底下的實體證書，構成一條完整的證書信任鏈。

某些時候用戶在使用瀏覽器訪問某些網站時，可能會被提示是否信任對方的證書。這說明該網站證書無法被當前系統中的證書信任鏈進行驗證，需要進行額外檢查。另外，當信任鏈上任一證書不可靠時，則依賴它的所有後繼證書都將失去保障。

可見，證書作為公鑰信任的基礎，對其生命週期進行安全管理十分關鍵。後面章節將介紹的 PKI 體系提供了一套完整的證書管理的框架，包括生成、頒發、撤銷過程等。
