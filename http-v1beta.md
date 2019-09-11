Hermes 测试环境接口

API endpoint: https://hermes.ocx.im

# 目前支持的币种

1. ETH
2. ERC20 代币
3. BTC

# 通过官方途径获取

1. appkey:  用于 HTTP API 认证

2. appsecret:  用于 HTTP API 签名

# 认证与签名

Hermes 认证基于 OAuth2 bearer token 认证，签名使用 HMAC sha256，即发起 HTTP 请求前需要设置 header 

1. X-Hermes-Key: `<appkey>`

2. X-Hermes-Signature: 参考下述签名步骤得到的 hash 值

## 签名步骤

1. 签名 payload 是对 HTTP verb, HTTP path 和 HTTP params 进行组装, 如

```
payload = "POST|/v1beta/addresses|currency_code=eth&sn=user_id"
```

注意 HTTP params 由 HTTP URL queries 和 HTTP Body 请求参数组成键值对, 对其中键进行排序并用 `&` 连接


2. 对上述组装后的字符串使用 `HMAC-SHA256` 加密算法进行 `hash` 计算:

```
hash = HMAC-SHA256(payload, appsecret).to_hex
```

# API 列表

1. `POST /v1beta/addresses` 生成特定币种的地址

```json
# Request

{
  "currency_code": "eth",
  "sn": "user id",
}

# Response
{
  "data": {
    "currency_code": "eth",
    "address": "0x84Eaeddd4952c22fE69222f8382200F3048bE24C"
  }
}
```
| Name          | Type      |  Explain       |
| ------------- | --------- | -------------- |
| currency_code |  String   |                |
| address       |  String   |                |


2. `GET /v1beta/accounts/{currency_code}`  获取特定币种的余额

```json
# Response
{
  "data":{
    "currency_code": "eth",
    "balance": 99.5,
    "locked": 0.5
   }
}
```

| Name          | Type      |  Explain       |
| ------------- | --------- | -------------- |
| currency_code |  String   |                |
| balance       |  Double   |  币种余额                      |
| locked        |  Double   |  币种冻结数量，但用于提现时会先冻结 |



3. `POST /v1beta/withdraws` 申请提现

```json
# Request

{
  "currency_code": "eth",
  "amount": 1.0,
  "address": "valid currency address to which your withdrawal wants to send",
  "external_uuid": "uuid on merchant side",
}

# Response

{
  "data": {
    "currency_code": "eth",
    "txid": "blockchain txid",
    "state": "withdrawing",    // withdrawing|sent
    "from": "0x0001",
    "to": "0x0002",
    "value": 1.0,
    "memo": "withdrawal memo",
    "confirmations": 1,
    "external_uuid": "" 
  }
}
```

| Name          | Type      |  Explain       |
| ------------- | --------- | -------------- |
| currency_code |  String   |                |
| txid          |  String   |  区块链真实交易号 |
| state         |  String   |  状态, 有 submitted, withdrawing, sent |
| from          |  String   |  提现发出地址    |
| to            |  String   |  提现接收地址    |
| memo          |  String   |  提现备注       |
| confirmations |  Integer  |  提现当前确认数  |
| external_uuid |  String   |  提现对应商户系统的 ID 值 |



# 提供回调接口

Hermes 通过商户提供的回调接口，将充值信息或者提现状态反馈给商户，为了保证安全。回调同样采用上述的认证签名机制, 即发送交易会在 HTTP header 传入


1. X-Hermes-Key: `<appkey>`

2. X-Hermes-Signature: 同理, `hash = HMAC-SHA256(payload, appsecret).to_hex` 结果

3. 回调采用 POST 进行调用


## 充值回调

`POST https://merchant-server.com/webhook`


```json
# Request
{
  "type": "Deposit",
   "currency_code": "eth",
   "txid": "0xethereumtxid",
   "address": "0xfromaddress",
   "state": "depositing",  // depositing|done
   "amount": 1.0,
   "memo": "blockchain memo"
}
```


## 提现回调

`POST https://merchant-server.com/webhook`

```json
# Request
{
  "type": "Withdraw",
  "currency_code": "eth",
  "txid": "0xethereumtxid",
  "state": "withdrawing",  // withdrawing|sent
  "external_uuid": "merchant withdraw uuid"
}
```








