# BlueOcean Pay Api Document

## 1 接口規則

### 1.1 接口協議

#### 調用BlueOcean Pay接口須遵守以下規則：

1. 請求方式一律使用 POST 並將請求的數據參數的JSON字符串以 http body的方式傳遞

2. 請求的數據格式統一使用JSON格式，若參數有json字符串請轉義雙引號（\"）

3. 字符串編碼請統一使用UTF-8

4. 簽名算法MD5
  
 
### 1.2 參數簽名

1.2.1. 假設請求參數如下：

```
{
    "appid":"1000010",
    "name":"BlueOcean Pay",
    "region":"HK",
    "business":"Online payment"
}
```

1.2.2. 將參數按照鍵值對（key=value）的形式排列,按照參數名ASCII字典序排序,並用&連接

```
str = "appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK"
```

1.2.3. 在最後拼接上密鑰字符串 `&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb`

```
str += "&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb"
```
即 str 為:

```
appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb
```

   
1.2.4. 最後計算MD5值並將md5字符串轉換成大寫

```
sign = strtoupper(md5(str))
```

sign的值為:

```
08C612FB4D2D52C8C913EA00E3DABC8B
```

### 1.3 請求頭

為了便於追蹤、統計API請求
需在http頭部設置User Agent

#### User Agent 示例

```
# English
BoPayPos/1.1.0 NetType/WIFI Language/en_US

# 簡體中文
BoPayPos/1.1.0 NetType/WIFI Language/zh_CN

# 繁體中文
BoPayPos/1.1.0 NetType/WIFI Language/zh_HK

```
  
### 1.4 PHP簽名示例代碼：


```
<?php
/**
 * 參數數組數據簽名
 * @param array $data 參數
 * @param string $key 密鑰
 * @return string 簽名
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


### 1.5 接口域名

在不同的業務區域，使用不同的域名

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

### 1.6 響應碼說明

響應碼| 響應碼業務說明 | 邏輯處理
-----|--------------|-----------------------------------------------
0   |通用錯誤|未歸類的錯誤，需提示用戶或做相應的容錯處理
200|成功|按照具體業務邏輯處理
201|刷卡交易未知狀態|系統繁忙、用戶支付中、輸入密碼等原因，需調用 `/order/query` 查詢交易狀態
404|接口地址錯誤|根據文檔檢查接口地址
4001|請求數據格式錯誤|請使用JSON數據格式
4002|無效AppId|檢查AppId以及相應的密鑰是否正確
4003|缺少參數|按照接傳遞正確參數
4004|簽名錯誤|檢查簽名邏輯以及相應的密鑰是否正確
4005|通訊出錯|按提示排查代碼錯誤
4006|服務器繁忙|稍後再重新調用該接口
4007|請求方式錯誤|請使用POST方式
40100|無效請求||根據提示處理
40101|訂單已支付|
40102|訂單不存在|
40103|訂單已關閉|
40104|訂單已撤銷|
40105|訂單已退款|
40106|訂單號重復|
40107|餘額不足|檢查商戶賬戶餘額是否不足
40108|訂單超過退款期限|無法完成退款
40109|缺少參數|按照接口提示補全參數
40110|編碼格式錯誤|請使用UTF8編碼格式
40111|每個二維碼只可以用一次|重新獲取用戶支付二維碼
40400|系統繁忙|系統繁忙，稍後重試
40500|支付錯誤|提示用戶相應的信息
40600|未知錯誤|未歸類的錯誤
490001|時間參數格式非法(非法時間戳)|傳遞正確的時間參數
491001|不是合法的json參數|傳遞正確的json格式參數
491002|參數長度不能超過限定長度|檢查參數長度
491003|參數不能為空|傳遞參數值
499999|未知錯誤
492001|參數格式不符合指定要求|如不符合規則運算式
492002|數據不存在|如訂單數據不存在
492003|上游廠商不支持此功能|排查上游是否開通這項功能


### 1.7 通用響應數據結構

響應數據為Json格式,Key-Map 描述如下:

屬性    | 說明 | 示例
-------|------|-------
code   | 業務響應碼 | 200
message| 業務提示消息，根據業務結果，可直接使用該屬性值提示用戶 | Success
data   | 業務數據，需根據相應接口進行邏輯處理,有時為空(不存在該屬性) | 

正常示例

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": "1000258",
    "attach": "",
    "bank_type": "",
    "body": "Shopping"
  }
}

```


失敗示例

```
{
  "code": 40500,
  "message": "餘額不足"
}
```


## 2 接口列表

### 2.1 支付

Pos機根據使用場景，提交相應請求參數，完成相應支付業務

### Api: 

```
/payment/pay
```
### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String|appid,由商戶後台獲取，或者登錄獲取
簽名|sign|是|String參考簽名方法
支付方式|payment|是|String|micropay,alipay.qrcode,wechat.qrcode
訂單金額|total_fee|是|Int|支付金額 單位為"分" 如10即0.10元
支付授權碼|code|可選|String|payment為"micropay"時填寫 如支付寶 288271620985824610 微信 134519771100657507 服務端據此參數值區分
商戶自身訂單號|out\_trade\_no|可選|String|如果商戶有自己的訂單系統，可以自己生成訂單號，否則建議交由藍海支付後台自動生成
異步通知url|notify_url|可選|String|異步通知url
openid|sub_openid|可選|String|商戶公眾號、小程序獲取的openid 
appid|sub_appid|可選|String|商戶公眾號、小程序AppId
sub\_blue\_mch\_id|sub\_blue\_mch\_id|No|BlueOceanPay Sub merchant Id|BlueOceanPay Sub merchant Id
store\_id|store\_id|可选|Store Id|Store Id
body|body|可選|商品名称|商品名称
附加數據|attach|可選|String|附加數據，在查詢API和支付通知中原樣返回，該字段主要用於商戶攜帶訂單的自定義數據
h5\_redirect\_url|h5\_redirect\_url|可選|String|微信香港錢包公眾號支付跳轉url,支付寶WAP跳轉url
钱包|wallet|可選|String|限定支付錢包如HKD，CN

#### payment 參數說明

參數值|描述
----------|----------
micropay|刷卡支付 此時需傳遞支付授權碼 `code` 參數
alipay.qrcode|支付寶二維碼
alipay.wappay|支付寶WAP線上
wechat.qrcode|微信二維碼
blueocean.qrcode|混合二維碼 可以直接跳轉到qrcode對應的網址支付，也可以生成二維碼供用戶掃描
wechat.jsapi|公眾號、小程序支付
wechat.app|微信APP支付
unionpay.qrcode|銀聯二維碼
unionpay.link|銀聯UPOP

支付返回後，檢查交易狀態trade_state,並根據其結果，決定是否調用訂單查詢接口進行結果查詢處理

### 訂單支付狀態 trade_state 說明

```
NOTPAY:未支付
SUCCESS:支付成功
REFUND:已退款
CLOSED:已關閉
REVOKED:已撤銷
USERPAYING:支付中
PAYERROR:支付異常
```

### 正確響應數據說明

響應結果response.data數據說明

字段|變量名|類型|描述
----|----|---|----
訂單id|id|Int|如:10357
藍海訂單編號|sn|String|示例: 11201802091347484054542598 唯一標號，可以用於查詢，關閉，退款等操作
貨幣|fee_type|String|交易貨幣 如 HKD,AUD
商戶名稱|mch_name|String|如 "BlueOcean Pay"
商戶Id|out\_trade\_no|String|商戶訂單號 如: "11201802091347484054542598"
支付方交易號|transaction_id|String| P5631VZG299QZN94JD
支付提供方|provider|String|如:alipay,wechat
訂單時間|create_time|Int|時間戳 如: 1518155270
支付時間|time_end|Int|成功支付的時間戳 如1518155297
交易狀態|trade_state|String| NOTPAY
二維碼文本|qrcode|String|掃碼支付時存在，客戶端使用第三方工具，將該內容生成二維碼，供用戶掃描付款 如 "https://qr.alipay.com/bax03112k12liy7lrysg2004", "weixin://wxpay/bizpayurl?pr=HBXdDeM"
實際支付金額|total_fee|Int|用戶需要支付的金額 如:10
優惠金額|discount|Int|優惠金額，用於商家自身系統集成，顯示 如:2
數據簽名|sign|String|如"7FB42F08C85670A86431F97109DE8683",用於本地校驗



### 請求示例

#### 支付寶二維碼示例:

請求參數

```
{
  "appid": "1000322",
  "payment": "alipay.qrcode",
  "total_fee": 10,
  "wallet": "CN",
  "sign": "520489313B46B5D403CCD8A01267B6C7"
}
```

響應結果

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
如果在app中實現alipay，需使用Alipay sdk喚起alipay

參考支付寶: https://docs.open.alipay.com/204/105695/

#### 支付寶WAP線上示例

```
{
  "appid": "1000343",
  "payment": alipay.wappay,
  "total_fee": "20",
  "wallet": "CN",
  "notify_url": "http://blueocean.com/notify",
  "h5_redirect_url": "http://blueocean.com/notify",
  "store_id" : "1000342",
  "body" : "shopping",
  "sign": "1FBFA9773ACEA258829477ED333381BD"
}
```

响应

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 1000343,
    "attach": "",
    "bank_type": "",
    "body": "shopping",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": "1557469674",
    "detail": "",
    "discount": "0",
    "fee_type": "HKD",
    "id": "1006228",
    "is_subscribe": "N",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "FBWRgJU1GC",
    "out_trade_no": "11201905101427535280230440",
    "pay_amount": "20",
    "provider": "alipay",
    "qrcode": "https://api.yedpay.com/o-wap/NMJVPOM78RMMK70RL8",
    "sn": "11201905101427535280230440",
    "time_end": 0,
    "total_fee": "20",
    "total_refund_fee" : 0,
    "trade_state": "NOTPAY",
    "trade_type": "WAPPAY",
    "transaction_id": "",
    "wallet": "CN",
    "sign": "5355B47A4F99F86E46658F2CF3446941"
  }
}
```

#### 混合二維碼示例

請求參數

```
{
  "appid": "1000343",
  "discount": 0,
  "notify_url": "https://payment.comenix.com/index/debug",
  "payment": "blueocean.qrcode",
  "total_fee": 13,
  "sign": "1FBFA9773ACEA258829477ED333381BD"
}
```

響應

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 1000343,
    "attach": "",
    "bank_type": "",
    "body": "",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": "1526957792",
    "detail": "",
    "discount": "0",
    "fee_type": "HKD",
    "id": "97736",
    "is_subscribe": "N",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "kROW6XeRn6",
    "out_trade_no": "11201805221056323300637881",
    "pay_amount": "13",
    "provider": "blueocean",
    "qrcode": "http://api.hk.blueoceanpay.com/order/qrcode/97736",
    "sn": "11201805221056323300637881",
    "time_end": 0,
    "total_fee": "13",
    "trade_state": "NOTPAY",
    "trade_type": "NATIVE",
    "transaction_id": "",
    "wallet": "",
    "sign": "8663CC409008CA4ED66D1F97BDB94414"
  }
}
```



#### 刷卡支付示例 

請求參數

```
{
  "appid": "1000322",
  "code": "134602370743606195",
  "payment": "micropay",
  "total_fee": 5,
  "sign": "5D8883E85FB4D721A0CF53BBD0A12905"
}
``` 

響應

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



### 公眾號、小程序示例

請求參數

```
{
  "appid": "1000258",
  "payment": "wechat.jsapi",
  "sub_appid": "wx6f4b43e962dbb48c",
  "sub_openid": "oxoPW5SUhI5l_Szaef2Rfd4qE57w",
  "total_fee": 2,
  "wallet": "CN",
  "sign": "D6BF87F2831B3F66AB55ADF96470A550"
}
```

響應結果

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 1000258,
    "attach": "",
    "bank_type": "",
    "body": "Shopping",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": "1530669821",
    "detail": "",
    "discount": "0",
    "fee_type": "HKD",
    "id": "171811",
    "is_subscribe": "N",
    "jsapi": "{\"appId\":\"wx6f4b43e962dbb48c\",\"timeStamp\":\"1530669821\",\"nonceStr\":\"lNTHG8Uf5K72iVgZEyJH\",\"package\":\"prepay_id=wx04100341876500f618f0c8c61198797061\",\"signType\":\"MD5\",\"paySign\":\"CED5552DA5C377F3E65818C7A66AF45C\"}",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "1jnQ6C4rfk",
    "openid": "oxoPW5SUhI5l_Szaef2Rfd4qE57w",
    "out_trade_no": "11201807041003410641249865",
    "pay_amount": "2",
    "provider": "wechat",
    "qrcode": "",
    "refundable": 0,
    "sn": "11201807041003410641249865",
    "time_end": 0,
    "total_fee": "2",
    "total_refund_fee": 0,
    "trade_state": "NOTPAY",
    "trade_type": "JSAPI",
    "transaction_id": "",
    "wallet": "CN",
    "sign": "5C22D88E332511B4FBD2B620322BC24A"
  }
}

```
### 微信APP示例

請求參數

```
{
  "appid": "1000258",
  "payment": "wechat.app",
  "sub_appid": "wx6f4b43e962dbb48c",
  "total_fee": 501,
  "wallet": "CN",
  "store_id": "1000342",
  "body": "test",
  "sign": "0BDCCE6962C76082E6FBA5FA07878929"
}
```
響應結果

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 1000343,
    "attach": "",
    "bank_type": "",
    "body": "test",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": "1554110899",
    "detail": "",
    "discount": "0",
    "fee_type": "HKD",
    "id": "829084",
    "is_subscribe": "N",
    "app": "{\"appid\":\"wxa4dc6a4e80965a1d\",\"partnerid\":\"1503284471\",\"prepayid\":\"wx20190401172818340140\",\"package\":\"Sign=WXPay\",\"noncestr\":\"63L8okvqA33lCR2eCBfR\",\"timestamp\":\"1554110898\",\"paySign\":\"9C48C140210DA72B1DED250606721585\"}",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "3YcBzhHncs",
    "out_trade_no": "11201904011728183454944814",
    "pay_amount": "2",
    "provider": "wechat",
    "qrcode": "",
    "refundable": 0,
    "sn": "11201807041003410641249865",
    "time_end": 0,
    "total_fee": "501",
    "total_refund_fee": 0,
    "trade_state": "NOTPAY",
    "trade_type": "APP",
    "transaction_id": "",
    "wallet": "CN",
    "sign": "1A878D7E06559821C4BC9BBBE1657436"
  }
}

```


### 银联UPOP示例

請求參數

```
{
  "appid": "1000258",
  "payment": "unionpay.link",
  "total_fee": 10,
  "wallet": "CN",
  "notify_url":"http://blueoceanpay.com/",
  "sign": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```
響應結果

```
{
  "code": 200,
  "message": "success",
  "data": {
    "adapter": "chinaums",
    "appid": 1000258,
    "attach": "",
    "bank_type": "",
    "body": "",
    "cash_fee": "0",
    "cash_fee_type": "",
    "create_time": "1606806091",
    "detail": "",
    "discount": "0",
    "fee_type": "HKD",
    "id": "2275866",
    "is_print": "0",
    "is_subscribe": "N",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "rZblCN3Tn5",
    "out_trade_no": "1120201201150xxxxxxxxxxxxxxxx",
    "pay_amount": "10",
    "provider": "unionpay",
    "qrcode": "https://apigw.gnete.com.hk/easyLinkApi/Payment/CreateChannelData?amount=0.1&currency=344&accessKey=1989fc10edf88de13c7176c2b3956b9e08b94cfe6d77231d0069fxxxxxxxxxxx&paymentInfoId=69cf4a55e6164cd9b293xxxxxxxxxxxx",
    "refundable": 0,
    "sn": "1120201201150xxxxxxxxxxxxxxxx",
    "time_end": 0,
    "total_fee": "10",
    "total_refund_fee": 0,
    "trade_state": "USERPAYING",
    "trade_type": "LINK",
    "transaction_id": "8a8994a975789xxxxxxxxxxxxxxxxxxx",
    "wallet": "CN",
    "sign": "670031A0FC96E723AA9xxxxxxxxxxxxx"
  }
}

```


### 拿到api數據(data.jsapi使用JSON.parse(data.jsapi)轉為JSON對象)後參考微信文檔，完成h5調用

[https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6)

### 香港錢包公眾號支付

#### 1. 引入wechat js

```
<script src="http://res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
```

#### 2.通過config接口註入權限驗證配置

```
<script type="text/javascript">
	wx.config({
        beta : true,
	    debug: true, // 調試作用，true為打開 false為關閉，開啟調試模式,調用的所有api的返回值會在客戶端alert出來，若要查看傳入的參數，可以在pc端打開，參數信息會通過log打出，僅在pc端時才會打印。
	    appId: '', // 必填，公眾號的唯壹標識
	    timestamp: , // 必填，生成簽名的時間戳
	    nonceStr: '', // 必填，生成簽名的隨機串
	    signature: '',// 必填，簽名參考：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115
	    jsApiList: ["getH5PrepayRequest"] // 必填，需要使用的JS接口列表
});
</script>
```

#### 3. 傳入預支付時返回的相關參數 ，通過ready接口處理成功驗證

```
<script type="text/javascript">
wx.ready(function(){
	    // config信息驗證後會執行ready方法，所有接口調用都必須在config接口獲得結果之後，config是壹個客戶端的異步操作，所以如果需要在頁面加載時就調用相關接口，則須把相關接口放在ready函數中調用來確保正確執行。對於用戶觸發時才調用的接口，則可以直接調用，不需要放在ready函數中。
	    //該參數為BOPAY服務端返回的數據
	    var payConfig = {
               "appId" : ""// 公眾號的唯壹標識，
               "timeStamp" : "",生成簽名的時間戳
               "nonceStr" : "", // 商戶生成的隨機字符串。由商戶生成後傳入
               "package" : "",//統壹下單接口返回的prepay_id參數值，並附加商戶歸屬地信息
               "signType" : "SHA1",//字段名稱：簽名方式；參數類型：字符串類型；字段來源：按照文檔中所示填入，目前僅支持SHA1
               "paySign" : "" // 簽名，詳見5.2簽名算法_JSAPI
         }
	     wx.invoke('getH5PrepayRequest', payConfig,function(res){//回调函数
             var msg = res.errMsg || res.err_msg;    //不同的版本定義的字段名不壹致
             if(!msg){}
             if(-1 != msg.indexOf("ok")){//調用成功
             }else if(-1 != msg.indexOf("cancel")){//用戶取消        
             }else{//失敗         
             }
        });
	    });
</script>

```

### 回调

支付完成後，平臺會把相關支付結果通過數據流的形式發送給商戶，商戶需要接收處理，並按文檔規範返回應答。
回調參數

字段|变量名|类型|描述
----|----|---|----
付款銀行|bank_type|String|付款銀行編碼,如:CFT
付款金額|cash_fee|Int|用戶付款的金額，如：20
支付貨幣類型|cash_fee_type|String|交易貨幣 如 CNY,HKD,AUD
幣種|fee_type|String|幣種 如 CNY,HKD,AUD
商戶訂單號|out\_trade\_no|String|商戶訂單號 如: "11201802091347484054542598"
支付方交易號|transaction_id|String| 如: P5631VZG299QZN94JD
支付完成時間|time_end|String|如:20190402162714
訂單金額|total_fee|Int|訂單金額 如：20
交易類型|trade_type|String|如: NATIVE
appid|appid|String| appid,由商戶後臺獲取，或者登錄獲取
隨機字符串|nonce_str|String|隨機字符串 如:O2r8GjZ46e
數據簽名|sign|String|如"7FB42F08C85670A86431F97109DE8683",用於本地校驗

響應參數
 SUCCESS

1. 同樣的通知可能會多次發送給商戶系統。商戶系統必須能夠正確處理重復的通知

2. 後臺通知交互時，如果平臺收到商戶的應答不符合規範或超時，平臺會判定本次通知失敗，按照機制重新發送通知，

### 2.2 退款

當交易發生之後一段時間內，由於買家或者賣家的原因需要退款時，賣家可以通過退款接口將支付款退還給支付用戶，

支付提供方(支付寶，微信等)將在收到退款請求並且驗證成功之後，按照退款規則將支付款按原路退還支付用戶帳號。

#### Api: 

```
/payment/refund
```
#### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String|appid,登錄時獲取
簽名|sign|是|String|
訂單編號|sn|否|String|與out\_trade\_no二選一,優先使用sn
商戶訂單號|out\_trade\_no|否|String|商戶訂單號
商戶退款單號|out\_refund\_no|否|String|商戶退款單號
退款金額|refund_fee|否|Int|可選參數，默認為訂單總額，即全額退款 支付寶不支持部分退款
退款描述|refund_desc|否|String|退款說明
退款密碼|password|否|String|退款密碼

#### 返回結果

字段|變量名|必填|類型|描述
----|----|----|----|----
返回碼|code|是|String(32)|返回碼，請參考返回碼表
返回信息|message|是|String(256)|返回信息，成功信息或錯誤信息
返回數據|data|否|Array/String|返回數據集或其他提示信息
  
###### 如果code=200,data参数：  

字段|變量名|必填|類型|描述
----|----|----|----|----
支付提供方訂單號|transaction_id|是|String(28)|微信，支付寶訂單號
商戶訂單號|out\_trade_no|是|String(32)|商戶訂單號
商戶退單號|out\_refund_no|是|String(32)|商戶退單號
微信退單號|refund_id|是|String(64)|微信退單號
標價金額|total_fee|是|Int|最小單位港幣分
標價幣種|fee_type|是|String(8)|一般是HKD
退款金額|refund\_fee|是|Int|最小單位港幣分
退款幣種|refund\_fee_type|是|String(8)|貨幣如 HKD
現金支付金額|cash_fee|是|Int|最小單位 人民幣分
現金支付幣種|cash\_fee_type|是|String(8)| 支付貨幣如 CNY
現金退款金額|cash\_refund_fee|是|Int|最小單位人民幣分
現金退款幣種|cash\_refund\_fee_type|是|String(8)|默認是CNY
匯率|rate|是|String(16)|匯率


#### 响应示例

```
{
   "appid": "1000258",
   "attach": "",
   "bank_type": "",
   "body": "",
   "cash_fee": "0",
   "cash_fee_type": "",
   "create_time": "1523501255",
   "detail": "",
   "discount": "0",
   "fee_type": "HKD",
   "id": "64260",
   "is_subscribe": "N",
   "mch_name": "BlueOcean Pay",
   "nonce_str": "LYuqnmuIlk",
   "out_refund_no": "11201804121050043286306633",
   "out_trade_no": "11201804121047337839520818",
   "pay_amount": "10",
   "provider": "alipay",
   "qrcode": "",
   "refund_desc": "",
   "refund_fee": "10",
   "refund_status": "SUCCESS",
   "refund_time": "1523501406",
   "sn": "11201804121047337839520818",
   "time_end": 1523501255,
   "total_fee": "10",
   "trade_state": "REFUND",
   "trade_type": "MICROPAY",
   "transaction_id": "2NMJVPOM96DMZ70RL8",
   "wallet": "",
   "sign": "FC173A8B25C8AACF1BD3CFFF908F3632"
}

```

### 2.2.1 退款査詢

#### Api:

```
/order/refundquery
```
#### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String|
簽名|sign|是|String|
退款單號|customer_refund_no|是|String|退款單號

#### 返回結果

字段|變量名|必填|類型|描述
----|----|----|----|----
返回碼|code|是|String(32)|返回碼，請參考返回碼表
返回信息|message|是|String(256)|返回信息，成功信息或錯誤信息
返回數據|data|否|Array/String|返回數據集或其他提示信息

###### 如果code=200,data参数：

字段|變量名|必填|類型|描述
----|----|----|----|----
商戶退單號|out\_refund_no|是|String（32）|商戶退單號
商戶訂單號|out\_trade_no|是|String（32）|商戶訂單號
藍海訂單號|sn|是|String（32）|藍海訂單號
標價金額|total_fee|是|Int|最小組織港幣分
標價幣種|fee_type|是|String（8）|一般是HKD
退款金額|refund\_fee|是|Int|最小組織港幣分
現金支付金額|cash_fee|是|Int|最小組織人民幣分
現金支付幣種|cash\_fee_type|是|String（8）|支付貨幣如CNY
退款時間|refund_time|是|String（15）|退款時間
退款狀態|refund_status|是|String（16）| SUCCESS

#### 請求參數

```
{
    "appid": "1000258",
    "customer_refund_no": "11202012101315545xxxxxxxxxxxxxx",
    "sign": "03818F947E2398376B2xxxxxxxxxxxxx"
}
```

#### 返回結果示例

```
{
   "adapter": "wantu",
   "appid": 1000258,
   "cash_fee": "0",
   "cash_fee_type": "CNY",
   "discount": "0",
   "fee_type": "HKD",
   "mch_name": "BlueOcean Pay",
   "nonce_str": "0124tYLSSH",
   "out_refund_no": "11202012101315545xxxxxxxxxxxxxx",
   "out_trade_no": "11202012101314xxxxxxxxxxxxxxx",
   "pay_amount": "10",
   "provider": "alipay",
   "refund_desc": "",
   "refund_fee": "5",
   "refund_status": "SUCCESS",
   "refund_time": "2020-12-10 13:14:49",
   "sn": "11202012101314xxxxxxxxxxxxxxx",
   "total_fee": "10",
   "trade_type": "MICROPAY",
   "wallet": "CN",
   "sign": "9957257C48C00CA4Bxxxxxxxxxxxxxxx"
}

```


### 2.3 訂單操作

訂單狀態操作(查詢，關閉，撤銷)，使用類似的參數

### Api: 

```
查詢
/order/query
關閉
/order/close
撤銷
/order/reverse
```
### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String|appid,登錄時獲取
簽名|sign|是|String|
訂單編號|sn|否|String|與out\_trade\_no二選一,優先使用sn
商戶訂單號|out\_trade\_no|否|String|商戶訂單號


請求示例

查詢

```
{
  "appid": "1000322",
  "sn": "11201802071854269363947431",
  "sign": "9CC7EB3EBE33D11421FE63298F2EEB94"
}

```

結果

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
    "refundable": "10",
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


### 2.4 訂單列表

訂單列表

### Api: 

```
/order/list
```
### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
接口ID|appid|是|String|由藍海支付提供的接口應用ID(或通過登錄接口獲取)
簽名|sign|是|String|詳情見簽名規則
支付服務提供方|provider|否|String|默認查詢所有 alipay,wechat
分頁參數|page|否|Int|分頁參數:第幾頁 默認值:1
每頁記錄數|limit|否|Int|每頁條目數 默認值:10
門店Id|store_id|否|Int|門店Id, 登錄接口獲取的store_id > 0 時填寫
交易狀態|trade_state|否|String|交易狀態 如 SUCCESS,REFUND
開始時間|start_time|否|String|按照創建時間查詢訂單 如 2017-12-12
結束時間|end_time|否|String|按照創建時間查詢訂單 如2017-12-13
sub\_blue\_mch\_id|sub\_blue\_mch\_id|No|BlueOceanPay Sub merchant Id|BlueOceanPay Sub merchant Id

#### 返回結果

字段|變量名|必填|類型|描述
----|----|----|----|----
返回碼|code|是|String|返回碼，請參考返回碼表
返回信息|message|是|String|返回信息，成功信息或錯誤信息
返回數據|data|否|Array|返回數據集

#### 請求參數示例

```
{
    "appid": "1000322",
    "limit": 10,
    "page": 1,
    "store_id": 1000321,
    "sign": "55F40D7150AD3013E4AB479745FAE542"
}
```

### 返回結果示例

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


### 2.5 退款列表

api：

```
/refund/list
```

#### 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
接口ID|appid|是|String|
簽名|sign|是|String|詳情見簽名規則
分頁參數|page|否|Int|分頁參數:第幾頁 默認值:1
每頁記錄數|limit|否|Int|每頁條目數 默認值:10
支付提供方|provider|否|String|alipay,wechat
退款狀態|refund_status|否|String|退款狀態 如 SUCCESS
開始時間|start_time|否|String|按照退款時間查詢訂單 如 2017-12-12
結束時間|end_time|否|String|按照退款時間查詢訂單 如2017-12-13
sub\_blue\_mch\_id|sub\_blue\_mch\_id|否|BlueOceanPay Sub merchant Id|BlueOceanPay Sub merchant Id
商戶訂單號|out_trade_no|否|String|商戶訂單號（傳遞該值，則只會査詢指定訂單的退款清單）

#### trade_state
```
SUCCESS:退款成功
PROCESSING:退款處理中
REFUNDCLOSE:退款關閉
CHANGE:退款異常
```

#### 返回結果

字段|變量名|必填|類型|描述
----|----|----|----|----
返回碼|code|是|String(32)|返回碼，請參考返回碼表
返回信息|message|是|String(256)|返回信息，成功信息或錯誤信息
返回數據|data|否|Array|返回數據集或其他提示信息


請求參數
```
{
  "appid": "1000322",
  "limit": 2,
  "page": 1,
  "sign": "74CD2AAB31B3FB76A68FEBF8FC56AF70"
}
```

響應結果

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


### 2.6 用戶登錄

### api

```
/user/login
```

### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
email|email|是|String(32)|登錄帳號
password|password|是|String(32)|密碼

#### 使用 POST 並將請求的數據參數的json字符串以 http body的方式傳遞

#### 該方法無需簽名，可以在chrome 瀏覽器插件 Advanced REST client 中測試

請求參數示例:

```
{"email":"abc@blueoceanpay.com","password":"123456"}
```
該帳號不存在，實際使用中，請由商戶後台獲取。

### Response 響應示例

#### 商戶帳號登錄正確返回示例

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 100018,
    "app_key": "TVqec7ZcuG59HepqjQbTi68S78sdLvUh",
    "name": "御龍集團",
    "store_id": 0,
    "store_name": ""
  }
}

```

#### 門店帳號登錄正確返回示例

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 100018,
    "app_key": "TVqec7ZcuG59HepqjQbTi68S78sdLvUh",
    "name": "御龍集團",
    "type":"store",//登錄的帳號類型: merchant -> 商戶, store -> 門店, cashier -> 收銀
    "store_id": 100011,
    "store_name": "中環店"
  }
}
```

### 2.7 添加用戶

### api

```
/user/create
```

### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登錄時獲取
sign|sign|是|String(32)| 
email|email|是|String(32)|登錄帳號
password|password|是|String(32)|
type|type|是|String(32)| merchant -> 商戶 , store -> 門店, cashier -> 收銀

請求參數示例:

```
{
  "appid": "100002",
  "email": "test@blueoceanpay.com",
  "password":"123456",
  "type": "store",  
  "sign": "C770EDDEC3F173CE69B5A46D7A009A59"
}
```


### Response 響應示例

#### 成功示例

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

#### 失敗示例

```
{
  "code": 0,
  "message": "帳號已存在"
}
```

### 2.8 設置密碼

### api

```
/user/password
```

### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登錄時獲取
sign|sign|是|String(32)|
email|email|是|String(32)|登錄郵箱帳號
old password|old|是|String(32)|密碼
new password |new|是|String(32)|
repeat password|repeat|是|String(32)|



請求參數示例:

```
{
  "old": "123456",
  "new": "654321",
  "repeat": "654321",
  "sign": "355954524EA6FF44D8582DC50BECF85F"
}
```


### Response 響應示例

#### 修改成功示例

```
{
  "code": 200,
  "message": "success"
}

```

#### 修改失敗示例

```
{
  "code": 0,
  "message": "兩次密碼錯誤"
}
```


### 2.9 刪除用戶

### api

```
/user/remove
```

### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登錄時獲取
簽名|sign|是|String|簽名
要刪除的用戶Id|user_id|否|Int| 可選參數 如果不傳遞，則刪除appid對應的用戶

請求參數示例:

```
{
  "appid":"1000050",
  "user_id":"1000051",
  "sign":"BCA64F4B0A34753497C9DC8943DDFCC9"
}
```

### Response 響應示例

#### 修改成功示例

```
{
  "code": 200,
  "message": "success"
}

```

#### 修改失敗示例

```
{
  "code": 0,
  "message": "用戶已經刪除"
}
```

### 2.10 用戶列表
### api

```
/user/list
```

### Parameters 請求參數

字段|變量名|必填|類型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登錄時獲取
簽名|sign|是|String|簽名
分頁參數|page|否|Int|分頁參數:第幾頁 默認值:1
每頁記錄數|limit|否|Int|每頁條目數 默認值:10
用戶類型|type|否|String|默認值: cashier(收銀員) 目前設計 只有store類型的用戶能夠查看同店鋪的收銀員並對其進行相應操作


請求參數示例:

```
{
  "appid":"83",
  "sign":"BCA64F4B0A34753497C9DC8943DDFAED"
}
```

### Response 響應示例

#### 獲取成功示例

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

2019.12.12