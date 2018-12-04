# REST API

APIはプログラムによる取引を実装、可能にします。

APIを使えば以下の機能が使用できます。

- マーケット情報の取得（チャート、深度、リアルタイム約定、24時間チャートのモニタリング）
- アカウント資産情報の取得
- 注文、キャンセル操作
- 注文情報の取得


# セキュリティ認証
目下apikeyの申請や修正に関して、“アカウント - API管理”の画面で操作をします。その中でもAccessKeyはAPIアクセスに必要な秘密鍵で、SecretKeyはユーザーのリクエストに対してサインするための秘密鍵です。（申請時に表示されます）

**重要な注意事項：これらの二つの秘密鍵はアカウントのセキュリティに関する重要なものなので絶対に他人に教えないでください**



## リクエストに必要な構成
セキュリティ上の理由で、Quotes API以外のAPIリクエストには全て署名が必要となります。正当なリクエストは以下で構成されています。

→リクエスト時のURL：HOST(サーバ)の後ろにメソッド名が続きます。
例）https://{HOST}/v1/order/orders
　

* AccessKeyId： 適用するアクセスキー

* SignatureMethod:署名の演算時に用いるハッシュベースプロトコル、ここではHmacSHA256を指定します。

* SignatureVersion：署名プロトコルのバージョン、ここでは2を指定します。

*Timestamp：リクエスト時のタイムスタンプ(UTC 時間) 。タイムスタンプをクエリに含めることで、第三者がリクエストを傍受するのを防ぐことができます。例：2017-05-11T16:22:06。タイムスタンプはUTC 時間であることに注意してください。 

* 必須、オプションのパラメーター各メソッドには、API呼び出しを定義するための一連の必須、オプションのパラメーターがあります。これらのパラメータとその意味は、各メソッドの説明で確認できます。
GETリクエストの場合、各メソッドのパラメータに署名する必要があることに注意してください。POSTリクエストの場合、各メソッドのパラメータは署名されていません。つまり、POSTリクエストにはAccessKeyIdとSignatureMethodだけが必要となり、SignatureVersion、タイムスタンプの4つのパラメータ、その他のパラメータはbodyに配置されます。


* Signature：署名に基づいて計算された値。署名が有効で改ざんされていないことを保証するために使用されます。


以下、リクエスト例：
```
https://{host}/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15%3A19%3A30
&order-id=1234567890
&Signature=calculated value
```
# 署名のオペレーション
APIリクエストは、インターネット経由で送信されている間に改ざんされる可能性が最も高いです。要求が変更されていないことを確認するため、各リクエスト（Quote APIを除く）に署名を含めて、転送中にパラメータまたはパラメータの値が変更されたことを確認します。

### 署名の演算に必要な手順：

1. 署名を演算するための仕様要求
HMACは署名演算に使用されるため、異なるコンテンツを使用して演算された結果は完全に異なります。 したがって、署名演算の前に、要求を標準化してください。 以下は注文詳細要求を照会する例です。
```
https://{HOST}/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15:19:30
&order-id=1234567890
```

2. リクエスト方法（GET 或いは POST），続けて改行を追加する\n。
```
GET\n
```
3. 小文字のアクセスアドレスに続けて改行を追加する\n。
```
{host}\n
```
4. アクセスメソッドへのパス、続けて改行を追加する\n。
```
/v1/order/orders\n
```
5. パラメータ名は、ASCIIコードの順にソートされます（UTF-8エンコーディングとURIエンコーディングを使用すると、16進文字は大文字にする必要があり、 '：'は '％3A'、スペースは'％20'といった具合にエンコードされます）。
例えば、エンコード後の要求パラメータの元の順序は次のとおりです。```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
order-id=1234567890
SignatureMethod=HmacSHA256
SignatureVersion=2
Timestamp=2017-05-11T15%3A19%3A30
```

これらのパラメータは、次のようにソートされます。：
```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
SignatureMethod=HmacSHA256
SignatureVersion=2
Timestamp=2017-05-11T15%3A19%3A30
order-id=1234567890

```
上記の順序で、各パラメーターは文字 '＆'を使用して接続されます。
```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
````
計算のために署名される最後の文字列は
次のとおりです。```
GET\n
{host}\n
/v1/order/orders\n
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
````
署名を計算し、次の2つのパラメータを暗号化ハッシュ関数に渡します。
計算の対象となる文字列
```
GET\n
{host}\n
/v1/order/orders\n
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
```
署名された秘密鍵（SecretKey）
```
b0xxxxxx-c6xxxxxx-94xxxxxx-dxxxx
```
署名演算結果を取得し、Base64エンコーディングを実行する
```
4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o=
```
上記の値をパラメータSignatureの値としてAPIリクエストに追加します。 このパラメータをリクエストに追加する場合、その値はURIエンコードされている必要があります。

最終的に、サーバーに送信されるAPIリクエストは次のようになります。
```
https://{host}/v1/order/orders?AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&order-id=1234567890&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&Signature=4F65x5A2bLyMWVQj3Aqp%2BB4w%2BivaA7n5Oi2SuYtCJ9o%3D
```

#  リクエストの説明

1. アクセスアドレス：ドキュメントの{HOST}をサービスプロバイダのホストに置き換えます
2. Content-Type：application / jsonはPOSTリクエストヘッダで宣言されなければならない; Content-Type：application / x-www-form-urlencodedはGETリクエストヘッダで宣言されなければならない。 （中国語のユーザーはAccept-Language：zh-cnを設定することをお勧めします）
3. すべてのリクエストパラメータは、APIの記述に従ってパラメータをカプセル化します。
4. パラメーターをカプセル化するAPIリクエストTまたはGETを介してサーバーに送信されます。
5. サーバーは要求を処理し、対応するJSON形式の結果を返します。
6. httpsリクエストを使用してください。
7. 頻度を制限する（各インターフェイス、トレーディングAPIのみ、マーケットAPIは制限されません）は100秒で100回です。
8. クエリアセットの詳細メソッドコールシーケンス：現在のユーザーのすべてのアカウントを照会 - >指定されたアカウントの残高を照会する


# API Reference

``` ヘッダーのユーザーエージェントを、次のように設定してください。 'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.71 Safari/537.36' ```

``` symbol 規則： 基盤銘柄（仮想通貨）+銘柄（仮想通貨）の評価額。例えばBTC/USDT，symbolはbtcusdt；ETH/BTC， symbolはethbtc。その他```

## インターフェースリスト
| インターフェースデータタイプ| リクエスト方法 | タイプ     | 説明  | 認証が必要  |子アカウントも使用可能|
| ------------ | ----- | ------ | ----- | ----- | ----|
| マーケット       | [GET /market/history/kline](#get-markethistorykline-%E8%8E%B7%E5%8F%96k%E7%BA%BF%E6%95%B0%E6%8D%AE)  | GET | チャート  | N |Y|
| マーケット      | [GET /market/detail/merged](#get-marketdetailmerged-%E8%8E%B7%E5%8F%96%E8%81%9A%E5%90%88%E8%A1%8C%E6%83%85ticker)  | GET | ローリング24時間取引と最適な見積集計チャート（単一シンボル） | N |Y|
| マーケット       | [GET /market/tickers](#get-markettickers)  | GET | 全symbolの取引チャート | N |Y|
| マーケット       | [GET /market/depth](#get-marketdepth-獲得-market-depth-データ)  | GET | 市場Depthチャート（単一symbol） | N |Y|
| マーケット       | [GET /market/trade](#get-markettrade-獲得-trade-detail-データ)  | GET | 単一symbol最新約定記録| N |Y|
| マーケット       | [GET /market/history/trade](#get-markethistorytrade-最近の取引履歴をまとめて取得する)  | GET | 単一symbol約定記録 | N |Y|
| トレード銘柄情報       | [GET /v1/common/symbols](#get-v1commonsymbols-サポートされている全てのトランザクションペアと精度を照会する)  | GET | 取引銘柄の価格設定通貨と見積もり精度  |N |Y|
| トレード銘柄情報      | [GET /v1/common/currencys](#get-v1commoncurrencys-サポートされているすべての通貨を照会)  | GET | トレード銘柄リスト  |N |Y|
| システム情報       | [GET /v1/common/timestamp](#get-v1commontimestamp-システムの現在の時刻を照会する)  | GET | 現在のシステム時刻を照会  |N |Y|
|アカウント情報	|[GET /v1/account/accounts](#get-v1accountaccounts)|	GET| ユーザーの全てのアカウント状況を照会する| Y|Y|
|アカウント情報	|[GET /v1/account/accounts/{account-id}/balance](#get-v1accountaccountsaccount-idbalance-指定されたアカウントの残高を照会する)|GET|	指定されたアカウントの残高を照会する	|Y|Y|
|トレード	|[POST/v1/order/orders/place](#post-v1orderordersplace-注文)|POST|	注文	|Y|Y|
|トレード	|[POST/v1/order/orders/{order-id}/submitcancel](#post-v1orderordersorder-idsubmitcancel--注文リクエストのキャンセルリクエスト)|POST|	order-id注文をキャンセルする	|Y|Y|
|トレード	|[POST /v1/order/orders/batchcancel](#post-v1orderordersbatchcancel-一括注文の取り消し)|POST|	order_id, 一括注文の取り消し（up to 50)	|Y|Y|
|トレード	|[POST /v1/order/orders/batchCancelOpenOrders](#post-v1orderbatchcancelopenorders-対象となる注文を一括でキャンセルする)|POST|	注文条件による一括注文取消（up to 100)	|Y|Y|
|ユーザー注文情報	|[GET /v1/order/orders/{order-id}](#get-v1orderordersorder-id-注文の詳細を問い合わせる)|GET|注文IDに基づく注文詳細の照会| Y | Y |
|ユーザー注文情報	|[GET /v1/order/orders/{order-id}/matchresults](#get-v1orderordersorder-idmatchresults--注文の取引明細を照会する)	|GET| オーダーIDに基づいてオーダーのオーダー詳細を照会する	|Y|Y|
|ユーザー注文情報	|[GET /v1/order/orders](#get-v1orderorders-現在の注文履歴を照会する)	|GET|ユーザーの現在の注文または過去の注文を照会する(up to 100)	|Y|Y|
|ユーザー注文情報	|[GET /v1/order/matchresults](#get-v1ordermatchresults-現在の約定履歴を照会する)	|GET|ユーザーの現在の約定と約定履歴を照会する|Y|Y|
|ユーザー注文情報|[GET /v1/order/openOrders](#get-v1orderopenorders-現アカウントで未約定の注文をすべて取得する)	|GET|ユーザーの未約定注文を照会する(up to 500)|Y|	Y|
|入出金	|[POST /v1/dw/withdraw/api/create](#post-v1dwwithdrawapicreate-仮想通貨の出金申请)	|POST|仮想通貨の出金申请|	Y|N|
|入出金	|[POST /v1/dw/withdraw-virtual/{withdraw-id}/cancel](#post-v1dwwithdraw-virtualwithdraw-idcancel-申请取消提现虚拟币)|POST| 	仮想通貨の出金申请のキャンセル|Y|N|
|入出金	|[GET /v1/query/deposit-withdraw](#get-v1querydeposit-withdraw-仮想通貨の入出金記録の照会)	|GET|入金記録の照会|Y|N|


## マーケット

クォートインタフェースを呼び出すときは、getパラメータを追加し、キーはAccessKeyIdで、valueはWebページに適用されたapikeyのアクセスキーです。

例：

```
https://{HOST}/market/history/kline?period=1day&size=200&symbol=btcusdt&AccessKeyId=fff-xxx-ssss-kkk

```

#### GET /market/history/kline チャートデータの獲得

リクエスパラメーター: 

| パラメーター名称 | 必須かどうか  | タイプ    | 説明  | デフォルト値   |値の範囲  |
| ------------ | ----- | ------ | ----- | ----- | ------- |
| symbol       | true  | string | 取引ペア  |  | btcusdt, bchbtc, rcneth ...   |
| period       | true  | string | チャートタイプ |    | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |
| size | false | integer | データの獲得 | 150 | [1,2000] |

応答データ: 

| パラメーター名称   | 必須かどうか | データタイプ   | 説明  | 値の範囲   |
| ------ | ---- | ------ | ----------- | ------ |
| status | true | string | リクエスト処理結果    | "ok" , "error" |
| ts     | true | number | 生成応答時間点，单位：ミリ秒  |    |
| tick   | true | object | KLine データ   |      |
| ch     | true | string | データの所属 channel，フォマット： market.$symbol.kline.$period |    |

data 説明: 

```
  "data": [
{
    "id": チャートid,
    "amount": 約定量,
    "count": 約定数,
    "open":始値,
    "close":終値,
    "low": 最安値,
    "high": 最高値,
    "vol": 取引高, 取引価格*約定の量＝取引量合計
  }
]
```

リクエスト応答例: 

```
/* GET /market/history/kline?period=1day&size=200&symbol=btcusdt */
{
  "status": "ok",
  "ch": "market.btcusdt.kline.1day",
  "ts": 1499223904680,
  "data": [
{
    "id": 1499184000,
    "amount": 37593.0266,
    "count": 0,
    "open": 1935.2000,
    "close": 1879.0000,
    "low": 1856.0000,
    "high": 1940.0000,
    "vol": 71031537.97866500
  },
// more data here
]
}

/* GET /market/history/kline?period=not-exist&size=200&symbol=ethusdt */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid period"
}

/* GET /market/history/kline?period=1day&size=not-exist&symbol=ethusdt */
{
  "ts": 1490758221221,
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid size, valid range: [1,2000]"
}

/* GET /market/history/kline?period=1day&size=200&symbol=not-exist */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```

#### GET /market/detail/merged 集約チャートの獲得(Ticker)

リクエストパラメーター: 

| リクエスト名称   | 必須かどうか  | タイプ    | 説明  | デフォルト値   | 値の範囲   |
| ------------ | ----- | ------ | -----  | ---  |  ----------- |
| symbol    | true  | string | トレードペア   |   | btcusdt, bchbtc, rcneth ...|

応答データ: 

| パラメータ名称   | 必須かどうか | データタイプ   | 説明   | 値の範囲   |
| ------ | ---- | ------ | -------  | ----  |
| status | true | string | リクエスト处理结果  | "ok" , "error" |
| ts     | true | number | 生成応答時間点，单位：ミリ秒    |     |
| tick   | true | object | チャートデータ    |      |
| ch     | true | string | データの所属 channel，フォマット： market.$symbol.detail.merged |     |

tick 説明: 

```
  "tick": {
    "id": チャートid,
    "amount": 約定量,
    "count": 約定数,
    "open": 始値,
    "close": 終値,
  "low": 最安値,
    "high": 最高値,
    "vol": 取引高, 取引価格*約定の量＝取引合計金額
    "bid": [売値],
    "ask": [買値]
  }

```

リクエスト応答例: 

```
/* GET /market/detail/merged?symbol=ethusdt */
{
"status":"ok",
"ch":"market.ethusdt.detail.merged",
"ts":1499225276950,
"tick":{
  "id":1499225271,
  "ts":1499225271000,
  "close":1885.0000,
  "open":1960.0000,
  "high":1985.0000,
  "low":1856.0000,
  "amount":81486.2926,
  "count":42122,
  "vol":157052744.85708200,
  "ask":[1885.0000,21.8804],
  "bid":[1884.0000,1.6702]
  }
}

/* GET /market/detail/merged?symbol=not-exist */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol”
}

```
#### GET /market/tickers 
```json
{  
    "status":"ok",
    "ts":1510885463001,
    "data":[  
        {  
            "open":0.044297,      // チャート（日） 始値
            "close":0.042178,     // チャート（日） 終値
            "low":0.040110,       // チャート（日） 最安値
            "high":0.045255,      // チャート（日） 最高値
            "amount":12880.8510,  // 24時間の約定量
            "count":12838,        // 24時間の約定数数
            "vol":563.0388715740, // 24時間の取引高
            "symbol":"ethbtc"     // 通貨ペア
        },
        {  
            "open":0.008545,
            "close":0.008656,
            "low":0.008088,
            "high":0.009388,
            "amount":88056.1860,
            "count":16077,
            "vol":771.7975953754,
            "symbol":"ltcbtc"
        }
    ]
}
```
注意：通貨ペアがまだ取引を生成していない場合、返されたデータは内部にあります `open` `close` `high` `low` `amount` `count` `vol` の値は全て `null`
#### GET /market/depth 獲得 Market Depth データ

リクエストパラメーター:

| パラメーター名称   | 必須かどうか  | タイプ     | 説明    | デフォルト値   | 値の範囲   |
| ------  | ----- | ------ | ------  | ----- | -------  |
| symbol     | true  | string | トレードペア    |       | btcusdt, bchbtc, rcneth ... |
| type    | true  | string | Depth タイプ     |       | step0, step1, step2, step3, step4, step5（Depth統合 0-5）；step0の時，Depth統合しない |

* ユーザーが“深度統合”を選択した時，特定の見積もり精度内の市場未確定注文が結合されて表示されます。深度統合は表示モードを変更するのみで、実際の取引価格は変更されません。

応答データ:

| パラメーター名称   | 必須かどうか | データタイプ   | 説明    | 値の範囲   |
| ------ | ---- | ------ | -------  | ---  |
| status | true | string |       | "ok" 或いは "error" |
| ts     | true | number | 生成応答時間点，单位：ミリ秒    |     |
| tick   | true | object | Depth データ    |     |
| ch     | true | string | データの所属 channel，フォマット： market.$symbol.depth.$type |  |

tick 説明:

```
  "tick": {
    "id": メッセージid,
    "ts": メッセージ生成時間，单位：ミリ秒,
    "bids": 売値,[price(約定額), amount(約定量)], price降順,
    "asks": 買値price(約定額), amount(約定量)], price昇順
  }
```

リクエスト応答例:

```json
/* GET /market/depth?symbol=ethusdt&type=step1 */
{
  "status": "ok",
  "ch": "market.btcusdt.depth.step1",
  "ts": 1489472598812,
  "tick": {
    "id": 1489464585407,
    "ts": 1489464585407,
    "bids": [
      [7964, 0.0678], // [price, amount]
      [7963, 0.9162],
      [7961, 0.1],
      [7960, 12.8898],
      [7958, 1.2],
      [7955, 2.1009],
      [7954, 0.4708],
      [7953, 0.0564],
      [7951, 2.8031],
      [7950, 13.7785],
      [7949, 0.125],
      [7948, 4],
      [7942, 0.4337],
      [7940, 6.1612],
      [7936, 0.02],
      [7935, 1.3575],
      [7933, 2.002],
      [7932, 1.3449],
      [7930, 10.2974],
      [7929, 3.2226]
    ],
    "asks": [
      [7979, 0.0736],
      [7980, 1.0292],
      [7981, 5.5652],
      [7986, 0.2416],
      [7990, 1.9970],
      [7995, 0.88],
      [7996, 0.0212],
      [8000, 9.2609],
      [8002, 0.02],
      [8008, 1],
      [8010, 0.8735],
      [8011, 2.36],
      [8012, 0.02],
      [8014, 0.1067],
      [8015, 12.9118],
      [8016, 2.5206],
      [8017, 0.0166],
      [8018, 1.3218],
      [8019, 0.01],
      [8020, 13.6584]
    ]
  }
}

/* GET /market/depth?symbol=ethusdt&type=not-exist */
{
  "ts": 1490759358099,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid type"
}
```


#### GET /market/trade 取引詳細データを取得する
リクエストパラメーター:

| パラメーター名称   | 必須かどうか  | タイプ | 説明   | デフォルト値   | 値の範囲  |
| -------  | ----- | ------ | ------ | ----- | ---- |
| symbol   | true  | string | 取引通貨ペア   | btcusdt, bchbtc, rcneth ... |

応答リクエスト:

| パラメーター名称   | 必須かどうか | データタイプ   | 説明   | 値の範囲    |
| ------ | ---- | ------ | ----------| --------------- |
| status | true | string |    | "ok" 或いは "error" |
| ts     | true | number | 生成応答時間点，单位：ミリ秒    |      |
| tick   | true | object | Trade データ      |     |
| ch     | true | string | データの所属 channel，フォマット： market.$symbol.trade.detail |     |

tick 説明：

```
  "tick": {
    "id": メッセージid,
    "ts": 最新約定時間,
    "data": [
      {
        "id": 約定id,
        "price": 約定価格,
        "amount": 約定量,
        "direction": 売り:sellもしくは買い:buy方向の指定
        "ts": 約定時間
      }
    ]
  }
```

リクエスト応答例:

```json
/* GET /market/trade?symbol=ethusdt */
{
  "status": "ok",
  "ch": "market.btcusdt.trade.detail",
  "ts": 1489473346905,
  "tick": {
    "id": 600848670,
    "ts": 1489464451000,
    "data": [
      {
        "id": 600848670,
        "price": 7962.62,
        "amount": 0.0122,
        "direction": "buy",
        "ts": 1489464451000
      }
    ]
  }
}

/* GET /market/trade?symbol=not-exist */
{
  "ts": 1490759506429,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```
#### GET /market/history/trade 最近の取引履歴をまとめて取得する

リクエストパラメーター:

| パラメーター名称   | 必須かどうか  | タイプ   | 説明   | デフォルト値   | 値の範囲    |
| ------- | ----- | ------ | ---- | ----- | ---  |
| symbol   | true  | string | 取引通貨ペア   |       | btcusdt, bchbtc, rcneth ... |
| size  | false  |integer| トレード数を取得する    |  1   | [1, 2000]    |

応答データ:

| パラメーター名称   | 必須かどうか  | タイプ  | 説明  | デフォルト値  | 値の範囲   |
| -------- | ----- | ------ | --------  | ----- | ----  |
| status   | true  | string |    |    | ok, error   |
| ch     | true | string | データの所属 channel，フォマット： market.$symbol.trade.detail  |    |
| ts    | true  |integer| 発信時間  |    |      |
| data  | true  | object | 約定記録    |    |    |

data 説明：

```
  "data": {
    "id": メッセージid,
    "ts": 最新約定時間,
    "data": [
      {
        "id": 約定id,
        "price": 約定価格,
        "amount": 約定量,
        "direction": アクティブな約定方向,
        "ts": 約定时间
      }
    ]
  }
```

リクエスト応答例:

```
/* GET /market/history/trade?symbol=ethusdt */
{
    "status": "ok",
    "ch": "market.ethusdt.trade.detail",
    "ts": 1502448925216,
    "data": [
        {
            "id": 31459998,
            "ts": 1502448920106,
            "data": [
                {
                    "id": 17592256642623,
                    "amount": 0.04,
                    "price": 1997,
                    "direction": "buy",
                    "ts": 1502448920106
                }
            ]
        }
    ]
}
```

### パブリックAPI


####  GET /v1/common/symbols サポートされている全ての取引ペアと精度を照会する

 リクエストパラメーター:
(無し)

応答データ:

| パラメーター名称    | 必須かどうか | データタイプ   | 説明    | 値の範囲 |
| -------------- | ---- | ------ | ----- | ---- |
| base-currency  | true | string |主軸通貨（仮想通貨）  |      |
| quote-currency | true | string |決済通貨  |      |
| price-precision | true | string | 価格桁数（0は1桁） |      |
| amount-precision | true | string | 数量桁数（0は1桁）|      |
| symbol-partition | true | string | トレードエリア | mainエリア，innovationイノベーションゾーン（マイナーアルトコインエリア），bifurcation分岐エリア（ハードフォークエリア）  |

 リクエスト応答例:

```
/* GET /v1/common/symbols */
{
  "status": "ok",
  "data": [
    {
      "base-currency": "eth",
      "quote-currency": "usdt",
      "symbol": "ethusdt"
    }
    {
      "base-currency": "etc",
      "quote-currency": "usdt",
      "symbol": "etcusdt"
    }
  ]
}
```

####  GET /v1/common/currencys サポートされているすべての銘柄を照会

 リクエストパラメーター:

(無し)

 応答データー:

```
currency list
```

リクエスト応答例:

```json
/* GET /v1/common/currencys */
{
  "status": "ok",
  "data": [
    "usdt",
    "eth",
    "etc"
  ]
}
```

####  GET /v1/common/timestamp システムの現在の時刻を照会する

リクエストパラメータ:

(無し)

 応答データー:

```
システムタイムスタンプ
```

リクエスト応答例

```
/* GET /v1/common/timestamp */
{
  "status": "ok",
  "data": 1494900087029
}
```

### ユーザーアセットAPI

####  GET /v1/account/accounts 

リクエストパラメーター:

無し

応答データ:

| パラメーター名称  | 必須かどうか | データタイプ | 説明 | 値の範囲 |
| ----- | ---- | ------ | ----- | ----  |
| id    | true | long   | account-id |    |
| state | true | string | アカウントステータス  | working：正常, lock：アカウントがロック状態 |
| type  | true | string | アカウントタイプ  | spot：現物用アカウント    |

リクエスト応答例:

```json
/* GET /v1/account/accounts */
{
  "status": "ok",
  "data": [
    {
      "id": 100009,
      "type": "spot",
      "state": "working",
      "user-id": 1000
    }
  ]
}
```

####  GET /v1/account/accounts/{account-id}/balance 指定されたアカウントの残高を照会する

リクエストパラメーター

| パラメーター名称   | 必須かどうか | タイプ  | 説明   | デフォルト値  | 値の範囲 |
| ---------- | ---- | ------ | --------------- | ---- | ---- |
| account-id | true | string | account-id，を path に記入， GET使用可能 /v1/account/accounts 獲得 |      |      |

* もし自分のアカウントを知らない場合， ```GET /v1/account/accounts``` 照会を使って下さい

応答データー:

| パラメーター名称  | 必須かどうか  | データータイプ   | 説明    | 値の範囲   |
| ----- | ----- | ------ | ----- | ----- |
| id    | true  | long   | アカウント ID |      |
| state | true  | string | アカウントステータス  | working：正常  lock：アカウントロック |
| type  | true  | string | アカウントタイプ  | spot：現物アカウント              |
| list  | false | Array  | 子アカウント配列 |     |

List欄の説明

| パラメーター名称   | 必須かどうか | データータイプ   | 説明   | 値の範囲   |
| -------- | ---- | ------ | ---- |  ------ |
| balance  | true | string | 残高   |    |
| currency | true | string | 銘柄（仮想通貨）   |    |
| type     | true | string | タイプ  | trade: トレード残高，frozen: 凍結残高 |

リクエスト応答例:

```json
/* GET /v1/account/accounts/'account-id'/balance */
{
  "status": "ok",
  "data": {
    "id": 100009,
    "type": "spot",
    "state": "working",
    "list": [
      {
        "currency": "usdt",
        "type": "trade",
        "balance": "500009195917.4362872650"
      },
      {
        "currency": "usdt",
        "type": "frozen",
        "balance": "328048.1199920000"
      },
     {
        "currency": "etc",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "etc",
        "type": "frozen",
        "balance": "9786.6783000000"
      }
     {
        "currency": "eth",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "eth",
        "type": "frozen",
        "balance": "9786.6783000000"
      }
    ],
    "user-id": 1000
  }
}
```

## トレードAPI

#### POST /v1/order/orders/place 注文

#### リクエストパラメーター

| パラメーター名称   | 必須かどうか  | タイプ     | 説明    | デフォルト値  | 値の範囲    |
| ----- | ----- | ------ | --------  | ---- | -------  |
| account-id | true  | string | アカウント ID，accountsを使用して獲得する方法。仮想通貨ペアのトレードを使用‘spot’アカウントのaccountid |      |     |
| amount     | true  | string | 指値の表示注文数量，成り行きで買う時は幾らで買うかを表示，成り行きで売る時に何枚売るかを表示 |   |   |
| price      | false | string | 注文価格，成り行き注文はこのパラメーターを通さない   |      |       |
| source     | false | string | 注文のソース    | api |    |
| symbol     | true  | string | トレードペア    |      | btcusdt, bchbtc, rcneth ...   |
| type       | true  | string | 注文タイプ   |    | buy-market：成り行き買い, sell-market：成り行き売り, buy-limit：指値買い, sell-limit：指値売り, buy-ioc：IOC買い注文, sell-ioc：IOC売り注文, buy-limit-maker, sell-limit-maker(詳細の説明は以下)|

**buy-limit-maker**　

“注文価格”>=“市場最低売り価格”である場合，注文送信後，システムはこの注文を約定拒否します。

“注文価格”<“市場最低売り価格”である場合，送信成功後，この注文はシステムによって受け入れられます。

**sell-limit-maker**

“注文価格”<=“市場最高買い入れ価格” である場合，注文送信後，システムはこの注文を受け入れることを拒否します。

“注文価格”>“市場最高買い入れ価格” である場合， 送信成功後，この注文はシステムによって受け入れられます。


#### 応答データー:

| パラメーター名称 | 必須かどうか | データータイプ | 説明   | 値の範囲 |
| ---- | ---- | ---- | ---- | ---- |
| data | false | string | 订单ID  |      |

#### リクエスト応答例:

```json
/* POST /v1/order/orders/place */
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
{
  "status": "ok",
  "data": "59378"
}
```

####  GET /v1/order/openOrders 現時点、アカウントで未約定の注文をすべて取得する

####  リクエストパラメーター: 

`“account-id” と “symbol” 同時に二つとも指定する必要があるかないか。もし二つとも指定しないのであれば，注文番号で降順でソートされた未約定注文を最大500個返します。 `

| パラメーター名称     | 必須かどうか | タイプ    | 説明           | デフォルト値  | 値の範囲 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true | string | アカウントID |      |      |
| symbol | true | string | トレードペア |      |   単一トレードのペア文字列，デフォルトでは、条件に満たされていない全ての注文が返されます。   |
| side | false | string | 取引方向 |      |   “buy”か“sell”， デフォルトでは、条件に満たされていない全ての注文が返されます。   |
| size | false | int | 必要な記録数 |   10   |    [0,500]  |

####  応答データ: 

| パラメーター名称 | 必須かどうか | データタイプ   | 説明    | 値の範囲 |
| ---- | ---- | ------ | ----- | ---- |
| id | true | long | 注文番号 |  |
| symbol| true | string | トレードペア |     |
| price | true | string | 注文価格 |    |
| created-at | true | int | 注文時間（ミリ秒） |  Unixタイムスタンプ  |
| type | true | string | 注文タイプ |  buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc  |
| filled-amount | true | string | 注文時間（ミリ秒） |  部分取引注文でない場合、このフィールドは0 になります |
| filled-cash-amount | true | string | 約定された部品の注文価格（=約定された注文の数量x注文の価格） | 部分取引注文でない場合、このフィールドは0 になります|
| filled-fees | true | string | 取引の一部における手数料 | 部分取引注文でない場合、このフィールドは0 になります
| source | true | string | 注文ソース |  sys, web, api, app  |
| state | true | string | この注文ステータス |  submitted（約定済）, partial-filled（部分約定）, cancelling（キャンセル）  |

####  応答例:

```json
/* GET /v1/orders/openOrders */
{
  "status": "ok",
  "data": [
    {
      "id": 5454937,
      "symbol": "ethusdt",
      "account-id": 30925,
      "amount": "1.000000000000000000",
      "price": "0.453000000000000000",
      "created-at": 1530604762277,
      "type": "sell-limit",
      "filled-amount": "0.0",
      "filled-cash-amount": "0.0",
      "filled-fees": "0.0",
      "source": "web",
      "state": "submitted"
    }
  ]
}
```

####  POST /v1/order/orders/{order-id}/submitcancel  注文リクエストのキャンセルリクエスト

リクエストパラメーター: 

| リクエスト名称     | 必須かどうか | タイプ     | 説明           | デフォルト値  | 値の値 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| order-id | true | string | 注文ID，pathの中に記入|      |      |

応答データー: 

| パラメーター名称 | 必須かどうか | データタイプ   | 説明    | 値の範囲 |
| ---- | ---- | ------ | ----- | ---- |
| data | true | string | 注文 ID |      |

リクエスト応答例:

```
/* POST /v1/order/orders/{order-id}/submitcancel */
{
  "status": "ok",//注意，OKであれば、出金依頼が成功したことを示します。注文が正常に取り消された場合は、注文状況照会インターフェースを呼び出して注文状況を照会してください。  "data": "59378"
}
```

#### POST /v1/order/orders/batchcancel 一括注文の取り消し

リクエストパラメーター:

| パラメーター名称  | 必須かどうか | タイプ  | 説明  | デフォルト値  | 値の範囲 |
| ---- | ---- | ---- | ----  | ---- | ---- |
| order-ids | true | list | キャンセル注文IDリスト |  | 1回の注文ID数は50を超えない|

応答データー:

| パラメーター名称 | 必須かどうか | データタイプ | 説明    | 値の範囲 |
| ---- | ----- | ---- | ----- | ---- |
| data | false | map | キャンセル結果 |      |

リクエスト応答例:

```json
/* POST /v1/order/orders/batchcancel */
{
  "order-ids": [
    "1", "2", "3"
  ]
}
```
---
```json
{
  "status": "ok",
  "data": {
    "success": [
      "1",
      "3"
    ],
    "failed": [
      {
        "err-msg": " Invalid record",
        "order-id": "2",
        "err-code": "base-record-invalid"
      }
    ]
  }
}
```

####  POST  /v1/order/orders/batchCancelOpenOrders  対象となる注文を一括でキャンセルする

リクエストパラメーター: 

| パラメーター名称     | 必須かどうか | タイプ     | 説明           | デフォルト値  | 値の範囲 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true  | string | アカウントID     |     |      |
| symbol     | false | string | 取引通貨ペア     |      |   単一トレードペアの文字列は、デフォルトでは条件が満たされていない全ての注文を返します  |
| side | false | string | 取引方向 |      |   “buy”か“sell”， デフォルトでは、条件が満たされていない全ての注文が返されます。   |
| size | false | int | 必要な記録数  |  100 |   [0,100]   |

応答データ: 

| パラメーター名称 | 必須かどうか | データータイプ   | 説明    | 値の範囲 |
| ---- | ---- | ------ | ----- | ---- |
| success-count | true | int | キャンセル注文成功数 |     |
| failed-count | true | int | キャンセル注文失敗数　 |     |
| next-id | true | long | キャンセル基準を満たす次の注文番号 |    |

####  応答例:

```json
/* POST /v1/order/orders/batchCancelOpenOrders */
{
  "status": "ok",
  "data": {
    "success-count": 2,
    "failed-count": 0,
    "next-id": 5454600
  }
}
```

####  GET /v1/order/orders/{order-id} 注文の詳細を照会

リクエストパラメーター: 

| パラメーター名称     | 必須かどうか | タイプ  | 説明   | デフォルト値  | 値の範囲 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | パスに記載された注文ID |      |      |

応答データー: 

| パラメーター名称     | 必須かどうか  | データータイプ   | 説明   | 値の範囲     |
| ----------------- | ----- | ------ | -------  | ----  |
| account-id        | true  | long   | アカウント ID    |       |
| amount            | true  | string | 注文数量              |    |
| canceled-at       | false | long   | 注文キャンセル時間    |     |
| created-at        | true  | long   | 注文作成時間    |   |
| field-amount      | true  | string | 約定数量    |     |
| field-cash-amount | true  | string | 約定総額     |      |
| field-fees        | true  | string | 約定手数料（買い入れは仮想通貨のために，売り出しは法定通貨のために） |     |
| finished-at       | false | long   | 注文の終了時間は約定した時間ではなく、キャンセル済の時も含みます   |     |
| id                | true  | long   | 注文ID    |     |
| price             | true  | string | 注文価格       |     |
| source            | true  | string | 注文ソース   | api |
| state             | true  | string | 注文ステータス   | submitting , submitted 送信済, partial-filled 部分約定, partial-canceled 部分約定キャンセル, filled 完全約定, canceled キャンセル済み |
| symbol            | true  | string | トレードペア   | btcusdt, bchbtc, rcneth ... |
| type              | true  | string | 注文種類   | buy-market：成り行き買い, sell-market：成り行き売り, buy-limit：指値買い, sell-limit：指値売り, buy-ioc：IOC買い注文, sell-ioc：IOC売り注文 |


リクエスト応答例:

```json
/* GET /v1/order/orders/{order-id} */
{
  "status": "ok",
  "data": {
    "id": 59378,
    "symbol": "ethusdt",
    "account-id": 100009,
    "amount": "10.1000000000",
    "price": "100.1000000000",
    "created-at": 1494901162595,
    "type": "buy-limit",
    "field-amount": "10.1000000000",
    "field-cash-amount": "1011.0100000000",
    "field-fees": "0.0202000000",
    "finished-at": 1494901400468,
    "user-id": 1000,
    "source": "api",
    "state": "filled",
    "canceled-at": 0,
    "exchange": "xxx",
    "batch": ""
  }
}
```

####  GET /v1/order/orders/{order-id}/matchresults  注文の約定詳細を照会する

リクエストパラメーター:

| パラメーター名称  | 必須かどうか | タイプ  | 説明  | デフォルト値  | 値の範囲 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | 注文ID，path中に記入 |      |      |

応答データー:

| パラメーター名称    | 必須かどうか | データータイプ   | 説明   | 値の範囲     |
| ------------- | ---- | ------ | -------- | -------- |
| created-at    | true | long   | 約定時間     |    |
| filled-amount | true | string | 約定数量     |    |
| filled-fees   | true | string | 約定手数料    |     |
| id            | true | long   | 約定注文記録ID |     |
| match-id      | true | long   | マッチングID     |     |
| order-id      | true | long   | 注文 ID    |      |
| price         | true | string | 約定価格  |    |
| source        | true | string | 注文ソース  | api      |
| symbol        | true | string | トレードペア   | btcusdt, bchbtc, rcneth ...  |
| type          | true | string | 注文タイプ   | buy-market：成り行き買い, sell-market：成り行き売り, buy-limit：指値買い, sell-limit：成り行き売り, buy-ioc：IOC買い注文, sell-ioc：IOC売り注文 |

リクエスト応答例:

```json
/* GET /v1/order/orders/{order-id}/matchresults */
{
  "status": "ok",
  "data": [
    {
      "id": 29553,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "9.1155000000",
      "filled-fees": "0.0182310000",
      "created-at": 1494901400435
    }
  ]
}
```

####  GET /v1/order/orders 注文の約定詳細を照会する

リクエストパラメーター:

| パラメーター名称   | 必須かどうか  | タイプ    | 説明  | デフォルト値  | 値の範囲   |
| ---------- | ----- | ------ | ------  | ---- | ----  |
| symbol     | true  | string | トレードペア      |      |btcusdt, bchbtc, rcneth ...  |
| types      | false | string | オーダーのタイプの組み合わせ照会，使用','分割  |      | buy-market：成り行き買い, sell-market：成り行き売り, buy-limit：指値買い, sell-limit：指値売り, buy-ioc：IOC買い注文, sell-ioc：IOC売り注文 |
| start-date | false | string | 開始日時照会, 日時フォマットyyyy-mm-dd |      |      |
| end-date   | false | string | 終了日時照会, 日時フォマットyyyy-mm-dd |      |    |
| states     | true  | string | オーダーのタイプの組み合わせ照会，使用','分割  |      | submitted 提出済み, partial-filled 部分約定, partial-canceled 部分約定キャンセル, filled 完全約定, canceled キャンセル済み |
| from       | false | string | 開始 ID照会   |      |    |
| direct     | false | string | 方向照会   |      | prev 前，next 後ろ    |
| size       | false | string | 大小記録の照会      |      |         |

応答データー: 

| パラメーター名称    | 必須かどうか  | データータイプ  | 説明   | 値の範囲   |
| ----------------- | ----- | ------ | ----------------- | ----  |
| account-id        | true  | long   | アカウント ID    |     |
| amount            | true  | string | 注文数量    |   |
| canceled-at       | false | long   | キャンセル申請を受ける時間   |    |
| created-at        | true  | long   | 注文作成時間  |    |
| field-amount      | true  | string | 約定数量   |    |
| field-cash-amount | true  | string | 約定総金額    |    |
| field-fees        | true  | string | 約定済み手数料（買いは仮想通貨の為に，売りはお金のために） |       |
| finished-at       | false | long   | 最終的な約定時間    |   |
| id                | true  | long   | 注文ID    |    |
| price             | true  | string | 注文価格  |    |
| source            | true  | string | 注文ソース   | api  |
| state             | true  | string | 注文ステータス    | submitting , submitted 提出済, partial-filled 部分約定, partial-canceled 部分約定キャンセル, filled 完全約定, canceled キャンセル済 |
| symbol            | true  | string | トレードペア    | btcusdt, bchbtc, rcneth ... |
| type              | true  | string | 注文タイプ  | submit-cancel：キャンセル申請済  ,buy-market：成り行き買い, sell-market：成り行き売り, buy-limit：指値買い, sell-limit：指値売り, buy-ioc：IOC買い注文, sell-ioc：IOC売り注文 |


リクエスト応答例:

```json
/* GET /v1/order/orders */
{
  "status": "ok",
  "data": [
    {
      "id": 59378,
      "symbol": "ethusdt",
      "account-id": 100009,
      "amount": "10.1000000000",
      "price": "100.1000000000",
      "created-at": 1494901162595,
      "type": "buy-limit",
      "field-amount": "10.1000000000",
      "field-cash-amount": "1011.0100000000",
      "field-fees": "0.0202000000",
      "finished-at": 1494901400468,
      "user-id": 1000,
      "source": "api",
      "state": "filled",
      "canceled-at": 0,
      "exchange": "xxx",
      "batch": ""
    }
  ]
}
```

####  GET /v1/order/matchresults 現在の約定、約定履歴の照会

リクエストパラメーター:

| パラメーター名称   | 必須かどうか  | タイプ  | 説明   | デフォルト値  | 値の範囲    |
| ---------- | ----- | ------ | ------ | ---- | ----------- |
| symbol     | true  | string | トレードペア   | btcusdt, bchbtc, rcneth ... |    |
| types      | false | string | オーダータイプの組み合わせ照会，使用','分割   |      | buy-market：成り行き買い, sell-market：成り行き売り, buy-limit：指値買い, sell-limit：指値売り, buy-ioc：IOC買い注文, sell-ioc：IOC売り注文 |
| start-date | false | string | 開始日の照会, 日時フォマットyyyy-mm-dd | -61 days     | [-61day, now] |
| end-date   | false | string | 終了日の照会, 日時フォマットyyyy-mm-dd |   Now   |  [start-date, now]  |
| from       | false | string | 開始日の照会 ID    |   注文約定記録ID（最大值）   |     |
| direct     | false | string | 照会方向    |   デフォルトnext， 約定記録ID大から小へ並ぶ   | prev 前に，next 後ろに   |
| size       | false | string | 大小記録の照会    |   100   | <=100  |

応答データー: 

| パラメーター名称   | 必須かどうか | データータイプ  |説明   |値の範囲   |
| ------------- | ---- | ------ | -------- | ------- |
| created-at    | true | long   | 約定時間     |    |
| filled-amount | true | string | 約定数量     |    |
| filled-fees   | true | string | 約定手数料    |    |
| id            | true | long   | 約定注文記録ID |    |
| match-id      | true | long   | マッチングID     |    |
| order-id      | true | long   | 注文 ID    |    |
| price         | true | string | 約定価格     |    |
| source        | true | string | 注文ソース     | api   |
| symbol        | true | string | 取引通貨ペア      | btcusdt, bchbtc, rcneth ...  |
| type          | true | string | 注文タイプ     | buy-market：成り行き買い, sell-market：成り行き売り, buy-limit：指値買い, sell-limit：指値売り, buy-ioc：IOC買い注文, sell-ioc：IOC売り注文 |

リクエスト応答例:

```json
/* GET /v1/orders/matchresults */
{
  "status": "ok",
  "data": [
    {
      "id": 29555,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "0.9845000000",
      "filled-fees": "0.0019690000",
      "created-at": 1494901400487
    }
  ]
}
```


## 仮想通貨の出金API

> **引出しのみサポート【Pro出金アドレスリストの中の出金アドレス】**


####  POST /v1/dw/withdraw/api/create 仮想通貨の出金申請

リクエストパラメーター:

| パラメーター名称       | 必須かどうか | タイプ     | 説明    | デフォルト値  | 値の範囲 |
| ---------- | ---- | ------ | ------ | ---- | ---- |
| address | true | string   | 出金アドレス |      |      |
| amount     | true | string | 出金数量   |      |      |
| currency | true | string | 通貨種別   |   |  btc, ltc, bch, eth, etc ...(サポート銘柄) |
| fee     | false | string | 送金手数料  |      |      |
| addr-tag|false | string | 仮想通貨アドレスの共有tag，に適しているのは xrp，xem，bts，steem，eos，xmr |  | フォーマット, "123"類の整数|

応答データ: 

| パラメーター名称 | 必須かどうか  | データータイプ | 説明  | 値の範囲 |
| ---- | ----- | ---- | ---- | ---- |
| data | false | long | 出金ID |      |

リクエスト応答例:

```
/* POST /v1/dw/withdraw/api/create*/
{
  "address": "0xde709f2102306220921060314715629080e2fb77",
  "amount": "0.05",
  "currency": "eth",
  "fee": "0.01"
}
{
  "status": "ok",
  "data": 700
}
```

####  POST /v1/dw/withdraw-virtual/{withdraw-id}/cancel 仮想通貨の出金をキャンセル

リクエストパラメーター:

| パラメーター名称        | 必須かどうか | タイプ   | 説明 | デフォルト値  | 値の範囲 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| withdraw-id | true | long | 出金ID，pathの中に記入 |      |      |

応答データー:

| パラメーター名称 | 必須かどうか  | データータイプ | 説明    | 値の範囲 |
| ---- | ----- | ---- | ----- | ---- |
| data | false | long | 提现 ID |      |

リクエスト応答例:

```
/* POST /v1/dw/withdraw-virtual/{withdraw-id}/cancel */
{
  "status": "ok",
  "data": 700
}
```

####  GET /v1/query/deposit-withdraw 仮想通貨の入出金記録

リクエスパラメータ:

| パラメーター名称        | 必須かどうか | タイプ   | 説明 | デフォルト値  | 値の範囲 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| currency | true | string | 銘柄  |  |  |
| type | true | string | 'deposit' or 'withdraw'  |     |    |
| from   | false | string | 照会開始 ID  |    |     |
| size   | false | string | 大小記録の照会  |    |     |

応答データー:

| パラメーター名称 | 必須かどうか | データータイプ | 説明 | 値の範囲|
|-----|-----|-----|-----|------|
|   id  |  true  |  long  |   | |
|   type  |  true  |  long  | タイプ | 'deposit' 'withdraw' |
|   currency  |  true  |  string  |  銘柄 | |
| tx-hash | true |string | トレードハッシュ | |
| amount | true | long | 個数 | |
| address | true | string | アドレス | |
| address-tag | true | string | アドレスラベル | |
| fee | true | long | 手数料 | |
| state | true | string | ステータス | ステータスは下の表を参考に |
| created-at | true | long | 開始時間 | |
| updated-at | true | long | 最後に更新した時間 | |

###### 仮想通貨出金ステータスの定義：

| ステータス | 説明  |
|--|--|
| submitted | 提出済 |
| reexamine | 審査中 |
| canceled  | キャンセル済 |
| pass    | 承認 |
| reject  | 拒否 |
| pre-transfer | 处理中 |
| wallet-transfer | 送金済 |
| wallet-reject   | ウオレットの拒否 |
| confirmed      | ブロックチェーン上で承認済 |
| confirm-error  | ブロックチェーン上で承認エラー |
| repealed       | キャンセル済 |

###### 仮想通貨入金ステータスの定義：

|ステータス|説明|
|--|--|
|unknown|不明|
|confirming|確認中|
|confirmed|確認済み|
|safe|完了|
|orphan| |

リクエスト応答例:

```
/* GET /v1/query/deposit-withdraw?currency=xrp&type=deposit&from=5&size=12 */

{
    
    "status": "ok",
    "data": [
      {
        "id": 1171,
        "type": "deposit",
        "currency": "xrp",
        "tx-hash": "ed03094b84eafbe4bc16e7ef766ee959885ee5bcb265872baaa9c64e1cf86c2b",
        "amount": 7.457467,
        "address": "rae93V8d2mdoUQHwBDBdM4NHCMehRJAsbm",
        "address-tag": "100040",
        "fee": 0,
        "state": "safe",
        "created-at": 1510912472199,
        "updated-at": 1511145876575
      },
     ...
    ]
}
```



# エラーコード

## チャート API エラーコード

| エラーコード |  説明 |
|-----|-----|
| bad-request | リクエストエラー |
| invalid-parameter | パラメーターエラー |
| invalid-command | 命令エラー|
code の具体的な詳細については `err-msg`にて対応しているもの参照ください.

## トレード API エラーコード

| エラーコード  |  説明 |
|-----|-----|
| base-symbol-error |  トレードペアが存在していない |
| base-currency-error |  銘柄が存在していない |
| base-date-error | 日時フォマットのエラー |
| account-transfer-balance-insufficient-error | 残高不足により凍結できません |
| bad-argument | パラメーターの期限切れ |
| api-signature-not-valid | API署名エラー |
| gateway-internal-error | システムビジー，少し時間をあけて再度お試しください|
|security-require-assets-password|資金パスワードの入力が必要|
|audit-failed| 注文失敗|
|ad-ethereum-addresss| 有効なETHのアドレスを入力してください|
|order-accountbalance-error| アカウント残高不足|
| order-limitorder-price-error|指値の注文価格が制限を超えている |
|order-limitorder-amount-error|指値の注文量が制限を超えている |
|order-orderprice-precision-error|注文価格が制限精度を超えている |
|order-orderamount-precision-error|注文量が制限精度を超えている|
|order-marketorder-amount-error|注文量が制限を超えている|
|order-queryorder-invalid|この注文を見つけることができない |
|order-orderstate-error|注文ステータスエラー|
|order-datelimit-error|照会時間制限を超えた|
|order-update-error|注文の更新失敗|

