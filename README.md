# 集成推送平台接口说明

# API接口规范
## 接口响应规范 
> HTTP接口遵循魅族API协议规范。返回数据格式统一如下：


```
{
    “code”:”“, //必选,返回码
    “message”:”“, //可选，返回消息，网页端接口出现错误时使用此消息展示给用户，手机端可忽略此消息，甚至服务端不传输此消息
    “value”:”“,// 必选，返回结果
}
```
> Api returnCode定义


code|value
---|---
200|正常
500|其他异常
1001|系统错误
1003|服务器忙
1005|参数错误，请参考API文档
1006|签名认证失败
110000|appId不合法
110001|appKey不合法
110004|参数不能为空
110010|应用推送速率过快
110053|透传超过限制


## 接口签名规范
> 请求参数分别是“k1”、“k2”、“k3”，它们的值分别是“v1”、“v2”、“v3”，计算方法如下所示：
> 
> 1. 将参数以其参数名的字典序升序进行排序,如 对 k1 k2 k3 排序
> 1. 遍历排序后的字典，将所有参数按"key=value"格式拼接在一起，如“k1=v1k2=v2k3=v3”
> 1. 在拼接好的字符串末尾追加上应用的Secret Key
> 
> 上述字符串的MD5值即为签名的值。（32位小写）
> 
> 将签名值放在请求的参数中例如sign=MD5_SIGN
> 
> 服务端SDK调用API的应用的私钥Secret Key为 appSecret

```java
   /**
     * @param paramMap 请求参数
     * @param secret   密钥
     * @return md5摘要
     */
    public static String getSignature(Map<String, String> paramMap, String secret) {
        // 先将参数以其参数名的字典序升序进行排序
        Map<String, String> sortedParams = new TreeMap<String, String>(paramMap);
        Set<Entry<String, String>> entrys = sortedParams.entrySet();

        // 遍历排序后的字典，将所有参数按"key=value"格式拼接在一起
        StringBuilder basestring = new StringBuilder();
        for (Entry<String, String> param : entrys) {
            basestring.append(param.getKey()).append("=").append(param.getValue());
        }
        basestring.append(secret);

        logger.debug("basestring is:{}", new Object[]{basestring.toString()});

        // 使用MD5对待签名串求签
        return MD5Util.MD5Encode(basestring.toString(),"UTF-8");
    }
    
   //示例，注意是针对接口中所有参数做签名，并且是原始字符串（非urlencode）
    public static void main(String[] args) {
        //本示例为三个参数 appId、pushIds、messageJson
        Map<String, String> paramMap = new HashMap<String, String>();
        paramMap.put("appId", "10000");
        paramMap.put("pushIds", "RA50c6348036344485d01776773577c64740465480a6b");
        paramMap.put("messageJson", "{\"title\":\"title\",\"content\":\"content\",\"pushTimeInfo\":{\"offLine\":1,\"validTime\":24}}");
        String sign = SignUtils.getSignature(paramMap, "<APP_SECRET>");
    }
    //MD5原始字符串为
    appId=10000messageJson={"title": "title","content": "content","pushTimeInfo": {"offLine": 1,"validTime": 24}}pushIds=RA50c6348036344485d01776773577c64740465480a6b<APP_SECRET>
    //MD5摘要 sign为
    ac076ff25d9900015a681cb5172aa53b
```
## 接口请求示例

```
POST http://server-api-mzups.meizu.com/garcia/api/server/push/unvarnished/pushByAlias HTTP/1.1
Host: server-api-push.meizu.com
Connection: keep-alive
Content-Length: 226
Cache-Control: no-cache
Content-Type: application/x-www-form-urlencoded
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8

alias=xxx&appId=xxx&messageJson=%7B%22title%22%3A%22title%22%2C%22content%22%3A%22hello+test%22%2C%22pushTimeInfo%22%3A%7B%22offLine%22%3A1%2C%22validTime%22%3A24%7D%7D&sign=a68b75e5d5b30e35536f130cf1cae14a


HTTP/1.1 200 OK
Server: nginx
Date: Wed, 28 Dec 2016 03:34:53 GMT
Content-Type: application/json; charset=UTF-8
Content-Length: 87
Connection: keep-alive
Content-Language: zh-CN
Set-Cookie: JSESSIONID=1wl3nhcfqroiicj6pvxwdvjx6;Path=/
Expires: Thu, 01 Jan 1970 00:00:00 GMT


{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171123143026239_100000002",
        "respTarget": {
            "110003": [
                "pushId"
            ]
        }
    }
}
```



# API说明 

## 前言

> 消息推送结果接口响应部分value是map集合的json格式且只返回推送非法的pushId，合法的pushId不予返回，一般情况下，pushId未注册则视为非法。

map部分code定义

code|value
---|---
110002|pushId无效
110003|pushId非法
110005|alias无效


**注：平台使用pushId来标识每个独立的用户，每一台终端上每一个app拥有一个独立的pushId**


## 客户端说明

[客户端消息自定义行为](https://github.com/comsince/ups_meizu_pushsdk/blob/master/UpsIntegrateReadme.md#三-消息自定义行为分析)

## 推送API
### pushId推送（透传消息)

描述|内容
---|---
接口功能|pushId推送（透传）
请求方法|Post
请求路径|/ups/api/server/push/unvarnished/pushByPushId
请求HOST|server-api-mzups.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID   必填
pushIds|推送设备，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "content": 推送内容,  【string 必填，字数限制2000以内】
    "pushTimeInfo": {
        "offLine": 是否进离线消息 0 否 1 是[validTime] 【int 非必填，默认值为1】
        "validTime": 有效时长 (1- 72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    }
}
```

响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171204155029658_100000000",
        "respTarget": {},
        // cp log
        "logs": {
            "1": "msgId:NS20171204155029599_0_11168408",
            "2": "msgId:sdm15b455123738301602T",
            "3": "requestId:151237383037157123121"
        }
    }
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171123143026239_100000002",
        "respTarget": {
            "110003": [
                "flyme"
            ]
        }
    }
}
```

> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": ""
}
```


### pushId推送（通知栏）

描述|内容
---|---
接口功能|根据pushId推送（通知栏）
请求方法|Post
请求路径|/ups/api/server/push/varnished/pushByPushId
请求HOST|server-api-mzups.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
appId|推送应用ID 必填
pushIds|推送设备，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "noticeBarInfo": {
        "title": 推送标题, 【string 必填，字数限制1~32字符】
        "content": 推送内容, 【string 必填，字数限制1~100字符】
    },
    //参考客户端参数定义说明
    "clickTypeInfo": {
        "clickType": 点击动作 (0,"打开应用"),(1,"打开应用页面"),(2,"打开URI页面"),(3, "应用客户端自定义"),(4, "打开自定Intent URI");【int 非必填,默认为0】
        "url": URI页面地址, 【clickType=2，必填】
        "parameters":参数 【JSON格式】【非必填】 
        "activity":应用页面地址 应用页面地址【clickType=1，必填 格式 pkg.activity eg: com.meizu.upspushdemo.TestActivity】
        "customAttribute":应用客户端自定义【clickType=3，必填 】
        "customUri":打开自定Intent URI 【clickType=4，必填 eg:upspushscheme://com.meizu.upspush/notify_detail?title=ups title&content=ups content】
    },
    "pushTimeInfo": {
        "offLine": 是否进离线消息(0 否 1 是[validTime]) 【int 非必填，默认值为1】
        "validTime": 有效时长 (1到72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    },
    "advanceInfo": {
        "suspend":是否通知栏悬浮窗显示 (1 显示  0 不显示) 【int 非必填，默认1】
        "clearNoticeBar":是否可清除通知栏 (1 可以  0 不可以) 【int 非必填，默认1】
        "fixDisplay":是否定时展示 (1 是  0 否) 【int 非必填，默认0】
        "fixStartDisplayTime": 定时展示开始时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "fixEndDisplayTime ": 定时展示结束时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【int 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【int 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【int 非必填，默认1】
        }
    }
}

```


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171204155029658_100000000",
        "respTarget": {},
        // cp log
        "logs": {
            "1": "msgId:NS20171204155029599_0_11168408",
            "2": "msgId:sdm15b455123738301602T",
            "3": "requestId:151237383037157123121"
        }
    }
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171123143026239_100000002",
        "respTarget": {
            "110003": [
                "flyme"
            ]
        }
    }
}
```

> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": "",
}
```
### 别名推送接口（透传消息）

描述|内容
---|---
接口功能|根据别名推送
请求方法|Post
请求路径|/ups/api/server/push/unvarnished/pushByAlias
请求HOST|server-api-mzups.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
alias|推送别名，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "content": 推送内容,  【string 必填，字数限制2000字节以内】
    "pushTimeInfo": {
        "offLine": 是否进离线消息 0 否 1 是[validTime] 【int 非必填，默认值为1】
        "validTime": 有效时长 (1- 72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    }
}
```

响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171123143026239_100000002",
        "respTarget": {}
    }
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171123143026239_100000002",
        "respTarget": {
            "110003": [
                "flyme"
            ]
        }
    }
}
```

> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": "",
    "redirect": ""
}
```


### 别名推送接口（通知栏消息）

描述|内容
---|---
接口功能|根据别名推送
请求方法|Post
请求路径|/ups/api/server/push/varnished/pushByAlias
请求HOST|server-api-mzups.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
appId|推送应用ID 必填
alias|推送别名，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "noticeBarInfo": {
        "title": 推送标题, 【string 必填，字数限制1~32字符】
        "content": 推送内容, 【string 必填，字数限制1~100字符】
    },
    "clickTypeInfo": {
        "clickType": 点击动作 (0,"打开应用"),(1,"打开应用页面"),(2,"打开URI页面"),(3, "应用客户端自定义"),(4, "打开自定Intent URI");【int 非必填,默认为0】
        "url": URI页面地址, 【clickType=2，必填】
        "parameters":参数 【JSON格式】【非必填】 
        "activity":应用页面地址 应用页面地址【clickType=1，必填 格式 pkg.activity eg: com.meizu.upspushdemo.TestActivity】
        "customAttribute":应用客户端自定义【clickType=3，必填】
        "customUri":打开自定Intent URI 【clickType=4，必填 eg:upspushscheme://com.meizu.upspush/notify_detail?title=ups title&content=ups content】
    },
    "pushTimeInfo": {
        "offLine": 是否进离线消息(0 否 1 是[validTime]) 【int 非必填，默认值为1】
        "validTime": 有效时长 (1到72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    },
    "advanceInfo": {
        "suspend":是否通知栏悬浮窗显示 (1 显示  0 不显示) 【int 非必填，默认1】
        "clearNoticeBar":是否可清除通知栏 (1 可以  0 不可以) 【int 非必填，默认1】
        "fixDisplay":是否定时展示 (1 是  0 否) 【int 非必填，默认0】
        "fixStartDisplayTime": 定时展示开始时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "fixEndDisplayTime ": 定时展示结束时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【int 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【int 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【int 非必填，默认1】
        }
    }
}

```

响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171204155029658_100000000",
        "respTarget": {},
        // cp log
        "logs": {
            "1": "msgId:NS20171204155029599_0_11168408",
            "2": "msgId:sdm15b455123738301602T",
            "3": "requestId:151237383037157123121"
        }
    }
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "msgId": "UPSDEV20171123143026239_100000002",
        "respTarget": {
            "110003": [
                "flyme"
            ]
        }
    }
}
```

> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": "",
    "redirect": ""
}
```


## 统计API
### 获取应用推送统计 


描述|内容
---|---
接口功能|获取应用推送统计（最长跨度30天）
请求方法|Get
请求路径|/ups/api/server/push/statistics/dailyPushStatics
请求HOST|server-api-mzups.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
appId|推送应用ID 必填
startTime|开始日期, 如20140214 必填
endTime|结束日期, 如20140218 必填
sign|签名  必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
    "value": [
        {
            "acceptNo": 609,//接收数
            "clickNo": 30,//点击数
            "date": "2017-05-03",//推送日期
            "pushedNo": 691287,//推送总数
        },
        {
            "acceptNo": 228,
            "clickNo": 31,
            "date": "2017-05-02",
            "pushedNo": 228463,
        }
    ]
}

```

> 失败情况：


```
{
    "code": "500",
    "message": "结束时间不能早于开始时间",
    "redirect": "",
    "value": ""
}

{
    "code": "500",
    "message": "开始时间和结束时间不能相差30天以上",
    "redirect": "",
    "value": ""
}
```
