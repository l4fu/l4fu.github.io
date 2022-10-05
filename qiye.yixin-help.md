# qiye.yixin_help
# 建立连接

oauth服务的地址：[https://oauth.qiye.yixin.im](https://oauth.qiye.yixin.im/)

开放平台服务的地址：[https://open.qiye.yixin.im](https://open.qiye.yixin.im/)

如无说明，相关接口都是调用开放平台服务地址。比如：

```
https://open.qiye.yixin.im/cgi-bin/token
https://open.qiye.yixin.im/cgi-bin/appmsg/send

```

## 获取access_token

**调用类型** ：Get

**接口地址** ：/cgi-bin/token

**参数列表**

| 参数 | 参数类型 | 是否必须 | 说明 |
| --- | --- | --- | --- |
| grant_type | string | 是 | 获取access\_token填写client\_credential |
| appKey | string | 是 | 应用凭证 |
| appSecret | string | 是 | 应用凭证密码 |
| permAuth | string | 是 | 永久授权码 |

**返回对象**

| 参数 | 说明 |
| --- | --- |
| access_token | 获取到的凭证 |
| expires_in | 凭证有效时间，单位：秒 |

**示例**

```
{"access_token":"ACCESS_TOKEN","expires_in":86400}

```

## 获取jssdk_ticket

调用类型 : Get

接口地址 : /cgi-bin/jssdk/ticket

参数列表

| 参数 | 参数类型 | 是否必须 | 说明 |
| --- | --- | --- | --- |
| access_token | string | 是 | 调用接口凭证 |

返回对象

| 参数 | 说明 |
| --- | --- |
| ticket | 用于jssdk的临时票据 |
| expires_in | 凭证有效时间，单位：秒 |

示例

```
{"ticket":"TICKET","expires_in":7200}

```

## JSSDK权限签名算法

如果开发者想使用马上办提供的jssdk接口，首先需要获取jssdk_ticket，获得之后，就可以生成jssdk权限验证的签名了。

#### 1.获取jssdk_ticket

jssdk_ticket,是开发者调用马上办JS接口的临时授权码，其作用主要用于生成签名，这个签名在[jssdk权限验证配置接口](https://doc.qiye.yixin.im/open/JSSDK%E6%96%87%E6%A1%A3.html#2%E6%B3%A8%E5%85%A5%E6%9D%83%E9%99%90%E9%AA%8C%E8%AF%81%E9%85%8D%E7%BD%AE)中使用。

正常的情况下，jssdk\_ticket的有效期为7200秒，通过access\_token来获取。由于频繁刷新jssdk_ticket会导致api调用受限，影响自身业务。

**|注意：** 开发者必须在自己的服务全局缓存jssdk_ticket。

#### 2.签名生成算法

签名生成规则如下：参与签名的字段包括nonce（随机字符串）, 有效的jssdk_ticket, timestamp（时间戳）, url（当前网页的URL，**不能包含#及其后面部分**） 。对所有待签名参数进行字典序排序，将四个参数字符串拼接成一个字符串进行sha1加密

示例

```
nonce=7470274696946504
jssdk_ticket=74de1561cd58481b9c8417ede23168e0
timestamp=1467705915427
url=https://debug.qiye.yixin.im/jssdk

```

步骤1\. 对所有待签名参数进行**字典序排序**，将四个参数字符串拼接成一个字符串string1

```
1467705915427747027469694650474de1561cd58481b9c8417ede23168e0https://debug.qiye.yixin.im/jssdk

```

步骤2\. 对string1进行sha1签名，得到signature。

```
e181e93475b5de115969313a77f968e919ee3c91

```

示例java代码

```
import java.security.MessageDigest;
import java.util.Arrays;

import org.apache.commons.codec.binary.Hex;

/**
 * JSSDK生成签名示例
 */
public class JssdkDemo {

    /**
     * 生成jssdk权限验证签名
     */
    public static String generateJssdkSignature(String nonce, String timestamp, String ticket, String url) {
        String[] arr = new String[] { nonce, timestamp, ticket, url };
        return generateSign(arr);
    }

    public static String generateSign(String[] arr){
        if(null == arr || arr.length == 0){
            return null;
        }
        // 将参数进行字典序排序
        Arrays.sort(arr);
        //将参数字符串拼接成一个字符串进行sha1签名
        StringBuilder content = new StringBuilder();
        for (int i = 0; i < arr.length; i++) {
            content.append(arr[i]);
        }
        //进行sha1签名
        return digest(content.toString(), "SHA1");
    }

    //进行数据签名
    public static String digest(String value, String algorithm) {
        if (value == null) {
            return null;
        }
        try {
            MessageDigest messageDigest = MessageDigest.getInstance(algorithm);
            messageDigest.update(value.getBytes("UTF-8"));
            return Hex.encodeHexString(messageDigest.digest());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}

```

## 主动调用的频率限制

当您获取到AccessToken时，您的应用后台就可以成功调用马上办开发平台所提供的各种接口或访问相应企业的资源或给成员发消息。

为了防止应用的程序错误而引发马上办服务器负载异常，默认情况下，每个服务端调用接口都有一定的频率限制，当超过此限制时，调用对应接口会收到相应错误码。

以下是当前默认的频率限制，马上办后台可能会根据运营情况调整此阈值：

- 每个企业调用单个接口的频率不可超过1500次/分
    
- 每个ISV（应用提供商）调用单个接口的频率不可超过2000次/分
    
- 每个ISV（应用提供商）调用单个企业的单个接口频率不可超过1500次/分
    
- 每个应用调用单个企业的单个接口频率不可超过1000次/分
    

# 免登流程

通过免登流程，当用户访问应用时，应用可以获取用户的身份信息。该免登流程只能用于客户端内使用。

用户访问应用的主要路径：

1. 马上办 – 工作 – 具体应用
2. 马上办 – 消息 – 工作助手

**工作面板**

![工作面板.jpg](_v_images/20200623153636529_26523.jpg)

**应用消息体**

![工作面板.jpg](_v_images/20200623153636222_21385.jpg)

## 1\. 应用免登流程

马上办中打开应用的H5页面，如果应用需要获取当前的用户信息，就需要走免登流程。

流程时序图（推荐图片另存放大查看）：

![](_v_images/20200623153635912_24556.png)

具体步骤为：

### 1.1. 获取authorization code

马上办的浏览器访问应用服务端，应用服务端需要重定向到马上办OAuth服务的authorize接口

```
https://oauth.qiye.yixin.im/authorize?response_type=code&client_id=da393115ae6945888a38fe9e1bab7000&state=xyz&redirect_uri=http%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb

```

请求参数说明

| 参数字段 | 类型 | 说明 |
| --- | --- | --- |
| response_type | String | 固定参数 response_type=code |
| client_id | String | 应用的appkey |
| state | String | 应用服务端自定义值，OAuth服务在回调时会原值返回。**\ 注意：**state参数如果包含特殊字符，请UrlEncode。 |
| redirect_uri | String | 应用的回调地址。**\ 注意：**请UrlEncode。 |

马上办OAuth服务会重定向到应用的回调地址，并带上code（该code短时间内一次有效）

```
http://client.example.com/cb?state=xyz&code=71e9a96cb8b3442cbc045c74b833c60c

```

如果发生错误，马上办OAuth服务会重定向到应用的回调地址，并带上error

```
http://client.example.com/cb?state=xyz&error=invalid_request

```

error字段说明

| 返回字段 | 说明 |
| --- | --- |
| invalid_request | 无效的请求 |
| unauthorized_client | 未授权的客户 |
| access_denied | 访问禁止 |
| server_error | 服务错误 |

### 1.2. 换取用户openid和公司corpOpenid

应用服务端拿上面步骤的code以及client\_id、client\_secret到OAuth服务换取用户openid和公司corpOpenid（使用POST方式）

```
https://oauth.qiye.yixin.im/token

```

```
grant_type=authorization_code&code=71e9a96cb8b3442cbc045c74b833c60c&client_id=da393115ae6945888a38fe9e1bab7000&client_secret=96e67d8c79824579a1ed6651efbd60a8

```

请求参数说明

| 参数字段 | 类型 | 说明 |
| --- | --- | --- |
| grant_type | String | 固定参数 grant\_type=authorization\_code |
| code | String | 第一步返回的code |
| client_id | String | 应用的appkey |
| client_secret | String | 应用的appsecret |

OAuth服务会返回

```
{"openid":"68e146b2d2b30131","corpOpenid":"b03f0456fb953668"}

```

返回字段说明

| 返回字段 | 类型 | 说明 |
| --- | --- | --- |
| openid | String | 用户openid |
| corpOpenid | String | 公司openid |

如果发生错误，OAuth服务会返回

```
{"error":"invalid_request"}

```

error字段说明参考上面

开发者应该处理OAuth服务返回的错误，并跳转到出错页面。

![auth_tips.png](_v_images/20200623153635390_13066.png)

## 2\. 应用后台管理员免登流程

和应用免登类似，马上办后台中打开应用的管理页面，也需要走免登流程。该免登流程用于浏览器使用的管理后台中的应用后台跳转。

具体步骤为：

### 2.1. 获取authorization code

浏览器中访问马上办后台并登录后，如果用户点击应用的“进入后台”链接，OAuth服务会从重定向到应用的管理页面（该地址必须在创建应用时指定）并带上code（该code短时间内一次有效）

```
http://admin.example.com/cb?code=71e9a96cb8b3442cbc045c74b833c60c

```

如果发生错误，马上办OAuth服务会重定向到应用的回调地址，并带上error

```
http://admin.example.com/cb?error=invalid_request

```

error字段说明

| 返回字段 | 说明 |
| --- | --- |
| invalid_request | 无效的请求 |
| unauthorized_client | 未授权的客户 |
| access_denied | 访问禁止 |
| server_error | 服务错误 |

### 2.2. 换取用户openid和公司corpOpenid

应用服务端拿上面步骤的code以及client\_id、client\_secret到OAuth服务换取用户openid和公司corpOpenid（使用POST方式）

```
https://oauth.qiye.yixin.im/token

```

```
grant_type=authorization_code&code=71e9a96cb8b3442cbc045c74b833c60c&redirect_uri=https%3A%2F%2Fadmin%2Eexample%2Ecom%2Fcb&client_id=da393115ae6945888a38fe9e1bab7000&client_secret=96e67d8c79824579a1ed6651efbd60a8

```

请求参数说明

| 参数字段 | 类型 | 说明 |
| --- | --- | --- |
| grant_type | String | 固定参数 grant\_type=authorization\_code |
| code | String | 第一步返回的code |
| client_id | String | 应用的appkey |
| client_secret | String | 应用的appsecret |

OAuth服务会返回

```
{"openid":"68e146b2d2b30131","corpOpenid":"b03f0456fb953668"}

```

返回字段说明

| 返回字段 | 类型 | 说明 |
| --- | --- | --- |
| openid | String | 用户openid |
| corpOpenid | String | 公司openid |

如果发生错误，OAuth服务会返回

```
{"error":"invalid_request"}

```

error字段说明参考上面

# 发送消息

应用可以主动发消息给用户，所有应用消息通过统一通道“工作助手”发送到具体用户。

## 发送应用消息

调用类型 : Post,异步

接口地址 : /cgi-bin/appmsg/send?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| to | String |  | 是 | 接受方(用户openid),多个以英文逗号隔开,限100人 |
| type | String |  | 是 | oa:OA消息，mi:文图消息，numen:单图文，numenmulti:多图文 |
| body | String |  | 是 | 消息内容json串，根据不同的type需要不同的参数，详见下面的示例 |

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 成功：0；其他为失败 | Integer |
| errmsg |  | String |

示例

```
{
          "errcode": 0,
          "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

示例

### OA消息

其中img需要先调用上传图片接口获取到fileKey

```
{
      "to": "c0d66467f83a5df1",
      "type": "oa",
      "body": {
          "title": "内容标题(标题)",
          "img": "fileKey",
          "content": "消息正文(必填)",
          "form": [
            "开始时间:3月2日",
            "结束时间:3月3日"
          ],
          "mobileUrl": "客户端点击消息时跳转或拉取操作层的地址(手机端)(必填)",
          "pcUrl": "客户端点击消息时跳转或拉取操作层的地址(PC端)(必填)",
          "useSysBrowser": 0        --说明：PC端是否使用系统浏览器打开url，0不使用，1使用
      }
}

```

### 文图消息

img和content至少需要填写一个

```
{
      "to": "c0d66467f83a5df1",
      "type": "mi",
      "body": {
          "img": "fileKey",
          "content": "消息内容"
      }
}

```

oa消息图和文图消息图

![发送消息.png](_v_images/20200623153635082_6774.png)

文图只上传文字或图片的显示

![](_v_images/20200623153634564_17153.png)

### 单图文消息

这里的img可以直接使用外网地址，图片格式允许jpg和png格式.

```
{
    "to": "UUFSGmKgI+8=",
    "type": "numen",
    "body": {
        "title": "单图文标题",
        "desc": "描述内容",
        "img": "https://nos.netease.com/kolibri//2.7.5.png",
        "mobileUrl": "http://www.163.com",
        "pcUrl": "http://www.163.com"
    }
}

```

![单图文](_v_images/20200623153634257_20920.png)

### 多图文消息

注意这里有两个body的嵌套，第一个位置为主图文，后面的为子图文。图片格式允许jpg和png格式。

```
{
    "to": "UUFSGmKgI+8=",
    "type": "numenmulti",
    "body": {
        "body":[
            {
                "title": "多图文主标题",
                "desc": "多图文主描述内容",
                "img": "https://nos.netease.com/kolibri//2.7.5.png",
                "mobileUrl": "http://www.163.com",
                "pcUrl": "http://www.163.com"
            },
            {
                "title": "子图文标题",
                "desc": "子描述内容",
                "img": "https://nos.netease.com/kolibri//2.7.5.png",
                "mobileUrl": "http://www.163.com",
                "pcUrl": "http://www.163.com"
            }
        ]
    }
}

```

![多图文](_v_images/20200623153633849_1735.png)

## 上传图片

调用类型 : Post

接口地址 : /cgi-bin/file/upload?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| type | String |  | 是 | 媒体文件类型，填为图片字符串，上传图片大小限制为2M，支持png和jpg |
| media | String |  | 是 | 文件二进制，form-data中的文件标识 |

示例图片

![](_v_images/20200623153633510_4699.png)

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| fileKey | 文件id | String |

返回示例

```
{
    "errcode": 0,
    "errmsg": null,
    "fileKey": "MTU2OTMwNjg3OTcxOTc0NTAzLnBuZw=="
}

```

示例错误码对应的信息： errcode: -1 errmsg:上传图片失败

## 获得图片

调用类型 : Get

接口地址 : /cgi-bin/file/get?access\_token=ACCESS\_TOKEN&fileKey=FILEKEY

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| fileKey | String |  | 是 | 文件id |

返回对象

和普通的http下载相同，请根据http头做相应的处理。

示例

```
          HTTP/1.1 200: OK
          Connection: close
          Content-Type: /jpeg
          Content-disposition: attachment; filename="123.jpg"
          Date: Sun, 04 Jan 2015 12:00:00 GMT
          Cache-Control: no-cache, must-revalidate
          Content-Length: 1234567
          ...

```

示例错误码对应的信息： errcode: -1 errmsg:获得图片失败

# 管理通讯录

可获取该企业的通讯录相关信息，主要包括部门信息和成员信息。

## 部门管理

### 查询部门列表

调用类型

Post, 异步

接口地址

/cgi-bin/department/list?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| id | String |  | 是 | 部门ID，根节点传0 |
| hasAllChild | int |  | 否 | 0不包含，1则包含该部门的所有的子节点，如果id=0，则返回全部部门 |

示例

```
{
    "id":"85",
    "hasAllChild":1
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| id | 部门id | String |
| name | 部门名称 | String |
| sort | 排序 | String |
| parentId | 父部门id | String |

示例

```
{
    "errcode": 0,
    "errmsg": "success",
    "depList": [
        {
            "name": "测试公司",
            "id": 43974,
            "sort": 1,
            "parentId": 0
        },
        {
            "name": "财务部",
            "id": 81184,
            "sort": 2,
            "parentId": 43974
        },
        {
            "name": "销售部",
            "id": 81185,
            "sort": 3,
            "parentId": 43974
        },
        {
            "name": "产品部",
            "id": 81186,
            "sort": 4,
            "parentId": 43974
        },
        {
            "name": "华东销售部",
            "id": 81187,
            "sort": 1,
            "parentId": 81185
        }
    ]
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 获取部门详情

调用类型

Post,异步

接口地址

/cgi-bin/department/get?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| id | String |  | 是 | 节点ID |

示例

```
{
    "id": "1"
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| id | 部门id | String |
| name | 部门名称 | String |
| sort | 排序 | String |
| parentId | 父部门id | String |

示例

```
{
    "errcode": 0,
    "name": "财务部",
    "errmsg": "success",
    "id": 81184,
    "sort": 2,
    "parentId": 43974
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 更新部门

调用类型

Post,异步

接口地址

/cgi-bin/department/update?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| name | String |  | 是 | 部门名 |
| id | long |  | 是 |  |
| sort | int |  | 是 | 排序号，小序号优先，升序 |

示例

```
{
    "name":"测试部门2",
    "id":81188,
    "sort":3
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| id | 部门id | String |
| name | 部门名称 | String |
| sort | 排序 | String |
| parentId | 父部门id | String |

示例

```
{
    "errcode": 0,
    "name": "测试部门2",
    "errmsg": "success",
    "id": 81188,
    "sort": 3,
    "parentId": 43974
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

## 成员管理

### 获取部门成员列表

调用类型

Post, 异步

接口地址

/cgi-bin/contact/simplelist?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| departmentId | String |  | 是 | 部门ID，0则为公司节点下的成员 |

示例

```
{
    "departmentId": "1234"
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| guid | 用户openid | String |
| name | 成员姓名 | String |
| departmentIds | 所在部门id | String |
| sex | 性别 | String |
| position | 职位 | String |
| bindMobile | 手机号码(ISV调用返回结果不包括此字段信息) | String |
| countryCode | 国际码(ISV调用返回结果不包括此字段信息) | String |
| email | 邮箱(ISV调用返回结果不包括此字段信息) | String |
| mobile | 办公电话 | String |
| extInfo | 扩展信息，需要在管理后台先创建扩展字段 | JSON |

示例

```
{
    "errcode": 0,
    "contactList": [
        {
            "bindMobile": "18612311111",
            "countryCode": "86",
            "sex": 0,
            "name": "admin",
            "mobile": "",
            "guid": "UUFSGmKgI+8=",
            "position": "",
            "departmentIds": "43974",
            "extInfo": {
                "自定1": "123"
            }
        },
        {
            "bindMobile": "18612311112",
            "countryCode": "86",
            "sex": 0,
            "name": "赵六",
            "mobile": "",
            "guid": "nAsfxfQS6V0=",
            "position": "",
            "departmentIds": "43974",
            "email": "",
            "extInfo": {
                "自定1": "234"
            }
        },
        {
            "bindMobile": "18612311114",
            "countryCode": "86",
            "sex": 0,
            "name": "李四",
            "mobile": "",
            "guid": "l1D0/Hl3w1M=",
            "position": "",
            "departmentIds": "43974",
            "email": "",
            "extInfo": {
                "自定1": "567"
            }
        },
        {
            "bindMobile": "18612311115",
            "countryCode": "86",
            "sex": 0,
            "name": "张三",
            "mobile": "",
            "guid": "EKSO0tCarVI=",
            "position": "",
            "departmentIds": "43974",
            "email": ""
        }
    ],
    "errmsg": "success"
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 获取成员详情

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/get?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| guid | String |  | 是 | 用户openid |

示例

```
{
    "guid": "vJdba="
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| guid | 用户openid | String |
| name | 成员姓名 | String |
| departmentIds | 所在部门id | String |
| sex | 性别 | String |
| position | 职位 | String |
| bindMobile | 手机号码(ISV调用返回结果不包括此字段信息) | String |
| countryCode | 国际码(ISV调用返回结果不包括此字段信息) | String |
| email | 邮箱(ISV调用返回结果不包括此字段信息) | String |
| mobile | 办公电话 | String |
| extInfo | 扩展信息，需要在管理后台先创建扩展字段 | JSON |

示例

```
{
    "errcode": 0,
    "bindMobile": "1598888888",
    "countryCode": "86",
    "sex": 0,
    "name": "admin",
    "mobile": "",
    "guid": "UUFSGmKgI+8=",
    "errmsg": "success",
    "position": "",
    "departmentIds": "43974",
    "extInfo": {
        "自定1": "123"
    }
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 获取超管详情

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/getAdmin?access\_token=ACCESS\_TOKEN

不需要上传其他参数，根据access_token匹配到公司超管信息

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| guid | 用户openid | String |
| name | 成员姓名 | String |
| departmentIds | 所在部门id | String |
| sex | 性别 | String |
| position | 职位 | String |
| bindMobile | 手机号码(ISV调用返回结果不包括此字段信息) | String |
| countryCode | 国际码(ISV调用返回结果不包括此字段信息) | String |
| email | 邮箱(ISV调用返回结果不包括此字段信息) | String |
| mobile | 办公电话 | String |
| extInfo | 扩展信息，需要在管理后台先创建扩展字段 | JSON |

示例

```
{
    "errcode": 0,
    "bindMobile": "1598888888",
    "countryCode": "86",
    "sex": 0,
    "name": "admin",
    "mobile": "",
    "guid": "UUFSGmKgI+8=",
    "errmsg": "success",
    "position": "",
    "departmentIds": "43974",
    "extInfo": {
        "自定1": "123"
    }
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 更新成员

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/update?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| guid | String |  | 是 | 用户guid |
| name | String |  | 是 | 用户名 |
| departmentIds | String |  | 是 | 部门id字符串，多个以英文逗号分隔。如1,2,3 |
| sex | int |  | 是 | 性别：1男，2女 |
| position | String |  | 是 | 职位 |
| mobile | String |  | 是 | 办公电话 |

示例

```
{
    "name":"姓名2",
    "departmentIds":"43974",
    "sex":1,
    "position":"",
    "mobile":"",
    "guid":"j0PEt1ef+AE="
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| guid | 用户openid | String |
| name | 成员姓名 | String |
| departmentIds | 所在部门id | String |
| sex | 性别 | String |
| position | 职位 | String |
| bindMobile | 手机号码(ISV调用返回结果不包括此字段信息) | String |
| countryCode | 国际码(ISV调用返回结果不包括此字段信息) | String |
| email | 邮箱(ISV调用返回结果不包括此字段信息) | String |
| mobile | 办公电话 | String |
| extInfo | 扩展信息，需要在管理后台先创建扩展字段 | JSON |

示例

```
{
    "errcode": 0,
    "bindMobile": "1598888888",
    "sex": 1,
    "name": "姓名2",
    "guid": "j0PEt1ef+AE=",
    "errmsg": "success",
    "departmentIds": "43974",
    "email": ""
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 禁用成员

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/block?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| guid | String |  | 是 | 用户guid |

示例

```
{
    "guid":"j0PEt1ef+AE="
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": "success"
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 启用成员

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/unblock?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| guid | String |  | 是 | 用户guid |

示例

```
{
    "guid":"j0PEt1ef+AE="
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": "success"
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

## 角色管理

### 获取分类列表

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/category/list?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| cateType | 分类类型：0正常类型，1同步类型 | int |
| name | 分类名称 | String |
| id | 分类id | long |
| sort | 分类序号 | int |

示例

```
{
    "errcode": 0,
    "categoryList": [
        {
            "cateType": 0,
            "name": "业务角色",
            "id": 446,
            "sort": 1
        },
        {
            "cateType": 0,
            "name": "岗位角色",
            "id": 447,
            "sort": 1
        },
        {
            "cateType": 0,
            "name": "自定分类1",
            "id": 7013,
            "sort": 3
        }
    ],
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 创建分类

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/category/create?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| categoryName | String |  | 是 | 分类名 |
| sort | int |  | 是 | 分类序号 |

示例

```
{
    "categoryName":"分类名1",
    "sort":1
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| id | 分类id | long |

示例

```
{
    "errcode": 0,
    "errmsg": null,
    "id": 7014
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 更新分类

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/category/update?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| categoryName | String |  | 是 | 分类名 |
| id | long |  | 是 | 分类id |

示例

```
{
    "categoryName":"分类名2",
    "id":7014
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| id | 分类id | long |

示例

```
{
    "errcode": 0,
    "errmsg": null,
    "id": 7014
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 删除分类

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/category/delete?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| id | long |  | 是 | 分类id |

示例

```
{
    "id":7014
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 获取角色列表

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/list?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| roleType | 角色类型：0正常类型，1同步类型 | int |
| roleName | 角色名称 | String |
| id | 角色id | long |
| sort | 角色序号 | int |
| categoryId | 所属分类id | long |

示例

```
{
    "errcode": 0,
    "errmsg": null,
    "roleList": [
        {
            "roleName": "人事",
            "id": 1471,
            "sort": 0,
            "roleType": 0,
            "categoryId": 446
        },
        {
            "roleName": "行政",
            "id": 1472,
            "sort": 0,
            "roleType": 0,
            "categoryId": 446
        },
        {
            "roleName": "财务",
            "id": 1473,
            "sort": 0,
            "roleType": 0,
            "categoryId": 446
        },
        {
            "roleName": "出纳",
            "id": 1474,
            "sort": 0,
            "roleType": 0,
            "categoryId": 446
        },
        {
            "roleName": "主管",
            "id": 1475,
            "sort": 0,
            "roleType": 0,
            "categoryId": 447
        },
        {
            "roleName": "总监",
            "id": 1476,
            "sort": 0,
            "roleType": 0,
            "categoryId": 447
        },
        {
            "roleName": "总经理",
            "id": 1477,
            "sort": 0,
            "roleType": 0,
            "categoryId": 447
        }
    ]
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 创建角色

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/create?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| categoryId | long |  | 是 | 所属分类id |
| roleName | String |  | 是 | 角色名 |
| sort | int |  | 是 | 角色序号 |

示例

```
{
    "categoryId":447,
    "roleName":"角色名1",
    "sort":1
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| id | 角色id | long |

示例

```
{
    "errcode": 0,
    "errmsg": null,
    "id": 12024
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 更新角色

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/update?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| roleName | String |  | 是 | 角色名 |
| id | long |  | 是 | 角色id |

示例

```
{
    "roleName":"角色名2",
    "id":12024
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| id | 角色id | long |

示例

```
{
    "errcode": 0,
    "errmsg": null,
    "id": 12024
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 删除角色

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/role/delete?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| id | long |  | 是 | 角色id |

示例

```
{
    "id":12024
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 获取角色成员列表

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/roleuser/list?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| roleId | long |  | 是 | 角色id |

示例

```
{
    "roleId": 13017
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| roleUserList |  | Array |

示例

```
{
    "errcode": 0,
    "errmsg": null,
    "roleUserList": [
        "UUFSGmKgI+8=",
        "EKSO0tCarVI=",
        "nAsfxfQS6V0="
    ]
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 添加角色成员

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/roleuser/add?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| roleId | long |  | 是 | 角色id |
| guids | String\[\] |  | 是 | 成员guid数组 |

示例

```
{
    "roleId":13017,
    "guids":["UUFSGmKgI+8=","EKSO0tCarVI="]
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 移除角色成员

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/roleuser/delete?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| roleId | long |  | 是 | 角色id |
| guids | String\[\] |  | 是 | 成员guid数组 |

示例

```
{
    "roleId":13017,
    "guids":["UUFSGmKgI+8=","EKSO0tCarVI="]
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

## 通讯录集成（只有对接开发才能使用）

### 获取本次对接更新的通讯录结果

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/getSyncInfo?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| syncType | int |  | 是 | 1增量，2全量 |

示例

```
{
    "syncType":1
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| addDepCount | 新增部门数 | int |
| updateDepCount | 更新部门数 | int |
| delDepCount | 删除部门数 | int |
| addCuCount | 新增成员数 | int |
| updateCuCount | 更新成员数 | int |
| delCuCount | 删除成员数 | int |

示例

```
{
    "errcode": 0,
    "updateDepCount": 20,
    "delCuCount": 1236,
    "errmsg": null,
    "addDepCount": 24,
    "delDepCount": 31,
    "addCuCount": 762,
    "updateCuCount": 339
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

### 执行通讯录对接更新

调用类型 : Post, 异步

接口地址 : /cgi-bin/contact/sync?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| syncType | int |  | 是 | 1增量，2全量 |

示例

```
{
    "syncType":1
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

# 群组管理

## 创建群

调用类型 : Post, 异步

接口地址 : /cgi-bin/team/create?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| tname | String |  | 是 | 群名称 |
| owner | String |  | 是 | 群主guid |
| members | String\[\] |  | 是 | 成员guid数组，一次不能超过200个 |

示例

```
{
    "tname":"测试群1",
    "owner":"nAsfxfQS6V0=",
    "members":["UUFSGmKgI+8=","EKSO0tCarVI="]
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| tid | 群id | String |

示例

```
{
    "errcode": 0,
    "errmsg": null,
    "tid":"nAsfxfQxx"
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

## 拉人进群

调用类型 : Post, 异步

接口地址 : /cgi-bin/team/member/add?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| tid | String |  | 是 | 群id |
| owner | String |  | 是 | 群主guid |
| members | String\[\] |  | 是 | 成员guid数组，一次不能超过200个 |

示例

```
{
    "tid":"AsfxfQS6xx",
    "owner":"nAsfxfQS6V0=",
    "members":["UUFSGmKgI+8=","EKSO0tCarVI="]
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

## 踢人出群

调用类型 : Post, 异步

接口地址 : /cgi-bin/team/member/kick?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| tid | String |  | 是 | 群id |
| owner | String |  | 是 | 群主guid |
| members | String\[\] |  | 是 | 成员guid数组，一次不能超过200个 |

示例

```
{
    "tid":"nAsfxfQxx",
    "owner":"nAsfxfQS6V0=",
    "members":["UUFSGmKgI+8=","EKSO0tCarVI="]
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

## 解散群组

调用类型 : Post, 异步

接口地址 : /cgi-bin/team/remove?access\_token=ACCESS\_TOKEN

参数列表

| 参数 | 数据类型 | 默认值 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| tid | String |  | 是 | 群id |
| owner | String |  | 是 | 群主guid |

示例

```
{
    "tid":"nAsfxfQxx",
    "owner":"nAsfxfQS6V0="
}

```

返回对象

| 属性 | 说明 | 数据类型 |
| --- | --- | --- |
| errcode | 返回错误码 | int |
| errmsg | 错误信息 | String |

示例

```
{
    "errcode": 0,
    "errmsg": null
}

```

errcode详见 [code状态码](https://doc.qiye.yixin.im/open/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#code%E7%8A%B6%E6%80%81%E7%A0%81)

# ISV开发者消息

## 1.使用之前

在使用回调接口之前开发商首先要拿到应用的"消息接收URL"，"Token"，"数据加密密钥(EncodingAESKey)"

1. 消息接收URL(notifyURL):服务提供商接收马上办推送请求的协议和地址
2. Token:马上办创建服务时分配给服务提供商，用来生成signature
3. 数据加密密钥(EncodingAESKey):用于消息体的加解密

## 2.推送消息及返回

### 2.1.推送地址及参数

```
http://ipAddress:port/app/isvreceive?signature=111108bb8e6dbce3c9671d6fdb69d15066227608×tamp=1783610513&nonce=u82p7

```

### 2.2.收到消息体内容

马上办会将加密后的消息体以JSON形式发送到ISV的消息接收URL，其中的encrypt字段是经过加密的消息体（见【术语说明】），经过一系列解密步骤后获得明文消息，解密流程见【5.解密流程】，解密后的具体信息详见【6.推送消息】。

收到消息体示例：

```
{ "encrypt":"1ojQf0NSvw2WPvW7LijxS8UvISr8pdDP+rXpPbcLGOmIBNbWetRg7IP0vdhVgkVwSoZBJeQwY2zhROsJq/HJ+q6tp1qhl9L1+ccC9ZjKs1wV5bmA9NoAWQiZ+7MpzQVq+j74rJQljdVyBdI/dGOvsnBSCxCVW0ISWX0vn9lYTuuHSoaxwCGylH9xRhYHL9bRDskBc7bO0FseHQQasdfghjkl"}

```

### 2.3.术语说明

| 参数 | 说明 | 描述 |
| --- | --- | --- |
| signature | 消息签名 | 用于验证调用者的合法性和消息完整性 |
| timeStamp | 时间戳 | Unix时间戳，长度13位 |
| nonce | 随机字符串 | 大写、小写字母和数字组成，长度5位 |
| AESKey | AES密钥 | 长度32字节，马上办AES算法采用CBC模式，数据采用PKCS#7填充；IV初始向量大小为16字节，取AESKey前16字节 |
| EncodingAESKey | 加密密钥 | EncodingAESKey是AESKey的Base64编码。长度为44个字符，解码后即为AESKey，可在创建应用时获得 |
| msg | 消息明文 | 格式为JSON |
| encrypt | 消息密文 | msg经过加密后获得的密文，以JSON格式传给ISV，字段名为encrypt |

### 2.4.消息返回

在服务提供商接收到马上办的推送消息之后，需要返回包含相应的加密结果的JSON消息表示收到了推送，如果服务商未能成功返回，马上办将会做出相应处理。返回的JSON消息包括属性如下

| 参数 | 类型 | 描述 |
| --- | --- | --- |
| msg_signature | String | 消息签名，算法参照 消息体签名 |
| timeStamp | String | 时间戳，使用推送请求中的timestamp，否则将会校验不通过 |
| nonce | String | 随机字符串，使用推送请求中的nonce，否则将会校验不通过 |
| encrypt | String | 加密后的返回结果，返回内容参照【推送消息】中的返回结果；加密算法参照 【明文加密方式】 |

## 3.明文加密方式

### 3.1.加密公式

```
encrypt = Base64_Encode(AES_Encrypt[random(16Byte) + msg_len(4Byte) + msg + $appKey]) (注：加密字符串不含+)

```

### 3.2.加密流程

1. random为16字节的随机字符串，msg\_len为msg的字节数（字符串长度值），网络字节序，4字节长度,将random，msg\_len，msg，appKey 四个字符串按顺序拼接转换为一个字符数组rand_msg
2. 将字符数组rand\_msg进行AES加密，生成加密后的字符数组aes\_msg，密钥和加密方式见【术语说明】AESKey
3. 将加密后的字符数组aes_msg进行Base64加密，得到密文encrypt

## 4.消息体签名

为了验证消息请求来自于马上办，服务器将会在回调url中提供消息签名(见【术语说明】signature)，企业应用需要先验证签名正确性后再进行解密。

签名校验流程：

1. 将token、timestamp、nonce、encrypt四个参数进行字典序从小到大排序拼接成一个字符串进行sha1加密
2. 开发者获得加密后的字符串可与signature对比，表示该请求来源于马上办

## 5.密文解密方式

密文解密是【3.明文加密方式】的逆向过程，具体过程如下：

密文解密流程：

1. 取出返回的JSON中的encrypt字段
2. 对密文BASE64解码：aes\_msg=Base64\_Decode(encrypt)
3. 使用AESKey做AES解密：rand\_msg=AES\_Decrypt(aes_msg)
4. 去掉rand\_msg头部的16个随机字节，4个字节的msg\_len,和尾部的$appKey即为最终的消息体原文msg
5. 验证$appKey、msg_len,核对消息的完整性和正确性

## 6.推送消息

服务商收到的推送消息，解密的后的消息体为JSON格式

### 6.1.企业开通应用

场景：企业开通应用时，服务提供商会收到一条由马上办推送的开通应用消息，用于告知开通企业的永久授权码，具体参数如下

参数列表

| 属性 | 数据类型 | 说明 |
| --- | --- | --- |
| EventType | String | 开通应用，sub_serv |
| AppKey | String | 应用的appKey |
| CorpOpenid | String | 企业ID |
| AuthCode | String | 开通企业的永久授权码 |
| TimeStamp | String | 时间戳 |

返回说明:

服务提供商在收到此事件推送后务必返回包含经过加密的字符串"success"的json数据，否则企业将不会开通服务。

返回数据方案见【消息返回】

### 6.2.企业删除应用

场景：企业删除应用时，服务提供商会收到由马上办推送的删除应用消息，用于告知服务商此企业关闭了服务，具体参数如下

参数列表

| 属性 | 数据类型 | 说明 |
| --- | --- | --- |
| EventType | String | 企业删除应用，unsub_serv |
| AppKey | String | 应用的appKey |
| CorpOpenid | String | 企业ID |
| TimeStamp | String | 时间戳 |

返回说明:

服务提供商在收到此事件推送后务必返回包含经过加密的字符串"success"的json数据，否则马上办将会向企业提示错误信息。

返回数据方案见【消息返回】

# code状态码

| code | 详细描述 |
| --- | --- |
| -1 | 统一的失败code，显示错误信息（具体视接口返回） |
| 0 | 请求成功 |
| 400 | 内部错误 |
| 414 | 参数错误 |
| 404 | 数据不存在 |
| 10431 | 输入email不是邮箱 |
| 10433 | 用户不存在 |
| 10435 | 不在同一个企业 |
| 10437 | 不能给同一个邮件地址同时发送多个邀请 |
| 10438 | 仅支持邀请163、QQ等公共邮箱 |
| 10439 | 邮箱已存在或已经加入了其他企业 |
| 10440 | 已经邀请过了 |
| 10441 | 解析link消息出错 |
| 10442 | 解析消息出错 |
| 10443 | 文件格式有误 |
| 40004 | 不合法的媒体文件类型 |
| 40005 | 不合法的文件类型 |
| 40006 | 不合法的文件大小 |
| 40008 | 不合法的消息类型 |
| 40009 | 不合法的图片文件大小 |
| 40013 | 不合法的appKey |
| 40014 | 不合法的access_token |
| 40015 | 不合法的授权码 |
| 40016 | 企业已禁用该应用 |
| 40017 | 开发者模式未启用 |
| 40018 | 企业已禁用该消息通道 |
| 40029 | access_token超时 |
| 40036 | appKey和appsecret不匹配 |
| 45009 | 接口调用超过限制 |
| 47001 | 解析JSON/XML内容错误 |
| 60015 | 企业没有订阅此服务 |
| 60025 | 此接口只能被企业自建应用调用 |
| 90014 | 请求参数中包含非法字符 |

# 示例Demo

java示例demo下载：[https://doc.qiye.yixin.im/open/kolibri-example.zip](https://doc.qiye.yixin.im/open/kolibri-example.zip)
