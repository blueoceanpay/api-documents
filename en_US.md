# BlueOcean Pay Api Documents

## 1 API Rules

### 1.1 API Protocol

#### Invoking the BlueOcean Pay interface is subject to the following rules:

1. Always use POST for the request method and pass the JSON string of the requested data parameter as the http body

2. The requested data format uses JSON format. If this parameter has a json string, use double quotes (\")

3. string encoding please use UTF-8

4. Signature algorithm MD5
  
 
### 1.2 Parameter signature

1.2.1. Assuming the request parameters are as follows:

```
{
    "appid":"1000010",
    "name":"BlueOcean Pay",
    "region":"HK",
    "business":"Online payment"
}
```

1.2.2. Arrange the parameters in the form of key-value pairs (key=value), sort them according to the ASCII name of the parameter name, and use the &

```
str = "appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK"
```

1.2.3. In the final stitching on the key string `&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb`

```
str += "&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb"
```
str is:

```
appid=1000010&business=Online payment&name=BlueOcean Pay&region=HK&key=sxkj0RH9qMxdaxo0sJ8xlbki4ssOjvXb
```

   
1.2.4. Finally calculate the MD5 value and convert it to uppercase

```
sign = strtoupper(md5(str))
```

Signed value:

```
08C612FB4D2D52C8C913EA00E3DABC8B
```

### 1.3 Request headers

To enable tracking and counting API requests
Need to set User Agent in http headers

#### User Agent Examples

```
# English
BoPayPos/1.1.0 NetType/WIFI Language/en_US

# Simplified Chinese
BoPayPos/1.1.0 NetType/WIFI Language/zh_CN

# Traditional Chinese
BoPayPos/1.1.0 NetType/WIFI Language/zh_HK

```
  
### 1.4 PHP Signature sample code:


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


### 1.5 API endpoint

Use different domain names in different business areas

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

### 1.6 Response Code Description

Response Code | Response Code Description | Logic Processing
-----|--------------|-----------------------------------------------
0 | Common Errors | Uncategorized Errors, Users Are Prompted or Tolerated
200 | Success | Dealing with specific business logic
201 | Swiping card transaction unknown status | system busy, user payment, input password, etc., need to call `/ order / query` query transaction status
404 | Interface address error | check interface address based on documentation
4001 | Request data format error | Please use JSON data format
4002 | Invalid AppId | Checked AppId and the corresponding key is correct
4003 | Missing Parameters | Pass Correct Parameters
4004 | Signature error | Check signature logic and the corresponding key is correct
4005 | Communication Error | Follow the prompts to troubleshoot the code
4006 | Server is busy | Recall the interface later
4007 | Request method error | Please use POST method
40100 | invalid request||
40101 | Order paid |
40102 | Order does not exist |
40103 | Order closed |
40104 | Order cancelled |
40105 | Order refunded |
40106 | Duplicate order number |
40107 | Insufficient balance | Check if the balance of the merchant account is insufficient
40108 | Order over refund period | Can't complete refund
40109 | Missing Parameters | Completing Parameters According to Interface Tips
40110 | Incorrect encoding format | Please use UTF-8 encoding format
40111 |Each QR code can only be used once |Reacquire user payment QR code
40400 | System is busy | The system is busy, try again later
40500 | Payment Error | Prompt User for Corresponding Information
40600 | Unknown Error | Uncategorized Error


### 1.7 Universal Response Data Structure

The response data is in Json format. The Key-Map description is as follows:

Property | Description | Example
------- | ------ | -------
Code | Business Response Codes | 200
Message|Business prompt message, according to the business result, the attribute value can be directly used to prompt the user | Success
Data | Business data, which needs to be processed logically according to the corresponding interface, sometimes empty (no attribute exists) |

Successful example

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


Failed example

```
{
  "code": 40500,
  "message": "Insufficient balance"
}
```




## 2 API List

### 2.1 Payment

The Pos machine submits corresponding request parameters and completes the corresponding payment service according to the usage scenario.

### Api: 

```
/payment/pay
```
### Request parameters

Parameter Name|Field|Required|Data Type|Description
----|----|----|----|----
appid|appid|Yes|String|appid,Get from the management background, or log in to get
Signature|sign|Yes|String
Payment Type|payment|Yes|String|micropay,alipay.qrcode,wechat.qrcode
Order Amount|total_fee|Yes|Int|Payment amount Unit "Cent"
Discount |discount|No|Int|Discount Unit "Cent" default value is "0"
Auth Code|code|No|String|requied when payment equal to "micropay" e.g alipay:288271620985824610  134519771100657507
Merchant's order No.|out\_trade\_no|No|String|If merchants have their own order system, they can generate their own order number, otherwise they are recommended to be automatically generated by BlueOcean Pay.
Notify url|notify_url|No|String|notify url
sub\_blue\_mch\_id|sub\_blue\_mch\_id|No|BlueOceanPay Sub merchant Id|BlueOceanPay Sub merchant Id
store\_id|store\_id|可选|Store Id|Store Id
body|body|No|product name|product name


#### Parameter Description

Parameter Value | Description
----------------|----------------
micropay| micropay At this point need to pass the payment authorization code `code`
alipay.qrcode|Alipay qrcode
wechat.qrcode|Wechat qrcode

After the payment is returned, check the transaction status trade_state, and based on the result, decide whether to invoke the order inquiry interface for result query processing.

### Description  of Order payment status `trade_state`

```
NOTPAY:Pending
SUCCESS:Paid
REFUND:Refund
CLOSED:Closes
REVOKED:Revoked
USERPAYING:User Paying
PAYERROR:Payment Error
```

### Success Response

response.data Description

Parameter Name|Field|Data Type|Description
--------------|-----|---------|------------
id|id|Int|e.g 10357
BlueOcean Pay Serial Number|sn|String|e.g 11201802091347484054542598 Unique Identifier,Can be used for queries, closes, refunds, etc.
Currency|fee_type|String|Currency e.g HKD,AUD,USD
Merchant Name|mch_name|String|e.g "BlueOcean Pay"
Merchant's Order No.|out\_trade\_no|String|Merchant's Order No. e.g "11201802091347484054542598"
Transaction Id |transaction_id|String| P5631VZG299QZN94JD
Payment Provider|provider|String|e.g alipay,wechat
Create time|create_time|Int|Timestamps e.g 1518155270
Pay time|time_end|Int|Successful payment timestamp e.g 1518155297
Trade State|trade_state|String| NOTPAY,PAID
Qrcode text|qrcode|String|qrcode e.g "https://qr.alipay.com/bax03112k12liy7lrysg2004", "weixin://wxpay/bizpayurl?pr=HBXdDeM"
payment amount|total_fee|Int|Unit "Cent" e.g 10
Discount|discount|Int|Unit "Cent" e.g 2
Signature |sign|String| e.g "7FB42F08C85670A86431F97109DE8683"



### Request example

#### Alipay QRcode Example:

Request parameters:

```
{
  "appid": "1000322",
  "payment": "alipay.qrcode",
  "total_fee": 10,
  "wallet": "CN",
  "sign": "520489313B46B5D403CCD8A01267B6C7"
}
```

Response Result:

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

#### Micropay Example 

Request parameters:

```
{
  "appid": "1000322",
  "code": "134602370743606195",
  "payment": "micropay",
  "total_fee": 5,
  "sign": "5D8883E85FB4D721A0CF53BBD0A12905"
}
``` 

Response Result:

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


### 2.2 Refund

When a refund is required due to the buyer or seller within a period of time after the transaction occurs, the seller can refund the payment to the paying user through the refund interface.

The payment provider (Alipay, WeChat, etc.) will receive the refund request and verify the success, and will refund the payment according to the refund rule according to the original payment user account.

#### Api: 

```
/payment/refund
```
#### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|--------------
appid|appid|Yes|String|appid
Signature|sign|Yes|String|
BlueOcean Pay Order Serial No.|sn|No|String|Select one of out\_trade\_no and sn,Use sn 
Merchant's Order No.|out\_trade\_no|No|String|Merchant's Order No.
Refund Amount|refund_fee|No|Int|Optional parameter, the default is the total order amount, which means full refund Alipay does not support partial refund
Refund Description|refund_desc|No|String|Refund Description
Password|password|No|String|Refund password




#### Response

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


### 2.3 Oder Operations

Order status operation (query, close, undo) using similar parameters

### Api: 

```
Order Query
/order/query

Order Close
/order/close

Order Reverse
/order/reverse
```
### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|-----------
Appid|appid| Yes |String|appid, obtained at login
Signature|sign| Yes ||
BlueOceanPay Serial No. |sn|No|String| and out\_trade\_no choose one, use sn preferentially
Merchant's Order No. |out\_trade\_no|No|String|Merchant's Order No.

Request Examples:

Order Query

```
{
  "appid": "1000322",
  "sn": "11201802071854269363947431",
  "sign": "9CC7EB3EBE33D11421FE63298F2EEB94"
}

```

Response Result:

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


### 2.4 Order List


### Api: 

```
/order/list
```
### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|-----------
appid|appid|Yes|String|
Signature |sign|Yes|String|
Payment Provider|provider|No|String|e.g alipay,wechat
Pagination|page|No|Int|Pagination parameter default:1
Page Limit|limit|No|Int|Pagination Limit default:10
Store Id|store_id|No|Int|Store Id
Trade State|trade_state|No|String|Trade State e.g SUCCESS,REFUND
Start Time|start_time|No|String|Find orders by creation time e.g 2017-12-12
End Time|end_time|No|String|Find orders by creation time e.g 2017-12-13
sub\_blue\_mch\_id|sub\_blue\_mch\_id|No|BlueOceanPay Sub merchant Id|BlueOceanPay Sub merchant Id

#### Response Data


Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|-----------
Code|code|Yes|String|Return Code
message|message|Yes|String|Return message
data|data|No|Array| data

#### Request parameters

```
{
    "appid": "1000322",
    "limit": 10,
    "page": 1,
    "store_id": 1000321,
    "sign": "55F40D7150AD3013E4AB479745FAE542"
}
```

### Response Result:

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


### 2.5 Refund List

api：

```
/refund/list
```

#### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|------------
AppId|appid|Yes|String|
Signature |sign|Yes|String|
Pagination|page|No|Int|Pagination parameter default:1
Page Limit|limit|No|Int|Pagination Limit default:10
Payment Provider|provider|No|String|alipay,wechat
Refund Status|refund_status|No|String|Refund Status e.g SUCCESS
Start Time|start_time|No|String|Query by refunded time e.g 2017-12-12
End Time|end_time|No|String|Query by refunded time e.g 2017-12-13
sub\_blue\_mch\_id|sub\_blue\_mch\_id|No|BlueOceanPay Sub merchant Id|BlueOceanPay Sub merchant Id


#### trade_state
```
SUCCESS: Refund Success
PROCESSING:Refund Processing
REFUNDCLOSE:Refund Closed
CHANGE:Refund Error
```

#### Response Result

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|------------
Code|code|Yes|String|
Message|message|Yes|String|message
Data|data|No|Array|


Request parameters:

```
{
  "appid": "1000322",
  "limit": 2,
  "page": 1,
  "sign": "74CD2AAB31B3FB76A68FEBF8FC56AF70"
}
```

Response Result:

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


### 2.6 User Login

### api

```
/user/login
```

### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|------------
email|email|Yes|String|Login Account
password|password|Yes|String| Login Password

#### Use POST and pass the json string of the requested data parameter as http body

#### This method requires no signature and can be tested in the chrome browser plug-in Advanced REST client

Request parameters Example:

```
{"email":"abc@blueoceanpay.com","password":"123456"}
```

The account does not exist. In actual use, please obtain it from the merchant's backend website.

### Response Examples

#### Successful Example

```
{
  "code": 200,
  "message": "success",
  "data": {
    "appid": 100018,
    "app_key": "TVqec7ZcuG59HepqjQbTi68S78sdLvUh",
    "name": "BlueOcean Pay",
    "store_id": 0,
    "store_name": ""
  }
}

```

#### Failure Example

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

### 2.7 Add User

### api

```
/user/create
```

### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|------------
appid|appid|Yes|String|appid
Signature |sign|Yes|String|
User Email|email|Yes|String|Login Account
User Password|password|Yes|String|Login Password
User Type|type|Yes|String| merchant,store,cashier

Request parameters:

```
{
  "appid": "100002",
  "email": "test@blueoceanpay.com",
  "password":"123456",
  "type": "store",  
  "sign": "C770EDDEC3F173CE69B5A46D7A009A59"
}
```


### Response Result Examples

#### Successful Example

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

#### Failure Example

```
{
  "code": 0,
  "message": "User Exists"
}
```

### 2.8 Setting Password

### api

```
/user/password
```

### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|------------
appid|appid|Yes|String|appid
Signature |sign|Yes|String|
User email|email|Yes|String|User Email
Old Password|old|Yes|String|Old Password
New Password|new|Yes|String|New Password
Repeat Password|repeat|Yes|String

Request parameters:

```
{
  "old": "123456",
  "new": "654321",
  "repeat": "654321",
  "sign": "355954524EA6FF44D8582DC50BECF85F"
}
```


### Response Result Example

#### Suceessful Example

```
{
  "code": 200,
  "message": "success"
}

```

#### Failure Example

```
{
  "code": 0,
  "message": "Two passwords are inconsistent"
}
```


### 2.9 Remove User

### api

```
/user/remove
```

### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|------------
appid|appid|Yes|String|appid
Signature|sign|Yes|String|
User Id to be removed|user_id|No|Int| Optional parameters If you do not pass, delete the user corresponding to appid

Request parameters:

```
{
  "appid":"1000050",
  "user_id":"1000051",
  "sign":"BCA64F4B0A34753497C9DC8943DDFCC9"
}
```

### Response Example

#### Successful Example

```
{
  "code": 200,
  "message": "success"
}

```

#### Failure Example

```
{
  "code": 0,
  "message": "User Removed"
}
```

### 2.10 User List
### api

```
/user/list
```

### Request parameters

Parameter Name|Field|Required|Data Type|Description
--------------|-----|--------|---------|------------
appid|appid|Yes|String|appid
Signature |sign|Yes|String|
Pagination|page|No|Int|Pagination parameter default:1
Page Limit|limit|No|Int|Pagination Limit default:10
User type|type|No|String|default: cashier


Request parameters:

```
{
  "appid":"83",
  "sign":"BCA64F4B0A34753497C9DC8943DDFAED"
}
```

#### Successful Example 

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

2018.03.21












