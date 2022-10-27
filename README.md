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
		"custNo": "客户编号（平台开户成功返回的客户编号）",
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


#### 代签名字符串

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
		"errorCode": "string",
		"errorMsg": "string",
		"result": {
			"cardCvv": "string",
			"cardId": "string",
			"cardNo": "string",
			"currency": "string",
			"expiryDate": "string",
			"orderNo": "string",
			"requestNo": "string"
		},
		"success": true
	}

<br>

### 卡信息查询

<br>

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

#### 代签名字符串

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
9  | alias | 别名 | String | 必填 |
10  | label | 标签 | String | 必填 |
11  | activeDate | 生效日期 | String | 必填 |
12  | limitAmount | 当前额度 | String | 必填 |
13 | balance | 卡余额 | String | 必填 |
14  | minAmt | 最低限额 | String | 必填 |
15  | maxAmt | 最高限额 | String | 必填 |
16  | usedAmt | 已用额度 | String | 必填 |
17  | multiUse | 多次使用 | String | 必填 |
18  | currencyCheck | 限制交易币种 | String | 必填 |
19  | status | 状态 | String | 必填 |


#### 响应示例
	{
		"errorCode": "string",
		"errorMsg": "string",
		"result": {
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
		},
		"success": true
	}	
	
<br>
	
### 卡片充值

<br>

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

#### 代签名字符串

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
	"result": "221020000012123123",
	"success": true
	}
	
<br>

### 卡片退款

<br>

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

#### 代签名字符串

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
	"result": "221020000012123123",
	"success": true
	}
	
<br>

### 销卡

<br>

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

#### 代签名字符串

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
	"result": "221020000012123123",
	"success": true
	}
	
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

	import cn.hutool.core.util.CharsetUtil;
	import cn.hutool.core.util.HexUtil;
	import cn.hutool.crypto.symmetric.SymmetricAlgorithm;
	import cn.hutool.crypto.symmetric.SymmetricCrypto;
	
	public class AesUtils {
	
	    public static String encode(String data, String apiKey) {
	        SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, HexUtil.decodeHex(apiKey));
	        return aes.encryptBase64(data,CharsetUtil.CHARSET_UTF_8);
	    }
	
	    public static String decode(String encodeData, String apiKey) {
	        SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, HexUtil.decodeHex(apiKey));
	        return aes.decryptStr(encodeData,CharsetUtil.CHARSET_UTF_8);
	    }
	
	    public static void main(String[] args) {
	        
	        String apiKey = "";
	        
	        String data = "requestNo=123&currency=USD&limitAmount=100&cardType=NORMAL";
	
	        System.out.println("原文：" + data);
	        String encodeData = encode(data, apiKey);
	        System.out.println("加密后：" + encodeData);
	
	        String decodeData = decode(encodeData, apiKey);
	        System.out.println("解密后：" + decodeData);
	
	        System.out.println("解密后与原文比较：" + data.equals(decodeData));
	    }
	    
	}



	
