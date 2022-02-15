<h1 align="center">WinDHP SDK for Java</h1>


欢迎使用 WinDHP SDK for Java。
WinDHP SDK for Java 让您不用复杂编程即可访问WinDHP产品。
这里向您介绍如何获取 WinDHP SDK for Java 并开始调用。
如果您在使用 WinDHP SDK for Java 的过程中遇到任何问题，欢迎联系我们提出疑问 [WinDHP官网](http://172.16.9.131/contact)，我们随时解答。

## 环境要求
1. 要使用 WinDHP SDK for Java ，您需要一个WinDHP账号以及一对`AppKey`和`AppSecret`。 请在WinDHP数字健康平台的[采购商中心管理页面](http://172.16.9.131/crm/#/index)上创建和查看您的AppKey，或者联系您的系统管理员。
2. 要使用 WinDHP SDK for Java 访问某个产品的API，您需要事先在[WinDHP云市场](http://172.16.9.131/market/#/api/list)中购买这个产品。
3. WinDHP SDK for Java 需要1.8以上的JDK。

## 安装依赖
作为采购商，需要调用WinDHP的产品，只需要安装`windhp-openapi-java-sdk-client`。
### 通过Maven来管理项目依赖(推荐)
如果您使用Apache Maven来管理Java项目，只需在项目的`pom.xml`文件加入相应的依赖项即可。您可以在[WinDHP开发资源](http://172.16.9.131/cooperate)中下载各云产品的Maven依赖。
以使用Client SDK为例，您只需在`pom.xml`中声明以下两个依赖：
```xml
<dependency>
    <groupId>com.winning</groupId>
    <artifactId>windhp-openapi-java-sdk-client</artifactId>
    <version>1.0.0</version>
</dependency>
```

如果 maven 没有从中央存储库下载 jar 包，则需要将这些依赖项添加到`pom.xml`文件中，否则将报告 NoClassDefFoundError 异常
```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>

<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.69</version>
</dependency>
```

## 使用诊断
通过响应头Response-header的`X-Trace-Id` 或 `X-Ca-Error-Message` ，帮助开发者快速定位，为开发者提供解决方案。

## 快速使用

以下这个代码示例向您展示了调用 WinDHP SDK for Java 的3个主要步骤：
1. 创建IProfile实例并初始化。
2. 创建API请求并设置参数。
3. 发起请求并处理应答或异常。

```java
package com.winning;

import com.winning.constant.Constants;
import com.winning.constant.SystemHeader;
import com.winning.exceptions.ClientException;
import com.winning.exceptions.ServerException;
import com.winning.profile.DHPProfile;
import com.winning.profile.IProfile;
import com.winning.request.Response;

public class Main {
    public static void main(String[] args) {
        IProfile profile = DHPProfile.getProfile(
                "<your-service-code>",   // 购买的产品的ServiceCode
                "<your-app-key>",        // WinDHP采购商账号的AppKey
                "<your-app-secret>");    // WinDHP采购商账号的AppSecret
        try {
            Response response = DHPHttpClient.get(profile)
                    .url("http://172.16.30.147/opengateway/call/simple")
                    .addHeader(SystemHeader.CONTENT_TYPE, Constants.APPLICATION_JSON)
                    .build()
                    .execute();
            System.out.println(response.string());
        } catch (ServerException e) {
            System.out.println(e.getMessage());
            e.printStackTrace();
        } catch (ClientException e) {
            System.out.println(e.getMessage());
            e.printStackTrace();
        }
    }
}
```
## 自己开发

### 签名规则

目前使用HMacSHA256算法计算签名，签名的计算需要appSecret，计算方法为:signature = hmacSHA256(stringToSign, appSecret)

### 需要参与签名的参数
|  参数 |  描述 |
| ------------ | ------------ |
| HTTPMethod  |  HTTP的方法，全部大写，比如POST |
| Content-Type  |  请求中的Content-Type头的值，可为空 |
| X-Service-Code | 服务code ，所有的API商品发布后均会由平台自动生成唯一的服务code  |
| X-Ca-Key  |  平台分配的AppKey |
| X-Ca-Nonce  |  请求唯一标识，15分钟内 AppKey+API+Nonce 不能重复，与时间戳结合使用才能起到防重放作用。 |
| X-Ca-Timestamp  |  请求的时间戳，值为当前时间的毫秒数，也就是从1970年1月1日起至今的时间转换为毫秒，时间戳有效时间为15分钟。 |
|  X-Conent-MD5 |  注意先进行MD5摘要再进行Base64编码获取摘要字符串，用于校验QueryParams/Body参数是否被篡改（post请求为body内容，get/delete请求为url参数（按照字典排序）) |


生成的签名串的格式如下：
```
HTTPMethod
Content-Type
headerStrToSign（带“X-”前缀的请求头组装而成）
```

headerStrToSign组装规则如下：
上述带“X-”前缀的请求头参数Key转成小写字母， 按照字典排序后使用如下方式拼接

- header 名转换为小写，跟上”:”英文冒号字符。
- 然后附加 header 值。
- 如果不是最后一条需构造签名的 header，以”&”符号进行连接。
``` java
string headerStrToSign="x-ca-key:wnw&x-ca-nonce:c45375bb-019f-45ae-81f1-cb214d8a8f25&x-ca-timestamp:1545675450395&x-content-md5:v0y1DCDvzr8w0+KVEjoIVA==&x-service-code:88249225355264"
```

- 按签名串格式拼接`HTTPMethod值`比如POST、`Content-Type头的值`比如application/json
```java
String stringToSign="POST" + "\n" + "application/json" + "\n" + "x-ca-key:wnw&x-ca-nonce:c45375bb-019f-45ae-81f1-cb214d8a8f25&x-ca-timestamp:1545675450395&x-content-md5:v0y1DCDvzr8w0+KVEjoIVA==&x-service-code:88249225355264"

String signature = hmacSHA256(stringToSign, appSecret))

private static final String ALGORITHM_NAME = "HmacSHA256";

public String hmacSHA256(String stringToSign, String accessKeySecret) {
    try {
        Mac hmacSha256 = Mac.getInstance(ALGORITHM_NAME);
        SecretKeySpec secret_key = new SecretKeySpec(accessKeySecret.getBytes(), ALGORITHM_NAME);
        hmacSha256.init(secret_key);
        return DatatypeConverter.printBase64Binary(hmacSha256.doFinal(stringToSign.getBytes(StandardCharsets.UTF_8)));
    } catch (NoSuchAlgorithmException e) {
        throw new IllegalArgumentException(e.toString());
    } catch (InvalidKeyException e) {
        throw new IllegalArgumentException(e.toString());
    }
}
```

将signature的值赋值给请求头参数X-Ca-Signature
- 签名错误排查方法
  当签名校验失败时，API网关会将服务端的StringToSign放到HTTP应答的Header中返回到客户端，Key为：`X-Ca-Error-Message`，只需要将本地计算的StringToSign与服务端返回的StringToSign进行对比即可找到问题；
  如果服务端与客户端的签名串是一致的，请检查用于签名计算的密钥是否正确；
  注意：因为HTTP Header中无法表示换行，因此返回的StringToSign中的换行符都被替换成#。

### X-Conent-MD5

注意先进行MD5摘要再进行Base64编码获取摘要字符串，用于校验QueryParams/Body参数是否被篡改（post请求为body内容，get/delete请求为url参数（按照字典排序）)
X-Conent-MD5 算法
``` java
String content-MD5 = Base64.encodeBase64(MD5(body.getbytes("UTF-8")));

public static String base64AndMd5(String str) {
      if (str == null) {
          throw new IllegalArgumentException("inStr can not be null");
      }
      try {
          return base64AndMd5(toBytes(str));
      } catch (ClientException e) {
          throw new RuntimeException(e);
      }
}

public static String base64AndMd5(byte[] bytes) {
      if (bytes == null) {
        throw new IllegalArgumentException("bytes can not be null");
      }
      try {
        final MessageDigest md = MessageDigest.getInstance("MD5");
        md.reset();
        md.update(bytes);
        Base64 base64 = new Base64();
        final byte[] enbytes = base64.encode(md.digest());
        return new String(enbytes);
      } catch (final NoSuchAlgorithmException e) {
        throw new IllegalArgumentException("unknown algorithm MD5");
      }
}

/**
 * String转换为字节数组
 */
private static byte[] toBytes(final String str) throws ClientException{
    if (str == null) {
        return new byte[0];
    }
    try {
        return str.getBytes(Constants.ENCODING_UTF8);
    } catch (final UnsupportedEncodingException e) {
        throw new ClientException("UnsupportedEncodingException", e.getMessage(), e);
    }
}

```


### 调用平台接口

请求协议：https

httpHeader 请求头参数，包含下方表格内参数

请求地址: https://域名/opengateway/call/simple

| key  | value(示例)  |说明   |  是否必填 |
| ------------ | ------------ | ------------ | ------------ |
|  X-Service-Code |  40862475087873  |  服务Code |  是 |
|  X-Ca-Timestamp | 1471864864235  | 请求的时间戳，值为当前时间的毫秒数，也就是从1970年1月1日起至今的时间转换为毫秒，时间戳有效时间为15分钟。  | 是  |
| X-Ca-Nonce  |  b931bc77-645a-4299-b24b-f3669be577ac | 请求唯一标识，15分钟内 AppKey+API+Nonce 不能重复，与时间戳结合使用才能起到防重放作用。  |  是 |
|  X-Content-MD5 |  Ndul11U4qbvgtNpEInWaDg== | 注意先进行MD5摘要再进行Base64编码获取摘要字符串 |  是 |
| X-Ca-Signature  |  FJleSrCYPGCU7dMlLTG+UD3Bc5Elh3TV3CWHtSKh1Ys= | 请求签名  | 是  |
| X-Ca-Key  | 8d3dgnzs87m4v2dme  |  平台分配的AppKey | 是   |


请求参数详见具体接口参数说明。

响应头
```
X-Trace-Id: 0ac9d34de2af41c5945ea9ab8f4bf9e0
//请求唯一 ID，请求一旦进入 API 网关应用后，API 网关就会生成请求 ID 并通过响应头返回给客户端，建议客户端与后端服务都记录此请求 ID，可用于问题排查与跟踪。
```
返回示例，body是服务返回的具体数据，返回格式，请参照各服务说明。

```
{
    "code":0,
    "data":{
       "organ_id":1,
       "name":"张三"
    }
}
```




