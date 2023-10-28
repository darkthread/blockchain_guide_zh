## 智能合約案例：投票

本節將介紹一個用 Solidity 語言編寫的智能合約案例。代碼來源於 [Solidity 官方文檔](https://solidity.readthedocs.io/en/latest/index.html) 中的示例。

該智能合約實現了一個自動化的、透明的投票應用。投票發起人可以發起投票，將投票權賦予投票人；投票人可以自己投票，或將自己的票委託給其他投票人；任何人都可以公開查詢投票的結果。

### 智能合約代碼

實現上述功能的合約代碼如下所示，並不複雜，語法跟 JavaScript 十分類似。

```js
pragma solidity ^0.4.11;

contract Ballot {
    struct Voter {
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }

    struct Proposal {
        bytes32 name;
        uint voteCount;
    }

    address public chairperson;
    mapping(address => Voter) public voters;
    Proposal[] public proposals;

    // Create a new ballot to choose one of `proposalNames`
    function Ballot(bytes32[] proposalNames) {
        chairperson = msg.sender;
        voters[chairperson].weight = 1;

        for (uint i = 0; i < proposalNames.length; i++) {
            proposals.push(Proposal({
                name: proposalNames[i],
                voteCount: 0
            }));
        }
    }

    // Give `voter` the right to vote on this ballot.
    // May only be called by `chairperson`.
    function giveRightToVote(address voter) {
        require((msg.sender == chairperson) && !voters[voter].voted);
        voters[voter].weight = 1;
    }

    // Delegate your vote to the voter `to`.
    function delegate(address to) {
        Voter sender = voters[msg.sender];
        require(!sender.voted);
        require(to != msg.sender);

        while (voters[to].delegate != address(0)) {
            to = voters[to].delegate;

            // We found a loop in the delegation, not allowed.
            require(to != msg.sender);
        }

        sender.voted = true;
        sender.delegate = to;
        Voter delegate = voters[to];
        if (delegate.voted) {
            proposals[delegate.vote].voteCount += sender.weight;
        } else {
            delegate.weight += sender.weight;
        }
    }

    // Give your vote (including votes delegated to you)
    // to proposal `proposals[proposal].name`.
    function vote(uint proposal) {
        Voter sender = voters[msg.sender];
        require(!sender.voted);
        sender.voted = true;
        sender.vote = proposal;

        proposals[proposal].voteCount += sender.weight;
    }

    // @dev Computes the winning proposal taking all
    // previous votes into account.
    function winningProposal() constant
            returns (uint winningProposal)
    {
        uint winningVoteCount = 0;
        for (uint p = 0; p < proposals.length; p++) {
            if (proposals[p].voteCount > winningVoteCount) {
                winningVoteCount = proposals[p].voteCount;
                winningProposal = p;
            }
        }
    }

    // Calls winningProposal() function to get the index
    // of the winner contained in the proposals array and then
    // returns the name of the winner
    function winnerName() constant
            returns (bytes32 winnerName)
    {
        winnerName = proposals[winningProposal()].name;
    }
}
```

### 代碼解析

#### 指定版本

在第一行，`pragma` 關鍵字指定了和該合約兼容的編譯器版本。

```
pragma solidity ^0.4.11;
```

該合約指定，不兼容比 `0.4.11` 更舊的編譯器版本，且 `^` 符號表示也不兼容從 `0.5.0` 起的新編譯器版本。即兼容版本範圍是 `0.4.11 <= version < 0.5.0`。該語法與 npm 的版本描述語法一致。

#### 結構體類型

Solidity 中的合約（contract）類似面向對象編程語言中的類。每個合約可以包含狀態變量、函數、事件、結構體類型和枚舉類型等。一個合約也可以繼承另一個合約。

在本例命名為 `Ballot` 的合約中，聲明瞭 2 個結構體類型：`Voter` 和 `Proposal`。

* `struct Voter`：投票人，其屬性包括 `uint weight`（該投票人的權重）、`bool voted`（是否已投票）、`address delegate`（如果該投票人將投票委託給他人，則記錄受委託人的帳戶地址）和 `uint vote`（投票做出的選擇，即相應提案的索引號）。
* `struct Proposal`：提案，其屬性包括 `bytes32 name`（名稱）和 `uint voteCount`（已獲得的票數）。

需要注意，`address` 類型記錄了一個以太坊帳戶的地址。`address` 可看作一個數值類型，但也包括一些與以太幣相關的方法，如查詢餘額 `<address>.balance`、向該地址轉帳 `<address>.transfer(uint256 amount)` 等。

#### 狀態變量

合約中的狀態變量會長期保存在區塊鏈中。通過調用合約中的函數，這些狀態變量可以被讀取和改寫。

本例中定義了 3 個狀態變量：`chairperson`、`voters`、`proposals`。

* `address public chairperson`：投票發起人，類型為 `address`。
* `mapping(address => Voter) public voters`：所有投票人，類型為 `address` 到 `Voter` 的映射。
* `Proposal[] public proposals`：所有提案，類型為動態大小的 `Proposal` 數組。

3 個狀態變量都使用了 `public` 關鍵字，使得變量可以被外部訪問（即通過消息調用）。事實上，編譯器會自動為 `public` 的變量創建同名的 getter 函數，供外部直接讀取。

狀態變量還可設置為 `internal` 或 `private`。`internal` 的狀態變量只能被該合約和繼承該合約的子合約訪問，`private` 的狀態變量只能被該合約訪問。狀態變量默認為 `internal`。

將上述關鍵狀態信息設置為 `public` 能夠增加投票的公平性和透明性。

#### 函數

合約中的函數用於處理業務邏輯。函數的可見性默認為 `public`，即可以從內部或外部調用，是合約的對外接口。函數可見性也可設置為 `external`、`internal` 和 `private`。

本例實現了 6 個 `public` 函數，可看作 6 個對外接口，功能分別如下。

##### 創建投票
函數 `function Ballot(bytes32[] proposalNames)` 用於創建一個新的投票。

所有提案的名稱通過參數 `bytes32[] proposalNames` 傳入，逐個記錄到狀態變量 `proposals` 中。同時用 `msg.sender` 獲取當前調用消息的發送者的地址，記錄為投票發起人 `chairperson`，該發起人投票權重設為 1。

##### 賦予投票權

函數 `function giveRightToVote(address voter)` 實現給投票人賦予投票權。

該函數給 `address voter` 賦予投票權，即將 `voter` 的投票權重設為 1，存入 `voters` 狀態變量。

這個函數只有投票發起人 `chairperson` 可以調用。這裡用到了 `require((msg.sender == chairperson) && !voters[voter].voted)` 函數。如果 `require` 中表達式結果為 `false`，這次調用會中止，且回滾所有狀態和以太幣餘額的改變到調用前。但已消耗的 Gas 不會返還。

##### 委託投票權

函數 `function delegate(address to)` 把投票委託給其他投票人。

其中，用 `voters[msg.sender]` 獲取委託人，即此次調用的發起人。用 `require` 確保發起人沒有投過票，且不是委託給自己。由於被委託人也可能已將投票委託出去，所以接下來，用 `while` 循環查找最終的投票代表。找到後，如果投票代表已投票，則將委託人的權重加到所投的提案上；如果投票代表還未投票，則將委託人的權重加到代表的權重上。

該函數使用了 `while` 循環，這裡合約編寫者需要十分謹慎，防止調用者消耗過多 Gas，甚至出現死循環。

##### 進行投票

函數 `function vote(uint proposal)` 實現投票過程。

其中，用 `voters[msg.sender]` 獲取投票人，即此次調用的發起人。接下來檢查是否是重複投票，如果不是，進行投票後相關狀態變量的更新。

##### 查詢獲勝提案

函數 `function winningProposal() constant returns (uint winningProposal)` 將返回獲勝提案的索引號。

這裡，`returns (uint winningProposal)` 指定了函數的返回值類型，`constant` 表示該函數不會改變合約狀態變量的值。

函數通過遍歷所有提案進行記票，得到獲勝提案。

##### 查詢獲勝者名稱

函數 `function winnerName() constant returns (bytes32 winnerName)` 實現返回獲勝者的名稱。

這裡採用內部調用 `winningProposal()` 函數的方式獲得獲勝提案。如果需要採用外部調用，則需要寫為 `this.winningProposal()`。
