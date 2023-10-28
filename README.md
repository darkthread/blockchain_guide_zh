# 區塊鏈技術指南 - 繁中
![https://github.com/yeasy/blockchain_guide](https://img.shields.io/badge/Version-1.1.8-blue.svg)
![](https://img.shields.io/badge/Language-Traditional%20Chinese-orange.svg)
![https://github.com/yeasy](https://img.shields.io/badge/Author-Baohua%20Yang-yellowgreen.svg)
![https://github.com/ChenPoWei](https://img.shields.io/badge/Translator-Chen%20Po%20Wei-blue.svg)

<center>
<img src="https://github.com/ChenPoWei/blockchain_guide_zh/raw/master/_images/blockchain_book.png" width="30%" height="30%" />
</center>

區塊鏈技術是現代金融科技（Fintech）領域的一項重要技術創新。

作為分散式記帳（Distributed Ledger Technology，DLT）系統的核心技術，區塊鏈被認為在金融、徵信、物聯網、經濟貿易、資產管理等眾多領域都擁有廣泛的應用前景。區塊鏈技術自身尚處於快速發展的初級階段，現有區塊鏈系統在設計和實現中利用了分散式系統、密碼學、博弈論、網路協議等諸多學科的知識，為學習原理和實踐應用都帶來了不小的挑戰。

本書希望可以探索區塊鏈概念的來龍去脈，剝繭抽絲，系統剖析關鍵技術原理，同時講解實踐應用。在開發相關開源分散式帳本平臺（如超級帳本），以及設計基於區塊鏈的企業方案過程中，筆者積累了一些實踐經驗，也通過本書一併分享出來，希望能推動區塊鏈技術的早日成熟和更多應用場景的出現。

## 目錄

* [前言](README.md)
* [修訂記錄](revision.md)
* [如何貢獻](contribute.md)
* [區塊鏈的誕生](born/README.md)
  * [記帳科技的千年演化](born/ledger_history.md)
  * [分散式記帳與區塊鏈](born/dlt_problem.md)
  * [站在前人肩膀上的比特幣](born/bitcoin.md)
  * [區塊鏈的商業價值](born/business.md)
  * [本章小結](born/summary.md)
* [核心技術概覽](overview/README.md)
  * [定義與原理](overview/definition.md)
  * [技術的演化與分類](overview/dlt.md)
  * [關鍵問題和挑戰](overview/challenge.md)
  * [趨勢與展望](overview/vision.md)
  * [認識上的誤區](overview/misunderstand.md)
  * [本章小結](overview/summary.md)
* [典型應用場景](scenario/README.md)
  * [應用場景概覽](scenario/overview.md)
  * [金融服務](scenario/finance.md)
  * [徵信和權屬管理](scenario/trust.md)
  * [資源共享](scenario/sharing.md)
  * [貿易管理](scenario/trading.md)
  * [物聯網](scenario/iot.md)
  * [其它場景](scenario/others.md)
  * [本章小結](scenario/summary.md)
* [分散式系統核心技術](distribute_system/README.md)
  * [一致性問題](distribute_system/intro.md)
  * [共識算法](distribute_system/consensus.md)
  * [FLP 不可能原理](distribute_system/flp.md)
  * [CAP 原理](distribute_system/cap.md)
  * [ACID 原則與多階段提交](distribute_system/acid.md)
  * [Paxos 算法與 Raft 算法](distribute_system/paxos.md)
  * [拜占庭問題與算法](distribute_system/bft.md)
  * [可靠性指標](distribute_system/availability.md)
  * [本章小結](distribute_system/summary.md)
* [密碼學與安全技術](crypto/README.md)
  * [密碼學簡史](crypto/history.md)
  * [Hash 算法與數位摘要](crypto/hash.md)
  * [加解密算法](crypto/algorithm.md)
  * [消息認證碼與數位簽名](crypto/signature.md)
  * [數位證書](crypto/cert.md)
  * [PKI 體系](crypto/pki.md)
  * [Merkle 樹結構](crypto/merkle_trie.md)
  * [Bloom Filter 結構](crypto/bloom_filter.md)
  * [同態加密](crypto/homoencryption.md)
  * [其它問題](crypto/others.md)
  * [本章小結](crypto/summary.md)
* [比特幣 —— 區塊鏈思想誕生的搖籃](bitcoin/README.md)
  * [比特幣項目簡介](bitcoin/intro.md)
  * [實體貨幣到加密數位貨幣](bitcoin/currency.md)
  * [基本原理和設計](bitcoin/design.md)
  * [挖礦過程](bitcoin/mining.md)
  * [共識機制](bitcoin/consensus.md)
  * [閃電網路](bitcoin/lightning_network.md)
  * [側鏈](bitcoin/sidechain.md)
  * [熱點問題](bitcoin/other.md)
  * [相關工具](bitcoin/tools.md)
  * [本章小結](bitcoin/summary.md)
* [以太坊 —— 掙脫加密貨幣的枷鎖](ethereum/README.md)
  * [以太坊項目簡介](ethereum/intro.md)
  * [核心概念](ethereum/concept.md)
  * [主要設計](ethereum/design.md)
  * [相關工具](ethereum/tools.md)
  * [安裝客戶端](ethereum/install.md)
  * [使用智能合約](ethereum/smart_contract.md)
  * [智能合約案例：投票](ethereum/contract_example.md)
  * [本章小結](ethereum/summary.md)
* [超級帳本 —— 面向企業的分散式帳本](hyperledger/README.md)
  * [超級帳本項目簡介](hyperledger/intro.md)
  * [社區組織結構](hyperledger/community.md)
  * [頂級項目介紹](hyperledger/top_project.md)
  * [開發必備工具](hyperledger/tools.md)
  * [貢獻代碼](hyperledger/contribute.md)
  * [本章小結](hyperledger/summary.md)

* [Fabric 部署與管理](fabric/README.md)
  * [簡介](fabric/intro.md)
  * [使用 Fabric 1.0 版本](fabric/1.0.md)
  * [使用 Fabric SDK Node](fabric/sdk-node.md)
  * [Fabric v0.6](fabric/v0.6/README.md)
    * [安裝部署](fabric/v0.6/install.md)
    * [使用 chaincode](fabric/v0.6/usage.md)
    * [權限管理](fabric/v0.6/membersrcv-usage.md)
    * [Python 客戶端](fabric/v0.6/hyperledger-py.md)
  * [小結](fabric/summary.md)
* [區塊鏈應用開發](app_dev/README.md)
  * [簡介](app_dev/intro.md)
  * [鏈上代碼工作原理](app_dev/chaincode.md)
  * [示例一：信息公證](app_dev/chaincode_example01.md)
  * [示例二：交易資產](app_dev/chaincode_example02.md)
  * [示例三：數位貨幣發行與管理](app_dev/chaincode_example03.md)
  * [示例四：學歷認證](app_dev/chaincode_example04.md)
  * [示例五：社區能源共享](app_dev/chaincode_example05.md)
  * [小結](app_dev/summary.md)
* [Fabric 架構與設計](fabric_design/README.md)
  * [簡介](fabric_design/intro.md)
  * [架構設計](fabric_design/design.md)
  * [消息協議](fabric_design/protocol.md)
  * [小結](fabric_design/summary.md)
* [區塊鏈服務平臺設計](baas/README.md)
  * [簡介](baas/intro.md)
  * [IBM Bluemix 雲區塊鏈服務](baas/bluemix.md)
  * [微軟 Azure 雲區塊鏈服務](baas/azure.md)
  * [使用超級帳本 Cello 搭建區塊鏈服務](baas/cello.md)
  * [本章小結](baas/summary.md)
* [性能與評測](evaluation/README.md)
  * [簡介](evaluation/intro.md)
  * [Hyperledger Fabric v0.6](evaluation/hyperledger.md)
  * [小結](evaluation/summary.md)
* [附錄](appendix/README.md)
  * [術語](appendix/terms.md)
  * [常見問題](appendix/faq.md)
  * [Golang 開發相關](appendix/golang/README.md)
    * [安裝與配置 Golang 環境](appendix/golang/install.md)
    * [編輯器與 IDE](appendix/golang/ide.md)
    * [高效開發工具](appendix/golang/tools.md)
  * [ProtoBuf 與 gRPC](appendix/grpc.md)
  * [參考資源鏈接](appendix/resources.md)

## 閱讀使用
本書適用於對區塊鏈技術感興趣，且具備一定資訊和金融基礎知識的讀者；無技術背景的讀者也可以從中瞭解到區塊鏈的應用現狀。

* 線上閱讀：[GitBook](https://poweichen.gitbook.io/blockchain-guide-zh/) 或 [GitHub](https://github.com/ChenPoWei/blockchain_guide/blob/master/SUMMARY.md)

如果發現疏漏，歡迎提交到 [勘誤表](https://github.com/yeasy/blockchain_guide/wiki/%E3%80%8A%E5%8C%BA%E5%9D%97%E9%93%BE%E5%8E%9F%E7%90%86%E3%80%81%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%BA%94%E7%94%A8%E3%80%8B%E5%8B%98%E8%AF%AF%E8%A1%A8)。

## 參與貢獻
區塊鏈技術自身仍在快速發展中，生態環境也在蓬勃成長。歡迎 [參與維護項目](contribute.md)。

* [修訂記錄](revision.md)
* [貢獻者名單](https://github.com/yeasy/blockchain_guide/graphs/contributors)

