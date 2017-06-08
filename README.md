# openapi

### 2.5. open api @云翔（本期暂无）client_id 
#### 2.5.1. open api概述
-open api的功能？
-open api适合谁使用？在什么情况下使用？


#### 2.5.2. 获取方式
##### 1. 接口访问签名
部分接口需要提供在HTTP Header里面提供签名才能被允许访问.
签名`X-IOT-Signature`生成方法如下:
`{nonce}:SHA1({nonce}+{apikey}+{secret}+{token})`
各参数说明:
- `nonce` (string) 由16个随机字符组成: a-z, A-Z, 0-9; 同一个Nonce只能使用一次, 每次请求接口都必须重新生成
- `apikey` (string) API KEY
- `secret` (string) 密钥
	- **WEB应用** 使用 APP Secret Key
	- **Android应用** 使用 包名+apk签名
	- **iOS应用** 使用 BundleID
- `token` (string) APP得到用户授权后拿到的`access_token`, 如果某些接口不需要验证该token, 则该值填空

##### 2. 接口标签说明
**部分接口会有★标签，调用这些接口需要前提条件**

|标签|说明|
|-|-|
|★|调用这些接口，需要设备与用户先进行绑定|

###### **使用接口必要的流程和配置**
+  先使用百度账号登录console云平台（http://......）
+  新建一个产品，默认会产生20个测试设备，或者通过创建批次获取更多的设备
	+  可有设备使用，主要使用设备的数据是deviceUuid和相应的token（bind_token）
+  为产品添加数据点（或者直接添加数据点模板）
	+  可有控制数据点使用
+  创建APP
	+  获得appId，AK，SK，根据不用的应用系统配置相应的参数，用于身份签名和验证
+  产品和APP做绑定
	+  产品绑定到APP后，APP才能绑定控制产品相应的设备
+  以上步骤都完成，此文档的接口才能使用
	+  目前为了测试此文档接口，我们已经把以上步骤全部做完，绑定产品的projectId为424，可从device表中选出424产品的设备做测试

##### 3. 测试OTA接口的必要流程
+  明确烧入设备所用profile的uuid，所属的批次id、批次号、固件版本号
+  添加一个固件版本，版本号大于现有最新版本号
+ 点击查看，进入详细信息，进入立马验证页面
+ 输入uuid，验证设备升级状态，验证通过后，点击通过验证
+ 返回详细信息页面，创建策略。如果想让设备有升级策略，需要注意以下填写方法
	+ 升级方式：选择用户确认升级，设备就不会静默升级
	+ 旧版本中必须包含测试设备现有版本号
	+ 批次筛选中必须包含测试设备所属的批次号
+ 确认策略后，点击策略发布
+  以上步骤都完成，OTA相关接口测试才会有效，否则设备不会正常升级

**错误码**定义如下:

|code|含义|
|-----|---|
|1204|project id error|
|1208|device uuid invalid|
|1209|device not exist|
|1211|accountUuid not found|
|1212|user and devices do not bind|
|1218|'user no permissions|
|1219|device token invalid|
|1223|app not bind project|
|1228|token and device do not match|
|1305|command name invalid|
|1308|method invalid|
|1401|data query params invalid|
|1506|package id invalid|
|1801|device id is empty|
|1802|account has not power to access this device|
|1803|the number of device beyond 20|
|1902|lack of strategy id or valid|
|1905|user can not use this strategy id|
|2001|create ota task params is error|
|2004|device is running to update|
|2005|task id or device uuid is error|
|2007|device version is higher than package version|


#### 2.5.3. open api功能
##### 1、移动客户端(Android/iOS/SDK) OAuth 相关接口
###### **1.1用户登陆授权 [GET /account/authorize]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/account/authorize?client_id=[client_id]&response_type=[response_type]&state=[state]&scope=[scope]&redirect_uri=[redirect_uri]

+ Parameters
|名称|类型|说明|
|-|-|
|client_id|string|填写appid
|response_type| string| 直接填写值: token, 使用简化模式|
|state|string| 客户端的当前状态, 可以指定任意字符串，认证服务器会原封不动地返回这个值|
|scope|string|APP所需要的权限范围, 多个权限范围用逗号’,’分隔; 该参数内容必须不多于APP在开发者中心申请的权限范围|
|redirect_uri|string|直接填写http://iam.iot.baidu.com/v1/iam/oauth/receive_implicit_token, 使用urlencode编码,拼接示例:/account/authorize?response_type=token&client_id=1234567&state=123&redirect_uri=http://iam.iot.baidu.com/v1/iam/oauth/receive_implicit_token,用户登录（百度账号）并授权操作完成后会跳转到redirect_uri回调地址并带上相关返回参数; access_token,state,expires_in

**Response 302**

+ Headers 
Location: http://iam.iot.baidu.com/v1/iam/oauth/receive_implicit_token/#access_token=eed3d99f7122cee3bcf7ce5d500a0ffb&state=123&scope=allScope&expires_in=5184000

+ Parameters
|名称|类型|说明|
|-|-|
|access_token|string|API访问令牌
|state|string|返回请求时传入的state参数
|expire|int|过期时间(秒)

###### **1.2检查Token是否有效 [GET /account/validate]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/account/validate

+ Headers
```
http
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200**
```
{
    "data": true,
    "code": "0",
    "message": ""
}
```
**异常输出**
|status|说明|
|-|-|
|404|access_token不存在|
|426|access_token过期，需要重新授权|
|500|服务器错误|

###### **1.3获取用户信息 [GET /account/infos]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/account/infos

+ Headers
```
http
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200 (application/json)**
```
{
  "data": {
    "uuid": "5_2500292884",
    "originId": "2500292884",
    "originType": "USER_BAIDU_PASSPORT",
    "userName": "test7777",
    "displayName": "test7777",
    "status": "ENABLE",
    "createTime": "2017-03-22T05:46:19Z"
  },
  "code": "0",
  "message": ""
}
```
##### 2、设备绑定相关接口

###### **2.1 用户绑定设备 [POST /device/bind]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/bind

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备唯一的标识id|
|token|string|设备绑定的token|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

+ body
```
{
    "deviceUuid": "0083000000000a",
    "token": "6dd5790b2163a0132e6c65c6dd6f9ef2"
}
```

**Response 200 (application/json)**
```
{
    "data": true,
    "code": "0",
    "message": ""
}
```
+ 异常输出

|status|说明|
|-|-|
|404|	accountUuid not found|
|406|device has been bind!|
|500|	服务器错误|

###### **2.2*修改绑定设备名称 [PUT /device/name/update]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/name/update

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备唯一的标识id|
|name|string|设备名称|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

+ body
```
{
    "deviceUuid": "0083000000000a",
    "name": "testname"
}
```

**Response 201 (application/json)**
```
{
  "data": {
    "id": 22288,
    "uuid": "0083000000000a",
    "projectId": 131,
    "accountUuid": "5_27712168",
    "deviceBatchId": 106,
    "name": "testname",
    "bindToken": "6dd5790b2163a0132e6c65c6dd6f9ef2",
    "osVersionId": 160,
    "status": "UNACTIVATED",
    "version": 1,
    "activateTime": null,
    "lastHeartbeatTime": null,
    "createTime": "2017-06-07T02:46:30Z",
    "updateTime": "2017-06-07T06:10:14Z"
  },
  "code": "0",
  "message": "",
  "total": 1
}
```

###### **2.3*解除绑定 [POST /device/unbind]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/unbind

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备唯一的标识id|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

+ body
```
{
    "deviceUuid": "0083000000000a"
}
```

**Response 200 (application/json)**
```
{
  "data": true,
  "code": 0,
  "message": ""
}
```

###### **2.4获取用户所有绑定的设备 [GET /device/bind/list]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/bind/list

+ parameters

|名称|类型|说明|
|-|-|-|
|start |int|起始元素, 默认0|
|limit |int| 每页大小, 每页的个数，默认20且不得超过1000|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200 (application/json)**
```
{
  "data": [
    {
      "id": 22289,
      "projectId": 131,
      "accountUuid": "5_27712168",
      "deviceBatchId": 106,
      "name": "",
      "bindToken": "9614b7fd4af0305d412056cc0391d4b1",
      "osVersionId": 160,
      "status": "UNACTIVATED",
      "version": 1,
      "activateTime": null,
      "lastHeartbeatTime": null,
      "createTime": "2017-06-07T02:46:30Z",
      "updateTime": "2017-06-07T02:46:30Z",
      "osVersion": "0.0.0.0",
      "deviceUuid": "0083000000000b",
      "projectName": "doc_test",
      "type": "WIFI",
      "seriesName": "故事机"
    },
    {
      "id": 22288,
      "projectId": 131,
      "accountUuid": "5_27712168",
      "deviceBatchId": 106,
      "name": "testname",
      "bindToken": "6dd5790b2163a0132e6c65c6dd6f9ef2",
      "osVersionId": 160,
      "status": "UNACTIVATED",
      "version": 1,
      "activateTime": null,
      "lastHeartbeatTime": null,
      "createTime": "2017-06-07T02:46:30Z",
      "updateTime": "2017-06-07T06:10:14Z",
      "osVersion": "0.0.0.0",
      "deviceUuid": "0083000000000a",
      "projectName": "doc_test",
      "type": "WIFI",
      "seriesName": "故事机"
    }
  ],
  "code": 0,
  "message": "",
  "limit": 100,
  "start": 0,
  "total": 2
}
```

+ parameters

|名称|说明|
|-|-|
|version|设备使用版本：0为研发中的设备，1为投入使用的设备|

###### **2.5根据设备UUID和token获取指定设备详细信息 [GET /device/info/by_token]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/info/by_token?deviceUuid=[deviceUuid]&token=[token]

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备唯一的标识id|
|token|string|设备token|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200 (application/json)**
```
{
  "data": {
    "id": 22288,
    "uuid": "0083000000000a",
    "projectId": 131,
    "accountUuid": "5_27712168",
    "deviceBatchId": 106,
    "name": "testname",
    "bindToken": "6dd5790b2163a0132e6c65c6dd6f9ef2",
    "osVersionId": 160,
    "status": "UNACTIVATED",
    "version": 1,
    "activateTime": null,
    "lastHeartbeatTime": null,
    "createTime": "2017-06-07T02:46:30Z",
    "updateTime": "2017-06-07T06:10:14Z",
    "osVersion": "0.0.0.0",
    "deviceUuid": "0083000000000a",
    "projectName": "doc_test",
    "type": "WIFI",
    "seriesName": "故事机",
    "batchName": "0000"
  },
  "code": "0",
  "message": "",
  "total": 1
}
```

###### **2.6*根据设备UUID获取指定设备详细信息（需要用户绑定设备）[GET /device/info/by_uuid]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/info/by_uuid?deviceUuid=[deviceUuid]

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备唯一的标识id|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200 (application/json)**
```
{
  "data": {
    "id": 22288,
    "uuid": "0083000000000a",
    "projectId": 131,
    "accountUuid": "5_27712168",
    "deviceBatchId": 106,
    "name": "testname",
    "bindToken": "6dd5790b2163a0132e6c65c6dd6f9ef2",
    "osVersionId": 160,
    "status": "UNACTIVATED",
    "version": 1,
    "activateTime": null,
    "lastHeartbeatTime": null,
    "createTime": "2017-06-07T02:46:30Z",
    "updateTime": "2017-06-07T06:10:14Z",
    "osVersion": "0.0.0.0"
  },
  "code": "0",
  "message": "",
  "total": 1
}
```
##### 3、设备控制
###### **3.1*获取设备自描述资源 [GET /device/resource]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/resource?deviceUuid=[deviceUuid]

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备唯一的标识id|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200 (application/json)**
```
{
  "data": {
    "resources": [
      {
        "name": "play",
        "resourceType": "voice",
        "type": "string",
        "label": "播放内容",
        "description": "可通过“播放儿歌”“播放春天在哪里”“上一首”“下一首”等语音指令控制播放内容",
        "pattern": {},
        "uri": "/play",
        "protocol": [
          "COAP",
          "HTTP"
        ],
        "method": [
          "GET",
          "PUT"
        ]
      },
      {
        "name": "volume",
        "resourceType": "voice",
        "type": "int",
        "label": "音量",
        "description": "可通过“调高音量”“调低音量”等语音指令调节设备音量",
        "pattern": {
          "min": 0,
          "max": 16,
          "step": 1
        },
        "uri": "/volume",
        "protocol": [
          "COAP",
          "HTTP"
        ],
        "method": [
          "GET",
          "PUT"
        ]
      },
      {
        "name": "switch",
        "resourceType": "",
        "type": "bool",
        "label": "关机",
        "description": "关闭设备",
        "uri": "/switch",
        "protocol": [
          "COAP",
          "HTTP"
        ],
        "method": [
          "GET",
          "PUT"
        ]
      },
      {
        "name": "power",
        "resourceType": "",
        "type": "int",
        "label": "电量",
        "description": "查询电量",
        "pattern": {
          "min": 0,
          "max": 100,
          "step": 1
        },
        "uri": "/power",
        "protocol": [
          "COAP",
          "HTTP"
        ],
        "method": [
          "GET"
        ]
      }
    ],
    "report": []
  },
  "code": "0",
  "message": "",
  "total": 1
}
```
+ parameters

|resourceType|说明|
|-|-|
|voice|语音数据点|


###### **3.4*对指定设备发送控制命令 （同步）[POST /device/control/send]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/control/send

+ parameters

|名称|类型|说明|
|-|-|
|deviceUuid|string|设备唯一的标识id|
|method|string|GET读取数据点数据，PUT修改数据点数据|
|name|string|数据点标识名|
|content|string|method为PUT，修改的值|



+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

+ body
```
{
    "deviceUuid":"01a800000072af",
	"method":"PUT", 
	"name":"power", 
	"content":"1"
}
```

**Response 200 (application/json)**
```
{
  "data": {
     "code": "2.05",
     "content": "75"，
     "name":"power", 
  },
  "total": 1,
  "code": "0",
  "message": ""
}
```

##### 4、数据查询
###### **4.1查询设备是否在线 [POST /device/query/data/connect]** 
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/query/data/connect

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuids|string[ ]|设备uuid数组,一次最多查询20个设备uuid|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

+ body
```
{
    "deviceUuids":["0083000000000a","0083000000000b"]
}
```

**Response 200 (application/json)**
```
{
  "data": [
    {
      "deviceUuid": "0083000000000a",
      "status": false
    },
    {
      "deviceUuid": "0083000000000b",
      "status": false
    }
  ],
  "code": "0",
  "message": "",
  "total": 1
}
```

###### **4.2*查询设备历史数据 [GET /device/query/data/history]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/query/data/history?deviceUuid=[deviceUuid]&propertyKey=[propertyKey]&propertyType=[propertyType]&startTime=[startTime]&endTime=[endTime]&nextPage=[nextPage]

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string| 设备ID|
|propertyKey|string| 数据点，可选，不选默认全部数据点
|propertyType|string| 数据点类型（REPORT,RESOURCE），可选，不选默认全部数据点
|startTime|string|utc时间，例如为2017-04-14T08:31:10Z
|endTime|string|utc时间，limit (int) 默认20，不能超过100
|nextPage |long|下一页数据的参数，如果为null，则表示没有数据



+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200 (application/json)**
```
{
    "data": [
    {
      "timeStamp": 1495044775000,
      "value": "{id=108956,+rate=8000,+segment=15,+eof=true,+channel=1}",
      "key": "voice_control"
    },
    {
      "timeStamp": 1495034776000,
      "value": "{id=108956,+rate=8000,+segment=7,+voice_data=JgAAACwL9NxOU7tUy0tFYxxkcVz1h8fh02TJ/VEyVc9mtSD2fK0fsMl3JgAAACunuCmTZbF4FkujpJ4ZJXdZOckSLoOx5Y81du5dR8Roa99kmYlHJgAAAC+2OCt1oWqfuMN1dQUjF5kdiYX5vlJPsfCnXPVXiXk7MVMG9HFnJgAAACk65VyUSh4jsa8DdeoliWRmBhFkhhKncvUyC6B5FujAXXDmUvKXJgAAACkyl1yTTcqk3BKz43cmMU2B1bu6dpMXqiHOuz0ESYvJ2RU8X5bnJgAAACkyl1yyTF995XLf6sbe4XjP6BJjHFOHmCSPPOekCeJWi1IkRzdnJgAAACkii2nSTTLLh7aTF5ImRV9AlkFmd5MHjsqAUuNvKbxR55ZX2Eh3JgAAACk8i1XvS38iJoXiNbHCg8zZKfC/AJMbiNnYYdQREGrWDhhfrI4XJgAAAC24ZhLNIs9I/dEPzfB6q4HjUTa7At2H0FV32IyR4WPrkCy4Nh33JgAAAC+T5JTQJZ6Q9IyPxvJ9Ost7hOvO64T5g7s69SaiC8Tu8MopR7HHJgAAACoXH/fw0e9Yh8q1Sk7W4UTtl8rr6S1B6c/Uj4UC9W7r7pc2WGXHJgAAACoXWvfyMCZeZhKa05Gtk0cdwZ+bDSHLv10+d9adOYRi8EnmDaanJgAAACgTsNjSxas/2fqHcNk5n4F0WTmgQMwOqrGGF6On/lnq7ragwDbXJgAAAC4c/PDSOaOH9ZNsnhMCzWnVl9leQicj3Q4eoHNk4wHtk1rAv7vnJgAAACgLWtgTKI6Bi01uLkNmmUP6HcZmL8dB2B3FvWh+4E/R1VA+JyzH,+eof=false,+channel=1}",
      "key": "voice_control"
    },
    {
      "timeStamp": 1495034185000,
      "value": "{id=108955,+rate=8000,+segment=15,+eof=true,+channel=1}",
      "key": "voice_control"
    },
    {
      "timeStamp": 1495034085000,
      "value": "{id=108955,+rate=8000,+segment=14,+voice_data=JgAAACzTCkdJzVapC3RUm9emm6IxGPJJcuDH0UWIsjA4vvjtpoCENhb3JgAAACk3nlTJZcNGkOOmJPmxD75a+oNHLOLc3/bMqVlXQOHse+5pZU8HJgAAAC/fKr9Jerd7UihNgdzxN5gWPBATBj09yDo0/IhaAOP/ras+0KrnJgAAAC3Gk2SJRnehkLde64+4+fAbjpT2SkdV1FuzSEbMo27d8hxbbaunJgAAACzbZA5I9/s2Kul+xdeSD5CrV9v4yQeH2JyFXBavYEFnru55xMFHJgAAACzdSr5IdrbnqtKPqlnXzbQ7kR69bL9wwXXnOwaAaursi0JuO/jnJgAAAC3GnK+JpXcc0PWP6JfOO5QF4LsELq+N0W1xATL83D5tKB/yIvVHJgAAAC3crKtJGrvRsE8y6Wwpj5y15Bjrbo09yjKbHUnmXuboRYJzBOrXJgAAACzOFL9JtPdt9bMK/HVq4XrXnaod28xdoUwsDezFCZbpjnh/hFyHJgAAACzGir5poV9V9ADljYWUN7I9nXC5GO84qk4aEt3WBv5RHAd7/eQnJgAAACzear5pMzc9eP3IeXZoj2YeKQRguSUZ0A4lV2556w3Um07cVKJHJgAAACk2uFco3Y8KiiCLDWIt9ZhetT1ljfZRwV+3T2reSabukG1rlxlHJgAAACzGC9qKim5CJUQyPeEYMzTCMXrHQdmdqmonOTNPh4rnZC2GWB+nJgAAACzSvrtJAn9pFXsdHtVhW12QSROPpB5Tte1EEFcp483M71quHwvHJgAAACzd6r2IFX9GaZPG2dBPj4ir1RfpxRz2xa8Qi44Uky7X40BaqpLH,+eof=false,+channel=1}",
      "key": "voice_control"
    },
    ...
    ]
    "nextPage": 1495034775000,
    "limit": 4
}
```

###### **4.3*查询设备实时数据[GET /device/query/data/realtime]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/query/data/realtime

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string| 设备ID
| propertyKey |string| 设备自描述上报的字段，比如空调有温度属性（temperature），设备会上报temperature这个字段，那么对温度的历史数据查询就是：propertyKey = temperature


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```


**Response 200 (application/json)**
```
{
    "data": {
        "value": 94,
        "timeStamp": 1495034775000
    },
    "code": "0",
    "message": "",
    "total": 1
}
```
##### 5、用户设备升级
###### **5.1*查询设备最新版本 [GET /device/ota/new_version]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/ota/new_version?deviceUuid=[deviceUuid]

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备ID|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```


**Response 200 (application/json)**
```
{
  "data": {
    "deviceUuid": "01a800000072b0",
    "packageId": 484,
    "strategyId": 1132,
    "type": "PASSIVE",
    "newOsVersion": "1.1.1.1"，
    "originalOsVersion": "1.0.1.1"
  },
  "code": "0",
  "message": "",
  "total": 1
}
```
+ parameters

|data|otaTaskId|说明|
|-|-|-|
|null||设备不需要升级|
|不为null|大于0|设备正在升级|
|不为null|otaTaskId=0|type为 “PASSIVE”，需要用户手动升级|

###### **5.2获取版本详情 [GET /device/ota/version/info]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/ota/version/info?packageId=[packageId]

+ parameters

|名称|类型|说明|
|-|-|-|
|packageId|long|OTA版本id|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```


**Response 200 (application/json)**
```
{
     "data": {
          "id": 2,
          "projectId": 4,
          "osVersionId": 4,
          "description": "tt",
          "size": 521,
          "status": "RELEASED",
          "createTime": "2015-11-07T06:07:51Z",
          "updateTime": "2015-11-07T06:07:51Z",
          "osVersion": "1.0.0.0"
      },
      "total": 1,
      "code": "0",
      "message": ""
 }
```

###### **5.3*指定设备按照指定OTA版本升级 [POST /device/ota/update]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/ota/update

+ parameters

|名称|类型|说明|
|-|-|-|
|packageId|string|OTA版本id|
|deviceUuid|string|设备uuid|
|strategyId|string||


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

+ body
```
{
    "packageId": 480,
    "deviceUuid":"01a60000000004",
    "strategyId": 123 //可选
}
```

**Response 201 (application/json)**
```
{
  "data": true,
  "code": "0",
  "message": "",
  "total": 1
}
```

###### **5.4*查询升级状态 [GET /device/ota/update/status]**
**Request**

+ url 
https://openapi-iot.baidu.com/v1/device/ota/update/status?deviceUuid=[deviceUuid]

+ parameters

|名称|类型|说明|
|-|-|-|
|deviceUuid|string|设备uuid|


+ headers
```
X-IOT-APP: {apikey}
X-IOT-Signature: {sign}
X-IOT-Token: {token}
```

**Response 200 (application/json)**
```
{
  "data": {
    "id": 77,
    "taskId": 13450,
    "deviceUuid": "01a800000072db",
    "packageId": 508,
    "otaStrategyId": 1148,
    "osVersionId": 751,
    "status": "SUCCESS",
    "eventLog": "2017-04-10 10:26:02 [Success] OK\n2017-04-11 02:26:02 {\"transaction\":\"77\",\"event\":0}\n2017-04-11 02:26:20 {\"percent\":0,\"transaction\":\"77\",\"event\":4}\n2017-04-11 02:26:20 {\"transaction\":\"77\",\"event\":6}\n2017-04-11 02:26:20 {\"transaction\":\"77\",\"event\":2}\n2017-04-11 02:26:20 {\"transaction\":\"77\",\"event\":10}\n",
    "createTime": "2017-04-10T10:26:00Z",
    "updateTime": "2017-04-10T10:26:21Z",
    "osVersion": "4.0.0.2"
  },
  "code": "0",
  "message": "",
  "total": 1
}
```

+ parameters

|status|添加字段|说明|
|-|-|-|
|CREATED||暂未开始升级|
|SUCCESS||升级成功|
|INSTALLED||推送成功|
|RUNNING|补充percent字段|升级进度，0~1|
|FAILED|补充failure字段|错误描述|
