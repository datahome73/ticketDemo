# ticket-issue-demo

### 介绍
积分发行时的接口，描述了票号的生成、签名、验签、票号的验证等
增加了根据手机号、姓名、身份证号，三要素查询用户是否存在的接口

### 软件架构
3个文件：

- rsa_key_gen.py    生成RSA公私钥对
- client.py         客户端，商户方
- server.py         服务端，平台方


#### 架构图
![加密-解密-签名-验签](jpg/rsa_demo.jpg)

核心算法
###### [BIP39 分层确定性算法(https://iancoleman.io/bip39/)](https://iancoleman.io/bip39/)

###### [BIP32协议(https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)

###### [支付宝-开放平台-技术接入指南(https://opendocs.alipay.com/common/02mse2)](https://opendocs.alipay.com/common/02mse2)

### 加签和验签

#### 签名与验签介绍

##### 简介

平台的应用管理体系，使用了公私钥的机制，商家可在平台【密钥管理】中设置应用的【接口加签方式】中为自身应用配置【公钥】防止数据篡改，以此来保障商家应用和平台交互的安全性。

##### 名词解释

###### 公钥：
即 应用公钥（public_key），开发者通过工具自行生成的RSA公钥信息。 

###### 私钥：
即 应用私钥（private_key），开发者通过工具自行生成的RSA私钥信息。 

######  平台公钥：
开发者在开放平台配置 应用公钥 后由平台生成，供开发者验证来自平台的同步、异步信息签名。

#### 加签、验签机制说明
商家在应用中使用自己的【私钥】对消息加签之后，消息和签名会传递给平台，平台则使用应用的【公钥】验证消息的真实性（来自于合法应用的真实消息）。
对于平台返回消息给商家应用的情形，应用则使用【平台公钥】来验证返回消息的真实性。
![签名-验签](jpg/rsa_sign.png)

注意：
• 商家应用必须保障应用私钥的安全性，从而才能保障应用和平台交互的安全性。

#### 数据签名
本接口使用了RSA私钥进行签名方式，函数为 signature(appid+timestamp)，
其中
appid为分配商户号，由平台进行分配。
timestamp 发送消息的时间戳（格林威治时间1970年01月01日00时00分00秒起至现在的总秒数）

##### 传入参数格式
已签名的数据用post的方式发送数据，组合成json对象
```
{ "appid" : appid ,"timestamp" : timestamp ,"signature" : signature ,"data" : data }
```

data 里封装需要传输的查询数据，目前阶段未做加密处理
格式如下，其中
mobile 手机号码
id 身份证号
name 姓名 

```
{
	"request": [{
			"mobile": "17683853671",
			"id": "440402197101020015",
			"name": "测试人"
		},
		{
			"mobile": "17683853672",
			"id": "440402197101020016",
			"name": "测试二"
		}
	]
}
```

#### 数据验签
平台对传入的数据进行验签，函数为 sign_verify(appid+timestamp)，对符合规则的数据，返回查询结果

##### 传出参数格式
```
{
	"code": "200",
	"msg": "消息获取成功",
	"data": data
}
```

data 里封装查询结果，目前阶段未做加密处理
格式如下，其中
mobile 手机号码
exist 
如果为 true，对应的这条记录已经在平台完成实名认证；
如果为 false，对应的这条记录已经在平台不存在或者没有完成实名认证。

```
{
	"results": [{
			"mobile": "17683853671",
			"exist": true
		},
		{
			"mobile": "17683853672",
			"exist": false
		}
	]
}
```


### 票据发行的接口

#### 供票方操作步骤：
- 先获取根公钥 
- xpub_root = "xpub6EwLt1EDyThtuQV58UroAigCzRi8iwMRTSSs2peNiSwr2qTR1ffXEPYiGhMJ3pwSx8RKfaABjGhsu9raX27shVVzeDkZSRiz3JFhPCm6Czf"
- ![公钥扩展示意](jpg/pub2pub.png)
- 
1. 根据需要生成多张票据，每张票号有一个唯一的地址值（0x开头的地址），使用的算法为BIP32里的从公钥扩展子公钥
2. 对每张票号，用RSA私钥进行签名
3. 序号（sequence），票号地址（token），签名（sign）。组合成json对象
- 

.
.
##### 传入参数格式
签名和验签的方式同上
```
{ "appid" : appid ,"timestamp" : timestamp ,"signature" : signature ,"data" : data }
```

data 的json 对象
```
{
	"1": {
		"sequence": "1",
		"token": "0xAc85f833BfAC840eF2AAfD6006c5D51d8CF4848E",
		"sign": "NF7uJYCjW/Q+vP9vuSSwwdWKwSG5AvequbDhl9CckKqIlrPocfOel5iX7KQeH/v2vC78LNkxutb8Fuj4b7ZlHbx0kXNm/PazADzqy4tXnIr9LgfBvnqSqJPOArW8HXw1zJ1/55Z1vmNOmHouRZDvX6o2ys9WlzNVTWProcAmBwc="
	},
	"2": {
		"sequence": "2",
		"token": "0x6A72771cCF1ce5eCd8e10C2a66EC5f7607dAcC1b",
		"sign": "JukJiFdKPQQap4NeM8krYl8+zvBaKpMN8fu1VUpCzODlU0xvL9PMpeDgSmRxmcrHBYrgwfm5DiyntzdNKmD8Ef/cmlFUysgxRnmiThEwdefxvIL13LTHye243KmGgC1mPHTd4nbHmcIPCPFV/HwRAQUAycFixkyAK5RnqON2mSo="
	},
	"3": {
		"sequence": "3",
		"token": "0x0d2539E0fc299965613cD52A8fBe5f1137a01BeD",
		"sign": "Mso8hFn6t2z0h/doCgXaA1qIAnf56RsuzHKuUKJFy9O1u6shuGQnNtPigka+0E7tlrpiPw/K9CAx2dLmAqKSDo8R1C71bQMBk5j7cwuYxXea3qehCrRjz3ZfOChntY3Cpkfg+eX0fuuX5FwgpFs4gKYLryidBlgAVVNsMHnsdWI="
	},
	"4": {
		"sequence": "4",
		"token": "0x6982eCD13f80034823Ed7FBb7139b70D2c7eD737",
		"sign": "i0WQmVfCU3HB8F5rieFWv1g3pvJV8LFaO5CtvH9lAofPKDNXhfehBM1r2LMZHY0763bBd97o5HZywly/92+UZZolKtJm3dGtkUgqicIh0ApxfGsHF/g/kcpVQzDCn/22TlxIdA6CkH5vVRYcUw5k0/BWpTDj63+UlNk1wvrQA6g="
	},
	"5": {
		"sequence": "5",
		"token": "0xBAF5238bbd4A6083345Bb9348f5d22af05eCeb68",
		"sign": "aoHt2HhpE9gfJ4FYk0W9Dr0j7IGz3ukwUZEUBs/E6K53WDeB3iXuzfi1nGxuugYQIdTOUG8JCxreFq6Z48L1cSX1cfwDCdnDnUOv8jM1p6eFyPVVFjL4SIOe4jbD3pyz/txQW9y/F0G1tMABWgUbLPZiDL9l3SlbWzamJQENaJ4="
	},
	"6": {
		"sequence": "6",
		"token": "0xEA60932fdEc4d31D9C244e881514F189A374A2D4",
		"sign": "PO6Wz+V/+0A/R+Sq1IDv5YrPy+XX+H4Bza8cSMNYxBe6KrO3QSje9KsXQLH/JJRDRGPUPhvgNDA10RtKJV+vEM/KkJgvhJ0wquhnj8Y9UD8s3BcDY/7AhjdH9MQQ8LRs8w0kNILfgZe4x9GT8Ry5aZXeAHwuuBuJJpbAslPaJks="
	},
	"7": {
		"sequence": "7",
		"token": "0x47AEc4596643f071a4538150D78d680425693ef4",
		"sign": "GX+e9btvbAPAKdq0J8hfT/KWq98Rz2FQnptFT0sOf7I9RSenS4GbfgqciHD9mk7BD7S6BRLbSJFJP+UnOtRlrybSi1LOcObbUlBzwgf7VZH9XMJbC50D6f5ENsgkPghQoNC26rg/Aw00g5glEENngoVTIVAClgRpjkPcp4j1E5c="
	},
	"8": {
		"sequence": "8",
		"token": "0xE6d3EB60b7d190EABD0119c11dE66FD27bc642Ee",
		"sign": "FCQ6ApeJejUwFJRLLbpmeb2/gefptaMVrglGNOkrnZ5GZ/cl1k2Cb85kjTnTxCsWuaJNUF0HNq3J6lAeR7yFk7qLBiCXA/Ep+yYvfDeptgywpxuNvqvI+fQvGjQXPlpyht/uhOMv839u4WaeXIuaRxtdiRXEan3aPw9vlmVw2bA="
	},
	"9": {
		"sequence": "9",
		"token": "0x951E0D9bA7AAA7B6207fee35Aa2Cf6402aD054F6",
		"sign": "hx9WOpDMPy7D/++eKcklh+Wcu+atNofS4/w8ygtLC0O5JfNFhNlHuUiORo+YE9jN6VwySTNHts/+Q8+OKLY8uTm0rzel8SsL0WNio8LSUyxeLV7wZzzy7CZ4X4wO+X5V6nZ6z4+ri4rnUR1H7FWSn0SRIL/sRGwAqf9rlBhnQy4="
	}
}
```

#### 网络传输数据 
. 在DEMO中是使用了 http协议的api接口，正式生产环境中预计会提供 jsonRPC 接口

#### 服务端操作步骤：
1. 检查消息是否过期（1小时前的消息，视为过期）
2. 对传输过来的消息进行RSA验签，供票方（appid）+时间戳（time_stamp）
3. 接收json对象包含的数据，存盘做后续的处理。异步方式反馈每张票的流通情况。

##### 传出的数据格式
```
{
	"1": {
		"path": "m/44'/60'/1'/0/1",
		"token": "0xAc85f833BfAC840eF2AAfD6006c5D51d8CF4848E"
	},
	"2": {
		"path": "m/44'/60'/1'/0/2",
		"token": "0x6A72771cCF1ce5eCd8e10C2a66EC5f7607dAcC1b"
	},
	"3": {
		"path": "m/44'/60'/1'/0/3",
		"token": "0x0d2539E0fc299965613cD52A8fBe5f1137a01BeD"
	},
	"4": {
		"path": "m/44'/60'/1'/0/4",
		"token": "0x6982eCD13f80034823Ed7FBb7139b70D2c7eD737"
	},
	"5": {
		"path": "m/44'/60'/1'/0/5",
		"token": "0xBAF5238bbd4A6083345Bb9348f5d22af05eCeb68"
	},
	"6": {
		"path": "m/44'/60'/1'/0/6",
		"token": "0xEA60932fdEc4d31D9C244e881514F189A374A2D4"
	},
	"7": {
		"path": "m/44'/60'/1'/0/7",
		"token": "0x47AEc4596643f071a4538150D78d680425693ef4"
	},
	"8": {
		"path": "m/44'/60'/1'/0/8",
		"token": "0xE6d3EB60b7d190EABD0119c11dE66FD27bc642Ee"
	},
	"9": {
		"path": "m/44'/60'/1'/0/9",
		"token": "0x951E0D9bA7AAA7B6207fee35Aa2Cf6402aD054F6"
	}
}
```

### 安装教程

1.  用 python rsa_key_gen.py 在工作目录下生成4个公私钥对
2.  用 python server.py 启动服务端
3.  用 python client.py 启动客户端 

#### 使用说明

1.  服务端和客户端分别在两个不同的终端窗口打开
2.  在 python 3.10.5 环境下测试通过
3.  pip install bip44  
4.  pip install flask

##### 其他需要说明的内容
- DEMO中的助记词是用 purity tunnel grid error scout long fruit false embody caught skin gate
- 不同的助记词会生成不同是公私钥，但从公钥扩展到子公钥，从公钥生成地址的算法不变。

### 参与贡献

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request


### 特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  Gitee 官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解 Gitee 上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是 Gitee 最有价值开源项目，是综合评定出的优秀开源项目
5.  Gitee 官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  Gitee 封面人物是一档用来展示 Gitee 会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)
