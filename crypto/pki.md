## PKI 體系

按照 X.509 規範，公鑰可以通過證書機制來進行保護，但證書的生成、分發、撤銷等步驟並未涉及。

實際上，要實現安全地管理、分發證書需要遵循 PKI（Public Key Infrastructure）體系。該體系解決了證書生命週期相關的認證和管理問題。

需要注意，PKI 是建立在公私鑰基礎上實現安全可靠傳遞消息和身份確認的一個通用框架，並不代表某個特定的密碼學技術和流程。實現了 PKI 規範的平臺可以安全可靠地管理網絡中用戶的密鑰和證書。目前包括多個具體實現和規範，知名的有 RSA 公司的 PKCS（Public Key Cryptography Standards）標準和 OpenSSL 等開源工具。

### PKI 基本組件

一般情況下，PKI 至少包括如下核心組件：

* CA（Certification Authority）：負責證書的頒發和吊銷（Revoke），接收來自 RA 的請求，是最核心的部分；
* RA（Registration Authority）：對用戶身份進行驗證，校驗數據合法性，負責登記，審核過了就發給 CA；
* 證書數據庫：存放證書，多采用 X.500 系列標準格式。可以配合LDAP 目錄服務管理用戶信息。

其中，CA 是最核心的組件，主要完成對證書信息的維護。

常見的操作流程為，用戶通過 RA 登記申請證書，提供身份和認證信息等；CA 審核後完成證書的製造，頒發給用戶。用戶如果需要撤銷證書則需要再次向 CA 發出申請。

### 證書的簽發

CA 對用戶簽發證書實際上是對某個用戶公鑰，使用 CA 的私鑰對其進行簽名。這樣任何人都可以用 CA 的公鑰對該證書進行合法性驗證。驗證成功則認可該證書中所提供的用戶公鑰內容，實現用戶公鑰的安全分發。

用戶證書的簽發可以有兩種方式。可以由用戶自己生成公鑰和私鑰，然後 CA 來對公鑰內容進行簽名（只有用戶持有私鑰）；也可以由 CA 直接來生成證書（內含公鑰）和對應的私鑰發給用戶（用戶和 CA 均持有私鑰）。

前者情況下，用戶一般會首先自行生成一個私鑰和證書申請文件（Certificate Signing Request，即 csr 文件），該文件中包括了用戶對應的公鑰和一些基本信息，如通用名（common name，即 cn）、組織信息、地理位置等。CA 只需要對證書請求文件進行簽名，生成證書文件，頒發給用戶即可。整個過程中，用戶可以保持私鑰信息的私密性，不會被其他方獲知（包括 CA 方）。

生成證書申請文件的過程並不複雜，用戶可以很容易地使用開源軟件 openssl 來生成 csr 文件和對應的私鑰文件。

例如，安裝 openssl 後可以執行如下命令來生成私鑰和對應的證書請求文件：

```bash
$ openssl req -new -keyout private.key -out for_request.csr
Generating a 1024 bit RSA private key
...........................++++++
............................................++++++
writing new private key to 'private.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Blockchain
Organizational Unit Name (eg, section) []:Dev
Common Name (e.g. server FQDN or YOUR name) []:example.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

生成過程中需要輸入地理位置、組織、通用名等信息。生成的私鑰和 csr 文件默認以 PEM 格式存儲，內容為 base64 編碼。

如生成的 csr 文件內容可能為：

```bash
$ cat for_request.csr                                                                                                                                       
-----BEGIN CERTIFICATE REQUEST-----
MIIBrzCCARgCAQAwbzELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaWppbmcxEDAO
BgNVBAcTB0JlaWppbmcxEzARBgNVBAoTCkJsb2NrY2hhaW4xDDAKBgNVBAsTA0Rl
djEZMBcGA1UEAxMQeWVhc3kuZ2l0aHViLmNvbTCBnzANBgkqhkiG9w0BAQEFAAOB
jQAwgYkCgYEA8fzVl7MJpFOuKRH+BWqJY0RPTQK4LB7fEgQFTIotO264ZlVJVbk8
Yfl42F7dh/8SgHqmGjPGZgDb3hhIJLoxSOI0vJweU9v6HiOVrFWE7BZEvhvEtP5k
lXXEzOewLvhLMNQpG0kBwdIh2EcwmlZKcTSITJmdulEvoZXr/DHXnyUCAwEAAaAA
MA0GCSqGSIb3DQEBBQUAA4GBAOtQDyJmfP64anQtRuEZPZji/7G2+y3LbqWLQIcj
IpZbexWJvORlyg+iEbIGno3Jcia7lKLih26lr04W/7DHn19J6Kb/CeXrjDHhKGLO
I7s4LuE+2YFSemzBVr4t/g24w9ZB4vKjN9X9i5hc6c6uQ45rNlQ8UK5nAByQ/TWD
OxyG
-----END CERTIFICATE REQUEST-----
```

openssl 工具提供了查看 PEM 格式文件明文的功能，如使用如下命令可以查看生成的 csr 文件的明文：

```bash
$ openssl req -in for_request.csr -noout -text
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=CN, ST=Beijing, L=Beijing, O=Blockchain, OU=Dev, CN=yeasy.github.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (1024 bit)
                Modulus (1024 bit):
                    00:f1:fc:d5:97:b3:09:a4:53:ae:29:11:fe:05:6a:
                    89:63:44:4f:4d:02:b8:2c:1e:df:12:04:05:4c:8a:
                    2d:3b:6e:b8:66:55:49:55:b9:3c:61:f9:78:d8:5e:
                    dd:87:ff:12:80:7a:a6:1a:33:c6:66:00:db:de:18:
                    48:24:ba:31:48:e2:34:bc:9c:1e:53:db:fa:1e:23:
                    95:ac:55:84:ec:16:44:be:1b:c4:b4:fe:64:95:75:
                    c4:cc:e7:b0:2e:f8:4b:30:d4:29:1b:49:01:c1:d2:
                    21:d8:47:30:9a:56:4a:71:34:88:4c:99:9d:ba:51:
                    2f:a1:95:eb:fc:31:d7:9f:25
                Exponent: 65537 (0x10001)
        Attributes:
            a0:00
    Signature Algorithm: sha1WithRSAEncryption
        eb:50:0f:22:66:7c:fe:b8:6a:74:2d:46:e1:19:3d:98:e2:ff:
        b1:b6:fb:2d:cb:6e:a5:8b:40:87:23:22:96:5b:7b:15:89:bc:
        e4:65:ca:0f:a2:11:b2:06:9e:8d:c9:72:26:bb:94:a2:e2:87:
        6e:a5:af:4e:16:ff:b0:c7:9f:5f:49:e8:a6:ff:09:e5:eb:8c:
        31:e1:28:62:ce:23:bb:38:2e:e1:3e:d9:81:52:7a:6c:c1:56:
        be:2d:fe:0d:b8:c3:d6:41:e2:f2:a3:37:d5:fd:8b:98:5c:e9:
        ce:ae:43:8e:6b:36:54:3c:50:ae:67:00:1c:90:fd:35:83:3b:
        1c:86
```

需要注意，用戶自行生成私鑰情況下，私鑰文件一旦丟失，CA 方由於不持有私鑰信息，無法進行恢復，意味著通過該證書中公鑰加密的內容將無法被解密。

### 證書的吊銷

證書超出有效期後會作廢，用戶也可以主動向 CA 申請吊銷某證書文件。

由於 CA 無法強制收回已經頒發出去的數位證書，因此為了實現證書的作廢，往往還需要維護一個吊銷證書列表（Certificate Revocation List，CRL），用於記錄已經吊銷的證書序號。

因此，通常情況下，當對某個證書進行驗證時，需要首先檢查該證書是否已經記錄在列表中。如果存在，則該證書無法通過驗證。如果不在，則繼續進行後續的證書驗證過程。

為了方便同步吊銷列表信息，IETF 提出了在線證書狀態協議（Online Certificate Status Protocol，或 OCSP），支持該協議的服務可以實時在線查詢吊銷的證書列表信息。
