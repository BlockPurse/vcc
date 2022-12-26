# Block Purse VCC 开发文档
<br>
<br>

## 目录

* 网关地址

* 报文传输约定
  - 报文传输协议
  - 报文格式约定
  - 报文加解密

* API 接口
  - 开卡申请
  - 卡信息查询
  - 卡头列表查询(12-22修改)
  - 卡图信息查询
  - 卡片充值
  - 卡片退款
  - 销卡
  - 查询卡片订单信息(12-19新增)
  - 查询开卡结果
  - 查询充值结果
  - 查询退款结果
  - 查询销卡结果
  - 卡片交易明细
  - 卡片账单明细
  - 账户余额查询
  - 模拟卡片交易
  - Webhook 推送
* 开卡币种
* 交易类型
* 交易状态
* 工具类

<br>
<br>

## 网关地址

正式环境 https://vccapi.blockpurse.io

测试环境 https://test-vccapi.blockpurse.io


## 报文传输约定

<br>

### 报文传输协议

<br>

	接口采用https POST方式进行报文传输

<br>

### 报文格式约定

<br>

	交互过程中，接口请求参数及响应结果都采用JSON格式包装。
	如：
	
	{
      "custNo": "客户编号（平台开户成功返回的客户编号），必填",
      "request": 接口具体请求参数（泛型）
      "verify": "234rfre43...u763ewdft" 签名串
	}


<br>

### 报文加解密

	部分或全部接口需要对部分请求参数进行组装加密对其进行签名验证，具体拼接字段见接口内容。
	
	第一步、明文拼接相关字段字符串，如：
	
	"requestNo=2022...18240002&currency=USD&limitAmount=100&cardType=NORMAL"
	
	第二步、使用Base64对字符串进行Encode，得到待加密字符串。
	
	第三步、使用apiKey 对待加密字符串进行加密，加密方式为AES。

<br>
<br>
<br>
<br>

## API 接口

<br>

### 开卡申请

### URL
/api/v1.0/card/apply
<br>

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复
2 | currency  | 开卡币种  | String（16）| 必填 |  CNH-离岸人民币；CAD-加元；HKD-港币 USD-美元
3 | startDate | 生效日期 | String(10) | 必填 | 格式： yyyy-MM-dd
4 | endDate | 失效日期 | String(10) | 必填 | 格式： yyyy-MM-dd
5 | limitAmount | 开卡金额 | Number | 必填 |
6 | enableMultiUse | 多次卡 | String(16) | 必填 | YES-多次卡; NO-单次卡
7 | enableCurrencyCheck | 限制交易币种 | String(16) | 必填 | YES-限制;NO-不限制
8 | cardAlias | 卡片别名 | String(32) | / |
9 | binRangeId | 卡头 | String(16) | 必填 |
10 | cardType | 卡类型 | String(16) | 必填 | 卡类型(NORMAL-普通卡)

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "binRangeId": "522981",
        "cardAlias": "信息",
        "cardType": "NORMAL",
        "currency": "USD",
        "enableCurrencyCheck": "YES",
        "enableMultiUse": "YES",
        "endDate": "2023-10-25",
        "limitAmount": 100,
        "requestNo": "20222...240002",
        "startDate": "2022-10-26"
      },
      "verify": "234rfre43...u763ewdft"
	}


#### 待签名字符串

	"requestNo="+requestNo+"&currency="+currency+"&limitAmount="+limitAmount+"&cardType="+cardType


<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 |
2  | orderNo | 平台订单号 | String(18) | 必填 |
3  | cardId | 卡编号 | String | 必填 |
4  | currency | 卡币种 | String | 必填 |
5  | cardNo | 卡号 | String | 必填 |
6  | cardCvv | 卡CVV | String | 必填 |
7  | expiryDate | 失效日期 | String | 必填 |

#### 响应示例

    {
      "errorCode": null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
    }

result 解密后格式

    {
      "cardCvv": "123",
      "cardId": "221...0000001",
      "cardNo": "22292...764",
      "currency": "USD",
      "expiryDate": "202511",
      "orderNo": "221102...0000014",
      "requestNo": "20222...240002"
    }
<br>

### 卡信息查询

<br>

### URL
/api/v1.0/card/info

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号


#### 请求示例
	{
      "custNo": "15851...276017664",
      "request": {
        "cardId": "221...0000001"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"cardId ="+ cardId

<br>

#### 响应结果

<br>

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |
2  | customerId | 客户编号| String | 必填 |
3  | cardId | 卡编号 | String | 必填 |
4  | currency | 开卡币种 | String | 必填 |
5  | cardNo | 卡号 | String | 必填 |
6  | cardCvv | CVV | String | 必填 |
7  | cardType | 卡类型 | String | 必填 |
8  | expiryDate | 失效日期 | String | 必填 |
9  | alias | 别名 | String | / |
10  | label | 标签 | String | / |
11  | activeDate | 生效日期 | String | 必填 |
12  | limitAmount | 当前额度 | Number | 必填 |
13  | balance | 卡余额 | Number | 必填 |
14  | minAmt | 最低限额 | Number | 必填 |
15  | maxAmt | 最高限额 | Number | 必填 |
16  | usedAmt | 已用额度 | Number | 必填 |
17  | multiUse | 多次使用 | String | 必填 |
18  | currencyCheck | 限制交易币种 | String | 必填 |
19  | status | 状态 | String | 必填 |


#### 响应示例
	{
      "errorCode": null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}	

result 解密后格式

    {
      "activeDate": "2022-11-01",
      "alias": "测试卡",
      "balance": 10000.00,
      "cardCvv": "123",
      "cardId": "221...0000001",
      "cardNo": "22292...764",
      "cardType": "NORMAL",
      "currency": "USD",
      "currencyCheck": "YES",
      "customerId": "1587...000000001",
      "expiryDate": "202511",
      "label": "测试卡",
      "limitAmount": 10000.00,
      "maxAmt": 10000.00,
      "minAmt": 0.01,
      "multiUse": "YES",
      "orderNo": "22103...0000001",
      "status": "NORMAL",
      "usedAmt": 0.00
    }

<br>

### 卡头列表查询(12-22修改)

<br>

### URL
/api/v1.0/card/cardBins

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
/  | / | / | / | / | /


#### 请求示例
	{
      "custNo": "15851999...76017664",
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"custNo ="+ 1585199...76017664

<br>

#### 响应结果

<br>

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | cardBin | 卡头信息 | String | 必填 |
2  | cardType | 卡类型| String | 必填 |
3  | supportCcy | 卡币种 | String | 必填 |
4  | status | 状态 | String | 必填 |
5  | enDesc | 描述 | String | / |
6  | remark | 描述 | String | / |

<br>


#### 响应示例
	{
      "errorCode": null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}	

result 解密后格式

    [
      {
        "cardBin": "222222",
        "cardType": "visa",
        "supportCcy": "USD",
        "status": "yes",
        "enDesc": "U.S. Visa/Amazon, eBay, Alibaba, Wal-Mart and other platforms/FB advertisements/site building tools and other payment/image processing software payment, etc. /Malicious refunds and empty card consumption are prohibited, and violators are severely punished.",
        "remark": "美国Visa/亚马逊、eBay、Alibaba、沃尔玛等平台海淘/FB投放广告/建站工具等付费/图片处理软件付费等，禁止恶意退款和空卡消费，违规者严重处理",
      }
    ]

<br>

### 卡图信息查询

<br>

### URL
/api/v1.0/card/cardImg

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "cardId": "221...0000001",
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"cardId="+cardId

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | frontBase64 | 卡片正面图像 | String | 必填 | base64图片内容
2  | backBase64 | 卡片背面图像 | String | 必填 | base64图片内容


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}


result 解密后内容

    {
      "frontBase64": "asdfasdfasdfa23...erfgfre456yfdwq",
      "backBase64": "asdfasdfasdfa23...erfgfre456yfdwq"
    }

<br>

### 卡片充值

<br>

### URL
/api/v1.0/card/recharge

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复
2  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号
3  | amount | 充值金额| Number | 必填 |

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "amount": 100.00,
        "cardId": "221...0000001",
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo+"&cardId="+cardId+"&amount="+amount

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}


result 解密后内容

    2210200...23123

<br>

### 卡片退款

<br>

### URL
/api/v1.0/card/refund

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复
2  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号
3  | amount | 充值金额| Number | 必填 |

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "amount": 100.00,
        "cardId": "221...0000001",
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo+"&cardId="+cardId+"&amount="+amount

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}

result 解密后内容

    221020...23123

<br>

### 销卡

<br>

### URL
/api/v1.0/card/close

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复
2  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号
·


#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "cardId": "221...0000001",
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo+"&cardId="+cardId

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}


result 解密后内容

    2210200...3123

<br>

### 查詢开卡结果

<br>

### URL
/api/v1.0/card/getApplyInfo

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复


#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 |
2  | orderNo | 平台订单号 | String(18) | 必填 |
3  | cardId | 卡编号 | String | 必填 |
4  | currency | 卡币种 | String | 必填 |
5  | cardNo | 卡号 | String | 必填 |
6  | cardCvv | 卡CVV | String | 必填 |
7  | expiryDate | 失效日期 | String | 必填 |

#### 响应示例

    {
      "errorCode": null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
    }


result 解密后内容

    {
      "cardCvv": "123",
      "cardId": "221...0000001",
      "cardNo": "22292...764",
      "currency": "USD",
      "expiryDate": "202511",
      "orderNo": "221102...0000014",
      "requestNo": "20222...240002"
    }
<br>

### 查询卡片订单信息(12-19新增)

<br>

### URL
/api/v1.0/card/getOrderInfo

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复


#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String | 必填 |
2  | orderNo | 平台订单号 | String | 必填 |
3  | customerId | 客户编号 | String | 必填 |
4  | cardId | 卡编号 | String | / |
5  | cardNo | 卡号 | String | / |
6  | orderType | 订单类型 | String | 必填 | 见附录：订单类型
7  | orderDate | 订单日期 | Date | 必填 |
8  | orderAmount | 订单金额 | Number | 必填 |
9  | orderStatus | 订单状态 | String | 必填 | 见附录：订单状态

#### 响应示例

    {
      "errorCode": null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
    }


result 解密后内容

    {
      "requestNo": "221...0000001",
      "orderNo": "221...0000001",
      "customerId": "221...0000001",
      "cardId": "221...0000001",
      "cardNo": "22292...764",
      "orderType": "CARD_APPLY",
      "orderDate": "yy...",
      "orderAmount": 100.00,
      "orderStatus": "SUCCESS"
    }

<br>

### 查询充值结果

<br>

### URL
/api/v1.0/card/getRechargeInfo

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复


#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}


result 解密后内容

    221020...23123

<br>

### 查询退款结果

<br>

### URL
/api/v1.0/card/getRefundInfo

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复


#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}


result 解密后内容

    221020...3123

<br>

### 查询销卡结果

<br>

### URL
/api/v1.0/card/getCloseInfo

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复


#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "requestNo": "20222...240002"
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"requestNo="+requestNo

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "success": true
	}


result 解密后内容

    221020...123


### 卡片交易明细

<br>

### URL
/api/v1.0/card/trades

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号
2  | beginTime | 开始时间 | String | 必填 | 开始时间 格式yyyy-MM-dd
3  | endTime | 结束时间 | String | 必填 | 开始时间 格式yyyy-MM-dd
4  | currentPage | 当前页 | Number | 必填 | 正整数
5  | pageSize | 分页大小 | Number | 必填 | 最大50

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "beginTime": "2022-01-01",
        "cardId": "221...0000001",
        "currentPage": 1,
        "endTime": "2022-01-02",
        "pageSize": 50
      },
      "verify": "234rfre43...u763ewdft"
    }

#### 待签名字符串

	"beginTime="+beginTime+"&endTime="+endTime+"&currentPage="+currentPage

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | totalCount | 总条数 | String | 必填 |
2  | currentPage | 当前页 | String | 必填 |
3  | pageSize | 页行数 | String | 必填 |
4  | list | 数据集 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": {
        "list": "234rfre43...u763ewdft",
        "currentPage": 1,
        "pageSize": 50,
        "pageSize": 50
      },
      "success": true
    }

list 解密后内容

    [
      {
        "recordNo": "22110...00000001",
        "cardId": "221...0000001",
        "occurTime": "2022-11-01 14:39:19",
        "transCurrency": "USD",
        "transCurrencyAmt": 10.00,
        "transType": "AUTH",
        "transStatus": "APPROVED",
        "localCurrency": "USD",
        "localCurrencyAmt": 10.00,
        "crossBoardType": null,
        "respCode": "000000",
        "respCodeDesc": "交易成功",
        "approvalCode": "000000",
        "merchantName": "xxxxx...xxxxx"
      }
    ]

Result:

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | recordNo | 记录编号 | String | 必填 |
2  | cardId | 卡唯一编号 | String | 必填 |
3  | occurTime | 交易发生时间 | String | 必填 |
4  | transCurrency | 交易币种 | String | / |
5  | transCurrencyAmt | 交易币种金额 | Number | 必填 |
6  | localCurrency | 卡本币种 | String | 必填 |
7  | localCurrencyAmt | 卡本币种金额 | Number | 必填 |
8  | respCode | 交易响应码 | String | 必填 |
9  | respCodeDesc | 交易响应码描述 | String | 必填 |
10  | approvalCode | 授权码 | String | / |
11 | declineReason | 交易拒绝原因 | String | / |
12  | messageType | 交易类型 | String | 必填 |
13  | messageTypeDesc | 交易类型描述 | String | 必填 |
14  | merchantName | 商户名称 | String | 必填 |

<br>

### 卡片账单明细

<br>

### URL
/api/v1.0/card/settlements

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号
2  | beginTime | 开始时间 | String | 必填 | 开始时间 格式yyyy-MM-dd
3  | endTime | 结束时间 | String | 必填 | 开始时间 格式yyyy-MM-dd
4  | currentPage | 当前页 | Number | 必填 | 正整数
5  | pageSize | 分页大小 | Number | 必填 | 最大50

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "beginTime": "2022-01-01",
        "cardId": "221...0000001",
        "currentPage": 1,
        "endTime": "2022-01-02",
        "pageSize": 20
      },
      "verify": "234rfre43...u763ewdft"
    }

#### 待签名字符串

	"beginTime="+beginTime+"&endTime="+endTime+"&currentPage="+currentPage

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | totalCount | 总条数 | String | 必填 |
2  | currentPage | 当前页 | String | 必填 |
3  | pageSize | 页行数 | String | 必填 |
4  | list | 数据集 | String | 必填 |


#### 响应示例

	{
      "errorCode":  null,
      "errorMsg": null,
      "result": {
        "list": "234rfre43...u763ewdft",
        "currentPage": 1,
        "pageSize": 50,
        "totalCount": 500
      },
      "success": true
    }

list 解密后内容

    [
      {
        "approvalCode": "000000",
        "billCurrency": "USD",
        "billCurrencyAmt": 100.00,
        "cardId": "221...0000001",
        "isCredit": "1",
        "merchantName": "xxxxxxxxxxxxxxxx",
        "recordNo": "2212...0000627",
        "settleDate": "2022-11-01",
        "transCurrency": "USD",
        "transCurrencyAmt": 100.00
      }
    ]

Result:

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | recordNo | 记录编号 | String | 必填 |
2  | cardId | 卡唯一编号 | String | 必填 |
2  | settleDate | 账单日期 | String | 必填 |
2  | transCurrency | 交易币种 | String | 必填 |
2  | transCurrencyAmt | 交易金额 | Number | 必填 |
2  | billCurrency | 账单币种 | String | 必填 |
2  | billCurrencyAmt | 账单金额 | Number | 必填 |
2  | approvalCode | 授权码 | String | / |
2  | isCredit | 收付标识 | String | 必填 |
2  | merchantName | 商户名称 | String | 必填 |


### 账户余额查询

<br>

### URL
/api/v1.0/account/balance

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | currency | 账户币种 | String(16) | / |  不填查所有币种

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "currency": "USD",
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	"custNo="+custNo

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | result | 余额信息加密串 | String | 必填 |


#### 响应示例

    {
      "success": true,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "errorCode": null,
      "errorMsg": null
    }


result 解密后内容

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | accountId | 余额账户编号 | String | 必填 |
2  | accountType | 账户类型 | String | 必填 |
3  | currency | 账户币种 | String | 必填 |
4  | balance | 总余额 | Number | 必填 |
5  | availableBal | 可用余额 | Number | 必填 |
6  | freezeBal | 冻结余额 | Number | 必填 |
7  | customerId | 客户编号 | String | 必填 |
8  | status | 状态 | String | 必填 |


    [
      {
        "accountId": "1587...81670144",
        "accountType": "balance",
        "currency": "USD",
        "balance": 999870.34,
        "availableBal": 999870.34,
        "freezeBal": 0,
        "customerId": "15870...14565376",
        "status": "normal"
      }
    ]

<br>


### 模拟卡片交易

<br>

### URL
/api/v1.0/card/test/trade

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1 | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复
2 | cardId  | 卡片编号  | String（16）| 必填 | 非卡号
3 | tradeType | 交易类型 | String(10) | 必填 | 固定为：TRADE
4 | amount | 交易金额 | Number | 必填 | 仅支持2位小数，最小测试金额为 1.00

#### 请求示例

	{
      "custNo": "158519...017664",
      "request": {
        "requestNo": "20222...240002",
        "cardId": "221...0000001",
        "tradeType": "TRADE",
        "amount": 100.00
      },
      "verify": "234rfre43...u763ewdft"
	}

#### 待签名字符串

	requestNo="+requestNo+"&cardId="+cardId+"&tradeType="+tradeType+"&amount="+amount

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  |  result | 平台订单号加密串 | String | 必填 |


#### 响应示例

    {
      "success": true,
      "result": "dafsdjfnh239u4....odnfb2i3ope",
      "errorCode": null,
      "errorMsg": null
    }

<br>

result 解密后内容


序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |

22112...00001


<br>

### Webhook 推送

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | dataType | 数据类型 | String | 必填 | AUTH-交易，SETTLEMENT-清算
2  | request | 通知内容 | String | 必填 | 加密内容


#### 响应示例

	{
      "dataType": "AUTH",
      "request": "234rfde567uhgdw45678ijhgjhde",
    }

request  (dataType=AUTH) 交易解密后内容

    {
      "recordNo": "221101...0000001",
      "cardId": "221...0000001",
      "occurTime": "2022-11-01 14:39:19",
      "transCurrency": "USD",
      "transCurrencyAmt": 10.00,
      "transType": "AUTH",
      "transStatus": "APPROVED",
      "localCurrency": "USD",
      "localCurrencyAmt": 10.00,
      "crossBoardType": null,
      "respCode": "000000",
      "respCodeDesc": "交易成功",
      "approvalCode": "000000",
      "merchantName": "xxxxx...xxxxx"
    }


Result:

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | recordNo | 记录编号 | String | 必填 | 记录编号
2  | cardId | 卡唯一编号 | String | 必填 | 卡片编号
3  | occurTime | 交易时间 | String | 必填 | 交易时间
4  | transCurrency | 交易币种 | String | / | 交易币种
5  | transCurrencyAmt | 交易金额 | Number | 必填 | 交易金额
6  | transType | 交易类型 | String | 必填 | 交易类型: 见交易类型列表
7  | transStatus | 交易状态 | String | 必填 | 交易状态: 见交易状态列表
8  | localCurrency | 卡本位币种 | String | 必填 | 账单币种
9  | localCurrencyAmt | 卡本位币种金额 | Number | 必填 | 账单金额
10  | crossBoardType | 跨境类型 | String | / | 0 境内；1 境外
11  | respCode | 交易响应码 | String | 必填 | 交易响应码
12  | respCodeDesc | 交易响应码描述 | String | 必填 | 交易响应码描述
13  | approvalCode | 授权码 | String | / | 授权码
14 | merchantName | 商户名称 | String | / | 商户名称


request (dataType=SETTLEMENT) 清算解密后内容

    {
      "recordNo": "221101...000001",
      "cardId": "221...0000001",
      "settleDate": "2022-11-02",
      "transCurrency": "USD",
      "transCurrencyAmt": 0.00,
      "billCurrency": "USD",
      "billCurrencyAmt": 0.00,
      "approvalCode": "000000",
      "isCredit": "1",
      "merchantName": "xxxxxxxx"
    }


Result:

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | recordNo | 记录编号 | String | 必填 |
2  | cardId | 卡唯一编号 | String | 必填 |
3  | settleDate | 账单日期 | String | 必填 |
4  | transCurrency | 交易币种 | String | 必填 |
5  | transCurrencyAmt | 交易币种金额 | Number | 必填 |
6  | billCurrency | 账单币种 | String | 必填 |
7  | billCurrencyAmt | 账单金额 | Number | 必填 |
8  | approvalCode | 授权码 | String | 必填 |
9  | isCredit | 收付标志 | String | 必填 | 1:收(退款)  0:付(消费)
10  | merchantName | 授权码 | String | / |

<br>

request  (dataType=TYPE_CARD_OPERATE) 卡操作（目前只有被动销卡）

    {
      "orderId": "22281...23132123",
      "cardId": "221...0000001",
      "userReqNo": "124765...2312312",
      "opType": 0,
      "status": 2,
      "statusDesc": "成功",
      "amount": 100.00,
      "fee": 0.00,
      "createAt": "2022-11-29 12:00:00",
    }


Result:

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderId | 服务系统返回的单号 | String | 必填 | 记录编号
2  | cardId | 卡唯一编号 | String | 必填 | 卡片编号
3  | userReqNo | 请求流水号 | String | 必填 | 客户上送的请求流水号
4  | opType | 订单类型 | int | 必填 | 0-销卡
5  | status | 状态 | int | 必填 | 状态：0 - 待处理；1 - 处理中；2 - 成功；3 - 失败；
6  | statusDesc | 状态描述 | String | 必填 | 状态描述
7  | amount | 订单金额 | Number | 必填 | 销卡金额
8  | fee | 手续费 | Number | 必填 | 订单对应的手续费
9  | createAt | 创建时间 | String | 必填 | 创建的时间，格式：yyy-MM-dd HH:mm:ss

<br>

<br>
<br>
<br>
<br>


## 开卡币种

代码| 描述|
----|-----|
CNH | 离岸人民币
CAD | 加元
HKD | 港币
USD | 美元

<br>

## 交易类型

交易类型 | 中文描述| 英文描述
----|-----|----|
AUTH | 授权.消费 | AUTH |
AUTH_QUERY | 授权.查询 | AUTH QUERY(12-19新增) | 
REVERSAL | 冲正 | REVERSAL |
REFUND | 退款 | REFUND |
FEE | 手续费 | FEE |
FEE_REVERSAL | 手续费冲正 | FEE REVERSAL |
ORIGINAL_CREDIT | OCT退款 | ORIGINAL CREDIT |
ORIGINAL_CREDIT_REVERSAL | OCT退款冲正 | ORIGINAL CREDIT REVERSAL |

<br>

## 交易状态

交易状态 | 中文描述| 英文描述
----|-----|----|
APPROVED | 消费 | APPROVED |
DECLINED | 拒绝 | DECLINED |

<br>

## 订单类型 (12-19新增)

订单类型 | 中文描述| 英文描述
----|-----|----|
CARD_APPLY | 开卡 | CARD APPLY |
CARD_RECHARGE | 卡充值 | CARD RECHARGE |
CARD_REFUND | 卡退款 | CARD REFUND |
CARD_CLOSE | 销卡 | CARD CLOSE |

## 订单状态 (12-19新增)

订单状态 | 中文描述| 英文描述
----|-----|----|
INIT | 初始 | INIT |
SUCCESS | 成功 | SUCCESS |
FAILED | 失败 | FAILED |
UNKNOWN | 未知 | UNKNOWN |

<br>


## 错误码

错误码| 中文描述| 英文描述
----|-----|----|
MAIL_REGISTERED | 邮箱地址已注册 | The mailbox address has been registered
PHONE_REGISTERED|手机号已注册|Mobile phone number has been registered
LOGIN_ERROR|登录信息不存在或不正确|Login information does not exist or incorrect
TOKEN_EXPIRED|登录失效|Login failure
AUTHORIZATION_FAILED|权限校验失败|Permanent verification failed
OPERATION_NOT_ALLOW|不允许的操作|Operations that are not allowed
AUTH_ERROR|授权失败|Authorization failure
CAPTCHA_BUSY|验证码获取频繁，请稍后再试|The verification code is frequently obtained, please try again later
CAPTCHA_ERROR|验证码已过期或不正确|The verification code has expired or incorrect
ERROR_COUNTRY|不支持的国家或地区|Unwilling countries or regions
SIGN_VERIFY_FAIL|签名验证不通过|Signature verification is not approved
LACK_BALANCE|余额不足|Insufficient balance
EMPTY_PARAM|请求参数为空|The request parameter is blank
ACCOUNT_NOTFOUND|账户不存在|Account does not exist
CUSTOMER_NOTFOUND|客户不存在|Customers do not exist
ORDER_NOTFOUND|订单不存在|The order does not exist
KEY_NOT_CONFIG|密钥未配置|The key is not configured
BELOW_MINI_LIMIT|交易金额低于最小限定值|The transaction amount is lower than the minimum limited value
INVALID_CHAIN_ID|无效的网络ID|Infernal network ID
NETWORK_ERROR|网络连接异常|Network connection abnormal
CARD_NOT_FUND|卡片不存在|Card does not exist
RECORD_NOT_FUND|记录不存在|Record does not exist
CARD_LACK_BALANCE|卡片余额不足|Card Insufficient balance
REQUEST_FAILED|请求失败|Request failed
REQUEST_REPEAT|重复的请求|Repeated request
SIGNATURE_FAILED|签名校验失败|Signature verification failed
INVALID_CURRENCY|不支持的币种|Non -supported currency
INVALID_CARD|不支持的卡头|Unswerving card head
INVALID_COUNTRY|不支持的国家地区|Unwilling national and regions
INVALID_PARAM|参数校验错误|Parameter test error
INVALID_FILE_TYPE|不支持的文件类型|Unwilling file type
DATABASE_ERROR|系统繁忙，请稍后再试|The system is busy, please try again later
BILLING_ERROR|计费规则未配置|Unschere
CARD_CONFIG_ERROR|卡规则未配置|Card rules are not configured
BIZ_ERROR|业务异常|Business abnormalities
BIZ_STATUS_ERROR|订单状态未知，稍后重新查询订单状态 (12-19新增)|The order status is unknown. Query the order info again later please
REQUEST_BUSY|请求太过频繁，请稍后再试|The request is too frequent, please try it later
BAD_REQUEST_IP|不允许的请求IP，请使用预设的出口IP发起请求|If the request IP is not allowed, please use the preset export IP to initiate the request
ERROR_ENV|该操作只允许在测试环境执行|If the request IP is not allowed, please use the preset export IP to initiate the request
SYSTEM_ERROR|系统繁忙，请稍后再试|The system is busy, please try again later

### 工具类

<br>

#### Maven import：

    <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
      <version>5.8.4</version>
    </dependency>

<br>

#### Java Class

	import cn.hutool.core.codec.Base64;
	import cn.hutool.core.util.CharsetUtil;
	import cn.hutool.core.util.HexUtil;
	import cn.hutool.crypto.symmetric.SymmetricAlgorithm;
	import cn.hutool.crypto.symmetric.SymmetricCrypto;
	
	public class AesUtils {
	
	    public static String encode(String data, String apiKey) {
	        SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, HexUtil.decodeHex(apiKey));
	        return aes.encryptHex(Base64.encode(data),CharsetUtil.CHARSET_UTF_8);
	    }
	
	    public static String decode(String encodeData, String apiKey) {
	        SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, HexUtil.decodeHex(apiKey));
	        return Base64.decodeStr(aes.decryptStr(encodeData,CharsetUtil.CHARSET_UTF_8));
	    }
	
	    public static void main(String[] args) {
	
	        String apiKey = "a2450a85a4b00b626ed94490101a8613";
	
	        String data = "requestNo=123&currency=USD&limitAmount=100&cardType=NORMAL";
	
	        System.out.println("原文：" + data);
	        String encodeData = encode(data, apiKey);
	        System.out.println("加密后：" + encodeData);
	
	        String decodeData = decode(encodeData, apiKey);
	        System.out.println("解密后：" + decodeData);
	
	        System.out.println("解密后与原文比较：" + data.equals(decodeData));
	    }
	
	}
