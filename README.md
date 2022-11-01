# Block Purse VCC 开发文档
<br>
<br>

## 目录

* 报文传输约定
  - 报文传输协议
  - 报文格式约定
  - 报文加解密

* API 接口
  - 开卡申请
  - 卡信息查询
  - 卡片充值
  - 卡片退款
  - 销卡
  - 卡片交易明细
  - 卡片账单明细
* 开卡币种
* 工具类

<br>
<br>

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
      "verify": "7iu23kj ...... ew3i2o4r" 签名串
	}


<br>

### 报文加解密

	部分或全部接口需要对部分请求参数进行组装加密对其进行签名验证，具体拼接字段见接口内容。
	
	第一步、明文拼接相关字段字符串，如：
	
	"requestNo= 20222102618240002&currency=USD&limitAmount=100&cardType=NORMAL"
	
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
      "custNo": "1585199979276017664",
      "request": {
        "binRangeId": "522981",
        "cardAlias": "信息",
        "cardType": "NORMAL",
        "currency": "USD",
        "enableCurrencyCheck": "YES",
        "enableMultiUse": "YES",
        "endDate": "2023-10-25",
        "limitAmount": 100,
        "requestNo": "20222102618240002",
        "startDate": "2022-10-26"
      },
      "verify": "234rfre432wefgyu763ewdft"
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
      "cardCvv": "string",
      "cardId": "string",
      "cardNo": "string",
      "currency": "string",
      "expiryDate": "string",
      "orderNo": "string",
      "requestNo": "string"
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
      "custNo": "1585199979276017664",
      "request": {
        "cardId": "522981"
      },
      "verify": "234rfre432wefgyu763ewdft"
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
12  | limitAmount | 当前额度 | String | 必填 |
13  | balance | 卡余额 | String | 必填 |
14  | minAmt | 最低限额 | String | 必填 |
15  | maxAmt | 最高限额 | String | 必填 |
16  | usedAmt | 已用额度 | String | 必填 |
17  | multiUse | 多次使用 | String | 必填 |
18  | currencyCheck | 限制交易币种 | String | 必填 |
19  | status | 状态 | String | 必填 |


#### 响应示例
	{
      "errorCode": null,
      "errorMsg": null,
      "result": "23e9roifkjehyu......er8r9o3kjerufre3o",
      "success": true
	}	

result 解密后格式
  
    {
      "activeDate": "string",
      "alias": "string",
      "balance": 0,
      "cardCvv": 0,
      "cardId": "string",
      "cardNo": "string",
      "cardType": "string",
      "currency": "string",
      "currencyCheck": "string",
      "customerId": "string",
      "expiryDate": "string",
      "label": "string",
      "limitAmount": 0,
      "maxAmt": 0,
      "minAmt": 0,
      "multiUse": "string",
      "orderNo": "string",
      "status": "string",
      "usedAmt": 0
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
      "custNo": "string",
      "request": {
        "amount": 0,
        "cardId": "string",
        "requestNo": "string"
      },
      "verify": "string"
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
      "errorCode": "string",
      "errorMsg": "string",
      "result": "78ikejr........fhr3iekj",
      "success": true
	}


result 解密后内容

    221020000012123123

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
      "custNo": "string",
      "request": {
        "amount": 0,
        "cardId": "string",
        "requestNo": "string"
      },
      "verify": "string"
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
      "errorCode": "string",
      "errorMsg": "string",
      "result": "fe3456yhgfre56uhgfe45678ijhgfew3",
      "success": true
	}

result 解密后内容

    221020000012123123

<br>

### 销卡

<br>

### URL
/api/v1.0/card/cancel

#### 请求参数

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | requestNo | 请求流水号 | String(32) | 必填 | 唯一，不可重复
2  | cardId | 卡编号 | String(32) | 必填 | 开卡申请接口返回的卡编号

#### 请求示例

	{
      "custNo": "string",
      "request": {
        "cardId": "string",
        "requestNo": "string"
      },
      "verify": "string"
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
      "errorCode": "string",
      "errorMsg": "string",
      "result": "345yhgfdr65789ikjfder567890okjhgfe",
      "success": true
	}


result 解密后内容

    221020000012123123

<br>

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
      "custNo": "string",
      "request": {
      "beginTime": "string",
      "cardId": "string",
      "currentPage": 0,
      "endTime": "string",
      "pageSize": 0
      },
      "verify": "string"
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
      "errorCode": "string",
      "errorMsg": "string",
      "result": {
        "list": ""
        "currentPage": 0,
        "pageSize": 0,
        "totalCount": 0
      },
      "success": true
    }

result 解密后内容

    [
      {
        "approvalCode": "string",
        "billCurrency": "string",
        "billCurrencyAmt": "string",
        "cardId": "string",
        "isCredit": "string",
        "merchantName": "string",
        "recordNo": "string",
        "settleDate": "string",
        "transCurrency": "string",
        "transCurrencyAmt": "string"
      }
    ]

Result:

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | recordNo | 记录编号 | String | 必填 |
2  | cardId | 卡唯一编号 | String | 必填 |
3  | occurTime | 交易发生时间 | String | 必填 |
4  | transCurrency | 交易币种 | String | / |
5  | transCurrencyAmt | 交易币种金额 | String | 必填 |
6  | localCurrency | 卡本币种 | String | 必填 |
7  | localCurrencyAmt | 卡本币种金额 | String | 必填 |
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
      "custNo": "string",
      "request": {
      "beginTime": "string",
      "cardId": "string",
      "currentPage": 0,
      "endTime": "string",
      "pageSize": 0
      },
      "verify": "string"
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
      "errorCode": "string",
      "errorMsg": "string",
      "result": {
        "list": ""
        "currentPage": 0,
        "pageSize": 0,
        "totalCount": 0
      },
      "success": true
    }

result 解密后内容

    [
      {
        "approvalCode": "string",
        "billCurrency": "string",
        "billCurrencyAmt": "string",
        "cardId": "string",
        "isCredit": "string",
        "merchantName": "string",
        "recordNo": "string",
        "settleDate": "string",
        "transCurrency": "string",
        "transCurrencyAmt": "string"
      }
    ]

Result:

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | recordNo | 记录编号 | String | 必填 |
2  | cardId | 卡唯一编号 | String | 必填 |
2  | settleDate | 账单日期 | String | 必填 |
2  | transCurrency | 交易币种 | String | 必填 |
2  | transCurrencyAmt | 交易金额 | String | 必填 |
2  | billCurrency | 账单币种 | String | 必填 |
2  | billCurrencyAmt | 账单金额 | String | 必填 |
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
      "custNo": "string",
      "request": {
        "currency": "USD",
      },
      "verify": "string"
	}

#### 待签名字符串

	"custNo="+custNo

<br>

#### 响应结果

序号 | 字段 |  字段描述 | 字段类型   | 必填    | 备注
----|-----|-----------|--------|------------|-------|
1  | orderNo | 平台订单号 | String | 必填 |


#### 响应示例

    {
      "success": true,
      "result": nbhgYUnBGYumn...bgYUIKM,
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
        "accountId": "1587002211281670144",
        "accountType": "balance",
        "currency": "USD",
        "balance": 999870.34,
        "availableBal": 999870.34,
        "freezeBal": 0,
        "customerId": "1587002211214565376",
        "status": "normal"
      }
    ]

<br>

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
<br>
<br>
<br>

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


	
