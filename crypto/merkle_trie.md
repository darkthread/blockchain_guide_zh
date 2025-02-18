## Merkle 樹結構


[默克爾樹](https://en.wikipedia.org/wiki/Merkle_tree)（又叫雜湊樹）是一種典型的二叉樹結構，由一個根節點、一組中間節點和一組葉節點組成。默克爾樹最早由 Merkle Ralf 在 1980 年提出，曾廣泛用於文件系統和 P2P 系統中。

其主要特點爲：

* 最下面的葉節點包含存儲數據或其雜湊值。
* 非葉子節點（包括中間節點和根節點）都是它的兩個孩子節點內容的雜湊值。

進一步地，默克爾樹可以推廣到多叉樹的情形，此時非葉子節點的內容爲它所有的孩子節點的內容的雜湊值。

默克爾樹逐層記錄雜湊值的特點，讓它具有了一些獨特的性質。例如，底層數據的任何變動，都會傳遞到其父節點，一層層沿着路徑一直到樹根。這意味樹根的值實際上代表了對底層所有數據的“數位摘要”。

目前，默克爾樹的典型應用場景包括如下幾種。

### 快速比較大量數據

對每組數據排序後構建默克爾樹結構。當兩個默克爾樹根相同時，則意味着所代表的兩組數據必然相同。否則，必然不同。

由於 Hash 計算的過程可以十分快速，預處理可以在短時間內完成。利用默克爾樹結構能帶來巨大的比較性能優勢。

### 快速定位修改

以下圖爲例，基於數據 D0……D3 構造默克爾樹，如果 D1 中數據被修改，會影響到 N1，N4 和 Root。

![Merkle 樹示例](_images/Merkle_tree.png)

因此，一旦發現某個節點如 Root 的數值發生變化，沿着 Root --> N4 --> N1，最多通過 O(lgN) 時間即可快速定位到實際發生改變的數據塊 D1。

### 零知識證明

仍以上圖爲例，如何向他人證明擁有某個數據 D0 而不暴露其它信息。挑戰者提供隨機數據 D1，D2 和 D3，或由證明人生成（需要加入特定信息避免被人複用證明過程）。

證明人構造如圖所示的默克爾樹，公佈 N1，N5，Root。驗證者自行計算 Root 值，驗證是否跟提供值一致，即可很容易檢測 D0 存在。整個過程中驗證者無法獲知與 D0 相關的額外信息。
