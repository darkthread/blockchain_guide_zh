## 消息協議

節點之間通過消息來進行交互，所有消息都由下面的數據結構來實現。

```protobuf
message Message {
   enum Type {
        UNDEFINED = 0;

        DISC_HELLO = 1;
        DISC_DISCONNECT = 2;
        DISC_GET_PEERS = 3;
        DISC_PEERS = 4;
        DISC_NEWMSG = 5;

        CHAIN_STATUS = 6;
        CHAIN_TRANSACTION = 7;
        CHAIN_GET_TRANSACTIONS = 8;
        CHAIN_QUERY = 9;

        SYNC_GET_BLOCKS = 11;
        SYNC_BLOCKS = 12;
        SYNC_BLOCK_ADDED = 13;

        SYNC_STATE_GET_SNAPSHOT = 14;
        SYNC_STATE_SNAPSHOT = 15;
        SYNC_STATE_GET_DELTAS = 16;
        SYNC_STATE_DELTAS = 17;

        RESPONSE = 20;
        CONSENSUS = 21;
    }
    Type type = 1;
    bytes payload = 2;
    google.protobuf.Timestamp timestamp = 3;
}
```

消息分為四大類：Discovery（探測）、Transaction（交易）、Synchronization（同步）、Consensus（一致性）。

不同消息類型，對應到 payload 中數據不同，分為對應的子類消息結構。

### Discovery

包括 DISC_HELLO、DISC_GET_PEERS、DISC_PEERS。

DISC_HELLO 消息結構如下。

```protobuf
message HelloMessage { PeerEndpoint peerEndpoint = 1; uint64 blockNumber = 2;}message PeerEndpoint { PeerID ID = 1; string address = 2; enum Type { UNDEFINED = 0; VALIDATOR = 1; NON_VALIDATOR = 2; } Type type = 3; bytes pkiID = 4;}

message PeerID { string name = 1;}
```

節點新加入網路時，會向 `CORE_PEER_DISCOVERY_ROOTNODE` 發送 `DISC_HELLO` 消息，彙報本節點的信息（id、地址、block 數、類型等），開始探測過程。

探測後發現 block 數落後對方，則會觸發同步過程。

之後，定期發送 `DISC_GET_PEERS` 消息，獲取新加入的節點信息。收到 `DISC_GET_PEERS` 消息的節點會通過 `DISC_PEERS` 消息返回自己知道的節點列表。

### Transaction

包括 Deploy、Invoke、Query。消息結構如下：

```protobuf
message Transaction { enum Type { UNDEFINED = 0; CHAINCODE_DEPLOY = 1; CHAINCODE_INVOKE = 2; CHAINCODE_QUERY = 3; CHAINCODE_TERMINATE = 4; } Type type = 1; string uuid = 5; bytes chaincodeID = 2; bytes payloadHash = 3;

 ConfidentialityLevel confidentialityLevel = 7; bytes nonce = 8; bytes cert = 9; bytes signature = 10;

 bytes metadata = 4; google.protobuf.Timestamp timestamp = 6;}

message TransactionPayload { bytes payload = 1;}

enum ConfidentialityLevel { PUBLIC = 0; CONFIDENTIAL = 1;}
```

### Synchronization
當節點發現自己 block 落後網路中最新狀態，則可以通過發送如下消息（由 consensus 策略決定）來獲取對應的返回。

* SYNC_GET_BLOCKS（對應 SYNC_BLOCKS）：獲取給定範圍內的 block 數據；
* SYNC_STATE_GET_SNAPSHOT（對應 SYNC_STATE_SNAPSHOT）：獲取最新的世界觀快照；
* SYNC_STATE_GET_DELTAS（對應 SYNC_STATE_DELTAS）：獲取某個給定範圍內的 block 對應的狀態變更。

### Consensus

consensus 組件收到 `CHAIN_TRANSACTION` 類消息後，將其轉換為 `CONENSUS` 消息，然後向所有的 VP 節點廣播。

收到 `CONSENSUS` 消息的節點會按照預定的 consensus 算法進行處理。
