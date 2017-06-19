# OAUTH学习笔记

## WHY
为了让第三方应用获取资源服务器上用户所属资源，同时避免把用户名和密码交给第三方来登录资源服务器，而在这两者之间加入一层授权层(authorization layer)。

##协议
在HTTP/HTTPS基础之上

##角色
* Resource Owner：用户
* Resource Server：资源服务器
* Client：客户端(PC/APP)
* Authorization Server:授权服务器
* Protected Resource:受保护资源

*ps:Authorization Server和Resource Server可以是同一个*

## 基本流程
	+--------+                               +---------------+
	|        |--(A)- Authorization Request ->|   Resource    |
	|        |                               |     Owner     |
	|        |<-(B)-- Authorization Grant ---|               |
	|        |                               +---------------+
	|        |
	|        |                               +---------------+
	|        |--(C)-- Authorization Grant -->| Authorization |
	| Client |                               |     Server    |
	|        |<-(D)----- Access Token -------|               |
	|        |                               +---------------+
	|        |
	|        |                               +---------------+
	|        |--(E)----- Access Token ------>|    Resource   |
	|        |                               |     Server    |
	|        |<-(F)--- Protected Resource ---|               |
	+--------+                               +---------------+
	
	                Figure 1: Abstract Protocol Flow

## 术语定义
* Authorization Grant:client获取授权的方式
	* Authorization Code Grant Type Flow（授权码模式）
	* Implicit Grant Type Flow（简化模式）
	* Resource Owner Password Credentials Grant Type Flow（密码模式）
	* client credentials（客户端模式）
* Access Token:用来操作Protected Resource的唯一凭证，具体定义在RFC 6750
* Access Token Type:Access Token的类型，具体定义在RFC 6750
* Rrefresh Token:用来重新获取Access Token的Token,一般用在移动端，避免用户的多次授权而影响用户体验，必须随Access Token一并核发给client，具体流程
		
		+--------+                                           +---------------+
		|        |--(A)------- Authorization Grant --------->|               |
		|        |                                           |               |
		|        |<-(B)----------- Access Token -------------|               |
		|        |               & Refresh Token             |               |
		|        |                                           |               |
		|        |                            +----------+   |               |
		|        |--(C)---- Access Token ---->|          |   |               |
		|        |                            |          |   |               |
		|        |<-(D)- Protected Resource --| Resource |   | Authorization |
		| Client |                            |  Server  |   |     Server    |
		|        |--(E)---- Access Token ---->|          |   |               |
		|        |                            |          |   |               |
		|        |<-(F)- Invalid Token Error -|          |   |               |
		|        |                            +----------+   |               |
		|        |                                           |               |
		|        |--(G)----------- Refresh Token ----------->|               |
		|        |                                           |               |
		|        |<-(H)----------- Access Token -------------|               |
		+--------+           & Optional Refresh Token        +---------------+
		
		        Figure 2: Refreshing an Expired Access Token


## 外部环境依赖
* 全站使用TLS（HTTPS）
* User-Agent支持HTTP Redireaction(302)
* 规定Protected Resource的存取方式
* 定义详细的错误码以及错误信息

## Client的注册和认证

### 分类（基于保密能力） 
* public
* confidential

### 注册提供的基本信息
* Client Type
* Redirection URL
* name,index,logo。。。。

### Client Type
* public
	* User-Agent-based Application(browser puligins)
	* Native Application
* confidential
	* web application

### Client认证方式

#### 两个要素
* Client_id(保证唯一性）
* Client_secret

#### 认证方式
* HTTP Basic Auth

		Step 1 : client_id:client_secret
		Step 2 : base64_encode(Step1)
		Stpe 3 : Authorization: Basic Step2

* POST/GET

		client_id
		client_secret

**PS:1.不建议用POST/GET形式；2.要经过TLS；3.实际要考虑两种情况一起使用**


### Native Application

#### User-Agent
* 外部User-Agent
* 内部User-Agent

#### Grant Flow选择
* Authorization Code Grant 不要保存Client Credentials
* Implicit Grant 不要使用Refresh Token

### 安全性问题
* Client认证的安全性
* 伪造Client
* Open Redirector

## Endpoints
规定在互传Token的时候URI的格式，并不涉及到Resource Server

### 分类
* Authorization Server的Authorization Endpoint
* Authorization Server的Token Endpoint
* Client的Redirection Endpoint

**大致使用顺序：Authorization Endpoint → Redirection Endpoint → Token Endpoint**

### Authorization Endpoint
用户在授权时候，发给Authorization Server的请求

#### URI
可以有Query Component(`?xxx=yyy`)，不能有Fragment Component（`#zzz`）

#### HTTP Method
必须用GET，非必需用POST，要求使用HTTPS

#### Params
|参数 |是否必须 |含义 |可选值 |附加说明 |
|:---:|:------:|:---:|:----:|:------:|
|response_type |必 |指定grant的类型 |coe,token,。。 |不得欠缺，否则报错|
|state |建议有 |维持通信状态 |随意 |防CSRF攻击,维持Endpoint之间的状态|
|scope |可选 | 指定存取范围 |随意 |      

### Redirection Endpoint
交互code，acces token时候，回调地址的要求，必须要求Client接入时候预先设定，建议使用HTTPS

#### URI
scheme + hierarchical + query(可选,见下图),可以有Query Component(`?xxx=yyy`)，不能有Fragment Component（`#zzz`）

	  foo://username:password@example.com:8042/over/there/index.dtb?type=animal&name=narwhal#nose
	  \_/   \_______________/ \_________/ \__/            \___/ \_/ \______________________/ \__/
	   |           |               |       |                |    |            |                |
	   |       userinfo         hostname  port              |    |          query          fragment
	   |    \________________________________/\_____________|____|/ \__/        \__/
	   |                    |                          |    |    |    |          |
	scheme              authority                    path   |    |    interpretable as keys
	 name   \_______________________________________________|____|/       \____/     \_____/
	                             |                          |    |          |           |
	                     hierarchical part                  |    |    interpretable as values
	                                                        |    |
	                                interpretable as filename    interpretable as extension

### Token Endpoint
拿去Access Token时候， Client发个Authorization Server的请求， Implicit Grant Type不需要使用

#### HTTP Method
必须使用POST，且要使用HTTPS

#### URI
可以有Query Component(`?xxx=yyy`)，不能有Fragment Component（`#zzz`）

#### Params
|参数 |是否必须 |含义 |可选值 |附加说明 |
|:--:|:------:|:---:|:----:|:------:|
|grant_type |必 |表明grant |authorization_code,password,client_credentials,refresh_token |必须指明，否则报错|
|state |建议有 |维持通信状态 |随意 |防CSRF攻击,维持Endpoint之间的状态|
|scope |可选 | 指定存取范围 |随意 |  

## Authorization Code Grant Flow

### 流程图

	+----------+
	| Resource |
	|   Owner  |
	|          |
	+----------+
	     ^
		 |`
	    (B)
	+----|-----+          Client Identifier      +---------------+
	|         -+----(A)-- & Redirection URI ---->|               |
	|  User-   |                                 | Authorization |
	|  Agent  -+----(B)-- User authenticates --->|     Server    |
	|          |                                 |               |
	|         -+----(C)-- Authorization Code ---<|               |
	+-|----|---+                                 +---------------+ 
	  |    |                                         ^      v 
	 (A)  (C)                                        |      |
 	  |    |                                         |      |
	  ^    v                                         |      |
	+---------+                                      |      |
	|         |>---(D)-- Authorization Code ---------'      |
	|  Client |          & Redirection URI                  |
	|         |                                             |	
	|         |<---(E)----- Access Token -------------------'
	+---------+       (w/ Optional Refresh Token)
	
	註: (A), (B), (C) 這三步的線拆成兩段，因為會經過 user-agent

                  Figure 3: Authorization Code Flow

### Authorization Request
Client向Authorization Server发送请求，要求返回Authorization Code的过程

|请求方法|
|:-----:|
|GET    |

|请求参数|是否必选|意义 |
|:-----:|:-----:|:---:|
|response_type |必选 |code |
|client_id |必选 |Client_id |
|state |建议有 |内部状态 |
|redirect_uri |可选 |回调地址 |
|scope |可选 |存取范围 |

### Authorization Response
|请求头|参数|
|:---:|:--:|
|state |302 |
|location |url |

|返回参数|是否必选|意义 |
|:-----:|:-----:|:---:|
|code |必选 |code |
|state |建议有 |内部状态 |

#### 备注
* code生存时间必须短
* Code只能使用一次
* 要绑定Code<->Client ID<->Redirection URI关系

#### 例子
	HTTP/1.1 302 Found
	Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
	          &state=xyz

### Access Token Request
|请求方法|
|:-----:|
|POST   |

|请求参数|是否必选|意义 |
|:-----:|:-----:|:---:|
|grant_type |必选 |authorization_code |
|code |必选 |code |
|redirect_uri |可选 |回调地址， 必须与Authorization Request一模一样|
|client id |可选 |只有public Client才必填 |

#### Authoriztion Server的处理过程
* 认证Client
* 认证Authorization Code，必须是发给该Client的
* 验证Redirection URI

#### 例子
	POST /token HTTP/1.1
	Host: server.example.com
	Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
	Content-Type: application/x-www-form-urlencoded
	
	grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
	&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb

### Access Token Response
	
	HTTP/1.1 200 OK
	Content-Type: application/json;charset=UTF-8
	Cache-Control: no-store
	Pragma: no-cache
	
	{
	  "access_token":"2YotnFZFEjr1zCsicMWpAA",
	  "token_type":"example",
	  "expires_in":3600,
	  "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
	  "example_parameter":"example_value"
	}


## Issuing Access Token

### Status Code: `200 OK`

### Header
* `Pragma: no-cache`
* `Cache-Control: no-store`

### Response Body： JSON

### Response Param
|参数|是否必选|意义 |
|:-----:|:-----:|:---:|
|access_token |必选 |由AS核发的Access Token |
|token_type |必选 |token类型，如Bearer |
|expires_in |建议有 |过期时间 |
|scope |可选 |如果和申請的不同則要附上，如果一樣的話就不必附上 |
|refresh token |可选 |refresh token |

## Refreshing Access Token

### Request Method: POST

### Request Param
|参数|是否必选|意义 |
|:-----:|:-----:|:---:|
|grant_type |必选 |refresh token |
|refresh_token |必选 |refresh_token |
|scope |建议有 |	申請的存取範圍 |

### Authoriztion Server的处理过程
* 认证Client
* 认证Refresh Token，必须是发给该Client的
* 验证Refresh Token正确性

### 例子
	POST /token HTTP/1.1
	Host: server.example.com
	Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
	Content-Type: application/x-www-form-urlencoded
	
	grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA

## Bearer Token
Access Token一种类型

### 格式
	Bearer b64token
	b64token = 1*( ALPHA / DIGIT / "-" / "." / "_" / "~" / "+" / "/" ) *"="
	正则表达式
	/[A-Za-z0-9\-\._~\+\/]+=*/

### 传输方式
1. HTTP Header
	
		GET /resource HTTP/1.1
		Host: server.example.com
		Authorization: Bearer mF_9.B5f-4.1JqM

2. Request Body

		POST /resource HTTP/1.1
		Host: server.example.com
		Content-Type: application/x-www-form-urlencoded
		
		access_token=mF_9.B5f-4.1JqM

3. Query Parameter

		GET /resource?access_token=mF_9.B5f-4.1JqM HTTP/1.1
		Cache-Control: no-store/Cache-Control: private
		Host: server.example.com

## Oauth相关安全问题
### 应用方
* 授权认证时的参数选择不当
* CSRF劫持
* 账号体系固有的设计权限

### 平台方
* Implicit Grant场景
* 高权限的client_id和client_secret泄露
* 协议实现不全面
* Webview不信任
* HTTPS信任

## 参考链接
* [OAuth 2.0 笔记](https://blog.yorkxin.org/2013/09/30/oauth2-1-introduction "OAuth 2.0 笔记")
* [腾讯开发平台](http://wiki.open.qq.com/wiki/website/OAuth2.0%E7%AE%80%E4%BB%8B "腾讯开发平台")
* [理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html "理解OAuth 2.0")
* [OAuth的原理认证流程及访问资源流程](http://blog.csdn.net/diyagoanyhacker/article/details/6459210 "OAuth的原理认证流程及访问资源流程")
* [rfc6749：The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749#page-43 "rfc6749：The OAuth 2.0 Authorization Framework")
* [rfc6750：The OAuth 2.0 Authorization Framework: Bearer Token Usage0](https://tools.ietf.org/html/rfc6750 "rfc6750：The OAuth 2.0 Authorization Framework: Bearer Token Usage")
* [OAuth 2.0安全案例回顾](php.ph/wydrops/drops/OAuth%202.pdf "OAuth 2.0安全案例回顾")
