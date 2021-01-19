# 1. 接入指南

sdk技术问题沟通QQ群：609994083</br>

**注意事项：**

1. **一键登录服务必须打开蜂窝数据流量并且手机操作系统给予应用蜂窝数据权限才能使用**
2. **取号请求过程需要消耗用户少量数据流量（国外漫游时可能会产生额外的费用）**
3. **一键登录服务目前支持中国移动2/3/4/5G（2,3G因为无线网络环境问题，时延和成功率会比4G低） 和中国电信、中国联通4G（如有更新会在技术沟通QQ群上通知）**

## 1.1. 接入流程

**1.申请appid和appkey**

根据《开发者接入流程文档》，前往中国移动开发者社区（dev.10086.cn)，按照文档要求创建开发者账号并申请appid和appkey，并填写应用的包名和包签名。

**2.申请能力**

应用创建完成后，在能力配置页面上，勾选应用需要接入的能力类型，如一键登录，并配置应用的服务器出口IP地址。（如果在服务端需要用非对称加密方法对一些重要信息进行加密处理，请在能力配置页面填写RSA加密的公钥）

**3.添加appid白名单**

开发者在完成步骤1和步骤2后，将appid提供给移动认证工作人员，移动方将在2个工作日内将appid加入白名单，白名单添加完毕，开发者就可以开始联调对接。

**4.上线审核**

应用上线前，开发者需要将一键登录取号能力的场景所使用的授权页面（授权页面参考授权页面规范）提供给移动认证产品接口人，审核无误后可正式上线。

## 1.2. 开发流程

**第一步：获取SDK及相关文档**

请在业务联调群中联系移动认证运营获取最新的SDK包和文档

**第二步：搭建开发环境**

jar包集成方式：

1. 在Eclipse/AS中建立你的工程。 
2. 将`*.jar`拷贝到工程的libs目录下，如没有该目录，可新建。



远程依赖方式：

如果使用android studio进行开发，在app的主module的build.gradle中加入依赖配置：

```java
implementation 'com.cmictop.sso:sdk:x.x.x'
```

注：其中x.x.x是一键登录对应的SDK的版本号，例如9.0.7

**第三步：开始使用移动认证SDK**

**[1] AndroidManifest.xml设置**

必要的权限: 

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
```

建议的权限：

```java
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

<font color="red"><strong>本权限主要用于在双卡情况下，更精准的获取数据流量卡的运营商类型，缺少该权限，存在取号失败概率上升的风险。</strong></font>

权限说明：

| 权限                 | 说明                                       |
| -------------------- | ------------------------------------------ |
| INTERNET             | 允许应用程序联网，用于访问网关和认证服务器 |
| READ_PHONE_STATE     | 获取imsi用于判断双卡和换卡                 |
| ACCESS_WIFI_STATE    | 允许程序访问WiFi网络状态信息               |
| ACCESS_NETWORK_STATE | 获取网络状态，判断是否数据、wifi等         |
| CHANGE_NETWORK_STATE | 允许程序改变网络连接状态                   |

**[2] 创建一个AuthnHelper实例。**

`AuthnHelper`是SDK的功能入口，所有的接口调用都得通过AuthnHelper进行调用。因此，调用SDK，首先需要创建一个AuthnHelper实例

**示例代码：**

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }
```
**方法原型1：**（开发者默认使用该原型初始化SDK）

```java
public static AuthnHelper getInstance(Context context) 
```

**方法原型2：**

```java
public static AuthnHelper getInstance(Context context, String encrypType) 
```

**参数说明：**

| 参数       | 类型    | 说明                                               |
| ---------- | ------- | -------------------------------------------------- |
| context    | Context | 调用者的上下文环境，其中activity中this即可以代表。 |
| encrypType | string  | 暂时不传                                           |

**[3] 实现回调。**

所有的SDK接口调用，都会传入一个回调，用于接收SDK返回的调用结果。结果以`JsonObject`的形式传递，`TokenListener`的实现示例代码如下：

<font color="red">注意：在9.0.5以及以后版本，该回调方法会被sdk抛到主线程中调用</font>

```java
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        if (jObj != null) {
            mResultString = jObj.toString();
            mResultDialog.setResult(StringFormat.logcatFormat(mResultString));
            if (jObj.has("token")) {
                mtoken = jObj.optString("token");
            }
        }
    }
};
```
**[4] 混淆策略**

请避免混淆一键登录SDK，在Proguard混淆文件中增加以下配置：

```java
-dontwarn com.cmic.sso.sdk.**
-keep class com.cmic.sso.sdk.** {*;}
```

<div STYLE="page-break-after: always;"></div>

# 2. 一键登录&本机号码校验

## 2.1. 准备工作

在中国移动开发者社区进行以下操作：

1. 获得appid和appkey、APPSecret（服务端）；
2. 勾选一键登录能力；
3. 配置应用服务器的出口ip地址
4. 配置公钥（如果使用RSA加密方式）
5. **针对本机号码校验：**勾选本机号码校验短验辅助开关（可选）
6. <font color="red">商务对接签约（未签约应用使用体验版套餐，每个appid每天只能调用1000次，三个月到期）</font>

## 2.2. 流程说明 

![](\image\login_process.png)

## 2.3. 取号请求

本方法用于发起取号请求，SDK完成网络判断、蜂窝数据网络切换等操作并缓存凭证scrip。<font color="red">缓存允许用户在未开启蜂窝网络时成功取号。</font>

**取号方法原型：**

```java
public void getPhoneInfo(final String appId, 
                         final String appKey, 
                         final TokenListener listener)
```

**参数说明：**

| 参数     | 类型          | 说明                                                         |
| :------- | :------------ | :----------------------------------------------------------- |
| appId    | String        | 应用的AppID，在开发者社区创建应用时获取                      |
| appKey   | String        | 应用密钥，在开发者社区创建应用时获取                         |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**返回说明：**

OnGetTokenComplete的参数JSONObject，含义如下：

| 参数          | 类型   | 说明                          |
| ------------- | ------ | ----------------------------- |
| resultCode    | String | 接口返回码，“103000”为成功。  |
| operatorType | String        | 运营商类型：</br>未知；</br>移动；</br>联通；</br>电信 |
| desc          | String | 成功标识，true为成功。        |
| securityphone | String | 手机号码掩码，如“138XXXX0000” |
| scripExpiresIn | String | 缓存剩余有效时间，单位，秒，成功时返回 |

**示例代码：**

```java
//创建AuthnHelper实例
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }

//实现取号回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};

//调用取号方法
mAuthnHelper.getPhoneInfo(APP_ID, APP_KEY, mListener);
```

## 2.4. 创建授权页（一键登录）

为了确保用户在登录过程中将手机号码信息授权给开发者使用的知情权，一键登录需要开发者提供授权页登录页面供用户授权确认。开发者在调用授权登录方法前，必须弹出授权页，明确告知用户当前操作会将用户的本机号码信息传递给应用。授权页面的设计、布局、生成、弹出和消失，由开发者自行处理，但必须遵守移动认证授权页面设计规范。

注1：如果开发者需要使用**一键登录**服务，必须按照规定创建授权页面

注2：**本机号码校验**不需要创建授权页面，可以直接跳过2.4章

### 2.4.1. 页面规范细则

1、页面必须包含登录/注册按钮，授权登录方法`loginAuth`必须绑定该按钮使用。

2、登录按钮文字描述必须包含“登录”或“注册”等文字，不得诱导用户授权。

3、页面需要提示应用获取到的是用户的本机号码，例如，可以在页面显示本机号码的掩码（securityphone），或者提示用户将使用“本机号码”作为账号登录或注册。

4、页面必须包含移动认证协议条款，其中：
* 移动：
	* 条款名称：《中国移动认证服务条款》
	* 条款页面地址：https://wap.cmpassport.com/resources/html/contract.html 
* 电信：
	* 协议名称：《中国电信天翼账号服务条款》
	+ 协议链接：https://e.189.cn/sdk/agreement/detail.do
* 联通：
	* 协议名称：《中国联通认证服务协议》
	+ 协议链接：https://opencloud.wostore.cn/authz/resource/html/disclaimer.html?fromsdk=true

5、应用在上线前需将满足上述1~4的授权页面（正式上线版的）截图提供给产品接口人审核。

6、应用后续升级时，如果授权页面有较大改动（针对1~4内容进行修改），需将改动的授权页面截图提供给产品接口人审核。

7、对于未遵照1~4设计要求，或通过技术手段故意屏蔽不弹出授权页面但获得调用接口凭证token的行为，能力提供方有权限制APP一键登录取号能力的使用，待整改后再恢复提供服务。


## 2.5. 授权请求

取号请求成功后，应用在等待用户授权后，调用授权请求方法，获取取号token。

<font color="red">注意：务必在授权页上得到APP使用者的授权同意才允许调用此授权请求方法！</font>

**请求示例代码：**

```java
//1.调用取号方法getPhoneInfo

//2.实现回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};


//3.用户点击授权按钮时，调用loginAuth方法，传入应用定义的uaAuthactivity，获取token
    mAuthnHelper.loginAuth(Constant.APP_ID, Constant.APP_KEY, mListener);
    ……
```

**授权方法原型：**

```java
public void loginAuth(final String appId, 
                      final String appKey, 
                      final TokenListener listener)
```

**参数说明：**

| 参数     | 类型          | 说明                                                         |
| :------- | :------------ | :----------------------------------------------------------- |
| appId    | String        | 应用的AppID                                                  |
| appkey   | String        | 应用密钥                                                     |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**返回说明：**

TokenListener的参数JSONObject，含义如下：

| 参数          | 类型   | 说明                                                         |
| ------------- | ------ | ------------------------------------------------------------ |
| resultCode    | String | 接口返回码，“103000”为成功                                   |
| authType      | String | 登录方式：1：WIFI下网关鉴权</br>2：网关鉴权</br>3：其他      |
| authTypeDes   | String | 登录方式描述                                                 |
| securityphone | String | 手机号码掩码，如“138XXXX0000”                                |
| token         | String | 成功时返回：临时凭证，token有效期2min，一次有效；同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |
| scripExpiresIn | String | 缓存剩余有效时间，单位，秒，成功时返回。注意：电信联通卡调用本方法时，scripExpiresIn=0 |
| tokenExpiresIn | String | token剩余有效时间，单位，秒，成功时返回 |

## 2.6. 获取手机号码（服务端）

详细请开发者查看移动认证服务端接口文档说明。

## 2.7. 本机号码校验请求token

开发者可以在应用内部任意页面调用本方法，获取本机号码校验的接口调用凭证（token）

注意：通过本方法获取到的token，只能请求<font color="red">本机号码校验服务端接口</font>，不能请求<font color="red">获取手机号码接口</font>

**本机号码校验方法原型**

```java
public void mobileAuth(final String appId, 
                       final String appKey, 
                       final TokenListener listener)
```

**请求参数说明：**

| 参数        | 类型          | 说明                                                         |
| :---------- | :------------ | :----------------------------------------------------------- |
| appId       | String        | 应用的AppID                                                  |
| appkey      | String        | 应用密钥                                                     |
| listener    | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**响应参数：**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段        | 类型   | 含义                                                         |
| ----------- | ------ | ------------------------------------------------------------ |
| resultCode  | String | 接口返回码，“103000”为成功。                                 |
| authType    | String | 登录类型。                                                   |
| authTypeDes | String | 登录类型中文描述。                                           |
| token       | String | 成功返回:临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |
| scripExpiresIn | String | 缓存剩余有效时间，单位，秒，成功时返回。注意：电信联通卡调用本方法时，scripExpiresIn=0 |
| tokenExpiresIn | String | token剩余有效时间，单位，秒，成功时返回 |


## 2.8. 本机号码校验（服务端）

详细请开发者查看移动认证服务端接口文档说明。

# 3. SDK方法说明

## 3.1. 获取管理类的实例对象

获取管理类的实例对象

**原型**

```java
public static AuthnHelper getInstance(Context context)
```

**请求参数**

| 参数      | 类型      | 说明                              |
| ------- | ------- | ------------------------------- |
| context | Context | 调用者的上下文环境，其中activity中this即可以代表。 |

## 3.2. 取号请求

本方法用于发起取号请求，SDK完成网络判断、蜂窝数据网络切换等操作并缓存凭证scrip。

**请求参数**

| 参数       | 类型            | 说明                                       |
| :------- | :------------ | :--------------------------------------- |
| appId    | String        | 应用的AppID                                 |
| appkey   | String        | 应用密钥                                     |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**响应参数**

OnGetTokenComplete的参数JSONObject，含义如下：

| 参数          | 类型   | 说明                          |
| ------------- | ------ | ----------------------------- |
| resultCode    | String | 接口返回码，“103000”为成功。  |
| operatorType | String        | 运营商类型：</br>未知；</br>移动；</br>联通；</br>电信 |
| desc          | String | 成功标识，true为成功。        |
| securityphone | String | 手机号码掩码，如“138XXXX0000” |
| scripExpiresIn | String | 缓存剩余有效时间，单位，秒，成功时返回 |

**请求示例代码**

```java
//创建AuthnHelper实例
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }

//实现取号回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};

//调用取号方法
mAuthnHelper.getPhoneInfo(APP_ID, APP_KEY, mListener);
```

**响应示例代码**

```
{
	"resultCode":"103000",
	"desc":"true",
	"securityphone:"138XXXX0000"
}
```

## 3.3. 授权请求

取号请求成功后，应用在等待用户授权后，调用授权请求方法，获取取号token

**授权方法原型：**

```java
public void loginAuth(final String appId, 
                      final String appKey, 
                      final TokenListener listener)
```

**参数说明：**

| 参数     | 类型          | 说明                                                         |
| :------- | :------------ | :----------------------------------------------------------- |
| appId    | String        | 应用的AppID                                                  |
| appkey   | String        | 应用密钥                                                     |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**返回说明：**

TokenListener的参数JSONObject，含义如下：

| 参数          | 类型   | 说明                                                         |
| ------------- | ------ | ------------------------------------------------------------ |
| resultCode    | String | 接口返回码，“103000”为成功                                   |
| authType      | String | 登录方式：1：WIFI下网关鉴权</br>2：网关鉴权</br>3：其他      |
| authTypeDes   | String | 登录方式描述                                                 |
| securityphone | String | 手机号码掩码，如“138XXXX0000”                                |
| token         | String | 成功时返回：临时凭证，token有效期2min，一次有效；同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |
| scripExpiresIn | String | 缓存剩余有效时间，单位，秒，成功时返回。注意：电信联通卡调用本方法时，scripExpiresIn=0 |
| tokenExpiresIn | String | token剩余有效时间，单位，秒，成功时返回 |

**请求示例**

```java
//1.调用取号方法getPhoneInfo

//2.实现回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};


//3.用户点击授权按钮时，调用loginAuth方法，传入应用定义的uaAuthactivity，获取token
    mAuthnHelper.loginAuth(Constant.APP_ID, Constant.APP_KEY, mListener);
    ……
```

## 3.4. 设置取号超时

设置取号超时时间，默认为8000毫秒。

开发者设置取号请求方法（getPhoneInfo）、授权请求方法（loginAuth），本机号码校验请求token方法（mobileAuth）的超时时间。开发者在使用SDK方法前，可以通过本方法设置将要使用的方法的超时时间。

**原型**

```java
public void setOverTime(long overTime)
```

**请求参数**

| 参数     | 类型 | 说明                       |
| -------- | ---- | -------------------------- |
| overTime | long | 设置超时时间（单位：毫秒） |

**响应参数**

无



## 3.5. 获取网络状态和运营商类型

本方法用于获取用户当前的网络环境和运营商

**原型**

```java
public JSONObject getNetworkType(Context context)
```

**请求参数**

| 参数    | 类型    | 说明       |
| ------- | ------- | ---------- |
| context | Context | 上下文对象 |

**响应参数**

参数JSONObject，含义如下：

| 参数         | 类型   | 说明                                                         |
| ------------ | ------ | ------------------------------------------------------------ |
| operatorType | String        | 运营商类型：</br>0.未知；</br>1.移动流量；</br>2.联通流量；</br>3.电信流量 |
| networktype  | String | 网络类型：</br>0.未知；</br>1.流量；</br>2.wifi；</br>3.数据流量+wifi |

## 3.6. 删除临时取号凭证

开发者取号或者授权成功后，SDK将取号的一个临时凭证缓存在本地，<font color="red">缓存允许用户在未开启蜂窝网络时成功取号。</font>开发者可以使用本方法删除该缓存凭证。

注意1：删除临时取号凭证后，下次取号请求将不再使用缓存登录，SDK会再次发起网关请求取号。

注意2：用户更换SIM卡或者流量卡变更时，缓存将作废，SDK会再次发起网关请求取号。

**原型**

```java
public void delScrip()
```

## 3.7. 本机号码校验请求token

开发者可以在应用内部任意页面调用本方法，获取本机号码校验的接口调用凭证（token）

**本机号码校验方法原型**

```java
public void mobileAuth(final String appId, 
                       final String appKey, 
                       final TokenListener listener)
```

**请求参数说明：**

| 参数        | 类型          | 说明                                                         |
| :---------- | :------------ | :----------------------------------------------------------- |
| appId       | String        | 应用的AppID                                                  |
| appkey      | String        | 应用密钥                                                     |
| listener    | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**响应参数：**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段        | 类型   | 含义                                                         |
| ----------- | ------ | ------------------------------------------------------------ |
| resultCode  | String | 接口返回码，“103000”为成功。                                 |
| authType    | String | 登录类型。                                                   |
| authTypeDes | String | 登录类型中文描述。                                           |
| token       | String | 成功返回:临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |
| scripExpiresIn | String | 缓存剩余有效时间，单位，秒，成功时返回。注意：电信联通卡调用本方法时，scripExpiresIn=0 |
| tokenExpiresIn | String | token剩余有效时间，单位，秒，成功时返回 |



<div STYLE="page-break-after: always;"></div>

# 4. 返回码说明

## 4.1. SDK返回码

| 返回码 | 返回码描述                                                   |
| ------ | ------------------------------------------------------------ |
| 103000 | 成功                                                         |
| 102101 | 无网络                                                       |
| 102102 | 网络异常                                                     |
| 102103 | 未开启数据网络                                               |
| 102203 | 输入参数错误                                                 |
| 102223 | 数据解析异常，一般是卡欠费                                             |
| 102508 | 数据网络切换失败                                             |
| 103101 | 请求签名错误（若发生在客户端，可能是appkey传错，可检查是否跟appsecret弄混，或者有空格。若发生在服务端接口，需要检查验签方式是MD5还是RSA，如果是MD5，则排查signType字段，若为appsecret，需确认是否误用了appkey生签。如果是RSA，需要检查使用的私钥跟报备的公钥是否对应和报文拼接是否符合文档要求。）                        |
| 103102 | 包签名错误（社区填写的appid和对应的包名包签名必须一致）      |
| 103111 | 错误的运营商请求（可能是用户正在使用代理或者运营商判断失败导致） |
| 103119 | appid不存在（检查传的appid是否正确或是否有空格）                                     |
| 103211 | 其他错误，（常见于报文格式不对，先请检查是否符合这三个要求：a、json形式的报文交互必须是标准的json格式；b、发送时请设置content type为 application/json；c、参数类型都是String。如有需要请联系移动认证开发）     |
| 103412 | 无效的请求（1.加密方式错误；2.非json格式；3.空请求等）       |
| 103414 | 参数校验异常                                                 |
| 103511 | 服务器ip白名单校验失败                                       |
| 103811 | token为空                                                    |
| 103902 | scrip失效（短时间内重复登录）                                |
| 103911 | token请求过于频繁，10分钟内获取token且未使用的数量不超过30个 |
| 104201 | token已失效或不存在（重复校验或失效）                        |
| 105001 | 联通取号失败                                                 |
| 105002 | 移动取号失败（一般是物联网卡）                              |
| 105003 | 电信取号失败                                                 |
| 105012 | 不支持电信取号                                               |
| 105013 | 不支持联通取号                                               |
| 105018 | token权限不足（使用了本机号码校验的token获取号码）           |
| 105019 | 应用未授权（未在开发者社区勾选能力）                         |
| 105021 | 当天已达取号限额                                             |
| 105302 | appid不在白名单                                              |
| 105312 | 余量不足（体验版到期或套餐用完）                             |
| 105313 | 非法请求                                                     |
| 105315 | 不支持的运营商类型                                                     |
| 105317 | 受限用户                                                     |
| 200010 | 无法识别sim卡或没有sim卡                                     |
| 200023 | 请求超时                                                     |
| 200025 | 其他错误（socket、系统未授权数据蜂窝权限等，如需要协助，请联系移动认证开发）       |
| 200050 | EOF异常（iOS:Socket创建或发送接收数据失败）                  |
| 200072 | 证书校验异常                                                 |
| 200082 | 服务器繁忙，请稍后重试                                       |
