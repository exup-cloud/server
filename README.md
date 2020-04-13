
# 合约云接入文档

tags: 永续合约, 合约云


**文档版本: 1.12**

---
## 1. 接入流程说明

### 1.1 整体流程
1. 签订合同，确定技术对接开始时间
2. 商务人员拉群，技术人员入场
3. 技术对接（服务端+前端）
4. 在合约云测试环境下进行系统测试
5. 测试通过，系统正式上线
6. 上线后系统运维和数据统计

### 1.2 技术对接流程
1. 商户在tigermex测试官网注册tigermex账户,作为商户在合约云下的母账户
    (正式：https://bitwind.com/  测试： https://mybts.info/)    
2. 商户提供以下信息：
    - 商户在mybts测试环境的官网注册的mybts用户信息：注册名、UID
    - 服务器间通信中，商户数据的签名公钥（测试用）
3. 合约云人员在后台将商户注册的mybts账户开通交易所身份，并提供测试币
4. 商户在mybts网站上操作资金划转，将测试币资金转到自己的合约账户中，作为商户保证金
5. 合约云提供：
    - 服务器间通信中，平台数据的签名公钥（测试用）
    - 签名参数：加密串api_salt（测试用）
6. 商户根据接口格式调用合约云测试环境的API
7. 商户根据自身需要, 提供接收合约云通知调用的URL(通知包括:爆仓、强平、计划委托和ADL)

### 1.3 系统正式上线流程
1. 商户的系统开发和对接完成并测试通过（商户的用户在商户的交易所上从登录->资金划转->合约交易->余额信息变化整个流程都完整无误)
2. 商户在bitwind,作为商户在合约云下的母账户
3. 商户账户在bitwind,作为商户保证金
4. 商户向合约云提出正式上线申请，并提供信息包括：
    - 商户在bitwind正式环境的官网注册的tigermex用户信息：注册名、UID
    - 服务器间通信中，商户数据的签名公钥（正式）
    - 商户根据自身需要,提供接收合约云通知调用的URL(通知包括:爆仓、强平、计划委托和ADL)
5. 合约云提供：
    - 服务器间通信中，平台数据的签名公钥（正式）
    - 签名参数：加密串api_salt（正式）
6. 商户的服务器将调用API接口从测试地址切到正式地址


## 2. 服务端接入说明

### 2.1 交互流程图
![image](https://github.com/exup-cloud/server/blob/master/images/flowchart.png)



### 2.2 接口调用规范说明

- 协议: HTTPS
- 方法: 仅支持POST
- 字符编码: UTF-8
- 接口地址:
    - 合约云券商接口
        - 生产：
        - 测试：`http://swapcloud.mybts.info/gateway`
    - 交易接口地址
        合约云提供一个`cname`地址，商家可以将自己子域名解析到合约云提供的`cname`地址上面，测试示例提供的地址商家可以在没有指定域名解析时用来测试
        - 生产：
        - 测试：
        - 测试示例：
- HTTP头信息: `Content-Type: application/x-www-form-urlencoded`

### 2.3 POST请求格式

请求参数是通过POST方式发送, 请求参数包括: 公共参数 + 接口参数

公共参数如下:

|参数名      |类型  |必填 |说明                                                    |示例         |
|---        |---   |--- |---                                                     |---         |
|method     |string|Y   |接口名称                                                 |            |
|app_id     |string|Y   |接入交易所ID                                             |100001383729|
|nonce      |string|Y   |随机字符串配合timestamp防止重放攻击                         |            |
|timestamp  |string|Y   |时间戳(单位秒)确保调用方时间准确,服务端通过校验时间戳防止重放攻击|1542084086  |
|version    |string|Y   |版本号                                                   |固定填v1     |
|signature  |string|Y   |数据签名                                                 |            |

- 说明: 以上是所有接口调用的公共参数,不同的接口还有不同的额外参数,详情见下文接口说明

- 示例：

```
POST https://swapcloud.mybts.info/gateway
Accept: application/json
Content-Type: application/x-www-form-urlencoded

method=account.create&app_id=2017184040&nonce=24546&timestamp=1544897149&version=v1&signature={sign}&[接口参数1&接口参数2&...]

```

### 2.4 返回数据格式

- HTTP header

|  参数名   |  类型  | 必填 |                             说明                             |    示例    |
| -------- | ------ | ---- | ----------------------------------------------------------- | ---------- |
| Ex-Ts    | string | Y    | 时间戳(单位秒),调用方可通过校验时间戳防止重放攻击                | 1542084086 |
| Ex-Nonce | string | Y    | 随机字符串(不超过32字节),调用方可通过其配合timestamp防止重放攻击 | 123456     |
| Ex-Sign  | string | Y    | 数据签名,调用方请务必对sign做校验                              |            |


- Response Body
  
```
{
    "errno": "OK",           // 错误号(成功为OK)
    "message": "Success",    // 错误内容(成功为Success)
    "id": "",                // 请求ID(唯一标识本次请求)
    "data": {
        ...                  // 返回数据
    }
}
```


 
## 3. 合约云接口数据签名

### 3.1 请求数据的签名

1. 对所有请求参数（包括公共参数和接口参数，但不包括 signature 参数），按照`参数名`ASCII码表升序顺序排序。
如：foo=a1， bar=b2， foobar=c3， baz=d4 排序后的顺序是 bar=b2， baz=d4， foo=a1， foobar=c3 。
2. 将排序好的参数,按照参数值拼接构造成字符串，格式为：value1+value2+… 。
根据上面的示例得到的构造结果为：`b2d4a1c3` 。
3. 对构造结果进行SHA256签名(PKCS1填充)，然后对签名结果做base64encode即得到signature
  
### 3.2 返回数据的签名验证

1. 获得返回body(未反序列化)原始字符串,将其与返回的HTTP头的公共参数值按固定顺序(body+tigermex-Nonce+tigermex-Ts)构造待验签串 
    
    如：body={"errno": "OK","msg": "Success"}，tigermex-Ts=1542084086，tigermex-Nonce=123456。
    
    构造结果为`{"errno": "OK","msg": "Success"}1542084086123456`
    
2. 对构造字符串进行SHA256签名(PKCS1填充)，跟返回数据的HTTPS头中参数tigermex-Sign比对，以及tigermex公钥进行验签


### 3.3 合约云平台的签名公钥

- 测试环境

```
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCgnk5B2k9GQxRJJGPCbdU3yuRT
I6Yy4Mtb9dP4acYzkV2tvq3YFwaCoQKrLiMkfQBsglPRrSD/SGabiGrw6KbZ/xFs
jPDU4hlc6lCUbHmui+/qk0wiiOst7BQXmA8pGJVg+qlG8vlixnCxOtg+XoLB4r/p
hdtfynOwVs297s1InwIDAQAB
-----END PUBLIC KEY-----
```

- 正式环境

```
```


## 4. 接口列表

### 4.1 创建子账号

- 说明

    接入方通过调用此接口创建一个子账号(将自己的用户映射到tigermex), 并且为子账号生成一个api_key

- 请求参数

|参数名            |类型  |必填|说明|
|---              |---   |---|---|
|method           |string|Y|account.create|
|origin_uid       |string|Y|子账号在接入方的用户ID,同一个接入方origin_uid确保是唯一的|
|api_key_life_span |int64 |N|api_key的有效时长(单位秒)，如果不传，默认有效时长为30天|


- 返回数据(json)

```
{
    "account_id":0,             // int64    子账户用户ID(合约云)
    "app_id": 0,                // int64    母账户用户ID(合约云)
    "origin_uid":"",            // string   子用户ID(回传接入方的)
    "status":0,                 // int64    用户状态(1-可用，2-冻结)
    "api_key":"",               // string   API Key(用户通过API Key参与合约交易)
    "api_secret":""             // string   API 密钥
    "api_key_expired_at":"",    // string   API Key过期时间
    "created_at": "",           // string   创建时间
    "updated_at": ""            // string   更新时间
}
```


### 4.2 冻结子账号

- 说明

    接入端可以通过调用此接口解冻子账号，冻结之后，子账号无法参与合约交易

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.freeze|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|

`origin_uid`和`account_id`二者必须填其一

- 返回数据(json)

```
// 根据公共返回参数判断成功/失败
```


### 4.3 解冻子账号

- 说明

    接入端可以通过调用此接口解冻子账号

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.unfreeze|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|

`origin_uid`和`account_id`二者必须填其一


- 返回数据(json)

```
// 根据公共返回参数判断成功/失败
```


### 4.4 更换子账号api_key

- 说明

    更换子账号的api_key

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.api_key.update|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|
|api_key_life_span|int64|N|api_key的有效时长(单位秒)，如果不传，默认有效时长为30天|

`origin_uid`和`account_id`二者必须填其一


- 返回数据(json)

```
{
    "account_id":0,             // int64    用户ID(合约云)
    "origin_uid":"",            // string   接入方用户ID(回传)
    "uid":0,                    // int64    用户ID(合约云)
    "status":0,                 // int64    用户状态(1-可用，2-冻结)
    "api_key":"",               // string   API Key(用户通过API Key参与合约交易)
    "api_secret":""             // string   API 密钥
    "api_key_expired_at":"",    // string   API Key过期时间
    "created_at": "",           // string   创建时间
    "updated_at": ""            // string   更新时间
}
```


### 4.5 查询子账号api_key

- 说明

    查询子账号的api_key

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.api_key.query|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|

`origin_uid`和`account_id`二者必须填其一


- 返回数据(json)

```
{
    "account_id":0,             // int64    用户ID(合约云)
    "origin_uid":"",            // string   接入方用户ID(回传)
    "status":0,                 // int64    用户状态(1-可用，2-冻结)
    "api_key":"",               // string   API Key(用户通过API Key参与合约交易)
    "api_secret":""             // string   API 密钥
    "api_key_expired_at":"",    // int64    API Key过期时间
    "created_at": "",           // string   创建时间
    "updated_at": ""            // string   更新时间
}
```


### 4.6 划入资金到子账号

- 说明

    转入资金到子账号资产账户当中。

    一般流程是：

    1. 用户在交易所前端界面做划转操作，将自己币币账户（或现货账户）的资金划转到用户自己合约账户中
    2. 交易所收到请求后，在自己服务端记录这条划转记录，记录的交易号`out_trade_no`自己指定，不可重复
    3. 交易所服务端调用合约云该接口，实现将保证金从母账户到该子账户的划转
    4. 合约云平台会同时更新母账户余额和该子账户的余额

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.asset.transfer|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|
|coin_code|string|Y|划转币种|
|vol|string|Y|划入金额|
|out_trade_no|string|Y|第三方划转交易号|

`origin_uid`和`account_id`二者必须填其一

- 返回数据(json)

```
// 根据公共返回参数判断成功/失败
```


### 4.7 从子账号划出资金
- 说明

   从子账号转出资金

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.asset.transferout|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|
|coin_code|string|Y|划转币种|
|vol|string|Y|划入金额|
|out_trade_no|string|Y|第三方划转交易号|

`origin_uid`和`account_id`二者必须填其一

- 返回数据(json)

```
// 根据公共返回参数判断成功/失败
```


### 4.8 查询划转交易是否存在
- 说明

    通过第三方划转交易号查询划转交易是否存在

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.tradeno.query|
|out_trade_no|string|Y|第三方划转交易号|

- 返回数据(json)

```
// 根据公共返回参数判断成功/失败
```


### 4.9 查询母账号或子账号资产信息
- 说明

    查询母账号或子账号的资产信息

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.asset.query|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|
|coin_code|string|N|查询币种，不传默认查询所有|

`origin_uid`和`account_id`二者填其一, 如果二者都不填时，表示查询母账号的资产情况


- 返回数据(json)

```
{
    "account_id":0,         // int64    用户ID(合约云)
    "origin_uid":"",        // string   接入方用户ID(回传)
    "status":0,             // int64    用户状态
    "assets":[
        {
            "account_id": 14367463,
            "coin_code": "BTC",         // 币种
            "available_vol": "",        // 可用金额
            "cash_vol": "",             // 净现金余额
            "freeze_vol": "0",          // 冻结金额
            "realised_vol": "0",        // 已实现盈亏
            "earnings_vol": "0",        // 已结算收益
            "created_at": "2018-11-22T20:11:51.770168+08:00",
            "updated_at": "2018-11-23T16:54:15.192931+08:00"
        }
    ]
}
```


### 4.10 查询子账号订单列表
- 说明

    查询子账号的交易订单的列表

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.orders.query|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|
|contract_id|int64|Y|合约ID|
|status|int|Y|订单状态筛选，0: All; 1: 审批中; 2: 委托中; 4: 结束|
|offset|int|Y|查询位置偏移量|
|size|int|Y|查询订单数量|

`origin_uid`和`account_id`二者必须填其一

- 返回数据(json)

```
{
    "orders": [
        {
            "oid": 10539098,        // 订单id
            "company_id": 1000001,  // 商家ID(合约云客户端，company_id=app_id)
            "instrument_id": 1,     // 合约id
            "pid": 10539088,        // 平仓为id
            "uid": 10,              // 用户id
            "px": "16",             // 订单价格
            "qty": "1",             // 订单总量
            "hide_qty": "0",        // 订单隐藏量
            "avg_px": "16",         // 成交均价
            "cum_qty": "1",         // 成交量
            "side": 3,              // 订单方向
            "category": 1,          // 订单类型
            "make_fee": "0.00025",  // make fee
            "take_fee": "0.012",    // take fee
            "origin": "",           // 来源
            "status": 4,            // 状态
            "errno": 0,             // 订单结束原因
            "position_type": 2,     // 1:逐仓,2:全仓
            "time_in_force": 1,     // 1:普通,2:FOK,3:IOC
            "mfr": "0.00025",       // 系统的make手续费率
            "tfr": "0.00075",       // 系统的take手续费率
            "self_mfr": "0.00025",  // 用户自己的make手续费率,如果没有返回该字段,或者值为0,表示用户的手续费率以系统的配置为准
            "self_tfr": "0.00075",  // 用户自己的take手续费率,如果没有返回该字段,或者值为0,表示用户的手续费率以系统的配置为准
            "leverage": "10",       // 订单的杠杆大小,
            "client_id": 123123,    // 前端自定义id
            "created_at": "2018-07-23T11:55:56.715305Z",
            "finished_at": "2018-07-23T11:55:56.763941Z",
        }
    ]
}
```

- category 订单类型

该字段采用二进制按位表示法

|二进制位|含义|
|---    |---|
|0~5位  |订单的基本类型, 1:限价委托; 2:市价委托; 7:被动委托|
|第6位   |预留|
|第7位   |为1表示:强平委托单|
|第8位   |为1表示:爆仓委托单|
|第9位   |为1表示:自动减仓委托单|

- status 订单状态

|状态值|状态类型|
|---   |---   |
|1|申报中|
|2|委托中,表示订单已经进撮合队列,订单信息中的done_vol表示订单成交部分,只要done_vol不为0就表示订单有成交.|
|4|完成|

- errno 订单结束的原因

|errno|结束原因|
|---  |---|
|1|用户取消|
|2|超时,(暂时没有用)|
|3|用户资产不够,转撤销|
|4|用户冻结资产不够|
|5|系统部分转撤销|
|6|部分平仓导致的部分转撤销|
|7|自动减仓导致的部分转撤销|
|8|盈利补偿导致的部分转撤销(暂时没有用)|
|9|仓位错误导致的部分转撤销|
|10|类型非法|
|11|反方向订单存在|


### 4.11 查询子账号仓位信息列表
- 说明

    查询子账号的仓位信息列表

- 请求参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|account.positions.query|
|origin_uid|string|N|接入方的用户ID|
|account_id|int64|N|用户ID(合约云)|
|contract_id|int64|N|合约ID, 为空或为0表示所有合约|
|status|int|Y|状态筛选，//0: All; 1: 持仓中; 2: 系统代持中; 4: 已平仓|
|offset|int|Y|查询位置偏移量|
|size|int|Y|查询仓位数量|
|coin_code|string|N|筛选币种，为空表示所有币种|

`origin_uid`和`account_id`二者必须填其一

- 返回数据(json)

```
{
    "positions": [
        {
            "pid": 10116365,         // 仓位ID
            "company_id": 1000001,   // 商家ID(合约云客户端，company_id=app_id)
            "uid": 10,               // 用户ID
            "instrument_id": 1,      // 合约ID
            "cur_qty": "10",         // 当前持仓量
            "freeze_qty": "0",       // 冻结仓位
            "close_qty": "0",        // 已平仓位
            "avg_cost_px": "16",     // 持仓均价(当前持仓)
            "avg_open_px": "16",     // 开仓均价(包括已平仓位)
            "avg_close_px": "0",     // 已平仓均价
            "oim": "100.075",        // 原始开仓保证金
            "im": "100.075",         // 开仓保证金
            "mm": "50",              // 维持保证金
            "realised_pnl": "-0.075",// 已实现盈亏
            "earnings": "-0.075",    // 已结算收益
            "tax": "0",              // 持仓产生的资金费用
            "position_type": 1,      // 持仓类型(1-逐仓，2-全仓)
            "side": 2,               // 仓位方向(1-多仓，2-空仓)
            "status": 1,             // 状态(1-持仓中，2-系统托管，3-已平仓)
            "errno": 0,              // 平仓原因(见下表)
            "created_at": "2018-07-17T03:04:26.108983Z",
            "updated_at": "2018-07-17T03:04:26.098404Z"
        }
    ]
}
```

- errno 平仓原因:

|errno|平仓原因|
|---|---|
|1|平仓委托中|
|2|破产委托中|
|3|平仓委托结束|
|4|破产委托结束|
|5|爆仓|
|6|自动减仓(主动发起方)|
|7|自动减仓(被动接收方)|


### 4.12 按日期查询总交易量
- 说明

    通过日期获取券商下面所有子账户的交易数据统计

- 请求参数

    |参数名|类型|必填|说明|
    |---|---|---|---|
    |method|string|Y|app.trade_vols.query|
    |contract_id|int64|Y|合约ID,不能为空|
    |start|time|Y|开始时间(RFC3339标准)|
    |end|time|Y|结束时间(RFC3339标准)|

- 返回数据
```
[
    {
      "date": "2019-03-10",     // 日期
      "app_id": 1001,           // app_id
      "contract_id": 1,         // 合约ID
      "reward_fee": "0",        // 手续费分成(暂时没有统计,值为0)
      "total": {
        "date": "2019-03-09T00:00:00Z", // 日期(不用关注)
        "type": 2,                      // 交易量类型(值固定为2)
        "name": "BTCUSDT",              // 合约名称
        "coin_code": "USDT",            // 保证金币种
        "account": 2,                   // 交易子账户数(包括Maker/Taker)
        "count": 4,                     // 交易总次数(包括Maker/Taker)
        "vol": "10678",                 // 交易总量(包括Maker/Taker)
        "amount": "67066905.2",         // 交易额(包括Maker/Taker)
        "fee_coin_code": "USDT",        // 手续费币种
        "fee": "6.70669052"             // 手续费总额
      },
      "taker": {
        "date": "2019-03-09T00:00:00Z",
        "type": 2,
        "name": "BTCUSDT",
        "coin_code": "USDT",
        "account": 1,                   // 作为Taker参与交易的子账总数
        "count": 2,                     // 作为Taker参与交易的交易笔数
        "vol": "10676",                 // 作为Taker参与交易的交易量
        "amount": "67054366.2",         // 作为Taker参与交易的交易额
        "fee_coin_code": "USDT",         
        "fee": "6.70543662"             // 手作为Taker参与交易产生的手续费总额
      },
      "maker": {
        "date": "2019-03-09T00:00:00Z",
        "type": 2,
        "name": "BTCUSDT",
        "coin_code": "USDT",
        "account": 1,                   // 作为Maker参与交易的子账总数
        "count": 2,                     // 作为Maker参与交易的交易笔数
        "vol": "10676",                 // 作为Maker参与交易的交易量
        "amount": "67054366.2",         // 作为Maker参与交易的交易额
        "fee_coin_code": "USDT",         
        "fee": "6.70543662"             // 手作为Maker参与交易产生的手续费总额
      }
    }
]
```

- 注意
1. 合约云按照新加坡时间以天为单位统计券商每日的交易额。当日交易额两小时更新一次，并非实时数据
2. 请求参数start/end请按照RFC3339标准传递，服务端以start/end在新加坡(东八区)时区日期作为起始和截至日期查询交易量统计数据


### 4.13 按日期查询子账户交易量

- 说明

    通过日期查询单个子账户的交易量统计

- 请求参数

    |参数名|类型|必填|说明|
    |---|---|---|---|
    |method|string|Y|account.trade_vols.query|
    |contract_id|int64|Y|合约ID,不能为空|
    |origin_uid|string|N|接入方的用户ID|
    |account_id|int64|N|用户ID(合约云)account_id/origin_uid必须传递一个|
    |start|time|Y|开始时间(RFC3339标准)|
    |end|time|Y|结束时间(RFC3339标准)|

- 返回数据
```
[
    {
        "date": "2019-03-10",               // 日期
        "app_id": 1001,                     // app_id
        "contract_id": 1,                   // 合约ID
        "total": {
          "date": "0001-01-01T00:00:00Z",   
          "type": 2,
          "name": "BTCUSDT",
          "coin_code": "USDT",
          "account": 1,                     // 固定为1
          "count": 1,                       // 交易次数
          "vol": "1",                       // 交易总量
          "amount": "6277.7",               // 交易额
          "fee_coin_code": "USDT",          // 手续费币种
          "fee": "0.00062777"               // 产生手续费总额
        },
        "taker": {
          "date": "0001-01-01T00:00:00Z",
          "type": 2,
          "name": "BTCUSDT",
          "coin_code": "USDT",
          "account": 0,                     // Taker账户数(为0表示没有做Taker参与过交易)
          "count": 0,                       // Taker交易次数
          "vol": "0",                       // Taker交易量
          "amount": "0",                    // Taker交易额
          "fee_coin_code": "USDT",
          "fee": "0"
        },
        "maker": {
          "date": "0001-01-01T00:00:00Z",
          "type": 2,
          "name": "BTCUSDT",
          "coin_code": "USDT",
          "account": 1,
          "count": 1,
          "vol": "1",
          "amount": "6277.7",
          "fee_coin_code": "USDT",
          "fee": "0.00062777"
        }
    }
]
```


### 4.14 查询成交记录

- 说明

    通过时间查询单个子账户的交易量统计

- 请求参数

    |参数名|类型|必填|说明|
    |---|---|---|---|
    |method|string|Y|app.trades.query|
    |contract_id|int64|Y|合约ID,不能为空|
    |account_ids|[]int64|N|用户ID列表,如果为空表示所有用户|
    |start|time|N|开始时间(RFC3339标准)|
    |end|time|N|结束时间(RFC3339标准)|
    |limit|int|N|默认100,最大1000|
    |offset|int|N|默认0|

- 返回数据

```
 {
    "total": 4,   // 记录总数
    "trades": [
        {
            "trade_id": 2095843277,                     // TradeID
            "contract_id": 4,                           // 合约ID
            "sell_account_id": 2090204311,              // 卖家ID
            "buy_account_id": 2000124792,               // 买家ID
            "sell_order_id": 2095843276,                // 卖家OrderID
            "buy_order_id": 2082821054,                 // 买家OrderID
            "deal_price": "5246",                       // 成交价格
            "deal_vol": "1000",                         // 成交量
            "make_fee": "0.000019062142584827",         // Maker手续费
            "take_fee": "0.000038124285169653",         // Taker手续费
            "created_at": "2018-12-10T12:39:01.00475Z", // 成交时间
            "way": 5,                                   // Trade成交类型(见下方说明)
            "type": 0                                   // Trade类型(见下方说明)
        },
    ]
}
```

- 说明
  - way
  |值|说明|
  |-|-|
  |1|开多买 开空卖|
  |2|开多买 平多卖|
  |3|平空买 开空卖|
  |4|平空买 平多卖|
  |5|开空卖 开多买|
  |6|开空卖 平空买|
  |7|平多卖 开多买|
  |8|平多卖 平空买|
  - type
  |值|说明|
  |-|-|
  |0|普通交易记录|
  |1|部分强平的交易记录|
  |2|破产的交易记录  |
  |4|自动减仓的交易记录|
  - 获取Trade的Taker用户ID
  ```
  // 伪代码
  if way >= 1 && way <= 4 {
      taker = buy_account_id
  } else {
      taker = sell_account_id
  }
  ```
  - 手续费
  ```
  take_fee  // taker付出的手续费
  make_fee  // maker获得的手续费
  系统获得的手续费为 take_fee-make_fee, trade中只有taker是券商子账户，手续费才会参与分成
  ```
  

## 5. 合约云通知接口 

### 5.1 通知接口说明

用户在进行合约交易的过程中会出现各种事件, 例如:爆仓, 强平, ADL, 计划委托等.

合约云平台会将这类事件通知给交易所的服务器, 交易所收到通知调用后, 可以根据自身需求通过短信或邮件等形式告知用户.

如果交易所愿意接收通知,需在接入时提供接口通知调用的URL地址.

### 5.2 通知接口格式

合约云平台通知商户(交易所)的调用格式与商户调用合约云平台的接口格式`完全相同`!

所以,`调用方式`和`签名方式`,请参考上文的接入说明和格式规范说明.

出于性能考虑,现不支持通知失败重发机制,请确保自身服务器的可用性和可连通性.

- 通知接口参数

|参数名|类型|必填|说明|
|---|---|---|---|
|method|string|Y|notify|
|origin_uid|string|Y|接入方的用户ID|
|account_id|int64|Y|用户ID(合约云)|
|notify_type|int|Y|通知类型|
|contract_id|int64|N|合约ID|
|position_id|int64|N|用户仓位ID|
|contract_name_en|string|Y|合约英文名|
|contract_name_zh|string|Y|合约中文名|
|way_en|string|Y|仓位方向英文|
|way_zh|string|Y|仓位方向中文|
|modify_vol|string|N|相关交易量|
|order_id|string|N|订单号,计划委托中有效|

- 通知类型
  
|通知类型号|类型描述|
|---|---|
|1|爆仓预警通知|
|2|强平通知|
|3|ADL通知|
|4|计划委托成功|
|5|计划委托失败|
   

- 商户返回数据(json)

```
// 根据公共返回参数判断成功/失败
```




## 6. 前端交易说明

通过以上接口，商户(交易所)实现了接入到合约云系统, 具备:

- 在合约云上创建与自己用户账户一一对应的子账户。

- 通过资金划转接口，将母账户的保证金从子账户中转入转出。

- 查询所有子账户的交易情况

- 获取子账户交易操作所需的api_key值信息

商户获取到子账户的api_key相关信息，处理后传给自己前端界面，前端用户就可以做身份验证的签名，直接通过交易接口与合约交易系统通信了。

**注意: 交易接口不是合约云接口的范畴,是交易系统的功能. 本文提供了前端源代码,里面已经实现了前端和交易系统的交易接口交互的所有逻辑和功能.**

### 6.1 基本信息介绍

- 交易所下每个交易子账号都有一对api_key/api_secret，并且会有失效时间，这些数据通过接口获取

- 每个交易所都有一个api_salt，该数据在对接时候由合约云提供

- 交易所服务器通过子账号id向合约云服务器查询子账号key信息：api_key/api_secret，也可通过更新接口在失效前更新这些key信息。

- web端/移动端SDK及文档地址：[https://github.com/exup-cloud](https://github.com/exup-cloud)

- 合约交易接口详情：[https://documenter.getpostman.com/view/179017/SWECXanT?version=latest](https://documenter.getpostman.com/view/179017/SWECXanT?version=latest)


### 6.2 交易流程

- 交易所用户登录交易所后，前端向交易所服务端请求获取合约交易相关的key信息

- 交易所服务端向合约云服务器获取该子账号的`api_key`和`api_secret`

- 交易所服务端根据规则将`api_secret`生成`token`,以及`token`对应的过期时间戳`expired_ts`

- 交易所服务端将`api_key`、`token`、`expired_ts`传给前端

- 前端拿到`api_key`、`token`、`expired_ts`之后, 跟合约交易接口通信，用token来对消息做签名鉴权


### 6.3 交易所服务端生成token的方式

公式： token = md5(api_secret+api_salt+expired_ts)

expired_ts：由交易所服务器自己生成，表示token失效的时间戳（单位：`微秒`）

示例：

    ```
    api_secret = 'd07ed850-bc18-4768-ab07-bb3b4fe0c2fc'
    api_salt = '0ffe3c0ce92d0fd36c1d3809a3e05baf'
    expired_ts = 1542971159000000
    token=md5(api_secret+api_salt+expired_ts) = md5('d07ed850-bc18-4768-ab07-bb3b4fe0c2fc0ffe3c0ce92d0fd36c1d3809a3e05baf1542971159000000')
    ```

### 6.4 前端与交易接口通信的鉴权规则

本文提供的前端源代码中有实现逻辑,此处只做简要说明

在请求的http头加上以下头信息：

|Header Name    |Description|
|---            |---        |
|tigermex-Ver        |1.0|
|tigermex-Dev        |WEB|
|tigermex-Accesskey  |交易所服务端传给前端的api_key|
|tigermex-ExpiredTs  |交易所服务端传给前端的expired_ts（单位：`微秒`）|
|tigermex-Ts         |前端生成的当前时间的时间戳（单位：`微秒`）|
|tigermex-Sign       |MD5(PostData+token+tigermex-Ts)，其中PostData为POST请求时的body数据，GET请求时为空；token为交易所服务端传给前端的token|



## 7. 相关资料

### 7.1 币种信息

- 币种信息

    https://api.tigermex.com/v1/ifglobal/global

    币种精度数据在返回json数据中的data.spot_coins.vol_unit

- 币种精度

|name       |symbol |价格精度       |数量精度     |
|---        |---    |---           |---         |
|Bitcoin    |BTC    |0.00000001    |0.00000001  |
|Ethereum   |ETH    |0.00001       |0.00000001  |
|TetherUS   |USDT   |0.0001        |0.0001      |
|EOS        |EOS    |0.00001       |0.0001      |



  [1]: http://static.zybuluo.com/GuyunHy/uvtiyb5ht39ngx6c6hsglg5l/image_1csanpq561ufgsfdj878iqsus9.png
