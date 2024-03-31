---
title: JSON-RPC API
description: イーサリアムクライアント用のステートレスで軽量なリモートプロシージャコール(RPC)プロトコル。
lang: ja
---

ブロックチェーンデータの読み取りやネットワークへのトランザクションの送信など、ソフトウェアアプリケーションがイーサリアムブロックチェーンとやり取りするには、イーサリアムノードに接続する必要があります。

そのため、すべての[イーサリアムクライアント](/developers/docs/nodes-and-clients/#execution-clients)は[JSON-RPC 仕様](https://github.com/ethereum/execution-apis)を実装しており、ノードやクライアントの実装がどのようなものであっても、アプリケーションは統一されたメソッドのセットを使用できます。

[JSON-RPC](https://www.jsonrpc.org/specification)は、ステートレスで軽量なリモートプロシージャコール(RPC)プロトコルです。 いくつかのデータ構造とその処理に関するルールを定義しています。 トランスポートに依存しないため、同じプロセス内だけでなく、ソケット経由、HTTP 経由など、さまざまなメッセージパッシング環境で利用できます。 データ形式としては、JSON(RFC 4627)を使用します。

## クライアントの実装 {#client-implementations}

各イーサリアムクライアントでは、JSON-RPC 仕様を実装する際に異なるプログラミング言語を使用できます。 特定のプログラミング言語に関連する詳細については、各[クライアントのドキュメント](/developers/docs/nodes-and-clients/#execution-clients)を参照してください。 最新の API サポート情報についても、各クライアントのドキュメントを確認することをお勧めします。

## 便利なライブラリ {#convenience-libraries}

JSON-RPC API を介してイーサリアムクライアントと直接やり取りすることもできますが、dapp デベロッパーの作業が多くの場合に簡単になるオプションもあります。 [JavaScript](/developers/docs/apis/javascript/#available-libraries)と[バックエンド API](/developers/docs/apis/backend/#available-libraries)には、JSON-RPC API の上にラッパーを提供する多くのライブラリが存在します。 これらのライブラリを使用することで、デベロッパーは任意のプログラミング言語による直感的な 1 行のメソッドを作成するだけで、イーサリアムとやり取りする JSON-RPC リクエストを(内部的に)初期化できるようになります。

## コンセンサスクライアント API {#consensus-clients}

このページでは、主にイーサリアムの実行クライアントで使用される JSON-RPC API について説明します。 しかし、コンセンサスクライアントには、ユーザーがノードについての情報のクエリを行える RPC API が用意されており、ビーコンブロック、ビーコンの状態、その他のコンセンサス関連の情報を直接ノードにリクエストできます。 この API については 、[ビーコン API のウェブページ](https://ethereum.github.io/beacon-APIs/#/)に記載されています。

内部 API は、ノード内のクライアント間通信にも使用されます。 つまり、コンセンサスクライアントと実行クライアントとの間のデータ交換を可能にします。 これは「Engine API」と呼ばれており、仕様は[Github](https://github.com/ethereum/execution-apis/blob/main/src/engine/common.md)で参照できます。

## 実行クライアントの仕様 {#spec}

[GitHub 上で JSON-RPC API の全仕様を読む](https://github.com/ethereum/execution-apis)。

## 慣例 {#conventions}

### 16 進数のエンコーディング {#hex-encoding}

フォーマットされていないバイト配列とそのバイト数という、2 つのキーデータ型が JSON で渡されます。 どちらも 16 進数エンコーディングで渡されますが、フォーマットの要件は異なります。

#### バイト数 {#quantities-encoding}

バイト数(整数、数字)をエンコーディングする際は、接頭辞「0x」の 16 進数でエンコードするのが最も簡潔です。ただし、ゼロは「0x0」と表記する必要があります。

以下に、いくつかの例を示します。

- 0x41(10 進数で 65)
- 0x400(10 進数で 1024)
- 誤り: 0x(常に少なくとも 1 桁の数字が必要です。ゼロは「0x0」)
- 誤り: 0x0400(先頭にゼロは入力できません)
- 誤り: ff(接頭辞は 0x でなければなりません)

### フォーマットされていないデータ {#unformatted-data-encoding}

フォーマットされていないデータ(バイト配列、アカウントアドレス、ハッシュ、バイトコード配列)をエンコードする場合、16 進数としてエンコードし、接頭辞を「0x」とし、1 バイトごとに 2 桁の 16 進数でエンコードします。

以下に、いくつかの例を示します。

- 0x41(サイズ 1、「A」)
- 0x004200(サイズ 3、「\0B\0」)
- 0x(サイズ 0、「」)
- 誤り: 0xf0f0f(偶数でなければなりません)
- 誤り: 004200(接頭辞が 0x でなければなりません)

### デフォルトのブロックパラメータ {#default-block}

以下のメソッドには、追加のデフォルトブロックパラメータがあります。

- [eth_getBalance](#eth_getbalance)
- [eth_getCode](#eth_getcode)
- [eth_getTransactionCount](#eth_gettransactioncount)
- [eth_getStorageAt](#eth_getstorageat)
- [eth_call](#eth_call)

イーサリアムの状態に作用するリクエストを行う場合は、最後のデフォルトブロックパラメータによってブロックの高さが決定します。

デフォルトブロックパラメータには、以下のオプションがあります。

- `HEX String` - 整数のブロック番号
- `String "earliest"` - 最も古い/始まりのブロック
- `String "latest"` - 最も新しいマイニング済みブロック
- `String "safe"` - 最も新しい安全な先頭ブロック
- `String "finalized"` - 最も新しい確定済みブロック
- `String "pending"` - 保留中の状態/トランザクション

## 使用例

このページでは、コマンドラインツールである[curl](https://curl.se)を使用した、各 JSON-RPC API エンドポイントの使用方法の例を紹介します。 各エンドポイントの使用例は、[curl の使用例](#curl-examples)セクションで確認できます。 ページの下方には、Geth ノード、JSON-RPC API、curl を使用したスマートコントラクトのコンパイルとデプロイの[エンドツーエンドの例](#usage-example)もあります。

## Curl の使用例 {#curl-examples}

以下に、[curl](https://curl.se)でイーサリアムノードへのリクエストを行うことによって JSON-RPC API を使用する例をいくつか示します。 それぞれの例には、エンドポイントの説明、パラメータ、戻り値の型、使用方法の範例が含まれています。

curl リクエストは、コンテンツタイプに関するエラーメッセージを返すことがあります。 この理由は、`--data`オプションによって、コンテンツタイプが`application/x-www-form-urlencoded`に設定されるためです。 これによってノードがエラーになる場合は、呼び出しの最初に、手動で`-H "Content-Type: application/json"`と記述してヘッダーを設定してください。 また、使用例には、curl で最後に指定する引数である URL/IP とポートの組み合わせ(例: `127.0.0.1:8545`)が含まれていません 。 これらの追加データが含まれた完全な curl リクエストは、次の形式になります。

```shell
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}' 127.0.0.1:8545
```

## ゴシップ、状態、履歴 {#gossip-state-history}

少数のコア JSON-RPC メソッドは、イーサリアムネットワークからのデータを必要とします。該当メソッドは、*ゴシップ、状態、履歴*という、3 つの主要カテゴリーに明確に分類できます。 各メソッドは、セクションにあるリンクからジャンプするか、目次でメソッドの全リストを調べることで見つけられます。

### ゴシップメソッド {#gossip-methods}

> チェーンの先頭までたどるメソッドです。 これにより、トランザクションがどのようにネットワークを進み、ブロックへたどり着くのか、また、クライアントがどのようにして新しいブロックについての情報を得るのかがわかります。

- [eth_blockNumber](#eth_blocknumber)
- [eth_sendRawTransaction](#eth_sendrawtransaction)

### 状態メソッド {#state_methods}

> このメソッドは、すべての保存データの現在の状態を報告します。 「状態」は、RAM 内の 1 つの大きな共有部分です。アカウントの残高、コントラクトデータ、ガスの推定値が含まれています。

- [eth_getBalance](#eth_getbalance)
- [eth_getStorageAt](#eth_getstorageat)
- [eth_getTransactionCount](#eth_gettransactioncount)
- [eth_getCode](#eth_getcode)
- [eth_call](#eth_call)
- [eth_estimateGas](#eth_estimategas)

### 履歴メソッド {#history_methods}

> このメソッドは、各ブロックの履歴レコードを始まりのブロックまでさかのぼって取得します。 これは、1 つの大きな追加専用ファイルのようなもので、すべてのブロックヘッダー、ブロックボディ、アンクルブロック、トランザクションレシートが含まれます。

- [eth_getBlockTransactionCountByHash](#eth_getblocktransactioncountbyhash)
- [eth_getBlockTransactionCountByNumber](#eth_getblocktransactioncountbynumber)
- [eth_getUncleCountByBlockHash](#eth_getunclecountbyblockhash)
- [eth_getUncleCountByBlockNumber](#eth_getunclecountbyblocknumber)
- [eth_getBlockByHash](#eth_getblockbyhash)
- [eth_getBlockByNumber](#eth_getblockbynumber)
- [eth_getTransactionByHash](#eth_gettransactionbyhash)
- [eth_getTransactionByBlockHashAndIndex](#eth_gettransactionbyblockhashandindex)
- [eth_getTransactionByBlockNumberAndIndex](#eth_gettransactionbyblocknumberandindex)
- [eth_getTransactionReceipt](#eth_gettransactionreceipt)
- [eth_getUncleByBlockHashAndIndex](#eth_getunclebyblockhashandindex)
- [eth_getUncleByBlockNumberAndIndex](#eth_getunclebyblocknumberandindex)

## JSON-RPC API メソッド {#json-rpc-methods}

### web3_clientVersion {#web3_clientversion}

現在のクライアントバージョンを返します。

**パラメータ**

なし

**戻り値**

`String` - 現在のクライアントのバージョン

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}'
// Result
{
  "id":67,
  "jsonrpc":"2.0",
  "result": "Geth/v1.12.1-stable/linux-amd64/go1.19.1"
}
```

### web3_sha3 {#web3_sha3}

標準の SHA3-256 ではなく、指定されたデータの Keccak-256\_\_を返します。

**パラメータ**

1. `DATA` - SHA3 ハッシュに変換するデータ

```js
params: ["0x68656c6c6f20776f726c64"]
```

**戻り値**

`DATA` - 指定された文字列の SHA3 結果

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"web3_sha3","params":["0x68656c6c6f20776f726c64"],"id":64}'
// Result
{
  "id":64,
  "jsonrpc": "2.0",
  "result": "0x47173285a8d7341e5e972fc677286384f802f8ef42a5ec5f03bbfa254cb01fad"
}
```

### net_version {#net_version}

現在のネットワーク ID を返します。

**パラメータ**

なし

**戻り値**

`String` - 現在のネットワーク ID

現在のネットワーク ID の全リストは、[chainlist.org](https://chainlist.org)で入手できます。 よく使われるネットワーク ID は、以下の通りです。

- `1`: イーサリアムメインネット
- `5`: Goerli テストネット
- `11155111`: Sepolia テストネット

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"net_version","params":[],"id":67}'
// Result
{
  "id":67,
  "jsonrpc": "2.0",
  "result": "3"
}
```

### net_listening {#net_listening}

クライアントのネットワーク接続のリッスンがアクティブな場合に、`true`を返します。

**パラメータ**

なし

**戻り値**

`Boolean` - リッスンしている場合は`true`、その他の場合は`false`

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"net_listening","params":[],"id":67}'
// Result
{
  "id":67,
  "jsonrpc":"2.0",
  "result":true
}
```

### net_peerCount {#net_peercount}

クライアントに現在接続されているピアの数を返します。

**パラメータ**

なし

**戻り値**

`QUANTITY` - 接続されているピア数(整数)。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":74}'
// Result
{
  "id":74,
  "jsonrpc": "2.0",
  "result": "0x2" // 2
}
```

### eth_protocolVersion {#eth_protocolversion}

現在のイーサリアムプロトコルのバージョンを返します。 このメソッドは、[Geth では利用できないこと](https://github.com/ethereum/go-ethereum/pull/22064#issuecomment-788682924)に注意してください。

**パラメータ**

なし

**戻り値**

`String` - 現在のイーサリアムプロトコルのバージョン

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_protocolVersion","params":[],"id":67}'
// Result
{
  "id":67,
  "jsonrpc": "2.0",
  "result": "54"
}
```

### eth_syncing {#eth_syncing}

同期ステータスに関するデータを含むオブジェクトか、`false`を返します。

**パラメータ**

なし

**戻り値**

クライアント実装によって、厳密にはリターンデータが異なります。 ノードが同期していない場合、すべてのクライアントは`False`を返します。また、すべてのクライアントは次のフィールドを返します。

`Object|Boolean` - 同期ステータスデータを含むオブジェクト、または同期していない場合は`FALSE`

- `startingBlock`: `QUANTITY` - インポートを開始したブロック(同期がブロックの先頭に達したときのみリセットされる)
- `currentBlock`: `QUANTITY` - 現在のブロックであり、eth_blockNumber と同一
- `highestBlock`: `QUANTITY` - 最大ブロック高の推定値

ただし、クライアントによっては、追加のデータを提供する場合もあります。 例えば、Geth では以下のデータを返します。

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "currentBlock": "0x3cf522",
    "healedBytecodeBytes": "0x0",
    "healedBytecodes": "0x0",
    "healedTrienodes": "0x0",
    "healingBytecode": "0x0",
    "healingTrienodes": "0x0",
    "highestBlock": "0x3e0e41",
    "startingBlock": "0x3cbed5",
    "syncedAccountBytes": "0x0",
    "syncedAccounts": "0x0",
    "syncedBytecodeBytes": "0x0",
    "syncedBytecodes": "0x0",
    "syncedStorage": "0x0",
    "syncedStorageBytes": "0x0"
  }
}
```

それに対して、Besu では以下のデータを返します。

```json
{
  "jsonrpc": "2.0",
  "id": 51,
  "result": {
    "startingBlock": "0x0",
    "currentBlock": "0x1518",
    "highestBlock": "0x9567a3",
    "pulledStates": "0x203ca",
    "knownStates": "0x200636"
  }
}
```

詳細については、各クライアントのドキュメントを参照してください。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": {
    startingBlock: '0x384',
    currentBlock: '0x386',
    highestBlock: '0x454'
  }
}
// Or when not syncing
{
  "id":1,
  "jsonrpc": "2.0",
  "result": false
}
```

### eth_coinbase {#eth_coinbase}

クライアントのコインベースアドレスを返します。

**パラメータ**

なし

**戻り値**

`DATA`、20 バイト - 現在のコインベースアドレス。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_coinbase","params":[],"id":64}'
// Result
{
  "id":64,
  "jsonrpc": "2.0",
  "result": "0x407d73d8a49eeb85d32cf465507dd71d507100c1"
}
```

### eth_chainId {#eth_chainId}

リプレイ保護されたトランザクションの署名に使われるチェーン ID を返します。

**パラメータ**

なし

**戻り値**

`chainId`、16 進数の文字列で表された現在のチェーン ID の整数値。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":67}'
// Result
{
  "id":67,
  "jsonrpc": "2.0",
  "result": "0x1"
}
```

### eth_mining {#eth_mining}

クライアントの新規ブロックのマイニングがアクティブな場合に、`true`を返します。 プルーフ・オブ・ワークのネットワークに対してのみ `true`を返します。[マージ](/roadmap/merge/)以降、一部のクライアントでは使用できない場合があります。

**パラメータ**

なし

**戻り値**

`Boolean` - クライアントがマイニングしている場合は`true`、その他の場合は`false`

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_mining","params":[],"id":71}'
//
{
  "id":71,
  "jsonrpc": "2.0",
  "result": true
}
```

### eth_hashrate {#eth_hashrate}

ノードがマイニングしている 1 秒あたりのハッシュの数を返します。 プルーフ・オブ・ワークのネットワークに対してのみ `true`を返します。[マージ](/roadmap/merge/)以降、一部のクライアントでは使用できない場合があります。

**パラメータ**

なし

**戻り値**

`QUANTITY` - 1 秒あたりのハッシュの数

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_hashrate","params":[],"id":71}'
// Result
{
  "id":71,
  "jsonrpc": "2.0",
  "result": "0x38a"
}
```

### eth_gasPrice {#eth_gasprice}

現在のガス価格の見積りを wei で返します。 例えば、Besu クライアントではデフォルトで、最新の 100 ブロックを検査し、ガスのユニット価格における中央値を返します。

**パラメータ**

なし

**戻り値**

`QUANTITY` - 現在のガス価格(wei 単位の整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":73}'
// Result
{
  "id":73,
  "jsonrpc": "2.0",
  "result": "0x1dfd14000" // 8049999872 Wei
}
```

### eth_accounts {#eth_accounts}

クライアントが所有するアドレスのリストを返します。

**パラメータ**

なし

**戻り値**

`DATA配列`、20 バイト - クライアントが所有するアドレス

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"]
}
```

### eth_blockNumber {#eth_blocknumber}

最も新しいブロックの番号を返します。

**パラメータ**

なし

**戻り値**

`QUANTITY` - クライアントの現在のブロックの番号(整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}'
// Result
{
  "id":83,
  "jsonrpc": "2.0",
  "result": "0x4b7" // 1207
}
```

### eth_getBalance {#eth_getbalance}

指定されたアドレスのアカウントの残高を返します。

**パラメータ**

1. `DATA`、20 バイト - 残高を確認するアドレス
2. `QUANTITY|TAG` - ブロック番号 (整数)、もしくは文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください

```js
params: ["0x407d73d8a49eeb85d32cf465507dd71d507100c1", "latest"]
```

**戻り値**

`QUANTITY` - 現在の残高(wei 単位の整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1", "latest"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x0234c8a3397aab58" // 158972490234375000
}
```

### eth_getStorageAt {#eth_getstorageat}

指定されたアドレスのストレージの位置の値を返します。

**パラメータ**

1. `DATA`、20 バイト - ストレージのアドレス
2. `QUANTITY` - ストレージの位置(整数)
3. `QUANTITY|TAG` - ブロック番号 (整数)、もしくは文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください

**戻り値**

`DATA` - このストレージの位置の値

**例: ** 正確な位置計算は、取得するストレージによって変わります。 ここでは、アドレス`0x391694e7e0b0cce554cb130d723a9d27458f9298`で、`0x295a70b2de5e3953354a6a8344e616ed314d7251`にデプロイされているコントラクトについて考えます。

```
contract Storage {
    uint pos0;
    mapping(address => uint) pos1;
    function Storage() {
        pos0 = 1234;
        pos1[msg.sender] = 5678;
    }
}
```

pos0 の値の取得は簡単です。

```js
curl -X POST --data '{"jsonrpc":"2.0", "method": "eth_getStorageAt", "params": ["0x295a70b2de5e3953354a6a8344e616ed314d7251", "0x0", "latest"], "id": 1}' localhost:8545
{"jsonrpc":"2.0","id":1,"result":"0x00000000000000000000000000000000000000000000000000000000000004d2"}
```

ただし、マップの要素の取得は、より複雑になります。 マップの要素の位置は、次のように計算されます。

```js
keccak(LeftPad32(key, 0), LeftPad32(map position, 0))
```

つまり、pos1["0x391694e7e0b0cce554cb130d723a9d27458f9298"]のストレージを取得するためには、次のように位置を計算する必要があります。

```js
keccak(
  decodeHex(
    "000000000000000000000000391694e7e0b0cce554cb130d723a9d27458f9298" +
      "0000000000000000000000000000000000000000000000000000000000000001"
  )
)
```

これは、web3 ライブラリに付属する Geth コンソールを使用して、次のように計算できます。

```js
> var key = "000000000000000000000000391694e7e0b0cce554cb130d723a9d27458f9298" + "0000000000000000000000000000000000000000000000000000000000000001"
undefined
> web3.sha3(key, {"encoding": "hex"})
"0x6661e9d6d8b923d5bbaab1b96e1dd51ff6ea2a93520fdc9eb75d059238b8c5e9"
```

これにより、以下のようにしてストレージを取得できます。

```js
curl -X POST --data '{"jsonrpc":"2.0", "method": "eth_getStorageAt", "params": ["0x295a70b2de5e3953354a6a8344e616ed314d7251", "0x6661e9d6d8b923d5bbaab1b96e1dd51ff6ea2a93520fdc9eb75d059238b8c5e9", "latest"], "id": 1}' localhost:8545
{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000000000000000000000000000000000000000162e"}
```

### eth_getTransactionCount {#eth_gettransactioncount}

アドレスから*送信*されたトランザクションの数を返します。

**パラメータ**

1. `DATA`、20 バイト - アドレス
2. `QUANTITY|TAG` - ブロック番号 (整数)、もしくは文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください

```js
params: [
  "0x407d73d8a49eeb85d32cf465507dd71d507100c1",
  "latest", // state at the latest block
]
```

**戻り値**

`QUANTITY` - このアドレスから送信されたトランザクションの数(整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1","latest"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

### eth_getBlockTransactionCountByHash {#eth_getblocktransactioncountbyhash}

指定されたブロックハッシュに一致するブロック内のトランザクションの数を返します。

**パラメータ**

1. `DATA`、32 バイト - ブロックハッシュ

```js
params: ["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"]
```

**戻り値**

`QUANTITY` - このブロック内のトランザクションの数(整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xb" // 11
}
```

### eth_getBlockTransactionCountByNumber {#eth_getblocktransactioncountbynumber}

指定されたブロック番号に一致するブロックのトランザクションの数を返します。

**パラメータ**

1. `QUANTITY|TAG` - ブロックの番号(整数)、または文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください

```js
params: [
  "0xe8", // 232
]
```

**戻り値**

`QUANTITY` - このブロック内のトランザクションの数(整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByNumber","params":["0xe8"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xa" // 10
}
```

### eth_getUncleCountByBlockHash {#eth_getunclecountbyblockhash}

指定されたブロックハッシュに一致するブロックのアンクルの数を返します。

**パラメータ**

1. `DATA`、32 バイト - ブロックハッシュ

```js
params: ["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"]
```

**戻り値**

`QUANTITY` - このブロック内のアンクルの数(整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

### eth_getUncleCountByBlockNumber {#eth_getunclecountbyblocknumber}

指定されたブロック番号に一致するブロック内のアンクルの数を返します。

**パラメータ**

1. `QUANTITY|TAG` - ブロックの番号(整数)、または文字列"latest"、"earliest" 、"pending"のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください

```js
params: [
  "0xe8", // 232
]
```

**戻り値**

`QUANTITY` - このブロック内のアンクルの数(整数)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockNumber","params":["0xe8"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

### eth_getCode {#eth_getcode}

指定されたアドレスのコードを返します。

**パラメータ**

1. `DATA`、20 バイト - アドレス
2. `QUANTITY|TAG` - ブロック番号 (整数)、もしくは文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください

```js
params: [
  "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
  "0x2", // 2
]
```

**戻り値**

`DATA` - 指定されたアドレスからのコード

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getCode","params":["0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b", "0x2"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x600160008035811a818181146012578301005b601b6001356025565b8060005260206000f25b600060078202905091905056"
}
```

### eth_sign {#eth_sign}

この署名メソッドは、イーサリアム固有の署名を次のように計算します。`sign(keccak256("\x19Ethereum Signed Message:\n" + len(message) + message)))`。

メッセージに接頭辞を追加することで、計算された署名がイーサリアム固有の署名として認識されるようになります。 これにより、悪意のある dapp によって任意のデータ(トランザクションなど)が署名され、被害者のなりすましに悪用される被害を防ぐことができます。

注: 署名するアドレスのロックを解除する必要があります。

**パラメータ**

1. `DATA`、20 バイト - アドレス
2. `DATA`、N バイト - 署名するメッセージ

**戻り値**

`DATA`: 署名

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sign","params":["0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", "0xdeadbeaf"],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
}
```

### eth_signTransaction {#eth_signtransaction}

[eth_sendRawTransaction](#eth_sendrawtransaction)を使用して、ネットワークに後から送信可能なトランザクションに署名します。

**パラメータ**

1. `Object` - トランザクションオブジェクト

- `type`:
- `from`: `DATA`、20 バイト - トランザクションの送信元アドレス
- `to`: `DATA`、20 バイト - (新規コントラクトの作成時はオプション)トランザクションの宛先アドレス
- `gas`: `QUANTITY` - (オプション、デフォルトは 90000)トランザクションを実行するために提供されているガス(整数)で、 使用されなかったガスは返却される
- `gasPrice`: `QUANTITY` - (オプション、デフォルトは未確定)ガスごとに支払われるガス価格(wei 単位の整数)
- `value`: `QUANTITY` - (オプション)このトランザクションで送信された値(wei 単位の整数)
- `data`: `DATA` - コンパイルされたコントラクトのコード。または呼び出されたメソッドの署名とエンコードされたパラメータのハッシュ
- `nonce`: `QUANTITY` - (オプション)ノンス(nonce)の整数で、 同じノンス(nonce)を使用している保留中のトランザクションを上書きできる

**戻り値**

`DATA`、特定のアカウントによって署名され、RLP エンコードされたトランザクションオブジェクト。

**例**

```js
// Request
curl -X POST --data '{"id": 1,"jsonrpc": "2.0","method": "eth_signTransaction","params": [{"data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675","from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155","gas": "0x76c0","gasPrice": "0x9184e72a000","to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567","value": "0x9184e72a"}]}'
// Result
{
    "id": 1,
    "jsonrpc": "2.0",
    "result": "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
}
```

### eth_sendTransaction {#eth_sendtransaction}

データフィールドにコードが含まれている場合は、新しいメッセージコールトランザクションまたはコントラクト作成を行います。また、`from`で指定されたアカウントを使ってデータフィールドに署名します。

**パラメータ**

1. `Object` - トランザクションオブジェクト

- `from`: `DATA`、20 バイト - トランザクションの送信元アドレス
- `to`: `DATA`、20 バイト - (新規コントラクトの作成時はオプション)トランザクションの宛先アドレス
- `gas`: `QUANTITY` - (オプション、デフォルトは 90000)トランザクションを実行するために提供されているガス(整数)で、 使用されなかったガスは返却される
- `gasPrice`: `QUANTITY` - (オプション、デフォルトは未確定)ガスの支払いに適用されたガス価格(整数)
- `value`: `QUANTITY` - (オプション)このトランザクションで送信された値(整数)
- `data`: `DATA` - コンパイルされたコントラクトのコード。または呼び出されたメソッドの署名とエンコードされたパラメータのハッシュ
- `nonce`: `QUANTITY` - (オプション)ノンス(nonce)の整数で、 同じノンス(nonce)を使用している保留中のトランザクションを上書きできる

```js
params: [
  {
    from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
    to: "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
    gas: "0x76c0", // 30400
    gasPrice: "0x9184e72a000", // 10000000000000
    value: "0x9184e72a", // 2441406250
    data: "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675",
  },
]
```

**戻り値**

`DATA`、32 バイト - トランザクションのハッシュ、またはトランザクションがまだ使用可能でない場合はゼロハッシュ

コントラクトのアドレスを取得するには、コントラクトを作成したトランザクションがマイニングされた後に[eth_getTransactionReceipt](#eth_gettransactionreceipt)を使用します。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{see above}],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
}
```

### eth_sendRawTransaction {#eth_sendrawtransaction}

署名されたトランザクションについて、新規メッセージ呼び出しのトランザクションの作成、またはコントラクトの作成を行います。

**パラメータ**

1. `DATA` - 署名されたトランザクションのデータ

```js
params: [
  "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675",
]
```

**戻り値**

`DATA`、32 バイト - トランザクションのハッシュ、またはトランザクションがまだ使用可能でない場合はゼロハッシュ

コントラクトのアドレスを取得するには、コントラクトを作成したトランザクションがマイニングされた後に[eth_getTransactionReceipt](#eth_gettransactionreceipt)を使用します。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":[{see above}],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
}
```

### eth_call {#eth_call}

ブロックチェーン上にトランザクションを作成せずに、即座に新規メッセージ呼び出しを実行できます。 読み取り専用のスマートコントラクト関数を実行するために頻繁に使われます(例えば、ERC-20 コントラクトの`balanceOf`など)。

**パラメータ**

1. `Object` - トランザクションの呼び出しオブジェクト

- `from`: `DATA`、20 バイト - (オプション)トランザクションの送信元アドレス
- `to`: `DATA`、20 バイト - トランザクションの宛先アドレス
- `gas`: `QUANTITY` - (オプション)トランザクションを実行するために提供されているガス(整数) 。 eth_call はガスを消費しませんが、一部の実行ではこのパラメータが必要になる場合があります。
- `gasPrice`: `QUANTITY` - (オプション)ガスの支払いに適用されるガス価格(整数)
- `value`: `QUANTITY` - (オプション)このトランザクションで送信された値(整数)
- `input`: `DATA` - (オプション)メソッド署名とエンコードされたパラメータのハッシュ。 詳細については、[Solidity のドキュメントのイーサリアムコントラクト ABI](https://docs.soliditylang.org/en/latest/abi-spec.html)を参照してください。

2. `QUANTITY|TAG` - ブロック番号 (整数)、もしくは文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください

**戻り値**

`DATA` - 実行されたコントラクトの戻り値

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_call","params":[{see above}],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x"
}
```

### eth_estimateGas {#eth_estimategas}

トランザクションの完了に必要なガスの推定値を計算して返します。 このトランザクションはブロックチェーンに追加されません。 推定値は、EVM の仕組みやノードのパフォーマンスなどのさまざまな理由により、トランザクションによって実際に使われるガス量を大幅に上回る可能性があることに注意してください。

**パラメータ**

[eth_call](#eth_call)パラメータを参照してください。ただし、すべてのプロパティはオプションです。 ガスリミットが指定されていない場合、Geth では保留中のブロックのガスリミットが上限値として使用されます。 その結果、保留中のブロックのガスリミットよりガス量が多い場合には、返される推定値の量では、呼び出し/トランザクションを実行するには十分でないことがあります。

**戻り値**

`QUANTITY` - ガス使用量

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_estimateGas","params":[{see above}],"id":1}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x5208" // 21000
}
```

### eth_getBlockByHash {#eth_getblockbyhash}

ハッシュを使用して、ブロックに関する情報を返します。

**パラメータ**

1. `DATA`、32 バイト - ブロックのハッシュ
2. `Boolean` - `true`の場合は、完全なトランザクションオブジェクトを返します。 `false`の場合は、トランザクションのハッシュのみを返します

```js
params: [
  "0xdc0818cf78f21a8e70579cb46a43643f78291264dda342ae31049421c82d21ae",
  false,
]
```

**戻り値**

`Object` - ブロックオブジェクト、またはブロックが見つからなかった場合は`null`

- `number`: `QUANTITY` - ブロック番号。 保留中のブロックの場合は`null`
- `hash`: `DATA`、32 バイト - ブロックのハッシュ。 保留中のブロックの場合は`null`
- `parentHash`: `DATA`、32 バイト - 親ブロックのハッシュ
- `nonce`: `DATA`、8 バイト - 生成されたプルーフ・オブ・ワークのハッシュ。 保留中のブロックの場合は`null`
- `sha3Uncles`: `DATA`、32 バイト - ブロックのアンクルデータの SHA3
- `logsBloom`: `DATA`、256 バイト - ブロックのログのブルームフィルタ。 保留中のブロックの場合は`null`
- `transactionsRoot`: `DATA`、32 バイト - ブロックのトランザクションツリーのルート
- `stateRoot`: `DATA`、32 バイト - ブロックの最終状態ツリーのルート
- `receiptsRoot`: `DATA`、32 バイト - ブロックのレシートツリーのルート
- `miner`: `DATA`、20 バイト - マイニング報酬が支払われる受益者のアドレス
- `difficulty`: `QUANTITY` - このブロックの難易度(整数)
- `totalDifficulty`: `QUANTITY` - このブロックまでのチェーンの合計難易度(整数)
- `extraData`: `DATA` - このブロックの「追加データ」フィールド
- `size`: `QUANTITY` - このブロックのサイズのバイト数(整数)
- `gasLimit`: `QUANTITY` - このブロックで許可されているガスの最大量
- `gasUsed`: `QUANTITY` - このブロックのすべてのトランザクションで使用されたガスの合計量
- `timestamp`: `QUANTITY` - ブロックが照合されたときの UNIX タイムスタンプ
- `transactions`: `Array` - 最後に指定したパラメータに応じて、トランザクションオブジェクトの配列、または 32 バイトのトランザクションハッシュ
- `uncles`: `Array` - アンクルハッシュの配列

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByHash","params":["0xdc0818cf78f21a8e70579cb46a43643f78291264dda342ae31049421c82d21ae", false],"id":1}'
// Result
{
{
"jsonrpc": "2.0",
"id": 1,
"result": {
    "difficulty": "0x4ea3f27bc",
    "extraData": "0x476574682f4c5649562f76312e302e302f6c696e75782f676f312e342e32",
    "gasLimit": "0x1388",
    "gasUsed": "0x0",
    "hash": "0xdc0818cf78f21a8e70579cb46a43643f78291264dda342ae31049421c82d21ae",
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "miner": "0xbb7b8287f3f0a933474a79eae42cbca977791171",
    "mixHash": "0x4fffe9ae21f1c9e15207b1f472d5bbdd68c9595d461666602f2be20daf5e7843",
    "nonce": "0x689056015818adbe",
    "number": "0x1b4",
    "parentHash": "0xe99e022112df268087ea7eafaf4790497fd21dbeeb6bd7a1721df161a6657a54",
    "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "size": "0x220",
    "stateRoot": "0xddc8b0234c2e0cad087c8b389aa7ef01f7d79b2570bccb77ce48648aa61c904d",
    "timestamp": "0x55ba467c",
    "totalDifficulty": "0x78ed983323d",
    "transactions": [
    ],
    "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "uncles": [
    ]
}
}
```

### eth_getBlockByNumber {#eth_getblockbynumber}

ブロック番号を使用して、ブロックに関する情報を返します。

**パラメータ**

1. `QUANTITY|TAG` - ブロックの番号(整数)、または文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください
2. `Boolean` - `true`の場合は、完全なトランザクションオブジェクトを返します。 `false`の場合は、トランザクションのハッシュのみを返します

```js
params: [
  "0x1b4", // 436
  true,
]
```

**戻り値**については、[eth_getBlockByHash](#eth_getblockbyhash)を参照してください。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1b4", true],"id":1}'
```

結果については、[eth_getBlockByHash](#eth_getblockbyhash)を参照してください。

### eth_getTransactionByHash {#eth_gettransactionbyhash}

トランザクションハッシュを使用して、リクエストされたトランザクションに関する情報を返します。

**パラメータ**

1. `DATA`、32 バイト - トランザクションのハッシュ

```js
params: ["0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b"]
```

**戻り値**

`Object` - トランザクションオブジェクト、またはトランザクションが見つからなかった場合は`null`

- `blockHash`: `DATA`、32 バイト - このトランザクションが組み込まれていたブロックのハッシュ。 保留中の場合は`null`
- `blockNumber`: `QUANTITY` - このトランザクションが組み込まれていたブロックの番号 保留中の場合は`null`
- `from`: `DATA`、20 バイト - 送信者のアドレス
- `gas`: `QUANTITY` - 送信者が提供するガス
- `gasPrice`: `QUANTITY` - 送信者が指定したガス価格(wei 単位)
- `hash`: `DATA`、32 バイト - トランザクションのハッシュ
- `input`: `DATA` - トランザクションと共に送信されるデータ
- `nonce`: `QUANTITY` - 送信者がこのトランザクションより前に送信したトランザクションの数
- `to`: `DATA`、20 バイト - 受信者のアドレス。 コントラクト作成時のトランザクションは`null`
- `transactionIndex`: `QUANTITY` - ブロック内のトランザクションのインデックスの位置(整数)。 保留中の場合は`null`
- `value`: `QUANTITY` - 送金された価値(wei 単位)
- `v`: `QUANTITY` - ECDSA のリカバリ ID
- `r`: `QUANTITY` - ECDSA 署名 r
- `s`: `QUANTITY` - ECDSA 署名 s

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b"],"id":1}'
// Result
{
  "jsonrpc":"2.0",
  "id":1,
  "result":{
    "blockHash":"0x1d59ff54b1eb26b013ce3cb5fc9dab3705b415a67127a003c3e61eb445bb8df2",
    "blockNumber":"0x5daf3b", // 6139707
    "from":"0xa7d9ddbe1f17865597fbd27ec712455208b6b76d",
    "gas":"0xc350", // 50000
    "gasPrice":"0x4a817c800", // 20000000000
    "hash":"0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b",
    "input":"0x68656c6c6f21",
    "nonce":"0x15", // 21
    "to":"0xf02c1c8e6114b1dbe8937a39260b5b0a374432bb",
    "transactionIndex":"0x41", // 65
    "value":"0xf3dbb76162000", // 4290000000000000
    "v":"0x25", // 37
    "r":"0x1b5e176d927f8e9ab405058b2d2457392da3e20f328b16ddabcebc33eaac5fea",
    "s":"0x4ba69724e8f69de52f0125ad8b3c5c2cef33019bac3249e2c0a2192766d1721c"
  }
}
```

### eth_getTransactionByBlockHashAndIndex {#eth_gettransactionbyblockhashandindex}

ブロックのハッシュとトランザクションのインデックスの位置を使用して、トランザクションに関する情報を返します。

**パラメータ**

1. `DATA`、32 バイト - ブロックのハッシュ
2. `QUANTITY` - トランザクションのインデックスの位置(整数)

```js
params: [
  "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
  "0x0", // 0
]
```

**戻り値**については、[eth_getTransactionByHash](#eth_gettransactionbyhash)を参照してください。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockHashAndIndex","params":["0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b", "0x0"],"id":1}'
```

結果については、[eth_getTransactionByHash](#eth_gettransactionbyhash)を参照してください。

### eth_getTransactionByBlockNumberAndIndex {#eth_gettransactionbyblocknumberandindex}

ブロック番号とトランザクションのインデックスの位置を使用して、トランザクションに関する情報を返します。

**パラメータ**

1. `QUANTITY|TAG` - ブロックの番号、または文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください
2. `QUANTITY` - トランザクションのインデックスの位置

```js
params: [
  "0x9c47cf", // 10241999
  "0x24", // 36
]
```

**戻り値**については、[eth_getTransactionByHash](#eth_gettransactionbyhash)を参照してください。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockNumberAndIndex","params":["0x9c47cf", "0x24"],"id":1}'
```

結果については、[eth_getTransactionByHash](#eth_gettransactionbyhash)を参照してください。

### eth_getTransactionReceipt {#eth_gettransactionreceipt}

トランザクションのハッシュを使用して、トランザクションのレシートを返します。

**注** : 保留中のトランザクションでは、レシートは取得できません。

**パラメータ**

1. `DATA`、32 バイト - トランザクションのハッシュ

```js
params: ["0x85d995eba9763907fdf35cd2034144dd9d53ce32cbec21349d4b12823c6860c5"]
```

**戻り値** `Object` - トランザクションレシートのオブジェクト、またはレシートが見つからなかった場合は`null`

- `transactionHash`: `DATA`、32 バイト - トランザクションのハッシュ
- `transactionIndex`: `QUANTITY` - ブロック内のトランザクションのインデックスの位置(整数)。
- `blockHash`: `DATA`、32 バイト - このトランザクションが組み込まれていたブロックのハッシュ。
- `blockNumber`: `QUANTITY` - このトランザクションが組み込まれていたブロックの番号
- `from`: `DATA`、20 バイト - 送信者のアドレス
- `to`: `DATA`、20 バイト - 受信者のアドレス。 コントラクト作成時のトランザクションは null
- `cumulativeGasUsed`: `QUANTITY` - ブロック内でこのトランザクションの実行時に使用されたガスの総量
- `effectiveGasPrice` : `QUANTITY` - ガスユニットごとに支払われるベースフィーとチップの合計
- `gasUsed`: `QUANTITY` - この特定のトランザクションのみで使用されたガスの量
- `contractAddress`: `DATA`、20 バイト - コントラクト作成のトランザクションの場合は、作成されたコントラクトのアドレス、その他の場合は`null`
- `logs`: `Array` - このトランザクションが生成したログオブジェクトの配列
- `logsBloom`: `DATA`、256 バイト - 関連ログを迅速に取得するためのライトクライアント用のブルームフィルター。
- `type`: `QUANTITY` - トランザクションタイプの整数、`0x0`でレガシートランザクション、 `0x1`でアクセスリストタイプ、`0x2`で動的フィー。

また、以下のうち*いずれか*を返します。

- `root` : `DATA`、32 バイト - トランザクション後の状態ルート(ビザンチウム以前)
- `status`: `QUANTITY` - `1`(成功)、または`0`(失敗)

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["0x85d995eba9763907fdf35cd2034144dd9d53ce32cbec21349d4b12823c6860c5"],"id":1}'
// Result
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "blockHash":
      "0xa957d47df264a31badc3ae823e10ac1d444b098d9b73d204c40426e57f47e8c3",
    "blockNumber": "0xeff35f",
    "contractAddress": null, // string of the address if it was created
    "cumulativeGasUsed": "0xa12515",
    "effectiveGasPrice": "0x5a9c688d4",
    "from": "0x6221a9c005f6e47eb398fd867784cacfdcfff4e7",
    "gasUsed": "0xb4c8",
    "logs": [{
      // logs as returned by getFilterLogs, etc.
    }],
    "logsBloom": "0x00...0", // 256 byte bloom filter
    "status": "0x1",
    "to": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
    "transactionHash":
      "0x85d995eba9763907fdf35cd2034144dd9d53ce32cbec21349d4b12823c6860c5",
    "transactionIndex": "0x66",
    "type": "0x2"
  }
}
```

### eth_getUncleByBlockHashAndIndex {#eth_getunclebyblockhashandindex}

ハッシュとアンクルインデックスの位置を使用して、ブロックのアンクルに関する情報を返します。

**パラメータ**

1. `DATA`、32 バイト - ブロックのハッシュ
2. `QUANTITY` - アンクルのインデックスの位置

```js
params: [
  "0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b",
  "0x0", // 0
]
```

**戻り値**については、[eth_getBlockByHash](#eth_getblockbyhash)を参照してください。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockHashAndIndex","params":["0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b", "0x0"],"id":1}'
```

結果については、[eth_getBlockByHash](#eth_getblockbyhash)を参照してください。

**注**: アンクルには、個々のトランザクションは含まれていません。

### eth_getUncleByBlockNumberAndIndex {#eth_getunclebyblocknumberandindex}

ブロック番号とアンクルのインデックスの位置を使用して、ブロックのアンクルに関する情報を返します。

**パラメータ**

1. `QUANTITY|TAG` - ブロックの番号、または文字列`"latest"`、`"earliest"`、`"pending"`のいずれか。[デフォルトのブロックパラメータ](/developers/docs/apis/json-rpc/#default-block)を参照してください
2. `QUANTITY` - アンクルのインデックスの位置

```js
params: [
  "0x29c", // 668
  "0x0", // 0
]
```

**戻り値**については、[eth_getBlockByHash](#eth_getblockbyhash)を参照してください。

**注**: アンクルには、個々のトランザクションは含まれていません。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockNumberAndIndex","params":["0x29c", "0x0"],"id":1}'
```

結果については、[eth_getBlockByHash](#eth_getblockbyhash)を参照してください。

### eth_newFilter {#eth_newfilter}

(ログの)状態変更を通知するために、フィルターオプションに基づいてフィルターオブジェクトを作成します。 状態変更を確認するには、[eth_getFilterChanges](#eth_getfilterchanges)を呼び出します。

**トピックフィルターを指定する際の注:** トピックは順序に依存します。 トピック [A, B] を持つログのトランザクションは、下記のトピックフィルターによってマッチングされます。

- `[]` 「条件なし」
- `[A]` 「最初の位置が A(その後の位置については条件なし) 」
- `[null, B]`「最初の位置は条件なし、かつ 2 番目の位置が B(その後の位置については条件なし) 」
- `[A, B]`「最初の位置が A、かつ 2 番目の位置が B(その後の位置については条件なし) 」
- `[[A, B], [A, B]]`「最初の位置が(A または B)、かつ 2 番目の位置が(A または B)(その後の位置については条件なし) 」
- **パラメータ**

1. `Object` - フィルターオプション

- `fromBlock`: `QUANTITY|TAG` - (オプション、デフォルトは`"latest"`)ブロック番号(整数)、または`"latest"`(最後にマイニングされたブロック)、`"pending"`、`"earliest"`(まだマイニングされていないトランザクション)のいずれか
- `toBlock`: `QUANTITY|TAG` - (オプション、デフォルト: `"latest"`)ブロック番号(整数) 。 `"latest"`(最後にマイニングされたブロック)。`"pending"`、`"earliest"`(まだマイニングされていないトランザクション)のいずれか
- `address`: `DATA|Array`、20 バイト - (オプション)ログの生成元となるコントラクトアドレス、またはアドレスのリスト
- `topics`: `Array of DATA`、- (オプション)32 バイトの`DATA`トピックの配列。 トピックは順序に依存します。 各トピックは「or」オプションの DATA 配列にすることも可能

```js
params: [
  {
    fromBlock: "0x1",
    toBlock: "0x2",
    address: "0x8888f1f195afa192cfee860698584c030f4c9db1",
    topics: [
      "0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b",
      null,
      [
        "0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b",
        "0x0000000000000000000000000aff3454fce5edbc8cca8697c15331677e6ebccc",
      ],
    ],
  },
]
```

**戻り値** `QUANTITY` - フィルター ID

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newFilter","params":[{"topics":["0x12341234"]}],"id":73}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

### eth_newBlockFilter {#eth_newblockfilter}

新しいブロックの到着を通知するために、ノードにフィルターを作成します。 状態変更を確認するには、[eth_getFilterChanges](#eth_getfilterchanges)を呼び出します。

**パラメータ** なし

**戻り値** `QUANTITY` - フィルター ID

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newBlockFilter","params":[],"id":73}'
// Result
{
  "id":1,
  "jsonrpc":  "2.0",
  "result": "0x1" // 1
}
```

### eth_newPendingTransactionFilter {#eth_newpendingtransactionfilter}

新しい保留中のトランザクションの到着を通知するために、ノードにフィルターを作成します。 状態変更を確認するには、[eth_getFilterChanges](#eth_getfilterchanges)を呼び出します。

**パラメータ** なし

**戻り値** `QUANTITY` - フィルター ID

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newPendingTransactionFilter","params":[],"id":73}'
// Result
{
  "id":1,
  "jsonrpc":  "2.0",
  "result": "0x1" // 1
}
```

### eth_uninstallFilter {#eth_uninstallfilter}

指定された ID を使用して、フィルターをアンインストールします。 ウォッチが不要になったら、必ず呼び出す必要があります。 また、フィルターは一定期間、[eth_getFilterChanges](#eth_getfilterchanges)を使用してリクエストされなければタイムアウトします。

**パラメータ**

1. `QUANTITY` - フィルター ID

```js
params: [
  "0xb", // 11
]
```

**戻り値** `Boolean` - フィルターのアンインストールに成功した場合は`true`、その他の場合は`false`

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_uninstallFilter","params":["0xb"],"id":73}'
// Result
{
  "id":1,
  "jsonrpc": "2.0",
  "result": true
}
```

### eth_getFilterChanges {#eth_getfilterchanges}

フィルターのポーリングメソッド。最後のポーリング以降に発生したログの配列を返します。

**パラメータ**

1. `QUANTITY` - フィルター ID

```js
params: [
  "0x16", // 22
]
```

**戻り値** `Array` - ログオブジェクトの配列、または最後のポーリングから変化がない場合は空の配列

- `eth_newBlockFilter`で作成したフィルターでは、ブロックハッシュ(`DATA`、32 バイト)が返されます(例: `["0x3454645634534..."]`)
- `eth_newPendingTransactionFilter`で作成したフィルターでは、トランザクションハッシュ(`DATA`、32 バイト)が返されます(例: `["0x6345343454645..."]`)
- `eth_newFilter`で作成されたフィルターでは、ログは下記のパラメータを持つオブジェクトになります
  - `removed`: `TAG` - チェーンの再編成によってログが削除された場合は`true`。 ログが有効な場合は`false`
  - `logIndex`: `QUANTITY` - ブロック内のログインデックスの位置(整数)。 保留中のログの場合は`null`
  - `transactionIndex`: `QUANTITY` - ログの作成元のトランザクションのインデックスの位置(整数) 。 保留中のログの場合は`null`
  - `transactionHash`: `DATA`、32 バイト - このログが作成されたトランザクションのハッシュ。 保留中のログの場合は`null`
  - `blockHash`: `DATA`、32 バイト - このログが存在したブロックのハッシュ。 保留中の場合は`null` 保留中のログの場合は`null`
  - `blockNumber`: `QUANTITY` - このログが存在したブロックの番号。 保留中の場合は`null` 保留中のログの場合は`null`
  - `address`: `DATA`、20 バイト - このログの作成元のアドレス
  - `data`: `DATA` - ゼロまたは 32 バイト以上のインデックス付けがされていないログの引数を含む
  - `topics`: `Array of DATA` - 0 から 4 つの 32 バイトのインデックス付けされたログの引数の`DATA`配列。 (*Solidity*の場合: 最初のトピックはイベント(例: `Deposit(address,bytes32,uint256)`)の署名の*ハッシュ*。ただし、`anonymous`指定子でイベントを宣言した場合を除く)
- **例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getFilterChanges","params":["0x16"],"id":73}'
// Result
{
  "id":1,
  "jsonrpc":"2.0",
  "result": [{
    "logIndex": "0x1", // 1
    "blockNumber":"0x1b4", // 436
    "blockHash": "0x8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcfdf829c5a142f1fccd7d",
    "transactionHash":  "0xdf829c5a142f1fccd7d8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcf",
    "transactionIndex": "0x0", // 0
    "address": "0x16c5785ac562ff41e2dcfdf829c5a142f1fccd7d",
    "data":"0x0000000000000000000000000000000000000000000000000000000000000000",
    "topics": ["0x59ebeb90bc63057b6515673c3ecf9438e5058bca0f92585014eced636878c9a5"]
    },{
      ...
    }]
}
```

### eth_getFilterLogs {#eth_getfilterlogs}

指定された ID のフィルターに一致する、すべてのログの配列を返します。

**パラメータ**

1. `QUANTITY` - フィルター ID

```js
params: [
  "0x16", // 22
]
```

**戻り値**については、 [eth_getFilterChanges](#eth_getfilterchanges)を参照してください。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getFilterLogs","params":["0x16"],"id":74}'
```

結果については、[eth_getFilterChanges](#eth_getfilterchanges)を参照してください。

### eth_getLogs {#eth_getlogs}

指定されたフィルターオブジェクトに一致する、すべてのログの配列を返します。

**パラメータ**

1. `Object` - フィルターオプション

- `fromBlock`: `QUANTITY|TAG` - (オプション、デフォルトは`"latest"`)ブロック番号(整数)、または`"latest"`(最後にマイニングされたブロック)、`"pending"`、`"earliest"`(まだマイニングされていないトランザクション)のいずれか
- `toBlock`: `QUANTITY|TAG` - (オプション、デフォルト: `"latest"`)ブロック番号(整数) 。 `"latest"`(最後にマイニングされたブロック)。`"pending"`、`"earliest"`(まだマイニングされていないトランザクション)のいずれか
- `address`: `DATA|Array`、20 バイト - (オプション)ログの生成元となるコントラクトアドレス、またはアドレスのリスト
- `topics`: `Array of DATA`、- (オプション)32 バイトの`DATA`トピックの配列。 トピックは順序に依存します。 各トピックは「or」オプションの DATA 配列にすることも可能
- `blockhash`: `DATA`、32 バイト - (オプション、**実装予定**) EIP-234 が追加されたことにより、`blockHash`が新たなフィルターオプションになります。これは、返されるログを 32 バイトのハッシュ`blockHash`を持つ単一のブロックに制限します。 `blockHash`を使用することは、`fromBlock`と`toBlock`に`blockHash`のハッシュのブロック番号を指定することと同等です。 `blockHash`がフィルター条件にある場合、`fromBlock`と`toBlock`は使用できません。

```js
params: [
  {
    topics: [
      "0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b",
    ],
  },
]
```

**戻り値**については、 [eth_getFilterChanges](#eth_getfilterchanges)を参照してください。

**例**

```js
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getLogs","params":[{"topics":["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b"]}],"id":74}'
```

結果については、[eth_getFilterChanges](#eth_getfilterchanges)を参照してください。

## 使用例 {#usage-example}

### JSON_RPC を使用してコントラクトをデプロイする {#deploying-contract}

このセクションでは、RPC インターフェースのみを使用してコントラクトをデプロイする方法について、実例を交えて説明します。 コントラクトをデプロイする際には、複雑さを抽象化できる別の方法があります。例えば、RPC インターフェースを基盤としたライブラリ([web3.js](https://web3js.readthedocs.io/)、[web3.py](https://github.com/ethereum/web3.py)など)を使用できます。 通常、抽象化すると簡単に理解できるようになり、エラーも起こりにくくなります。それでも、内部の仕組みを知っておくことで、理解を深めることができます。

以下は、JSON-RPC インターフェースを使用してイーサリアムノードにデプロイされる、`Multiply7`と呼ばれる簡単なスマートコントラクトです。 このチュートリアルは、読者がすでに Geth ノードを実行していることを前提としています。 ノードとクライアントの詳細については、[こちら](/developers/docs/nodes-and-clients/run-a-node)をご覧ください。 また、Geth クライアント以外の HTTP JSON-RPC を起動する方法については、それぞれの[クライアント](/developers/docs/nodes-and-clients/)のドキュメントを参照してください。 ほとんどのクライアントは、デフォルトでは`localhost:8545`で動作します。

```javascript
contract Multiply7 {
    event Print(uint);
    function multiply(uint input) returns (uint) {
        Print(input * 7);
        return input * 7;
    }
}
```

まず、HTTP RPC インターフェースが有効になっていることを確認します。 つまり、Geth の起動時に`--http`フラグを設定します。 この例では、プライベート開発チェーン上の Geth ノードを使用します。 このアプローチを使用する際には、本物のネットワーク上の Ether は必要ありません。

```bash
geth --http --dev console 2>>geth.log
```

これにより、HTTP RPC インターフェースが`http://localhost:8545`で開始します。

[curl](https://curl.se)を使用して Coinbase アドレスと残高を取得することにより、インターフェースが実行されていることを確認できます。 例で示されているデータは、ローカルノードによって異なりますのでご注意ください。 これらのコマンドを試す場合は、2 番目の curl リクエストの request パラメータの値を、1 番目の curl リクエストから返された result パラメータに置き換えてください。

```bash
curl --data '{"jsonrpc":"2.0","method":"eth_coinbase", "id":1}' -H "Content-Type: application/json" localhost:8545
{"id":1,"jsonrpc":"2.0","result":["0x9b1d35635cc34752ca54713bb99d38614f63c955"]}

curl --data '{"jsonrpc":"2.0","method":"eth_getBalance", "params": ["0x9b1d35635cc34752ca54713bb99d38614f63c955", "latest"], "id":2}' -H "Content-Type: application/json" localhost:8545
{"id":2,"jsonrpc":"2.0","result":"0x1639e49bba16280000"}
```

数値は 16 進数でエンコードされているため、残高は 16 進数の wei で返されます。 Ether 単位で残高を確認したい場合は、Geth コンソールから Web3 を使用できます。

```javascript
web3.fromWei("0x1639e49bba16280000", "ether")
// "410"
```

これで、プライベート開発チェーンに Ether が存在するようになったため、コントラクトをデプロイできるようになりました。 最初のステップは、Multiply7 コントラクトを EVM に送信できるバイトコードにコンパイルすることです。 Solidity のコンパイラである solc をインストールするには、[Solidity ドキュメント](https://docs.soliditylang.org/en/latest/installing-solidity.html)を参照してください ([この例で使用しているコンパイラのバージョン](https://github.com/ethereum/solidity/releases/tag/v0.4.20)に適合するように、古い`solc`リリースを使用することをお勧めします)。

次のステップでは、Multiply7 コントラクトを、EVM に送信できるバイトコードにコンパイルします。

```bash
echo 'pragma solidity ^0.4.16; contract Multiply7 { event Print(uint); function multiply(uint input) public returns (uint) { Print(input * 7); return input * 7; } }' | solc --bin

======= <stdin>:Multiply7 =======
Binary:
6060604052341561000f57600080fd5b60eb8061001d6000396000f300606060405260043610603f576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063c6888fa1146044575b600080fd5b3415604e57600080fd5b606260048080359060200190919050506078565b6040518082815260200191505060405180910390f35b60007f24abdb5865df5079dcc5ac590ff6f01d5c16edbc5fab4e195d9febd1114503da600783026040518082815260200191505060405180910390a16007820290509190505600a165627a7a7230582040383f19d9f65246752244189b02f56e8d0980ed44e7a56c0b200458caad20bb0029
```

これで、コードがコンパイルされました。次に、デプロイに必要なガスの量を特定する必要があります。 RPC インターフェースには、推定値を取得するための`eth_estimateGas`メソッドがあります。

```bash
curl --data '{"jsonrpc":"2.0","method": "eth_estimateGas", "params": [{"from": "0x9b1d35635cc34752ca54713bb99d38614f63c955", "data": "0x6060604052341561000f57600080fd5b60eb8061001d6000396000f300606060405260043610603f576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063c6888fa1146044575b600080fd5b3415604e57600080fd5b606260048080359060200190919050506078565b6040518082815260200191505060405180910390f35b60007f24abdb5865df5079dcc5ac590ff6f01d5c16edbc5fab4e195d9febd1114503da600783026040518082815260200191505060405180910390a16007820290509190505600a165627a7a7230582040383f19d9f65246752244189b02f56e8d0980ed44e7a56c0b200458caad20bb0029"}], "id": 5}' -H "Content-Type: application/json" localhost:8545
{"jsonrpc":"2.0","id":5,"result":"0x1c31e"}
```

最後に、コントラクトをデプロイします。

```bash
curl --data '{"jsonrpc":"2.0","method": "eth_sendTransaction", "params": [{"from": "0x9b1d35635cc34752ca54713bb99d38614f63c955", "gas": "0x1c31e", "data": "0x6060604052341561000f57600080fd5b60eb8061001d6000396000f300606060405260043610603f576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063c6888fa1146044575b600080fd5b3415604e57600080fd5b606260048080359060200190919050506078565b6040518082815260200191505060405180910390f35b60007f24abdb5865df5079dcc5ac590ff6f01d5c16edbc5fab4e195d9febd1114503da600783026040518082815260200191505060405180910390a16007820290509190505600a165627a7a7230582040383f19d9f65246752244189b02f56e8d0980ed44e7a56c0b200458caad20bb0029"}], "id": 6}' -H "Content-Type: application/json" localhost:8545
{"id":6,"jsonrpc":"2.0","result":"0xe1f3095770633ab2b18081658bad475439f6a08c902d0915903bafff06e6febf"}
```

トランザクションがノードによって受け入れられると、トランザクションのハッシュが返されます。 このハッシュはトランザクションの追跡に使用されます。 次のステップは、コントラクトがデプロイされているアドレスを特定することです。 実行された各トランザクションによって、レシートが作成されます。 このレシートには、トランザクションが含まれていたブロックや、EVM によって使用されたガスの量など、トランザクションに関するさまざまな情報が含まれています。 トランザクション でコントラクトが作成される場合は、コントラクトのアドレスも含まれています。 レシートは、`eth_getTransactionReceipt`RPC メソッドで取得できます。

```bash
curl --data '{"jsonrpc":"2.0","method": "eth_getTransactionReceipt", "params": ["0xe1f3095770633ab2b18081658bad475439f6a08c902d0915903bafff06e6febf"], "id": 7}' -H "Content-Type: application/json" localhost:8545
{"jsonrpc":"2.0","id":7,"result":{"blockHash":"0x77b1a4f6872b9066312de3744f60020cbd8102af68b1f6512a05b7619d527a4f","blockNumber":"0x1","contractAddress":"0x4d03d617d700cf81935d7f797f4e2ae719648262","cumulativeGasUsed":"0x1c31e","from":"0x9b1d35635cc34752ca54713bb99d38614f63c955","gasUsed":"0x1c31e","logs":[],"logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","status":"0x1","to":null,"transactionHash":"0xe1f3095770633ab2b18081658bad475439f6a08c902d0915903bafff06e6febf","transactionIndex":"0x0"}}
```

これで、コントラクトが`0x4d03d617d700cf81935d7f797f4e2ae719648262`に作成されました。 レシートの代わりに null の結果が得られた場合は、トランザクションがまだブロックに含まれていないことを意味します。 しばらく待ってから、自分のマイナーが実行中になっていることを確認してから再試行してください。

#### スマートコントラクトとのやりとり {#interacting-with-smart-contract}

この例では、`eth_sendTransaction`を使用して、コントラクトの`multiply`メソッドにトランザクションを送信します。

`eth_sendTransaction`にはいくつかの引数、具体的には`from`、`to`、`data`が必要です。 `from`は、アカウントの公開アドレスで、`to`はコントラクトアドレスです。 引数`data`には、どのメソッドがどの引数で呼び出されるのかが定義されたペイロードが含まれています。 ここで、[ABI(アプリケーション・バイナリー・インターフェース)](https://docs.soliditylang.org/en/latest/abi-spec.html)を使用します。 ABI は、EVM のためにデータの定義とエンコードの方法を明示した JSON ファイルです。

ペイロードのバイトによって、コントラクトのどのメソッドが呼び出されるかが定義されています。 これは、関数名とその引数の型を 16 進数でエンコードした Keccak ハッシュの最初の 4 バイトです。 multiply 関数は uint256 のエイリアスである uint を受け取ります。 そのため、以下のようになります。

```javascript
web3.sha3("multiply(uint256)").substring(0, 10)
// "0xc6888fa1"
```

次のステップでは、引数をエンコードします。 uint256 は 1 つのみです。この例では 6 とします。 ABI には、uint256 型のエンコード方法を指定するセクションがあります。

`int<M>: enc(X)`は、X のビッグエンディアンの 2 の補数表現でのエンコーディングです。長さが 32 バイトの倍数になるように、X が負の場合は 0xff を使用して、正の場合は 0>バイトを使用して左詰めされます。

これにより、`00000000000000000000000000000000000000000000006`とエンコードされます。

関数セレクターとエンコードされた引数を組み合わせると、データは `0xc6888fa10000000000000000000000000000000000000000000000000000000000000006`になります。

これを、以下のようにノードに送信します。

```bash
curl --data '{"jsonrpc":"2.0","method": "eth_sendTransaction", "params": [{"from": "0xeb85a5557e5bdc18ee1934a89d8bb402398ee26a", "to": "0x6ff93b4b46b41c0c3c9baee01c255d3b4675963d", "data": "0xc6888fa10000000000000000000000000000000000000000000000000000000000000006"}], "id": 8}' -H "Content-Type: application/json" localhost:8545
{"id":8,"jsonrpc":"2.0","result":"0x759cf065cbc22e9d779748dc53763854e5376eea07409e590c990eafc0869d74"}
```

トランザクションが送信されたので、トランザクションハッシュが返されました。 取得したレシートには、以下が含まれています。

```javascript
{
   blockHash: "0xbf0a347307b8c63dd8c1d3d7cbdc0b463e6e7c9bf0a35be40393588242f01d55",
   blockNumber: 268,
   contractAddress: null,
   cumulativeGasUsed: 22631,
   gasUsed: 22631,
   logs: [{
      address: "0x6ff93b4b46b41c0c3c9baee01c255d3b4675963d",
      blockHash: "0xbf0a347307b8c63dd8c1d3d7cbdc0b463e6e7c9bf0a35be40393588242f01d55",
      blockNumber: 268,
      data: "0x000000000000000000000000000000000000000000000000000000000000002a",
      logIndex: 0,
      topics: ["0x24abdb5865df5079dcc5ac590ff6f01d5c16edbc5fab4e195d9febd1114503da"],
      transactionHash: "0x759cf065cbc22e9d779748dc53763854e5376eea07409e590c990eafc0869d74",
      transactionIndex: 0
  }],
  transactionHash: "0x759cf065cbc22e9d779748dc53763854e5376eea07409e590c990eafc0869d74",
  transactionIndex: 0
}
```

レシートには、ログが含まれています。 このログは、EVM によってトランザクションの実行時に生成され、レシートに含まれます。 `multiply`関数によって、`Print`イベントが入力の 7 倍で発生したことが示されています。 `Print`イベントの引数は uint256 であるため、ABI ルールに従ってデコードできます。これにより、期待される 10 進数である 42 が得られます。 このデータのほかに、トピックを使用することでどのイベントによってログが作成されたかを把握することもできます。

```javascript
web3.sha3("Print(uint256)")
// "24abdb5865df5079dcc5ac590ff6f01d5c16edbc5fab4e195d9febd1114503da"
```

ここまで、最も一般的なタスクをいくつか簡単に紹介し、JSON-RPC を直接使用するための方法を実例を交えて説明しました。

## 関連トピック {#related-topics}

- [JSON-RPC の仕様](http://www.jsonrpc.org/specification)
- [ ノードとクライアント](/developers/docs/nodes-and-clients/)
- [JavaScript APIs](/developers/docs/apis/javascript/)
- [バックエンド API](/developers/docs/apis/backend/)
- [実行クライアント](/developers/docs/nodes-and-clients/#execution-clients)
