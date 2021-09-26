# BlueOcean Pay Api Document

## 1 接口规则

### 1.1 接口协议

#### 调用BlueOcean Pay接口须遵守以下规则：

1. 请求方式一律使用 POST 并将请求的数据参数的JSON字符串以 http body的方式传递

2. 请求的数据格式统一使用JSON格式，若参数有json字符串请转义双引号（\"）

3. 字符串编码请统一使用UTF-8

4. 签名算法MD5

#### 注意：
1. 接口文档部分字段使用`xxxxxxxxxxx`做了信息脱敏处理，注意甄别

### 1.2 参数签名

1.2.1. 假设请求参数如下：

```json
{
    "appid":"1000010",
    "name":"BlueOcean Pay",
    "region":"HK",
    "business":"Online payment"
}
```

1.2.2. 将参数按照键值对（key=value）的形式排列,按照参数名ASCII字典序排序,并用&连接

```
str = "appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK"
```

1.2.3. 在最后拼接上密钥字符串 `&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb`

```
str += "&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb"
```
即 str 为:

```
appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb
```


1.2.4. 最后计算MD5值并将md5字符串转换成大写

```
sign = strtoupper(md5(str))
```

sign的值为:

```
08C612FB4D2D52C8C913EA00E3DABC8B
```

### 1.3 请求头

为了便于追踪、统计API请求
需在http头部设置User Agent

#### User Agent 示例

```
# English
BoPayPos/1.1.0 NetType/WIFI Language/en_US

# 简体中文
BoPayPos/1.1.0 NetType/WIFI Language/zh_CN

# 繁体中文
BoPayPos/1.1.0 NetType/WIFI Language/zh_HK

```

### 1.4 PHP签名示例代码：


```
<?php
/**
 * 参数数组数据签名
 * @param array $data 参数
 * @param string $key 密钥
 * @return string 签名
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

在不同的业务区域，使用不同的域名

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

### 1.6 响应码说明

响应码| 响应码业务说明 | 逻辑处理
-----|--------------|-----------------------------------------------
0   |通用错误|未归类的错误，需提示用户或做相应的容错处理
200|成功|按照具体业务逻辑处理
201|刷卡交易未知状态|系统繁忙、用户支付中、输入密码等原因，需调用 `/order/query` 查询交易状态
404|接口地址错误|根据文档检查接口地址
4001|请求数据格式错误|请使用JSON数据格式
4002|无效AppId|检查AppId以及相应的密钥是否正确
4003|缺少参数|按照接传递正确参数
4004|签名错误|检查签名逻辑以及相应的密钥是否正确
4005|通讯出错|按提示排查代码错误
4006|服务器繁忙|稍后再重新调用该接口
4007|请求方式错误|请使用POST方式
40100|无效请求||根据提示处理
40101|订单已支付|
40102|订单不存在|
40103|订单已关闭|
40104|订单已撤销|
40105|订单已退款|
40106|订单号重复|
40107|余额不足|检查商户账户余额是否不足
40108|订单超过退款期限|无法完成退款
40109|缺少参数|按照接口提示补全参数
40110|编码格式错误|请使用UTF8编码格式
40111|每个二维码只可以用一次|重新获取用户支付二维码
40400|系统繁忙|系统繁忙，稍后重试
40500|支付错误|提示用户相应的信息
40600|未知错误|未归类的错误
490001|时间参数格式非法(非法时间戳)|传递正确的时间参数
491001|不是合法的json参数|传递正确的json格式参数
491002|参数长度不能超过限定长度|检查参数长度
491003|参数不能为空|传递参数值
499999|未知错误
492001|参数格式不符合指定要求|如不符合正则表达式
492002|数据不存在|如订单数据不存在
492003|上游厂商不支持此功能|排查上游是否开通这项功能
492004|请求接口出现超时|请求上游的curl出现超时
492005|上游接口响应为空|上游没有返回值
492006|支付宝汇率接口报错|(比如: ILLEGAL_SIGN)
492007|解析支付宝汇率接口响应的报文失败|上游返回的数据解析失败
493001|退款单号重复|商户提交的退款单号重复

### 1.7 通用响应数据结构

响应数据为Json格式,Key-Map 描述如下:

属性    | 说明 | 示例
-------|------|-------
code   | 业务响应码 | 200
message| 业务提示消息，根据业务结果，可直接使用该属性值提示用户 | Success
data   | 业务数据，需根据相应接口进行逻辑处理,有时为空(不存在该属性) |

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


失败示例

```
{
  "code": 40500,
  "message": "余额不足"
}
```


## 2 接口列表

### 2.1 支付

Pos机根据使用场景，提交相应请求参数，完成相应支付业务

### Api:

```
/payment/pay
```
### Parameters 请求参数

| 字段 | 变量名 | 必填 | 类型 | 描述 |
|----|----|----|----|----|
| appid | appid | 是 | String | appid,由商户后台获取，或者登录获取 |
| 签名 | sign | 是 | String | 参考签名方法 |
| 支付方式 | payment | 是 | String | micropay,alipay.qrcode,wechat.qrcode等，详见参数说明 |
| 支付提供方 | provider | 可选 | String | provider取值:wechat,alipay 当payment为micropay时,如果指定了provider,则会检查相应的支付授权码是否是对应的支付提供方的code，如果不是，不能支付 |
| 订单金额 | total_fee | 是 | Int | 支付金额 单位为"分" 如10即0.10元 |
| 支付授权码 | code | 可选 | String | payment为"micropay"时填写 如支付宝 288271620985824610 微信 134519771100657507 服务端据此参数值区分 |
| 商户自身订单号 | out\_trade\_no | 可选 | String | 如果商户有自己的订单系统，可以自己生成订单号，否则建议交由蓝海支付后台自动生成 |
| 异步通知url | notify_url | 可选 | String | 异步通知url |
| 微信openid | sub_openid | 可选 | String | 商户公众号、小程序获取的openid |
| 微信APPID | sub_appid | 可选 | String | 商户公众号、小程序、APP的AppId(微信公众号、小程序、APP支付必传) |
| 门店ID | store\_id | 可选 | Int | Store Id |
| body | body | 可选 | String | 商品名称 |
| 附加数据 |attach | 可选 | String | 附加数据，在查询API和支付通知中原样返回，该字段主要用于商户携带订单的自定义数据 |
| h5\_redirect\_url | h5\_redirect\_url | 可选 | String | 微信香港钱包公众号支付跳转url,支付宝WAP跳转url |
| 钱包 | wallet | 可选 | String | 限定支付钱包地区如HK,CN（注意：仅支付宝online有效） |
| 客户IP | spbill_create_ip | 可选 | String | 支付用户的IP,微信H5(MWEB)必传 |


#### payment 参数说明

| 参数值 | 描述 |
|----------|----------|
|micropay | 刷卡支付 此时需传递支付授权码 `code` 参数 |
|alipay.qrcode | 支付宝二维码 |
|alipay.wappay | 支付宝WAP线上 |
|alipay.app | 支付宝APP |
|blueocean.qrcode | 混合二维码 可以直接跳转到qrcode对应的网址支付，也可以生成二维码供用户扫描 |
|wechat.qrcode | 微信二维码 |
|wechat.jsapi | 公众号、小程序支付 |
|wechat.mweb | 微信H5支付(WEB在手机浏览器打开的场景) |
|wechat.app | 微信APP支付 |
|unionpay.qrcode | 银联二维码 |
|unionpay.link | 银联UPOP |


支付返回后，检查交易状态trade_state,并根据其结果，决定是否调用订单查询接口进行结果查询处理

### 订单支付状态 trade_state 说明

```
NOTPAY:未支付
SUCCESS:支付成功
REFUND:已退款
CLOSED:已关闭
REVOKED:已撤销
USERPAYING:支付中
PAYERROR:支付异常
```

### 交易类型 trade_type 说明
```
NATIVE: 微信线下码/支付宝线下码
JSAPI: 微信小程序/公众号/商户静态码
MICROPAY: 微信B2C
APP: 微信APP/支付宝APP
WAPPAY: 支付宝H5
FACEPAY: 支付宝B2C
LINK: 银联在线
MWEB: 微信H5支付
```

### 正确响应数据说明

响应结果response.data数据说明

| 字段 | 变量名 | 类型 | 描述 |
|----|----|---|----|
| 订单ID | id | Int | 如:10357 |
| 蓝海订单编号 | sn | String | 示例: 1120180209xxxxxxxxxxxxxxxxxx 唯一标号，可以用于查询，关闭，退款等操作 |
| 下单币种 | fee_type | String | 交易货币 如：HKD,AUD |
| 实付币种 | cash_fee_type | String | 客户实际付款的币种 如：CNY,HKD |
| 商户名称 | mch_name | String | 如 "BlueOcean Pay" |
| 商户Id | out\_trade\_no | String | 商户订单号 如: "1120180209xxxxxxxxxxxxxxxx" |
| 支付方交易号 | transaction_id | String | P563xxxxxxxxxxxxx |
| 支付提供方 | provider | String | 如:alipay,wechat |
| 订单时间 | create_time | Int | 时间戳 如: 1518155270 |
| 支付时间 | time_end | Int | 成功支付的时间戳 如1518155297 |
| 交易状态 | trade_state | String| 如：NOTPAY |
| 二维码文本 | qrcode | String |扫码支付时存在，客户端使用第三方工具，将该内容生成二维码，供用户扫描付款 如 "https://qr.alipay.com/bax03112k12liy7lrysg2004", "weixin://wxpay/bizpayurl?pr=HBXdDeM" |
| 微信H5文本 | mweb_url | String | 使用微信H5支付时存在，直接跳转该URL即可完成微信H5支付 |
| 实际支付金额 | total_fee | Int | 用户需要支付的金额 单位为"分" 如:10 |
| 优惠金额 | discount | Int | 优惠金额，用于商家自身系统集成，显示 如:2 |
| 数据签名 | sign | String | 如"7FB42F08C85670A86431F9710xxxxxx",用于本地校验 |


### 请求示例

#### 微信H5实例：
请求参数
```
{
    "appid": "1000258",
    "out_trade_no": "11202103291520556xxxxxxxxxxxxx",
    "payment": "wechat.mweb",
    "h5_redirect_url": "https://www.blueoceanpay.com"
    "spbill_create_ip":"183.14.29.179",
    "total_fee": "20",
    "sign": "FSFHSDXXXXXXXXXXXX"
}
```
响应结果
```
{
    "code": 200,
    "message": "success",
    "data": {
        "adapter": "wechat",
        "appid": 1000258,
        "attach": "",
        "bank_type": "",
        "body": "Shopping",
        "cash_fee": "0",
        "cash_fee_type": "",
        "create_time": "1620467095",
        "detail": "{\"goods_detail\":[{\"wxpay_goods_id\":\"7372\"}]}",
        "discount": "0",
        "fee_type": "HKD",
        "id": "2492386",
        "is_print": "0",
        "is_subscribe": "N",
        "mch_name": "BlueOcean Pay",
        "mweb_url": "https://wx.tenpay.com/cgi-bin/mmpayweb-bin/checkmweb?prepay_id=wx081744554937890xxxxxxxxxxx&package=45542939",
        "nonce_str": "4GkNxPT0TR",
        "out_trade_no": "11202103291520556xxxxxxxxxxxxxxx",
        "pay_amount": "20",
        "provider": "wechat",
        "qrcode": "",
        "refundable": 0,
        "sn": "1120210508174455543xxxxxxxxxxx",
        "time_end": 0,
        "total_fee": "20",
        "total_refund_fee": 0,
        "trade_state": "USERPAYING",
        "trade_type": "MWEB",
        "transaction_id": "",
        "wallet": "CN",
        "sign": "83326E1EE0xxxxxxxxxxxxxxxxxx"
    }
}
```


#### 支付宝二维码示例:

请求参数

```
{
  "appid": "1000258",
  "payment": "alipay.qrcode",
  "total_fee": 10,
  "wallet": "CN",
  "sign": "520489313B46B5D403CCD8Axxxxxxxx"
}
```

响应结果

```json
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
    "out_trade_no": "1120180209xxxxxxxxxxxxxxxxx",
    "total_fee": 10,
    "discount": 0,
    "pay_amount": 10,
    "provider": "alipay",
    "qrcode": "https://qr.alipay.com/bax03112k12liy7lrysg2004",
    "sn": "1120180209xxxxxxxxxxxxxx",
    "time_end": 0,
    "trade_state": "NOTPAY",
    "trade_type": "NATIVE",
    "transaction_id": "P5631Vxxxxxxxxxxxxx",
    "sign": "7FB42F08C85670A86431xxxxxxxxxxx"
  }
}
```

如果在app中实现alipay，需使用Alipay sdk唤起alipay。

参考支付宝: https://docs.open.alipay.com/204/105695/

#### 支付宝WAP线上示例

```json
{
  "appid": "1000258",
  "payment": alipay.wappay,
  "total_fee": "20",
  "wallet": "CN",
  "notify_url": "http://blueocean.com/notify",
  "h5_redirect_url": "http://blueocean.com/notify",
  "store_id" : "1000342",
  "body" : "shopping",
  "sign": "1FBFA9773ACEA258829477xxxxxxxxxx"
}
```

响应

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 1000258,
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
    "out_trade_no": "1120190510xxxxxxxxxxxxxxxx",
    "pay_amount": "20",
    "provider": "alipay",
    "qrcode": "https://api.yedpay.com/o-wap/NMJVPOM78RMMK70RL8",
    "sn": "1120190510xxxxxxxxxxxxxxxx",
    "time_end": 0,
    "total_fee": "20",
    "total_refund_fee" : 0,
    "trade_state": "NOTPAY",
    "trade_type": "WAPPAY",
    "transaction_id": "",
    "wallet": "CN",
    "sign": "5355B47A4F99F86E46658Fxxxxxxxxxx"
  }
}
```

#### 混合二维码示例

请求参数

```json
{
  "appid": "1000258",
  "discount": 0,
  "notify_url": "https://payment.comenix.com/index/debug",
  "payment": "blueocean.qrcode",
  "total_fee": 13,
  "sign": "1FBFA9773ACEA258829477Exxxxxxxxxxx"
}
```

响应

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 1000258,
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
    "out_trade_no": "1120180522xxxxxxxxxxxxxxxx",
    "pay_amount": "13",
    "provider": "blueocean",
    "qrcode": "http://api.hk.blueoceanpay.com/order/qrcode/97736",
    "sn": "1120180522xxxxxxxxxxxxxxxxx",
    "time_end": 0,
    "total_fee": "13",
    "trade_state": "NOTPAY",
    "trade_type": "NATIVE",
    "transaction_id": "",
    "wallet": "",
    "sign": "8663CC409008CA4ED66D1F9xxxxxxxxx"
  }
}
```


#### 刷卡支付示例

请求参数

```json
{
  "appid": "1000258",
  "code": "134602370743606195",
  "payment": "micropay",
  "total_fee": 5,
  "sign": "5D8883E85FB4D721A0CFxxxxxxxxxxxxxx"
}
```

响应

```json
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
    "out_trade_no": "1120180207xxxxxxxxxxxxxxx",
    "provider": "wechat",
    "qrcode": "",
    "sn": "1120180207xxxxxxxxxxxxxxxx",
    "time_end": 0,
    "total_fee": 5,
    "discount":2,
    "pay_amount":3,
    "trade_state": "NOTPAY",
    "trade_type": "MICROPAY",
    "transaction_id": "",
    "sign": "9E93F481EBD5E06xxxxxxxxxxxxxxxxxxxx"
  }
}
```


#### 支付宝APP示例

请求参数

```json
{
    "appid":"1000258",
    "payment":"alipay.app",
    "total_fee":"20",
    "wallet":"CN",
    "notify_url":"http://blueoceanpay.com/",
    "h5_redirect_url":"http://blueoceanpay.com/",
    "body":"ITMobileTestProduct",
    "sign":"CDB471EEDACDF3661F7xxxxxxxxxxxxx"
}
```

响应结果


```
{
    "code":200,
    "message":"success",
    "data":{
        "app":"{"_input_charset":"UTF-8","body":"ITMobileTestProduct","currency":"HKD","forex_biz":"FP","it_b_pay":"30m","notify_url":"https:\/\/api.tlinx.hk\/mct1\/paystatus\/notify\/payment\/AlipayHKOL","out_trade_no":"915687149300xxxxxxxxxxxxxx","partner":"208853102xxxxxxx","payment_inst":"ALIPAYCN","payment_type":1,"product_code":"NEW_WAP_OVERSEAS_SELLER","refer_url":"http:\/\/api-mirror.hk.blueoceanpay.com\/alipay\/order\/entry","seller_id":"20885310xxxxxxxxxxxx","service":"mobile.securitypay.pay","subject":"ITMobileTestProduct","total_fee":0.2,"sign":"cyouJ7enNDZEU0P94jVQnJGZKloWPCMmsyYu6x7YWUr1UeCtyTmJc0VecE3JKw8Qr9%2Bfcg6VbCwpgFz63WpfibJ7gQT4dz98jjcvLm%2B6CL3ra4P%2FQ5nwlVPIZq8HmFZNRE%2BH90c9FZ18KEzgLUibS9AYCSuvh8SMep1jeZP8lshOV6ZoVB9myyQdzG9qruhAyE69w%2FhT6JI32Wrr3UAPKhYDd7zCbOboW2aXCtcONuL%2BEoiNBft%2BintUCxR4otvKJEwjeXDZfPsEobQioPIHQuTNflsK2BOfiwUcxROoy9Wc0LFt32GWni9MVpg9u5P2v%2FRBHxAdQ%3D%3D","sign_type":"RSA"}",
        "app_format": "_input_charset=\"UTF-8\"&body=\"测试\"&currency=\"HKD\"&forex_biz=\"FP\"&it_b_pay=\"30m\"&notify_url=\"https://api.tlinx.hk/mct1/paystatus/notify/payment/AlipayHKOL\"&out_trade_no=\"91607479xxxxxxxxxxxxxxxxx\"&partner=\"208853102xxxxxxx\"&payment_inst=\"ALIPAYCN\"&payment_type=\"1\"&product_code=\"NEW_WAP_OVERSEAS_SELLER\"&refer_url=\"http://summer.blueoceantech.co/alipay/order/entry\"&seller_id=\"20885310xxxxxxx\"&service=\"mobile.securitypay.pay\"&subject=\"测试\"&total_fee=\"0.1\"&sign=\"wW6B%2BLPJ1fURqcm1pK1HfO2aDv6%2BF%2F2G9TJJrV51X2QIlp5hmuOR9QhnPEcbo0qlCZQ0BIgS1M0v1zjAO7huX%2FwUYEWN%2FBl5UfF%2FI2%2BWolh9dnInJDek7hDSGyCpjhV0E6T8eHJTDD3%2F%2FYJZ%2F3O9em%2F5iOxnInnOaxvJM8WUc6zBVVyCQmq6JE94lpN7rBQL2zDss13iJUPVXkuCZ1OccbtcisZlWtj%2FIFxDrFgSOTVhbvEfN1Zj5vSwVHO4iKil1YZqJg9LaU%2BfYzuPwff9GYhcZ5vhAwDitEPse0LjrauLlPKVbDWZGQ2JRHwMqFzEJ7RmGTqB3xxxxxxxxxxxxx\"&sign_type=\"RSA\"",
        "appid":1000258,
        "attach":"",
        "bank_type":"",
        "body":"ITMobileTestProduct",
        "cash_fee":"0",
        "cash_fee_type":"",
        "create_time":"1568714930",
        "detail":"",
        "discount":"0",
        "fee_type":"HKD",
        "id":"1338084",
        "is_subscribe":"N",
        "mch_name":"BlueOcean Pay",
        "nonce_str":"r2340tF8Mv",
        "openid":"ow8Pv05TbDLNPxxxxxxxxxxxxxxxxxxxx",
        "out_trade_no":"1120190917xxxxxxxxxxxxxxxxxx",
        "pay_amount":"20",
        "provider":"alipay",
        "qrcode":"",
        "refundable":0,
        "sn":"1120190917xxxxxxxxxxxxxxxxxxx",
        "time_end":0,
        "total_fee":"20",
        "total_refund_fee":0,
        "trade_state":"NOTPAY",
        "trade_type":"APP",
        "transaction_id":"91568714xxxxxxxxxxxxxxxxxx",
        "wallet":"CN",
        "sign":"D2FBD87FDB2CB5727xxxxxxxxxxxxxxxxx"
    }
}
```

### 公众号、小程序示例

请求参数

```
{
  "appid": "1000258",
  "payment": "wechat.jsapi",
  "sub_appid": "wx6f4b43xxxxxxxxxxxx",
  "sub_openid": "oxoPW5SUhIxxxxxxxxxxxxxxx",
  "total_fee": 2,
  "wallet": "CN",
  "sign": "D6BF87F2831B3F66Axxxxxxxxxxxxxxxxx"
}
```

响应结果

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
    "jsapi": "{\"appId\":\"wx6f4b43exxxxxxxxxxxxx\",\"timeStamp\":\"1530669821\",\"nonceStr\":\"lNTHG8Uf5K72iVgZEyJH\",\"package\":\"prepay_id=wx04100341876500f618f0cxxxxxxxxxxxxxx\",\"signType\":\"MD5\",\"paySign\":\"CED5552DA5C377F3E65818C7A66AF45C\"}",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "1jnQ6C4rfk",
    "openid": "oxoPW5SUhI5lxxxxxxxxxxxxx",
    "out_trade_no": "1120180704xxxxxxxxxxxxxxxxxxxx",
    "pay_amount": "2",
    "provider": "wechat",
    "qrcode": "",
    "refundable": 0,
    "sn": "1120180704xxxxxxxxxxxxxxxxxxxxx",
    "time_end": 0,
    "total_fee": "2",
    "total_refund_fee": 0,
    "trade_state": "NOTPAY",
    "trade_type": "JSAPI",
    "transaction_id": "",
    "wallet": "CN",
    "sign": "5C22D88E332511B4FBD2xxxxxxxxxxxxxx"
  }
}

```

### 微信APP示例

请求参数

```
{
  "appid": "1000258",
  "payment": "wechat.app",
  "sub_appid": "wx6f4xxxxxxxxxxxxxxxxxx",
  "total_fee": 501,
  "store_id": "1000342",
  "body": "test",
  "sign": "0BDCCE6962C76082E6xxxxxxxxxxxxxxx"
}
```
响应结果

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
    "app": "{\"appid\":\"wxa4dxxxxxxxxxxxxxxx\",\"partnerid\":\"1503284471\",\"prepayid\":\"wx20190xxxxxxxxxxxxxxxx\",\"package\":\"Sign=WXPay\",\"noncestr\":\"63L8okvqA33lCR2eCBfR\",\"timestamp\":\"1554110898\",\"paySign\":\"9C48C140210DA72B1DED2xxxxxxx\"}",
    "mch_name": "BlueOcean Pay",
    "nonce_str": "3YcBzhHncs",
    "out_trade_no": "1120190401xxxxxxxxxxxxxxxxx",
    "pay_amount": "2",
    "provider": "wechat",
    "qrcode": "",
    "refundable": 0,
    "sn": "1120180704xxxxxxxxxxxxxxxxxxxx",
    "time_end": 0,
    "total_fee": "501",
    "total_refund_fee": 0,
    "trade_state": "NOTPAY",
    "trade_type": "APP",
    "transaction_id": "",
    "wallet": "CN",
    "sign": "1A878D7E06559821Cxxxxxxxxxx"
  }
}

```

### 银联UPOP示例

请求参数

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
响应结果

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

### 拿到api数据(data.jsapi使用JSON.parse(data.jsapi)转为JSON对象)后参考微信文档，完成h5调用

[https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6)


### 香港钱包公众号支付

#### 1. 引入wechat js

```
<script src="http://res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
```

#### 2. 通过config接口注入权限验证配置

```
<script type="text/javascript">
	wx.config({
        beta : true,
	    debug: true, // 调试作用，true为打开 false为关闭，开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
	    appId: '', // 必填，公众号的唯一标识
	    timestamp: , // 必填，生成签名的时间戳
	    nonceStr: '', // 必填，生成签名的随机串
	    signature: '',// 必填，签名参考：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115
	    jsApiList: ["getH5PrepayRequest"] // 必填，需要使用的JS接口列表
});
</script>
```

#### 3. 传入预支付时返回的相关参数 ，通过ready接口处理成功验证

```
<script type="text/javascript">
wx.ready(function(){
	    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
	    //该参数为BOPAY服务端返回的数据
	    var payConfig = {
               "appId" : ""// 公众号的唯一标识，
               "timeStamp" : "",生成签名的时间戳
               "nonceStr" : "", // 商户生成的随机字符串。由商户生成后传入
               "package" : "",//统一下单接口返回的prepay_id参数值，并附加商户归属地信息
               "signType" : "SHA1",//字段名称：签名方式；参数类型：字符串类型；字段来源：按照文档中所示填入，目前仅支持SHA1
               "paySign" : "" // 签名，详见5.2签名算法_JSAPI
         }
	     wx.invoke('getH5PrepayRequest', payConfig,function(res){//回调函数
             var msg = res.errMsg || res.err_msg;    //不同的版本定义的字段名不一致
             if(!msg){}
             if(-1 != msg.indexOf("ok")){//调用成功
             }else if(-1 != msg.indexOf("cancel")){//用户取消
             }else{//失败
             }
        });
	    });
</script>

```

### 回调

支付完成后，平台会把相关支付结果通过POST的形式发送给商户，商户需要接收处理，并按文档规范返回应答。

回调参数

| 字段 | 变量名 | 类型 | 描述 |
|----|----|---|----|
| 付款银行 | bank_type | String | 付款银行编码,如:CFT |
| 付款金额 | cash_fee | Int | 用户付款的金额，单位为"分" 如：20 |
| 支付货币类型 | cash_fee_type | String | 交易货币 如 CNY,HKD,AUD |
| 下单标价币种 | fee_type | String | 币种 如 CNY,HKD,AUD |
| 商户订单号 | out\_trade\_no | String | 商户订单号 如: "11201802091347484054542598" |
| 支付方交易号 | transaction_id | String | 如: P5631VZG299QZN94JD |
| 支付完成时间 | time_end | String | 如:20190402162714 |
| 订单金额 | total_fee | Int | 订单金额 如：20 |
| 交易类型 | trade_type | String | 如: NATIVE |
| 商户ID | appid | String | appid,由商户后台获取，或者登录获取 |
| 随机字符串 | nonce_str | String | 随机字符串 如:O2r8GjZ46e |
| 数据签名 | sign | String | 如"7FB42F08C85670A86431xxxxxxxxxxxx",用于本地校验 |
| 发送版本 | version | String | 蓝海回调版本(商户可忽略) |
| 蓝海订单号 | sn | String | 蓝海生成的订单号，可用于查询订单 |

响应参数
```
SUCCESS
```

1. 同样的通知可能会多次发送给商户系统。商户系统必须能够正确处理重复的通知

2. 后台通知交互时，如果平台收到商户的应答不符合规范或超时，平台会判定本次通知失败，按照机制重新发送通知，

3. 参数接收需使用`POST`的`x-www-form-urlencode`形式接收

### 2.2 退款

当交易发生之后一段时间内，由于买家或者卖家的原因需要退款时，卖家可以通过退款接口将支付款退还给支付用户，

支付提供方(支付宝，微信等)将在收到退款请求并且验证成功之后，按照退款规则将支付款按原路退还支付用户帐号。

#### Api:

```
/payment/refund
```
#### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String|
签名|sign|是|String|
订单编号|sn|否|String|与out\_trade\_no二选一,优先使用sn
商户订单号|out\_trade\_no|否|String|商户订单号
商户退款单号|out\_refund\_no|否|String|商户退款单号
退款金额|refund_fee|否|Int|可选参数，默认为订单总额，即全额退款
退款描述|refund_desc|否|String|退款说明
退款密码|password|否|String|退款密码

#### 返回结果

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array/String|返回数据集或其他提示信息

###### 如果code=200,data参数：

字段|变量名|必填|类型|描述
----|----|----|----|----
支付提供方订单号|transaction_id|是|String(28)|微信，支付宝订单号
商户订单号|out\_trade_no|是|String(32)|商户订单号
商户退单号|out\_refund_no|是|String(32)|商户退单号
微信退单号|refund_id|是|String(64)|微信退单号
标价金额|total_fee|是|Int|最小单位港币分
标价币种|fee_type|是|String(8)|一般是HKD
退款金额|refund\_fee|是|Int|最小单位港币分
退款币种|refund\_fee_type|是|String(8)|货币如 HKD
现金支付金额|cash_fee|是|Int|最小单位 人民币分
现金支付币种|cash\_fee_type|是|String(8)| 支付货币如 CNY
现金退款金额|cash\_refund_fee|是|Int|最小单位人民币分
现金退款币种|cash\_refund\_fee_type|是|String(8)|默认是CNY
汇率|rate|是|String(16)|汇率


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
   "out_refund_no": "1120180412xxxxxxxxxxxxxxxxxxx",
   "out_trade_no": "1120180412xxxxxxxxxxxxxxxxx",
   "pay_amount": "10",
   "provider": "alipay",
   "qrcode": "",
   "refund_desc": "",
   "refund_fee": "10",
   "refund_status": "SUCCESS",
   "refund_time": "1523501406",
   "sn": "1120180412xxxxxxxxxxxxxxxxxx",
   "time_end": 1523501255,
   "total_fee": "10",
   "trade_state": "REFUND",
   "trade_type": "MICROPAY",
   "transaction_id": "2NMJVPxxxxxxxxxxxxxxxxx",
   "wallet": "",
   "sign": "FC173A8B25C8AACF1xxxxxxxxxxxxxxxxxxx"
}

```


### 2.2.1 退款查询

#### Api:

```
/order/refundquery
```
#### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String|
签名|sign|是|String|
退款单号|customer_refund_no|是|String|退款单号

#### 返回结果

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array/String|返回数据集或其他提示信息

###### 如果code=200,data参数：

字段|变量名|必填|类型|描述
----|----|----|----|----
商户退单号|out\_refund_no|是|String(32)|商户退单号
商户订单号|out\_trade_no|是|String(32)|商户订单号
蓝海订单号|sn|是|String(32)|蓝海订单号
标价金额|total_fee|是|Int|最小单位 港币分
标价币种|fee_type|是|String(8)|一般是HKD
退款金额|refund\_fee|是|Int|最小单位 港币分
现金支付金额|cash_fee|是|Int|最小单位 人民币分
现金支付币种|cash\_fee_type|是|String(8)| 支付货币如 CNY
退款时间|refund_time|是|String(15)|退款时间
退款状态|refund_status|是|String(16)| SUCCESS


#### 请求参数

```
{
    "appid": "1000258",
    "customer_refund_no": "11202012101315545xxxxxxxxxxxxxx",
    "sign": "03818F947E2398376B2xxxxxxxxxxxxx"
}
```

#### 响应示例

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


### 2.3 订单操作

订单状态操作(查询，关闭，撤销)，使用类似的参数

### Api:

```
查询
/order/query
关闭
/order/close
撤销
/order/reverse
```
### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String|appid,登录时获取
签名|sign|是|String|
订单编号|sn|否|String|与out\_trade\_no二选一,优先使用sn
商户订单号|out\_trade\_no|否|String|商户订单号
微信/支付宝订单号|transaction_id|否|string|微信/支付宝订单号


请求示例

查询

```
{
  "appid": "1000258",
  "sn": "1120180207xxxxxxxxxxxxxxxxxxx",
  "sign": "9CC7EB3EBE33D11xxxxxxxxxxxxxxxxxxx"
}

```

结果

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
    "out_trade_no": "1120180207xxxxxxxxxxxxxxxxxxxxx",
    "provider": "wechat",
    "qrcode": "weixin://wxpay/bizpayurl?pr=wDmjTFq",
    "refundable": "10",
    "sn": "1120180207xxxxxxxxxxxxxxxxxxxx",
    "time_end": 0,
    "total_fee": 10,
    "discount":0,
    "pay_amount":10,
    "trade_state": "CLOSED",
    "trade_type": "NATIVE",
    "transaction_id": "",
    "sign": "C15B790CBCFE826821xxxxxxxxxxxxxxx"
  }
}
```


### 2.4 订单列表

订单列表

### Api:

```
/order/list
```
### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
接口ID|appid|是|String|由蓝海支付提供的接口应用ID(或通过登录接口获取)
签名|sign|是|String|详情见签名规则
支付服务提供方|provider|否|String|默认查询所有 alipay,wechat
分页参数|page|否|Int|分页参数:第几页 默认值:1
每页记录数|limit|否|Int|每页条目数 默认值:10
门店Id|store_id|否|Int|门店Id, 登录接口获取的store_id > 0 时填写
交易状态|trade_state|否|String|交易状态 如 SUCCESS,REFUND
开始时间|start_time|否|String|此字段后续会废弃,按照创建时间查询订单 如 2017-12-12
结束时间|end_time|否|String|此字段后续会废弃,按照创建时间查询订单 如2017-12-13
开始时间戳|unixtimestamp_start|否|Int|Unix时间戳,目前10位.按照创建时间查询订单 如1555475513
结束时间戳|unixtimestamp_end|否|Int|Unix时间戳,目前10位.按照创建时间查询订单 如1555475513
sub\_blue\_mch\_id|sub\_blue\_mch\_id|可选|蓝海子商户Id|蓝海子商户Id


#### 返回结果

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String|返回码，请参考返回码表
返回信息|message|是|String|返回信息，成功信息或错误信息
返回数据|data|否|Array|返回数据集

#### 请求参数示例

```
{
    "appid": "1000322",
    "limit": 10,
    "page": 1,
    "store_id": 1000321,
    "sign": "55F40D7150AD3013E4xxxxxxxxxxxxxxx"
}
```

### 返回结果示例

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
        "sn": "1120180206xxxxxxxxxxxxxxxxxx",
        "provider": "alipay",
        "blue_mch_id": "1000258",
        "store_id": "1000321",
        "body": "",
        "out_trade_no": "1120180206xxxxxxxxxxxxxxxxxxx",
        "transaction_id": "3PG7Yxxxxxxxxxxxxxxx",
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
        "sn": "1120180206xxxxxxxxxxxxxxxxx",
        "provider": "wechat",
        "blue_mch_id": "1000258",
        "store_id": "1000321",
        "body": "BlueOceanPay",
        "out_trade_no": "20180206xxxxxxxxxxxxxx",
        "transaction_id": "42000000xxxxxxxxxxxxxxxxxxxxxx",
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

#### 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
接口ID|appid|是|String|
签名|sign|是|String|详情见签名规则
分页参数|page|否|Int|分页参数:第几页 默认值:1
每页记录数|limit|否|Int|每页条目数 默认值:10
支付提供方|provider|否|String|alipay,wechat
退款状态|refund_status|否|String|退款状态 如 SUCCESS
开始时间|start_time|否|String|按照退款时间查询订单 如 2017-12-12
结束时间|end_time|否|String|按照退款时间查询订单 如2017-12-13
开始时间戳|unixtimestamp_start|否|Int|Unix时间戳,目前10位.按照创建时间查询订单 如1555475513
结束时间戳|unixtimestamp_end|否|Int|Unix时间戳,目前10位.按照创建时间查询订单 如1555475513
sub\_blue\_mch\_id|sub\_blue\_mch\_id|否|蓝海子商户Id|蓝海子商户Id
商户订单号|out_trade_no|否|String|商户订单号(传递该值，则只会查询指定订单的退款列表)

#### trade_state
```
SUCCESS:退款成功
PROCESSING:退款处理中
REFUNDCLOSE:退款关闭
CHANGE:退款异常
```

#### 返回结果

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array|返回数据集或其他提示信息


请求参数
```
{
  "appid": "1000258",
  "limit": 2,
  "page": 1,
  "sign": "74CD2AAB31B3FB76A68Fxxxxxxxxxxxxxx"
}
```

响应结果

```json
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
        "out_trade_no": "a257f51afe1xxxxxxxxxxxxxxxxxx",
        "transaction_id": "4200000060201xxxxxxxxxxxxxx",
        "out_refund_no": "a8d95666c735f4xxxxxxxxxxxxxxxxx",
        "refund_id": "500002057220xxxxxxxxxxxxxxxxxxxxx",
        "total_fee": "2",
        "refund_fee": "2",
        "discount": "0",
        "fee_type": "HKD",
        "refund_status": "SUCCESS",
        "refund_desc": "",
        "refund_time": 1518079526,
        "pay_amount": 2
      }
    ]
  }
}
```


### 2.6 用户登录

### api

```
/user/login
```

### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
email|email|是|String(32)|登录帐号
password|password|是|String(32)|密码

#### 使用 POST 并将请求的数据参数的json字符串以 http body的方式传递

#### 该方法无需签名，可以在chrome 浏览器插件 Advanced REST client 中测试

请求参数示例:

```
{"email":"abc@blueoceanpay.com","password":"123456"}
```
该帐号不存在，实际使用中，请由商户后台获取。

### Response 响应示例

#### 商户帐号登录正确返回示例

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 100018,
    "app_key": "TVqec7ZcuG59Hepqjxxxxxxxxxxxxxxx",
    "name": "御龍集團",
    "store_id": 0,
    "store_name": ""
  }
}

```

#### 门店帐号登录正确返回示例

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 100018,
    "app_key": "TVqec7ZcuG59Hexxxxxxxxxxxxxxxxxxx",
    "name": "御龍集團",
    "type":"store",//登录的帐号类型: merchant -> 商户, store -> 门店, cashier -> 收银
    "store_id": 100011,
    "store_name": "中环店"
  }
}
```

### 2.7 添加用户

### api

```
/user/create
```

### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登录时获取
sign|sign|是|String(32)|
email|email|是|String(32)|登录帐号
password|password|是|String(32)|
type|type|是|String(32)| merchant -> 商户 , store -> 门店, cashier -> 收银

请求参数示例:

```
{
  "appid": "100002",
  "email": "test@blueoceanpay.com",
  "password":"123456",
  "type": "store",
  "sign": "C770EDDEC3F173CE6xxxxxxxxxxxxxxx"
}
```


### Response 响应示例

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

#### 失败示例

```
{
  "code": 0,
  "message": "帐号已存在"
}
```

### 2.8 设置密码

### api

```
/user/password
```

### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登录时获取
sign|sign|是|String(32)|
email|email|是|String(32)|登录邮箱帐号
old password|old|是|String(32)|密码
new password |new|是|String(32)|
repeat password|repeat|是|String(32)|



请求参数示例:

```
{
  "old": "123456",
  "new": "654321",
  "repeat": "654321",
  "sign": "355954524EA6FFxxxxxxxxxxxxxxxxxxx"
}
```


### Response 响应示例

#### 修改成功示例

```
{
  "code": 200,
  "message": "success"
}

```

#### 修改失败示例

```
{
  "code": 0,
  "message": "两次密码错误"
}
```


### 2.9 删除用户

### api

```
/user/remove
```

### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登录时获取
签名|sign|是|String|签名
要删除的用户Id|user_id|否|Int| 可选参数 如果不传递，则删除appid对应的用户

请求参数示例:

```
{
  "appid":"1000050",
  "user_id":"1000051",
  "sign":"BCA64F4B0A34753xxxxxxxxxxxxxxxxxxxx"
}
```

### Response 响应示例

#### 修改成功示例

```
{
  "code": 200,
  "message": "success"
}

```

#### 修改失败示例

```
{
  "code": 0,
  "message": "用戶已經刪除"
}
```

### 2.10 用户列表
### api

```
/user/list
```

### Parameters 请求参数

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String(32)|appid,登录时获取
签名|sign|是|String|签名
分页参数|page|否|Int|分页参数:第几页 默认值:1
每页记录数|limit|否|Int|每页条目数 默认值:10
用户类型|type|否|String|默认值: cashier(收银员) 目前设计 只有store类型的用户能够查看同店铺的收银员并对其进行相应操作


请求参数示例:

```
{
  "appid":"83",
  "sign":"BCA64F4B0A3475349xxxxxxxxxxxxxxxxxx"
}
```

### Response 响应示例

#### 获取成功示例

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


### 2.11 POS机上传数据接口
### api

```
/datacollection/pos
```

### Parameters 请求方式
*HTTP POST*

### Parameters 请求参数(json格式，放在http请求body中进行传递)

字段|变量名|必填|类型|描述
----|----|----|----|----
终端编号|terminal_id|是|String(255)|终端编号
appid|appid|否|String|appid,由商户后台获取，或者登录获取
电量|battery|是|double(3,2)|pos机电量，如56.52
是否在充电|is_charge|是|int|pos机是否在充电，1在充电，0没在充电
是否运行BOP程序|is_run_bop|是|int|pos机是否在运行bop程序，1是，0否
是否登录|is_login|是|int|bop程序是否登录
预留扩展字段|ext|否|String(4096)|默认值空字符串
数据签名|sign|String|如"7FB42F08C85670A86431F9710xxxxxx",用于本地校验

请求参数示例:

```
{
  "terminal_id":"1903065464xxx",
  "is_charge":"1"
}
```

### Response 响应示例

#### 获取成功示例

```
{
    "code": 200,
    "message": "success",
    "data": {
        "ok": true
    }
}

```

### 2.12 海关报关接口
### api

```
/customs/newCustoms
```

### Parameters 请求方式
*HTTP POST*

### Parameters 请求参数(json格式，放在http请求body中进行传递)

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String|appid,登录时获取
签名|sign|是|String|
订单编号|sn|否|String|与out\_trade\_no二选一,优先使用sn
商户订单号|out\_trade\_no|否|String|商户订单号
类型|type|否|String|传wechat或者alipay,微信报关或者支付宝报关，不传自会根据订单区分
海关|customs|是|String|例如 GUANGZHOU_ZS (详情见列表)
海关备案号|mch_customs_no|是|String|例如 3122xxxxxxxx
商户海关备案名称|merchant_customs_name|是|String|例如 XXX有限公司
关税|duty|否|Int|关税，以分为单位，非必填项
报关类型|action_type|是|String|ADD 新增报关申请 MODIFY 修改
是否拆单|is_split|否|String|商户控制本单是否拆单报关。仅当该参数传值为T或者t时，才会触发拆单（报关海关必须支持拆单）。
商户子订单号|sub_order_no|否|String|商户子订单号，如有拆单则必传
币种|fee_type|否|String|微信支付订单支付时使用的币种，暂只支持人民币CNY,如有拆单则必传
应付金额|order_fee|否|Int|子订单金额，以分为单位，不能超过原订单金额，order_fee=transport_fee+product_fee（应付金额=物流费+商品价格），如有拆单则必传
物流费|transport_fee|否|Int|物流费用，以分为单位，如有拆单则必传
商品价格|product_fee|否|Int|商品费用，以分为单位，如有拆单则必传
证件类型|cert_type|否|String|IDCARD 身份证
证件号码|cert_id|否|String|证件号码 例 330821198xxxxxxxxx用户大陆身份证号，尾号为字母X的身份证号，请大写字母X
用户姓名|name|否|String|例如 张三

customs 参数说明如下:

参数值    | 所属类型 | 描述
-------|------|-------
GUANGZHOU_ZS     |  微信    |广州（总署版）
GUANGZHOU_HP_GJ  |  微信    |广州黄埔国检（需推送订单至黄埔国检的订单需分别推送广州（总署版）和广州黄埔国检，即需要请求两次报关接口）
GUANGZHOU_NS_GJ  |  微信    |广州南沙国检（需推送订单至南沙国检的订单需分别推送广州（总署版）和广州南沙国检，即需要请求两次报关接口）
HANGZHOU_ZS      |  微信    |杭州（总署版）
NINGBO           |  微信    |宁波
ZHENGZHOU_BS     |  微信    |郑州（保税物流中心）
CHONGQING        |  微信    |重庆
SHANGHAI_ZS      |  微信    |上海（总署版）
SHENZHEN         |  微信    |深圳
ZHENGZHOU_ZH_ZS  |  微信    |郑州综保（总署版）
TIANJIN          |  微信  |天津（需要推送订单至天津海关时，需要在商户管理后台同时配置天津海关备案信息与天津国检备案信息；调用报关接口时只需推送天津海关，即请求一次报关接口。)
zongshu          |  支付宝    |总署
zhengzhou        |  支付宝    |河南保税物流中心
ningbo           |  支付宝    |宁波海关
henan            |  支付宝    |新郑综合保税区（空港）,先推送henan，再推送zongshu
tianjin          |  支付宝    |天津海关,先推送tianjin，再推送zongshu
nanshagj         |  支付宝    |南沙国检
shanghai_cbt     |  支付宝    |上海海关
guangzhou_airport|  支付宝    |推送广州机场国检，备案信息需要传企业在广电的备案信息
guangzhou_nansha |  支付宝    |推送广州南沙国检，备案信息需要传企业在广电的备案信息
guangzhou_huangpu|  支付宝    |推送广州黄埔国检，备案信息需要传企业在广电的备案信息
guangzhou_shatian|  支付宝    |推送广州沙田国检，备案信息需要传企业在广电的备案信息
hangzhou_zongshu |  支付宝    |杭州海关





请求参数示例:

```
{
    "appid":"1000258",
    "out_trade_no":"112019xxxxxxxxxxxxxxxxxxx",
    "customs":"HANGZHOU_ZS",
    "mch_customs_no":"31xxxxxxxxxxxx",
    "duty":"2",
    "action_type":"ADD",
    "cert_type":"IDCARD",
    "cert_id":"33xxxxxxxxxxxxxxxxx",
    "name":"张三",
    "sign":"1865230D041C6802A7xxxxxxxxxxxxxxxxx"
}
```

### Response 响应示例

#### 获取成功示例

```
{
    "code":200,
    "message":"success",
    "data":{
        "appid":"1000258",
        "out_trade_no":"13xxxxxxxxxxxxxxx",
        "customs":"SHANGHAI_ZS",
        "mch_customs_no":"31xxxxxxxxx",
        "action_type":"ADD",
        "cert_type":"IDCARD",
        "cert_id":"33xxxxxxxxxxxxxxxxxxx",
        "name":"张三",
        "sign":"C25523D2575FDE713A30xxxxxxxxxxxx",
        "blue_mch_id":"1000258",
        "transaction_id":"4200xxxxxxxxxxxxxxxxxxxxxxxxxx",
        "fee_type":"CNY",
        "order_fee":"2",
        "depart_id":"1000258",
        "channel_id":"1",
        "state":"SUCCESS",
        "create_time":1563518890,
        "id":"75",
        "modify_time":"20190718181913",
        "cert_check_result":"DIFFERENT",
        "verify_department":"OTHERS",
        "verify_department_trade_id":"420000xxxxxxxxxxxxxxxxxx",
        "update_time":1563518891
    }
}

```
#### 返回结果

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array|返回数据集或其他提示信息
appid|appid|是|String
订单编号|sn|否|String|与out\_trade\_no二选一,优先使用sn
商户订单号|out\_trade\_no|否|String|商户订单号
海关|customs|是|String|例如	GUANGZHOU_ZS 广州
海关备案号|mch_customs_no|是|String|例如 3122xxxxxxxx
报关类型|action_type|是|String|ADD 新增报关申请 MODIFY 修改
关税|duty|否|Int|关税，以分为单位，非必填项
证件类型|cert_type|否|String|IDCARD 身份证
证件号码|cert_id|否|String|证件号码
用户姓名|name|否|String|例如 张三
物流费用|transport_fee|否|Int|以分为单位
三方订单号|transaction_id|是|String|微信或者支付宝订单号
币种|fee_type|是|String|微信支付订单支付时使用的币种
订单金额|order_fee|Int|以分为单位
状态码|state|是|String|状态码 UNDECLARED -- 未申报 SUBMITTED -- 申报已提交（订单已经送海关，商户重新申报，并且海关还有修改接口，那么记录的状态会是这个）PROCESSING -- 申报中 SUCCESS -- 申报成功 FAIL-- 申报失败 EXCEPT --海关接口异常
最后更新时间|modify_time|是|String|最后更新时间，格式为yyyyMMddhhmmss 该时间取自微信服务器
订购人和支付人身份信息校验结果|cert_check_result|是|String|NCHECKED 商户未上传订购人身份信息 SAME 商户上传的订购人身份信息与支付人身份信息一致 DIFFERENT 商户上传的订购人身份信息与支付人身份信息不一致
验核机构|verify_department|是|String|验核机构包括： 银联-UNIONPAY 网联-NETSUNION 其他-OTHERS(如余额支付，零钱通支付等)
验核机构交易流水号|verify_department_trade_id|是|String|交易流水号，来自验核机构，如银联记录的交易流水号，供商户报备海关

### 2.13 报关状态查询接口
### api

```
/customs/queryCustoms
```

### Parameters 请求方式
*HTTP POST*

### Parameters 请求参数(json格式，放在http请求body中进行传递)

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String|appid,登录时获取
签名|sign|是|String|
订单编号|sn|否|String|与out\_trade\_no二选一,优先使用sn
商户订单号|out\_trade\_no|否|String|商户订单号
海关|customs|是|String|例如 GUANGZHOU_ZS (详情见列表)
商户子订单号|sub_order_no|否|String|商户子订单号

请求参数示例:

```
{
    "appid":"1000258",
    "out_trade_no":"2019xxxxxxxxxxxxxxxx",
    "customs":"SHANGHAI_ZS",
    "sign":"F5B58D52465AFC5DB291xxxxxxxxxx"
}
```

### Response 响应示例

#### 获取成功示例

```
{
    "code":200,
    "message":"success",
    "data":{
        "depart_id":"1000258",
        "channel_id":"1",
        "out_trade_no":"2019xxxxxxxxxxxx",
        "transaction_id":"42000003xxxxxxxxxxxxxxxxxxxxx",
        "customs":"SHANGHAI_ZS",
        "mch_customs_no":"31xxxxxxxxxx",
        "duty":"0",
        "sub_order_no":"",
        "fee_type":"CNY",
        "order_fee":"2",
        "transport_fee":"0",
        "product_fee":"0",
        "cert_type":"IDCARD",
        "cert_id":"3308xxxxxxxxxxxxx",
        "name":"test",
        "state":"SUCCESS",
        "create_time":"1567654373",
        "update_time":"1567673022",
        "return_code":"SUCCESS",
        "return_msg":"成功",
        "sign":"0962301081A4107CBB28xxxxxxxxxxxxxxxxx",
        "appid":"wxaxxxxxxxxxxxxxxx",
        "mch_id":"150xxxxxxxxxx",
        "result_code":"SUCCESS",
        "err_code":"0",
        "err_code_des":"OK",
        "count":"1",
        "mch_customs_no_0":"312xxxxxxxxx",
        "customs_0":"SHANGHAI_ZS",
        "state_0":"SUCCESS",
        "explanation_0":"0|ok",
        "modify_time_0":"2019xxxxxxxxxx",
        "cert_check_result_0":"DIFFERENT",
        "verify_department_0":"OTHERS",
        "verify_department_trade_id_0":"4200000xxxxxxxxxxxxxxxxxxxxxxxx"
    }
}

```

#### 返回结果

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array|返回数据集或其他提示信息
商户号|depart_id|是|String|蓝海商户号
渠道号|channel_id|是|String|
订单编号|sn|否|String|与out\_trade\_no会其中一个有返回
商户订单号|out_trade_no|否|String|商户订单号
三方订单号|transaction_id|是|String|微信或者支付宝订单号
海关|customs|是|String|例如	GUANGZHOU_ZS 广州
海关备案号|mch_customs_no|是|String|例如 3122xxxxxxx
关税|duty|否|Int|关税，以分为单位
商户子订单号|sub_order_no|否|String|商户子订单号
币种|fee_type|否|String|微信支付订单支付时使用的币种
订单金额|order_fee|Int|以分为单位
物流费用|transport_fee|否|Int|以分为单位
商品价格|product_fee|否|Int|商品费用，以分为单位
证件类型|cert_type|否|String|IDCARD 身份证
证件号码|cert_id|否|String|证件号码
用户姓名|name|否|String|例如 张三
笔数|count|是|Int|笔数 例如 1
商户子订单号|sub_order_no_$n|否|String|商户子订单号
微信子订单号|sub_order_id_$n|否|String|微信子订单号
商户海关备案号|mch_customs_no_$n|否|String|商户在海关登记的备案号
海关|customs_$n|是|String|UANGZHOU 广州 HANGZHOU 杭州 NINGBO 宁波 ZHENGZHOU_BS 郑州（保税物流中心） CHONGQING 重庆 SHANGHAI 上海 ZHENGZHOU_ZH 郑州（综保区）
币种|fee_type_$n|否|String|币种
应付金额|order_fee_$n|否|Int|子单金额，以分为单位
关税|duty_$n|否|Int|关税，以分为单位，非必填项，不会提交给海关
物流费|transport_fee_$n|否|Int|物流费用，以分为单位
商品价格|product_fee_$n|否|Int|商品费用，以分为单位
状态码|state_$n|是|String|状态码 UNDECLARED -- 未申报 SUBMITTED -- 申报已提交（订单已经送海关，商户重新申报，并且海关还有修改接口，那么记录的状态会是这个） PROCESSING -- 申报中 SUCCESS -- 申报成功 FAIL -- 申报失败 EXCEPT --海关接口异常
申报结果说明|explanation_$n|否|String|申报结果说明，如果状态是失败或异常，显示失败原因
最后更新时间|modify_time_$n|是|String|最后更新时间，格式为yyyyMMddhhmmss
订购人和支付人身份信息校验结果|cert_check_result_$n|是|String|UNCHECKED 商户未上传订购人身份信息 SAME 商户上传的订购人身份信息与支付人身份信息一致 DIFFERENT 商户上传的订购人身份信息与支付人身份信息不一致
验核机构|verify_department|是|String|验核机构包括： 银联-UNIONPAY  网联-NETSUNION  其他-OTHERS(如余额支付，零钱通支付等)
验核机构交易流水号|verify_department_trade_id|是|String|交易流水号，来自验核机构，如银联记录的交易流水号，供商户报备海关

PS:$n表示记录的序号，取值为0~($ count -1)，例如count指示返回的退款记录有2条。第一条序号为“0”，第二条序号为“1”。

### 2.14 获取原始数据接口
### api

```
/open/getraw
```

### Parameters 请求方式
*HTTP POST*

### Parameters 请求参数(json格式，放在http请求body中进行传递)

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String|appid,登录时获取
订单编号|sn|否|String|与out\_trade\_no二选一,优先使用sn
商户订单号|out\_trade\_no|否|String|商户订单号
随机字符串|nonce_str|是|String|十六进制字符串，最长32位
签名|sign|是|String|

请求参数示例:

```
{
    "appid":"1000258",
    "out_trade_no":"11xxxxxxxxxxxxxxxxxxx",
    "nonce_str":"faf212hjjdhh5xxxxxxxxxxxxxxxx",
    "sign":"49614D0AEE9489EE7066F71xxxxxxxxxx"
}
```

### Response 响应示例

#### 获取成功示例

```
{
    "code":200,
    "message":"success",
    "data":"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"
}
```
其中data是加密数据，通过AES解密后，得到如下数据

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array|返回数据集或其他提示信息
蓝海商户号|blue_mch_id|是|String
商户订单号|out\_trade\_no|否|String|商户订单号
订单编号|sn|是|String|蓝海订单号
交易ID|transaction_id|是|String|微信订单号，或者支付宝订单号，或者银联订单号
订单类型|channel_type|是|String|订单类型，微信:wechat,支付宝:alipay,银联:unionpay
下单原始数据|request|是|String|原始数据,可能是json,可能是xml
下单返回原始数据|response|是|String|原始数据,可能是json,可能是xml
回调原始数据|notify|是|String|原始数据,可能是json,可能是xml
数据创建时间|created_time|是|String|例如 2019-08-26 18:46:16
数据最后更新时间|updated_time|是|String|例如 2019-08-27 15:55:11


### 2.15 获取支付宝汇率接口
### api

```
/exchangerate/fetch
```

### Parameters 请求方式
*HTTP POST*

### Parameters 请求参数(json格式，放在http请求body中进行传递)

字段|变量名|必填|类型|描述
----|----|----|----|----
appid|appid|是|String|appid,登录时获取
提供方|adapter|是|String|取值为`alipay`(目前只支持alipay)
签名|sign|是|String|参考签名说明

请求参数示例:

```
{
    "appid": "1000258",
    "adapter": "alipay",
    "sign": "3776FB394D85F914B3xxxxxxxxxxxxxx"
}
```

### Response 响应示例
#### 获取成功示例

```json
{
    "code": 200,
    "message": "success",
    "data": {
        "HKD": {
            "releaseDate": "2020-12-21",
            "releaseTime": "10:05:11",
            "currency": "HKD",
            "rate": "0.847370"
        },
        "GBP": {
            "releaseDate": "2020-12-21",
            "releaseTime": "10:05:11",
            "currency": "GBP",
            "rate": "8.790200"
        }
    }
}
```

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array|返回数据集或其他提示信息
更新日期|releaseDate|是|String|更新日期
更新时间|releaseTime|是|String|更新时间
币种|currency|是|String|币种 (例如 港币HKD)
汇率|rate|是|String|汇率(币种兑换的汇率 例如 HKD:CNY=1:0.847370)

PS:根据支付宝的官方文档，汇率值每天更新一次；返回的汇率是指该币种兑换人民币的汇率。

### 2.16 获取微信汇率接口
### api

```
/wechat/exchangerate/query
```

### Parameters 请求方式
*HTTP GET*

### Response 响应示例
#### 获取成功示例

```
{
    "code": 200,
    "message": "success",
    "data": {
        "currency": "HKD",
        "date": "20201221",
        "rate": "84570000"
    }
}
```

字段|变量名|必填|类型|描述
----|----|----|----|----
返回码|code|是|String(32)|返回码，请参考返回码表
返回信息|message|是|String(256)|返回信息，成功信息或错误信息
返回数据|data|否|Array|返回数据集或其他提示信息
币种|currency|是|String|币种 (港币HKD)
更新日期|date|是|String|更新日期
汇率|rate|是|String|汇率(币种兑换的汇率 例如 HKD:CNY=1:0.84570000)

PS:根据微信的官方文档，汇率值每天更新一次；返回是兑换人民币的汇率。

## Update
- By：YUN
- Time:2021.07.09












