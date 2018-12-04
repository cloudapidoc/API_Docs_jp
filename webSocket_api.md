# WebSocket APIの概要

 

WebSocket規格とはTCPに基づいた新しいコンピューターネットワーク用の通信規格である。1つのtcpコネクションでのクライアントとサーバ間の双方向通信を実装しています。サーバ側からクライアントにデータをプッシュ配信できるため、頻繁な認証等のオーバーヘッドを削減できる。主なメリットが2つ挙げられます。

 

 

- 2者間のヘッダデータサイズが約2Byteと非常に小さくなります。

 

- サーバ側はクライアント側からの送信要求を受けてからデータを送信するのではなく、クライアントに新しいデータをプッシュ配信できます。

 

以上のことからWebSocketプロトコルは、仮想通貨相場やその取引のようなリアルタイム性が求められる通信の要件を満たすには最適なインターフェースです。

 

# リクエストとサブスクリプションについて

 

 

### 1. アドレス

 

- マーケット情報の要求先アドレス：wss://{HOST}/ws

 

### 2. データ圧縮

WebSocket API 経由で返されるデータはすべてGZIP圧縮されており、データ受信者側のクライアントにて解凍する必要があります。Pakoを使用することをお勧めします。（[【pako】](https://github.com/nodeca/pako) とは圧縮・解凍できるGZIPのリポジトリである）

 

### 3. WebSocketライブラリ

[【ws】](https://github.com/websockets/ws) が Node.js のWebSocketライブラリになります。

 

### 4. ハートビート

WebSocket API は双方向のハートビートが可能です。 Server または Client のどちらかが`ping` messageを送信することが可能であり、相手より`pong` messageが返信されます。

 

WebSocket Server がpingを送信する：

```

{"ping": 18212558000}

```

 WebSocket Client がpongを返信する：

```

 {"pong": 18212558000}

```

注：`"pong"`の応答値は`"ping"` の受信時の値と同一の値となります。

注：WebSocket Client と WebSocket Server との間でコネクション確立後、WebSocket Server は `5s`（変更の可能性がある）ごとに  WebSocket Client にpingを送信し、WebSocket Client より2回連続Pingへの応答がなければ、WebSocket Server はコネクションを切断します。WebSocket Clientが直近2回のPingのうちの1つの`ping`に応答すれば、WebSocket Serverはコネクションを維持します。

```

┌────────┐                         ┌────────┐ 

│ Client │                         │ Server │

└───┬────┘                         └───┬────┘

    │         {"ping": 18212558000}  │

    │<─────────────────────────────────┤

    │                                  │ wait 5s

    │                                  ├───┐

    │                                  │<──┘

    │         {"ping": 18212558000}  │

    │<─────────────────────────────────┤

    │                                  │

    │ {"pong": 18212558000}          │

    ├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄>│

    │                                  │

```

注：WebSocket Clientが直近2回のmessageのうちの1つの`ping`を送信すれば、WebSocket Serverはコネクションを維持します。　

 

```

┌────────┐                         ┌────────┐ 

│ Client │                         │ Server │

└───┬────┘                         └───┬────┘

    │         {"ping": 1523778470416}  │

    │<─────────────────────────────────┤

    │                                  │ wait 5s

    │                                  ├───┐

    │                                  │<──┘

    │         {"ping": 1523778475416}  │

    │<─────────────────────────────────┤

    │                                  │ wait 5s

    │                                  ├───┐

    │                                  │<──┘

    │                                  │

    │                                  │ close WebSocket connection

    │                                  ├───┐

    │                                  │<──┘

    │                                  │

 

```

注：2回連続WebSocket Client からの応答がなければ、WebSocket Server はコネクションを切断します。

WebSocket Client がpingを送信する：

```json

{"ping": 18212553000}

```

注：必ずLong型の"ping"を送信しなければなりません。そうしなければ、誤ったデータが返信されます：

```json

{

  "ts": 1492420473027,

  "status": "error",

  "err-code": "bad-request",

  "err-msg": "invalid ping"

}

```

 

WebSocket Server がpongを返信する：

```

{"pong": 18212553000}

```

注：返信される`"pong"` の応答値は受信した"ping"` の値と同一の値となります。

 

エラー時の返信フォーマット

```json

{

  "id": "id generate by client",

  "status": "error",

  "err-code": "err-code",

  "err-msg": "err-message",

  "ts": 1487152091345

}

```

注：`ts`は誤ったデータより生成されたタイムスタンプになります。単位：ミリ秒

###  5. Topicのフォーマット

 

データのサブスクリプションとリクエストともに`topic`を使います。`topic` の構文は以下になります。





| topic タイプ   | topic 文法 | sub/req   | 説明 |
| ------------- | ---------------------------- | ---------------|------------------------- |
| KLine         | market.$symbol.kline.$period | sub/req |チャート データ　单位時間内の始値、終値、最高値、最安値、取引量、取引高、約定回数等のデータを含む $period 選択可能な値：{ 1min, 5min, 15min, 30min, 60min, 4hour,1day, 1mon, 1week, 1year } |
| Market Depth  | market.$symbol.depth.$type   | sub/req |板の精度、異なる stepで買いⅠ、買いⅡ、買いⅢ等及び売りⅠ、売りⅡ、売りⅢ等の価格を纏める$type 選択できるstep：{ step0, step1, step2, step3, step4, step5, percent10 } （精度0-5）；step0の場合，異なる価格の注文を纏めない|
| Trade Detail  | market.$symbol.trade.detail  |  sub/req |  取引履歴、約定価格、取引量、買/売等の情報を含む       |
| Market Detail | market.$symbol.detail        |  sub/req |   直近24時間の取引量、取引高、始値、終値、最高値、最安値、約定回数等   |
| Market Tickers | market.tickers  |  sub|   公開したすべての通貨ペアの1日のチャート、直近24時間の取引量等の情報                            |

 

* `$symbol` とは通貨ペアのことである。選択可能な值： { ethbtc, ltcbtc, etcbtc, bchbtc...... }

* ユーザーが「精度」を選ぶと、同じ精度範囲内の注文は纏まって表示される。ただし、表示の仕方に影響が出るのみであり、実際の約定価格に影響しない。

 

###  6. データリクエスト (req/rep)

 

データをリクエストした際、リクエストに対する返信は一度しかされません。

####   データリクエストのフォーマット

 

```json

{

  "req": "topic to req",

  "id": "id generate by client"

}

```

* `"req"` の値は****topic**** である。`「5. Topicのフォーマット」` の ****topic** **フォーマット**********参照**

 

********データリクエストの正しい例********

 

​```json

{

  "req": "market.btcusdt.kline.1min",

  "id": "id10"

}

```

 

返信データ例：

```json

{

  "status": "ok",

  "rep": "market.btcusdt.kline.1min",

  "tick": [

    {

      "amount": 1.6206,

      "count":  3,

      "id":     1494465840,

      "open":   9887.00,

      "close":  9885.00,

      "low":    9885.00,

      "high":   9887.00,

      "vol":    16021.632026

    },

    {

      "amount": 2.2124,

      "count":  6,

      "id":     1494465900,

      "open":   9885.00,

      "close":  9880.00,

      "low":    9880.00,

      "high":   9885.00,

      "vol":    21859.023500

    }

  ]

}

```

 

********データリクエストのエラー例********

 

```json

{

  "req": "market.invalidsymbo.kline.1min",

  "id": "id10"

}

```

 

エラーメッセージ応答例：

```json

{

  "status": "error",

  "id": "id10",

  "err-code": "bad-request",

  "err-msg": "invalid topic market.invalidsymbol.trade.detail",

  "ts": 1494483996521

}

```

 

 

###  7. データのサブスクリプション(sub)

 

####データのサブスクリプション(sub)及びサブスクリプションデータ受信の流れ

```

┌────────┐                         ┌────────┐ 

│ Client │                         │ Server │

└───┬────┘                         └───┬────┘

    │ {"sub": "topic"}                 │

    ├─────────────────────────────────>│

    │                                  │

    │              {"subbed": "topic"} │

    │<┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┤

    │                                  │

    │        {"tick": "data of topic"} │

    │<┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┤

    │                                  │

    │        {"tick": "data of topic"} │

    │<┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┤

    │                                  │

```

注： topic のサブスクリプション完了後、 topicのデータが更新された際、 Server は一定の頻度でtopicの更新データを Clientにプッシュ配信する

####  データのサブスクリプションのフォーマット

 

 WebSocket API とのコネクションを行った後、 Serverに以下のフォーマットのデータを送信することにより、データを購読する：



```json

{

   "id": "id generate by client",

  "sub": "topic to sub",

  "freq-ms": 1000

}

```

注：`id`の パラメータは選択可能です

注：`freq-ms` のパラメータは選択可能であり、選択値は{ `1000`, `2000`, `3000`, `4000`, `5000` }になります。`freq-ms` をアップロードしない限り、デフォルト値を` 0 `はとなり、データが更新された際、直ちにClientにプッシュ配信されることになります。 Server のプッシュ配信の頻度は`freq-ms`にて調整できます。

 

********サブスクリプションの正しい例********

 

正しいサブスクリプション：

```json

{

  "sub": "market.btcusdt.kline.1min",

  "id": "id1"

}

```

* `"sub"` の値は****topic****であり、 「5. Topicのフォーマット」の ****topic** **のフォーマット********参照のこと

 

サブスクリプション完了後、データを返信された例：

```json

{

  "id": "id1",

  "status": "ok",

  "subbed": "market.btcusdt.kline.1min",

  "ts": 1489474081631

}

```

 

その後、 KLine が更新されるごとに、clientはデータを受信する。例：


```json

{

  "ch": "market.btcusdt.kline.1min",

  "ts": 1489474082831,

  "tick": {

    "id": 1489464480,

    "amount": 0.0,

    "count": 0,

    "open": 7962.62,

    "close": 7962.62,

    "low": 7962.62,

    "high": 7962.62,

    "vol": 0.0

  }

}

```

tick の説明: 

 

```

  "tick": {

    "id": チャートid,

    "amount": 取引量,

    "count": 約定回数,

    "open": 始値

    "close": 終値。最後の一本のKラインは最新の約定価格である

    "low": 最安値,

    "high": 最高値,

    "vol": 取引高, 取引価格*約定の量＝取引量合計

  }

 

```

 

 

********サブスクリプションの誤った例********

 

誤ったサブスクリプション（誤った symbol）：

```json

{

  "sub": "market.invalidsymbol.kline.1min",

  "id": "id2"

}

```

 

サブスクリプションが失敗した場合、データが返送された例：

```json

{

  "id": "id2",

  "status": "error",

  "err-code": "bad-request",

  "err-msg": "invalid topic market.invalidsymbol.kline.1min",

  "ts": 1494301904959

}

```

 

誤ったサブスクリプション（間違った topic）：


```json

{

  "sub": "market.btcusdt.kline.3min",

  "id": "id3"

}

```

サブスクリプションが失敗した場合、データが返信された例：

```json

{

  "id": "id3",

  "status": "error",

  "err-code": "bad-request",

  "err-msg": "invalid topic market.btcusdt.kline.3min",

  "ts": 1494310283622

}

```

 

###  8. サブスクライブの停止(unsub)

 

####  サブスクライブの停止フォーマット

 

WebSocket Client はデータのサブスクライブ後、そのデータｍｐサブスクライブの停止が可能です。停止後は、WebSocket Serverは 当該topicのデータをプッシュしなくなります。サブスクライブ停止のフォーマットについては以下のとおりとなります。：

 

```json

{

  "unsub": "topic to unsub",

  "id": "id generate by client"

}

```

 

********サブスクライブ停止の正しい例********

 

サブスクライブ停止の正しい例：

```json

{

  "unsub": "market.btcusdt.trade.detail",

  "id": "id4"

}

```

 

サブスクライブ停止完了後、データが返信された例：

```json

{

  "id": "id4",

  "status": "ok",

  "unsubbed": "market.btcusdt.trade.detail",

  "ts": 1494326028889

}

```

 

********サブスクライブ停止の誤った例********

 

サブスクライブ停止の誤った例（未購読のtopicを中止する）：

```json

{

  "unsub": "market.btcusdt.trade.detail",

  "id": "id5"

}

```

 

誤ったデータが返信された例

```json

{

  "id": "id5",

  "status": "error",

  "err-code": "bad-request",

  "err-msg": "unsub with not subbed topic market.btcusdt.trade.detail",

  "ts": 1494326217428

}

```

 

サブスクライブ停止の誤った例（存在しないtopicを中止する）：

```json

{

  "unsub": "not-exists-topic",

  "id": "id5"

}

```

誤ったデータが返信された例：

```json

{

  "id": "id5",

  "status": "error",

  "err-code": "bad-request",

  "err-msg": "unsub with not subbed topic not-exists-topic",

  "ts": 1494326318809

}

```

 

# WebSocket API Reference

 

##  KLine データのサブスクリプション market.$symbol.kline.$period

 

 WebSocket API とのコネクション確立後、 データのサブスクライブを行うにあたって、 以下のフォーマットでデータをServerにを送信します。

```json

{

  "sub": "market.$symbol.kline.$period",

  "id": "id generate by client"

}

```

 

| パラメータ        | 必要性有無  | タイプ     | 説明                | デフォルト値   | 値の範囲 |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 通貨ペア               |       | ethbtc,btusdt... |
| period       | true  | string | チャートの周期          |       | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |

 

********サブスクリプションの正しい例********

正しいサブスクリプション

```json

{

  "sub": "market.btcusdt.kline.1min",

  "id": "id1"

}

```

 

サブスクリプション完了後、データを返信された例

```json

{

  "id": "id1",

  "status": "ok",

  "subbed": "market.btcusdt.kline.1min",

  "ts": 1489474081631

}

```

 

その後、 Klineが更新されるごとに、Clientは以下のデータを受信する 

```json

{

  "ch": "market.btcusdt.kline.1min",

  "ts": 1489474082831,

  "tick": {

    "id": 1489464480,

    "amount": 0.0,

    "count": 0,

    "open": 7962.62,

    "close": 7962.62,

    "low": 7962.62,

    "high": 7962.62,

    "vol": 0.0

  }

}

```

 

Tickの 説明

```

  "tick": {

    "id": チャートid,

    "amount": 取引量,

    "count": 約定回数,

    "open": 始値,

    "close": 終値,最後の1本のラインは最新の約定価格である

    "low": 最安値,

    "high": 最高値,

    "vol": 取引高、取引価格*約定の量＝取引量合計

  }

 

```

 

********サブスクリプションの誤った****例********

 

誤ったサブスクリプション (誤った symbol)

```json

{

  "sub": "market.invalidsymbol.kline.1min",

  "id": "id2"

}

```

 

サブスクリプションが失敗した場合、データを返信された例

```json

{

  "id": "id2",

  "status": "error",

  "err-code": "bad-request",

  "err-msg": "invalid topic market.invalidsymbol.kline.1min",

  "ts": 1494301904959

}

```

 

誤ったサブスクリプション(誤った topic)

```

{

  "sub": "market.btcusdt.kline.3min",

  "id": "id3"

}

```

 

サブスクリプションが失敗した場合の応答例

```json

{

  "id": "id3",

  "status": "error",

  "err-code": "bad-request",

  "err-msg": "invalid topic market.btcusdt.kline.3min",

  "ts": 1494310283622

}

```

 

##  KLine データのリクエスト market.$symbol.kline.$period  

 

```

{

  "req": "market.$symbol.kline.$period",

  "id": "id generated by client",

  "from": 1533536947, //optional, type: long, 2017-07-28T00:00:00+08:00 から2050-01-01T00:00:00+08:00 までの間の時刻。単位：秒

  "to": 1533536947 //optional, type: long, 2017-07-28T00:00:00+08:00 から2050-01-01T00:00:00+08:00 までの間の時刻。単位：秒。必ず fromより未来の日時である費用があります

}

```

 

| パラメータ         | 必要性有無  | タイプ     | 説明                | デフォルト値   | 値の範囲 |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 通貨ペア                |       | btcusdt, ethusdt, ltcusdt...|
| period       | true  | string | チャートの周期          |       | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |

 

####  KLine データのリクエストの例

 

```json

{

  "req": "market.btcusdt.kline.1min",

  "id": "id10"

}

```

 

データが返信された例

```

{

  "rep": "market.btcusdt.kline.1min",

  "status": "ok",

  "id": "id10",

  "tick": [

    {

      "amount": 17.4805,

      "count":  27,

      "id":     1494478080,

      "open":   10050.00,

      "close":  10058.00,

      "low":    10050.00,

      "high":   10058.00,

      "vol":    175798.757708

    },

    {

      "amount": 15.7389,

      "count":  28,

      "id":     1494478140,

      "open":   10058.00,

      "close":  10060.00,

      "low":    10056.00,

      "high":   10065.00,

      "vol":    158331.348600

    },

    // more KLine data here

  ]

}

```

 

##  Market Depthデータのサブスクリプション market.$symbol.depth.$type 

 

WebSocket API とのコネクション確立後、 データのサブスクライブを行うにあたって、 以下のフォーマットでデータをServerにを送信します。：

```json

{

  "sub": "market.$symbol.depth.$type",

  "id": "id generated by client"

}

```

 

| パラメータ         | 必要性有無  | タイプ     | 説明                | デフォルト値   | 値の範囲                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 通貨ペア                |       | btcusdt, ethusdt, ltcusdt, etcusdt, bchusdt, ethbtc, ltcbtc, etcbtc, bchbtc... |
| type         | true  | string | Depth タイプ          |       | step0, step1, step2, step3, step4, step5（精度0-5）；step0の場合，異なる価格の注文を纏めない |

 

* ユーザーが「深度結合」を選ぶ時、同じ深度範囲内の注文は纏まって表示される。ただし、表示の仕方に影響が出るのみであり、実際の約定価格に影響しない。

 

サブスクリプションの正しい例

```json

{

  "sub": "market.btcusdt.depth.step0",

  "id": "id1"

}

```

 

サブスクリプション完了後、データが返送された例

```json

{

  "id": "id1",

  "status": "ok",

  "subbed": "market.btcusdt.depth.step0",

  "ts": 1489474081631

}

```

 

その後、depthが更新されたごとに 、client は以下のようにデータを受信する

```
{

  "ch": "market.btcusdt.depth.step0",

  "ts": 1489474082831,

  "tick": {

    "bids": [

    [9999.3900,0.0098], // [price, amount]

    [9992.5947,0.0560],

    // more Market Depth data here

    ]

    "asks": [

    [10010.9800,0.0099]

    [10011.3900,2.0000]

    //more data here

    ]

  }

}

```

 

tick の説明


```

  "tick": {

    "bids": [

    [買い1の価格,買い1の注文量]

    [買い2の価格,買い2の注文量]

    //more data here

    ]

    "asks": [

    [売り1の価格,売り1の注文量]

    [売り2の価格,売り2の注文量]

    //more data here

    ]

  }

```

 

##  Market Depth のデータのリクエスト market.$symbol.depth.$type 

 

```json

{

  "req": "market.$symbol.depth.$type",

  "id": "id generated by client"

}

```

 

####   Market Depth のデータのリクエストの例

```json

{

  "req": "market.btcusdt.depth.step0",

  "id": "id10"

}

```

 

データが返信された例：

```

{

  "rep": "market.btcusdt.depth.step0",

  "status": "ok",

  "id": "id10",

  "tick": {

    "bids": [

    [9999.3900,0.0098], // [price, amount]

    [9992.5947,0.0560],

    // more Market Depth data here

    ]

    "asks": [

    [10010.9800,0.0099]

    [10011.3900,2.0000]

    //more data here
    ]

  }

}

```

 

##  取引詳細データのサブスクリプション market.$symbol.trade.detail  

 

```json

{

  "sub": "market.$symbol.trade.detail",

  "id": "id generated by client"

}

```

 



| パラメータ   | 必要性有無  | タイプ     | 説明               | デフォルト値   | 値の範囲      |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------- |
| symbol   | true| string | 取引通貨ペア | ----- | btcusdt, ethusdt, ltcusdt, etcusdt, bchusdt, ethbtc, ltcbtc, etcbtc, bchbtc... |

 

サブスクリプションの正しい例：

 

```json

{

  "sub": "market.btcusdt.trade.detail",

  "id": "id1"

}

```

 

サブスクリプション完了後、データが返信された例：


```json

{

  "id": "id1",

  "status": "ok",

  "subbed": "market.btcusdt.trade.detail",

  "ts": 1489474081631

}

```

 

その後、取引明細が更新されるごとに、clientは以下のようにデータを受信する：

 

```json

{

  "ch": "market.btcusdt.trade.detail",

  "ts": 1489474082831,

  "tick": {

        "id": 14650745135,

        "ts": 1533265950234,

        "data": [

            {

                "amount": 0.0099,

                "ts": 1533265950234,

                "id": 146507451359183894799,

                "price": 401.74,

                "direction": "buy"

            },
        ]
    }
}

```

 

data の説明：

```

  "data": [

    {

      "id":        情報ID,

      "price":     約定価格,

      "time":      約定時間,

      "amount":    取引量,

      "direction": 取引方向（買い/売り）

      "tradeId":   取引ID,

      "ts":        タイムスタンプ

    }

  ]

```

 

##  Trade Detail のデータのリクエストmarket.$symbol.trade.detail 

 

```

{

  "req": "market.$symbol.trade.detail",

  "id": "id generated by client"

}

```

 

* 取引明細のデータは直近の300 個しか取得できない

 

####   Trade Detail のデータのリクエストの例

```

{

  "req": "market.btcusdt.trade.detail",

  "id": "id11"

}

```

 

データが返信された例：

```json

{

  "rep": "market.btcusdt.trade.detail",

  "status": "ok",

  "id": "id11",

  "data": [

    {

      "id":        601595424,

      "price":     10195.64,

      "time":      1494495766,

      "amount":    0.2943,

      "direction": "buy",

      "tradeId":   601595424,

      "ts":        1494495766000

    },

    {

      "id":        601595423,

      "price":     10195.64,
      
      "time":      1494495711,

     "amount":    0.2430,

      "direction": "buy",

      "tradeId":   601595423,

      "ts":        1494495711000

    }

  ]

}

```

 

##  マーケット詳細データのリクエストmarket.$symbol.detail 

```json

{

  "req": "market.$symbol.detail",

  "id": "id generated by client"

}

```

 

* 現在のマーケット詳細しか返信されない

 

####   マーケット詳細 データのリクエストの例

 

```json

{

  "req": "market.btcusdt.detail",

  "id": "id12"

}

```

 

データが返信された例：

```json

{

  "rep": "market.btcusdt.detail",

  "status": "ok",

  "id": "id12",

  "tick": {

    "amount": 12224.2922,

   "open":   9790.52,

    "close":  10195.00,

    "high":   10300.00,

    "ts":     1494496390000,

    "id":     1494496390,

    "count":  15195,

    "low":    9657.00,

    "vol":    121906001.754751

  }

}

```

 