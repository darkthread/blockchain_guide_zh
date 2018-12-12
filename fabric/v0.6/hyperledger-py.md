## Python 客戶端
前面應用案例，都是直接通過 HTTP API 來跟 hyperledger 進行交互，操作比較麻煩。

還可以直接通過 [hyperledger-py](https://github.com/yeasy/hyperledger-py) 客戶端來進行更方便的操作。

### 安裝

```sh
$ pip install hyperledger --upgrade
```

或直接源碼安裝

```sh
$ git clone https://github.com/yeasy/hyperledger-py.git
$ cd hyperledger-py
$ pip install -r requirements.txt
$ python setup.py install
```

### 使用

```py
>>> from hyperledger.client import Client
>>> c = Client(base_url="http://127.0.0.1:7050")
>>> c.peer_list()
{u'peers': [{u'type': 1, u'ID': {u'name': u'vp1'}, u'address': u'172.17.0.2:30303'}, {u'type': 1, u'ID': {u'name': u'vp2'}, u'address': u'172.17.0.3:30303'}]}
```

更多使用方法，可以參考 [API 文檔](https://github.com/yeasy/hyperledger-py/blob/master/docs/api.md)。

### 其它客戶端

目前，HyperLedger Fabric 已經成立了 [SDK 工作組](https://wiki.hyperledger.org/groups/fabric-sdk/fabric-sdk-wg)。

目前在實現的客戶端 SDK 包括：

* [Python SDK](https://github.com/hyperledger/fabric-sdk-py)
* [Nodejs SDK](https://github.com/hyperledger/fabric-sdk-node)
