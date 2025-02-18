## 共識機制

比特幣網路是完全公開的，任何人都可以匿名接入，因此共識協議的穩定性和防攻擊性十分關鍵。

比特幣區塊鏈採用了 Proof of Work（PoW）的機制來實現共識，該機制最早於 1998 年在 [B-money](http://www.weidai.com/bmoney.txt) 設計中提出。

目前，Proof of X 系列中比較出名的一致性協議包括 PoW、PoS 和 DPoS 等，都是通過經濟懲罰來限制惡意參與。

### 工作量證明

工作量證明，通過計算來猜測一個數值（nonce），使得拼湊上交易數據後內容的 Hash 值滿足規定的上限（來源於 hashcash）。由於 Hash 難題在目前計算模型下需要大量的計算，這就保證在一段時間內，系統中只能出現少數合法提案。反過來，能夠提出合法提案，也證明提案者確實已經付出了一定的工作量。

同時，這些少量的合法提案會在網路中進行廣播，收到的用戶進行驗證後，會基於用戶認為的最長鏈基礎上繼續難題的計算。因此，系統中可能出現鏈的分叉（Fork），但最終會有一條鏈成為最長的鏈。

Hash 問題具有不可逆的特點，因此，目前除了暴力計算外，還沒有有效的算法進行解決。反之，如果獲得符合要求的 nonce，則說明在概率上是付出了對應的算力。誰的算力多，誰最先解決問題的概率就越大。當掌握超過全網一半算力時，從概率上就能控制網路中鏈的走向。這也是所謂 `51%` 攻擊的由來。

參與 PoW 計算比賽的人，將付出不小的經濟成本（硬件、電力、維護等）。當沒有最終成為首個算出合法 nonce 值的“幸運兒”時，這些成本都將被沉沒掉。這也保障了，如果有人嘗試惡意破壞，需要付出大量的經濟成本。也有設計試圖將後算出結果者的算力按照一定比例摺合進下一輪比賽考慮。

有一個很直觀的超市付款的例子，可以說明為何這種經濟博弈模式會確保系統中最長鏈的唯一性。

![Pow 保證一致性](_images/pow.png)

假定超市只有一個出口，付款時需要排成一隊，可能有人不守規矩要插隊。超市管理員會檢查隊伍，認為最長的一條隊伍是合法的，並讓不合法的分叉隊伍重新排隊。新到來的人只要足夠理智，就會自覺選擇最長的隊伍進行排隊。這是因為，看到多條鏈的參與者往往認為目前越長的鏈具備越大的勝出可能性，從而更傾向於選擇長的鏈。

### 權益證明

權益證明（Proof of Stake，PoS），最早在 2013 年被提出，最早在 [Peercoin]() 系統中被實現，類似現實生活中的股東機制，擁有股份越多的人越容易獲取記帳權（同時越傾向於維護網路的正常工作）。

典型的過程是通過保證金（代幣、資產、名聲等具備價值屬性的物品即可）來對賭一個合法的塊成為新的區塊，收益為抵押資本的利息和交易服務費。提供證明的保證金（例如通過轉帳貨幣記錄）越多，則獲得記帳權的概率就越大。合法記帳者可以獲得收益。

PoS 試圖解決在 PoW 中大量資源被浪費的缺點，受到了廣泛關注。惡意參與者將存在保證金被罰沒的風險，即損失經濟利益。

一般的，對於 PoS 來說，需要掌握超過全網 1/3 的資源，才有可能左右最終的結果。這個也很容易理解，三個人投票，前兩人分別支持一方，這時候，第三方的投票將決定最終結果。

PoS 也有一些改進的算法，包括授權股權證明機制（DPoS），即股東們投票選出一個董事會，董事會中成員才有權進行代理記帳。這些算法在實踐中得到了不錯的驗證，但是並沒有理論上的證明。

2017 年 8 月，來自愛丁堡大學和康涅狄格大學的 Aggelos Kiayias 等學者在論文《Ouroboros: A Provably Secure Proof-of-Stake Blockchain Protocol》中提出了 Ouroboros 區塊鏈共識協議，該協議可以達到誠實行為的近似納什均衡，認為是首個可證實安全的 PoS 協議。

