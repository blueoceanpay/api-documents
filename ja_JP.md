# BlueOcean Pay API ドキュメント

## 1 API ルール

### 1.1 API プロトコル

#### BlueOcean Payインターフェイスの呼び出しには、次の規則が適用されます。

1. リクエストメソッドには常にPOSTを使用し、リクエストされたデータパラメータのJSON文字列をhttpボディとして渡します

2. 要求されたデータ形式はJSON形式を使用します。 このパラメータにjson文字列がある場合は、二重引用符（\ "）を使用します。

3. 文字列エンコーディングはUTF-8を使用してください

4. 署名アルゴリズムMD5

### 1.2 パラメータ署名

1.2.1 要求パラメータを以下のように仮定します。

```
{
     "appid"： "1000010"、
     "name"： "BlueOcean Pay"、
     "region"： "HK"、
    "business":"Online payment"
}
```

1.2.2 パラメータをキーと値のペア（key = value）の形式で配置し、パラメータ名ASCIIの辞書順に従ってソートし、＆を使用して接続します

```
str = "appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK"
```

1.2.3 キーストリングの最終ステッチ `&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb`

```
str += "&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb"
```

つまり、str は:

```
appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb
```

   
1.2.4 最後にMD5値を計算し、md5文字列を大文字に変換します

```
sign = strtoupper(md5(str))
```

Signed value:

```
08C612FB4D2D52C8C913EA00E3DABC8B
```

### 1.3 ヘッダーを要求する

APIリクエストのトラッキングとカウントを有効にするには
HTTPヘッダーにユーザーエージェントを設定する必要がある

#### ユーザエージェントの例

```
# 英語
BoPayPos/1.1.0 NetType/WIFI Language/en_US

# 簡体字中国語
BoPayPos/1.1.0 NetType/WIFI Language/zh_CN

# 繁体字中国語
BoPayPos/1.1.0 NetType/WIFI Language/zh_HK

```
  
### 1.4 PHPシグネチャのサンプルコード:


```
<?php
/**
 * @param array $data
 * @param string $key
 * @return string
 */
function signData($data,$key){	
	$ignoreKeys = ['sign', 'key'];
    ksort($data);
    $signString = '';
    foreach ($data as $k => $v) {
        if (in_array($k, $ignoreKeys)) {
            unset($data[$k]);
            continue;
        }
        $signString .= "{$k}={$v}&";
    }
    $signString .= "key={$key}";
	return strtoupper(md5($signString));
}
```


### 1.5 APIエンドポイント

異なるビジネスエリアで異なるドメイン名を使用する

```
HongKong
https://api.hk.blueoceanpay.com

Australia
https://api.au.blueoceanpay.com

United States
https://api.us.blueoceanpay.com

United Kingdom
https://api.uk.blueoceanpay.com

```

### 1.6 応答エラーコード

戻りコード|メッセージ|理由|ソリューション
--------|---------|----|----------
200 |成功|正常|特定のビジネスロジックに従って
201 |カードスワイプの結果が不明な状態|システムがビジーで、中程度の理由でユーザーが支払いを行う| [クエリオーダーAPI]クエリで15秒待つ
404 |このインタフェースを見つけることができません|不正なインタフェースアドレス|ドキュメントに基づいてインタフェースアドレスを確認してください
4001 |データ形式のエラー|データ形式の不一致| JSONデータ形式を使用してください
4002 |無効なAPPID | APPIDは存在しません| APPIDが間違っているかどうかを確認してください
4003 |パラメータの不足|パラメータの不足|プロンプトに完全なパラメータ
4004 |署名エラー|署名検証エラー|コード署名ロジックのチェック
4005 |通信エラー| WeChatサーバーエラー|コードのトラブルシューティング後のトラブルシューティングコード、Blue Oceanテクニカルスタッフへのフィードバックの送信
4006 |サーバービジー|サーバー使用中|後でこのインターフェースを呼び出す
4007 |リクエストメソッドエラー|リクエストメソッドエラー| POSTメソッドを使用してください
40100 |プロンプト処理に基づく無効な要求||
40101 |注文が支払われました||特定のビジネスロジック
40102 |注文は存在しません| |特定のビジネスロジックに従って処理されました
40103 |注文は終了しました||特定のビジネスロジック
40104 |注文は取り消されました||特定のビジネスロジックを扱う
40105 |特定のビジネスロジックに従って払い戻された注文||
40106 |注文番号の重複||商人による注文番号の再生成
40107 |残高不足||商人口座の残高が不足していないか確認してください
40108 |注文は払い戻し期間を超過しました||払い戻しを完了できません
40109 |パラメータの不足||プロンプトに従ってパラメータを完成する
40110 |不正なエンコーディング形式|| UTF8エンコーディング形式を使用してください
40111 |各QRコードは一度しか使用できません||ユーザー支払いの再取得QRコード
40400 |システムがビジー状態です|システムがビジー状態です|システムがビジー状態で、後で再試行してください
40500 | Wechat Error || Blue Oceanテクニシャンにフィードバックして、問題解決に役立てる
40600 |不明なエラー|（エラーには含まれていません）|問題を解決するためのBlue Ocean技術者のフィードバック


## 2 インタフェースリスト

### 2.1 お支払い

Pos機械は、対応する要求パラメータを提出し、使用シナリオに従って対応する支払サービスを完了する。

### Api：

```
/payment/pay
```
### リクエストパラメータ

パラメータ名|フィールド|必須|データ型|説明
---- | ---- | ---- | ---- | ----
Appid | appid | はい |文字列| appid、管理の背景から取得する、またはログインする
署名|sign|はい|文字列
支払タイプ| payment |はい|文字列| micropay、alipay.qrcode、wechat.qrcode
注文金額| total_fee |はい| int |支払い金額単位 "Cent"
割引| discount |いいえ| Int |割引単位 "Cent"のデフォルト値は "0"です。
認証コード| code |いいえ|文字列|支払いが「マイクロペイ」、つまりアリペイの場合：288271620985824610 134519771100657507
商人の注文番号| out\_trade\_no |いいえ|文字列|加盟店に独自の注文システムがある場合、独自の注文番号を生成することができます。そうでない場合、BlueOcean Payによって自動的に生成されることが推奨されます。

#### パラメータ説明


#### 支払いパラメータの説明


パラメータ値|説明
----------|----------
micropay |支払いカードの支払いこの時点で、支払い認証コードの `code`パラメータを渡す必要があります
alipay.qrcode | Alipay QRコード
wechat.qrcode | Wechat QRコード

支払が返された後、トランザクションステータスtrade_stateを確認し、結果に基づいて、結果クエリ処理のために注文照会インタフェースを呼び出すかどうかを決定します。

### 注文支払いステータスtrade_state説明

```
NOTPAY：未払い
SUCCESS：支払いが成功しました
REFUND：払い戻し済み
CLOSED：クローズド
REVOKED：取り消されました
USERPAYING：支払い中
PAYERROR：支払い例外
```

### 正しい応答データの説明

レスポンスレスポンスresponse.dataデータの説明

パラメータ名|フィールド|データ型|説明
--------------|-----|---------|------------
Id | id | Int |たとえば10357
SN |sn |文字列|例えば11201802091347484054542598一意識別子、クエリのために使用することができ、閉じ、払い戻しなどblueoceanがシリアル番号を払います
通貨| fee_type |文字列|通貨（例：HKD、AUD、USD）
商人名| mch_name |文字列|たとえば "BlueOcean Pay"
文字列| out\_trade\_no| out\_trade\_noアウト|商人の注文番号マーチャントの注文番号例えば「11201802091347484054542598」
トランザクションID | transaction_id |文字列| P5631VZG299QZN94JD
支払いプロバイダ| provider |文字列|たとえば、alipay、wechat
作成時間| create_time | Int |タイムスタンプなど1518155270
支払い時間| time_end | Int |支払いの成功のタイムスタンプなど1518155297
貿易状態| trade_state |文字列| NOTPAY、PAID
QRコードのテキスト|qrcode|文字列| QRコード例えば "https://qr.alipay.com/bax03112k12liy7lrysg2004"、 "Weixin：// wxpay / bizpayurl PR = HBXdDeM"
支払額| total_fee | Int |単位 "Cent"、例えば10
割引| discount | Int |単位 "Cent"、例えば 2
署名|sign|文字列|たとえば "7FB42F08C85670A86431F97109DE8683"


###リクエストの例

#### Alipay QRコード例：

リクエストパラメータ：

```
{
  "appid": "1000322",
  "payment": "alipay.qrcode",
  "total_fee": 10,
  "wallet": "CN",
  "sign": "520489313B46B5D403CCD8A01267B6C7"
}
```

応答結果：

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": "1000258",
    "attach": "",
    "bank_type": "",
    "body": "",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": 1518155270,
    "detail": "",    
    "fee_type": "HKD",
    "id": "13152",
    "is_subscribe": "N",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "JnFOwTyJLm",
    "out_trade_no": "11201802091347484054542598",
    "total_fee": 10,
    "discount": 0,
    "pay_amount": 10,
    "provider": "alipay",
    "qrcode": "https://qr.alipay.com/bax03112k12liy7lrysg2004",
    "sn": "11201802091347484054542598",
    "time_end": 0,    
    "trade_state": "NOTPAY",
    "trade_type": "NATIVE",
    "transaction_id": "P5631VZG299QZN94JD",
    "sign": "7FB42F08C85670A86431F97109DE8683"
  }
}
```

####マイクロペイの例

リクエストパラメータ：

```
{
  "appid": "1000322",
  "code": "134602370743606195",
  "payment": "micropay",
  "total_fee": 5,
  "sign": "5D8883E85FB4D721A0CF53BBD0A12905"
}
``` 

応答結果：

```
{
  "code": 200,
  "message": "OK",
  "data": {
    "appid": "1000258",
    "attach": "",
    "bank_type": "",
    "body": "Shopping",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": 1517996545,
    "detail": "",
    "fee_type": "HKD",
    "id": "11763",
    "is_subscribe": "N",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "5L0CLFvTA1",
    "out_trade_no": "11201802071742242458779416",
    "provider": "wechat",
    "qrcode": "",
    "sn": "11201802071742242458779416",
    "time_end": 0,
    "total_fee": 5,
    "discount":2,
    "pay_amount":3,
    "trade_state": "NOTPAY",
    "trade_type": "MICROPAY",
    "transaction_id": "",
    "sign": "9E93F481EBD5E06081882E7CFAE2E0FF"
  }
}
```

### 2.2 払い戻し

取引が発生した後、ある期間内に買い手または売り手のために払い戻しが必要な場合、売り手は払い戻しのインターフェースを通じて支払いを払い戻すことができます。

支払いプロバイダ（Alipay、Wechatなど）は、払い戻しリクエストを受け取り、成功を確認した後、元のルートに従って払い戻しルールに従って支払いアカウントを払い戻します。

#### Api：

```
/payment/refund
```
####パラメータリクエストパラメータ

フィールド|変数名|必須|タイプ|説明
---- | ---- | ---- | ---- | ----
Appid | appid |はい|文字列| appidで、ログイン時に取得されます
署名|sign| はい ||
注文番号| sn |いいえ|文字列|と out\_trade\ _no は1つを選択し、優先的に使用します
商人注文番号| out\_trade\_no |いいえ|文字列|商人注文番号
払い戻し額| refund_fee |いいえ| Int |オプションのパラメータ、デフォルトは合計注文額です。つまり、全額払い戻しAlipayは部分払い戻しをサポートしていません
払い戻しの説明| refund_desc |いいえ|文字列|払い戻しの手順
払い戻しパスワード| password |はい|文字列|払い戻しパスワード


### 2.3 Oderの操作

類似のパラメータを使用したステータス操作の操作（クエリ、終了、元に戻す）

### Api：

```
注文照会
/order/query

オーダークローズ
/order/close

注文の逆転
/order/reverse
```
###リクエストパラメータ

パラメータ名|フィールド|必須|データ型|説明
--------------|-----|--------|---------|----------- 
Appid | appid | はい |文字列| appid、ログイン時に取得
署名|sign|はい||
BlueOceanPayシリアル番号| sn |いいえ|文字列|と出力 out\_trade\_no は1つを選択し、優先的に使用します
商人の注文番号| out\_trade\_no |いいえ|文字列|商人の注文番号

リクエストの例：

メモ

```

{
  "appid": "1000322",
  "sn": "11201802071854269363947431",
  "sign": "9CC7EB3EBE33D11421FE63298F2EEB94"
}

```

応答結果：

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": "1000258",
    "attach": "",
    "bank_type": "",
    "body": "Shopping",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": 1518000867,
    "detail": "",
    "fee_type": "HKD",
    "id": "11767",
    "is_subscribe": "N",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "Wu2vrRowHs",
    "out_trade_no": "11201802071854269363947431",
    "provider": "wechat",
    "qrcode": "weixin://wxpay/bizpayurl?pr=wDmjTFq",
    "sn": "11201802071854269363947431",
    "time_end": 0,
    "total_fee": 10,
    "discount":0,
    "pay_amount":10,
    "trade_state": "CLOSED",
    "trade_type": "NATIVE",
    "transaction_id": "",
    "sign": "C15B790CBCFE8268212D08A9F2AFA55D"
  }
}
```

### 2.4 注文リスト

注文リスト

### Api：

```
/order/list
```
### パラメータリクエストパラメータ

フィールド|変数名|必須|タイプ|説明
---- | ---- | ---- | ---- | ----
インタフェースID | appid | はい |文字列| Blue Ocean Paymentsが提供するインタフェースアプリケーションID（またはログインインタフェース経由で取得）
署名|sign| はい || |詳細については署名規則を参照
決済サービスプロバイダ| provider |いいえ|文字列|デフォルトのクエリAllipay、wechat
ページングパラメータ| page | いいえ | Int |ページングパラメータ：Pageデフォルト値：1
ページあたりのレコード数| limit | いいえ | Int | 1ページあたりのエントリ数デフォルト：10
ストアID | store_id |いいえ| Int |ストアID、login_interfaceで記入> store_id> 0
貿易状況| trade_state |いいえ|文字列|貿易状況はSUCCESS、REFUND
開始時刻| start_time | いいえ |文字列|作成時刻によるクエリの順番（例：2017-12-12）
終了時刻| end_time |いいえ|文字列|作成時刻によるクエリの順番（例：2017-12-13）


#### 結果を返す

フィールド|変数名|必須|タイプ|説明
---- | ---- | ---- | ---- | ----
戻りコード| code | はい |文字列|戻りコード、戻りコード表を参照してください
情報を返す|message|はい|文字列|情報、成功またはエラーメッセージを返す
データを返す|data|いいえ|配列|データセットを返す

#### サンプルリクエストパラメータ

```
{
    "appid": "1000322",
    "limit": 10,
    "page": 1,
    "store_id": 1000321,
    "sign": "55F40D7150AD3013E4AB479745FAE542"
}
```

### 応答結果：

```
{
  "code": 200,
  "message": "success",
  "data": {
    "page": 1,
    "limit": 10,
    "total": 43,
    "items": [
      {
        "id": "11316",
        "sn": "11201802061441100008508698",
        "provider": "alipay",
        "blue_mch_id": "1000258",
        "store_id": "1000321",
        "body": "",
        "out_trade_no": "11201802061441100008508698",
        "transaction_id": "3PG7YQKYM8J5KDXJRV",
        "total_fee": 10,
        "discount":2,
        "pay_amount":8,
        "trade_state": "NOTPAY",
        "trade_type": "QRCODE",
        "fee_type": "HKD",
        "create_time": 1517899273,
        "time_end": 0
      },
      {
        "id": "11315",
        "sn": "11201802061439532332362699",
        "provider": "wechat",
        "blue_mch_id": "1000258",
        "store_id": "1000321",
        "body": "BlueOceanPay",
        "out_trade_no": "20180206143947260296",
        "transaction_id": "4200000062201802068214812373",
        "total_fee": 2,
        "discount":0,
        "pay_amount":2,
        "trade_state": "SUCCESS",
        "trade_type": "MICROPAY",
        "fee_type": "HKD",
        "create_time": 1517899193,
        "time_end": 1517899193
      }
    ]
  }
}
```

### 2.5 払い戻しリスト

Api:

```
/refund/list
```

####リクエストパラメータ

フィールド|変数名|必須|タイプ|説明
---- | ---- | ---- | ---- | ----
インタフェースID | appid | is |文字列|
署名|sign| is || |詳細については署名規則を参照
ページングパラメータ| page | いいえ | Int |ページングパラメータ：Pageデフォルト値：1
ページあたりのレコード数| limit | いいえ | Int | 1ページあたりのエントリ数デフォルト：10
支払いプロバイダ|provider|いいえ|文字列| alipay、wechat
払い戻しのステータス| refund_status |いいえ|文字列| SUCCESSとして払い戻しステータス
開始時刻| start_time |いいえ|文字列|払い戻し時間による照会順序（例：2017-12-12）
終了時刻| end_time |いいえ|文字列|払い戻し時間に応じて注文を確認します。例：2017-12-13

#### trade_state

```
SUCCESS：払い戻しが成功しました
PROCESSING：払い戻し処理
REFUNDCLOSE：払い戻しを閉じました
CHANGE：異常払い戻し
```

#### 結果を返す

フィールド|変数名|必須|タイプ|説明
---- | ---- | ---- | ---- | ----
戻りコード|code| はい |文字列|戻りコード、戻りコード表を参照してください
情報を返す|message|はい|文字列 |情報、成功またはエラーメッセージを返す
データを返す|data|いいえ|配列|データセットやその他の情報を返す


リクエストパラメータ

```
{
  "appid": "1000322",
  "limit": 2,
  "page": 1,
  "sign": "74CD2AAB31B3FB76A68FEBF8FC56AF70"
}
```

応答結果：

```
{
  "code": 200,
  "message": "success",
  "data": {
    "page": 1,
    "limit": 2,
    "total": 88,
    "items": [
      {
        "id": "12662",
        "provider": "wechat",
        "blue_mch_id": "1000258",
        "store_id": "0",
        "out_trade_no": "a257f51afe1694b20180208164501288",
        "transaction_id": "4200000060201802089377289242",
        "out_refund_no": "a8d95666c735f412018020816452292",
        "refund_id": "50000205722018020803496119655",
        "total_fee": "2",
        "refund_fee": "2",
        "discount": "0",
        "fee_type": "HKD",
        "refund_status": "SUCCESS",
        "refund_desc": "",
        "refund_time": 1518079526,
        "pay_amount": 2
      },
      {
        "id": "12647",
        "provider": "alipay",
        "blue_mch_id": "1000258",
        "store_id": "1000321",
        "out_trade_no": "11201802081148468407623096",
        "transaction_id": "MJ86QYO39MXWOEP3RG",
        "out_refund_no": "11201802081449331411459094",
        "refund_id": "",
        "total_fee": "10",
        "refund_fee": "10",
        "discount": "0",
        "fee_type": "HKD",
        "refund_status": "SUCCESS",
        "refund_desc": "",
        "refund_time": 1518072577,
        "pay_amount": 10
      }
    ]
  }
}
```

### 2.6 ユーザーログイン

### api


```
/user/login
```

### リクエストパラメータ

パラメータ名|フィールド|必須|データ型|説明
--------------|-----|--------|---------|----------- 
メール|email|はい|文字列|ログインアカウント
パスワード|password|はい|文字列|ログインパスワード

#### POSTを使用して、要求されたデータパラメータのjson文字列をhttp bodyとして渡します

####このメソッドは署名を必要とせず、クロムブラウザのプラグインAdvanced RESTクライアントでテストすることができます

リクエストパラメータ例：

```
{"email"： "abc@blueoceanpay.com"、 "password"： "123456"}
```

アカウントは存在しません。実際に使用するには、販売者のバックエンドWebサイトから入手してください。

### レスポンスの例

#### 成功した例

```

{
   "code":200,
   "message":"Success",
   "data"：{
     "appid":100018,
     "app_key":"appkey",
     "name":"BlueOcean Pay",
     "store_id":0,
     "store_name": ""
   }
}

```

#### 失敗の例

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 100018,
    "app_key": "TVqec7ZcuG59HepqjQbTi68S78sdLvUh",
    "name": "BlueOcean Pay",
    "type":"store",//user type: merchant, store, cashier
    "store_id": 100011,
    "store_name": "BO Store Alipay"
  }
}
```

### 2.7ユーザーの追加

### api

```
/user/create
```

### パラメータリクエストパラメータ

フィールド|変数名|必須|タイプ|説明
---- | ---- | ---- | ---- | ----
Appid | appid | はい |文字列（32）| appid（ログイン時に取得）
サイン| sign |はい|ストリング（32）|
メール| email |はい|文字列（32）|ログインアカウント
パスワード|password|はい|文字列（32）|
タイプ|type| はい |文字列（32）|加盟店 - >加盟店、店舗 - >店舗、レジ係 - >レジ係

リクエストパラメータの例：

```
{
  "appid": "100002",
  "email": "test@blueoceanpay.com",
  "password":"123456",
  "type": "store",  
  "sign": "C770EDDEC3F173CE69B5A46D7A009A59"
}
```


###応答結果の例

#### Suceessfulの例

```
{
  "code": 200,
  "message": "success",
  "data":{
    "id": "1000047",
    "username": "test@blueoceanpay.com",
    "type": "store",
    ...
  }
}

```

####失敗の例

```
{
   "code":0,
   "message":"User Exists"
}
```


### 2.8 パスワードの設定

### api

```
/user/password
```

###パラメータリクエストパラメータ

フィールド|変数名|必須|タイプ|説明
---- | ---- | ---- | ---- | ----
Appid | appid | はい |文字列（32）| appid（ログイン時に取得）
サイン| sign |はい|ストリング（32）|
メール| email |はい|文字列（32）|ログインメールアカウント
元のパスワード|old|はい|文字列（32）|パスワード
新しいパスワード|new|はい|文字列（32）|
繰り返しパスワード|repeat|はい|文字列（32）|

リクエストパラメータ：:

```
{
  "old": "123456",
  "new": "654321",
  "repeat": "654321",
  "sign": "355954524EA6FF44D8582DC50BECF85F"
}
```


###応答結果の例

#### Suceessfulの例

```
{
  "code": 200,
  "message": "success"
}

```

#### 失敗の例

```
{
  "code": 0,
  "message": "2つのパスワードエラー"
}
```

### 2.9ユーザーの削除

### api

```
/user/remove
```

###パラメータリクエストパラメータ

フィールド|変数名|必須|タイプ|説明
----|----|----|----|----
Appid | appid | はい |文字列（32）| appid（ログイン時に取得）
署名|sign| はい ||署名
削除するUserId | user_id | いいえ | Int | Optional parameters渡されない場合、appidに対応するユーザーを削除します

リクエストパラメータの例：

```
{
  "appid":"1000050",
  "user_id":"1000051",
  "sign":"BCA64F4B0A34753497C9DC8943DDFCC9"
}
```

### レスポンスの例

#### 成功した例

```
{
  "code": 200,
  "message": "success"
}

```

#### 失敗した例の変更

```
{
  "code": 0,
  "message": "ユーザーが削除されました"
}
```


### 2.10ユーザーリスト

### api

```
/user/list
```

### パラメータリクエストパラメータ

フィールド|変数名|必須|タイプ|説明
----|----|----|----|----
Appid | appid | はい |文字列| appid（ログイン時に取得）
署名|sign| はい ||署名
ページングパラメータ|page|いいえ| Int |ページングパラメータ：ページデフォルト値：1
ページあたりのレコード数| limit | いいえ | Int | 1ページあたりのエントリ数デフォルト：10
ユーザータイプ|store_id|いいえ|文字列|デフォルト値：キャッシャー現在のデザイン店舗タイプのユーザーだけが同じ店舗内のキャッシャーをチェックし、それに応じて操作することができます
リクエストパラメータ：:

```
{
  "appid":"83",
  "sign":"BCA64F4B0A34753497C9DC8943DDFAED"
}
```

#### 成功例 

```
{
	"code": 200,
	"message": "success",
	"data": {
		"page": 1,
		"limit": 10,
		"total": 3,
		"items": [
			{
				"id": "1000112",
				"username": "qqqq.com",
				"name": "qqqq.com",
				"type": "cashier",
				"depart_id": "100002",
				"store_id": "4",
				"created_at": "2017-12-25 14:51:49",
				"updated_at": "2017-12-25 15:27:27"
			}
		],
	}
}

```

## Update

2018.03.29












