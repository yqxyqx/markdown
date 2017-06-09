**APP SDK文档**
# 1 SDK概述 ##
## 1.1 SDK功能概述
SDK包含设备绑定、设备控制、设备信息获取、设备解绑以及OTA升级等接口,方便开发者在App开发过程中对设备进行相关操作。SDK包含一个原型App开发者可以通过在原型App的基础上开发自己的App,也可以参照原型App开发自己的App。
## 1.2 名词解释
|Status Code|Method
|:-:|:-|
|AppKey|开发者在开放平台申请的账号后创建应用后会生成对应的AppKey|
|SecretKey|开发者在开放平台申请的账号后创建应用后会生成对应的SecretKey|
|AccessToken|表示用户身份的Token,调用各类App时需要,用户登录百度账号授权后可获得。注：也可使用第三方账号体系,登陆后将其传入即可|
|HttpCode|调用网络请求接口Http状态码|
|ResponseCode|调用网络请求接口时内部状态码|
|Message|调用网络请求时对应ResponseCode的状态信息|
### 1.2.1 HttpCode说明
|Status Code|Method|说明|
|:-:|:-:|-|
|200|GET/POST|成功返回用户数据|
|201|POST/PUT|用户新建或者修改数据成功|
|202|*|成功收到一个请求（异步处理）|
|204|DELETE|成功删除|
|401|*|用户验证错误（比如token错误,密码错误）|
|403|*|用户不具备权限（但是用户本身的验证是通过的）|
|404|*|URI或者资源不存在|
|405         |  * | method不支持|
|406|*|request格式错误（比如是xml格式,或者缺少某些关键参数）|
|500|*|服务器本身错误|
### 1.2.2 ResponseCode说明
|code|含义|
|-|-|
|0|正常返回,此时message为空|
|1001|认证失败|
|2001|资源不存在|
|2002|参数错误|
|2003|内部数据错误|
|3000|数据库错误|
|1201|项目名称无效|
|1202|项目描述无效|
|1203|项目名称不存在|
|1204|项目id错误|
|1205|注册版本无效|
|1206|注册硬件版本无效|
|1207|注册设备数目无效|
|1208|设备id无效|
|1210|设备已经被绑定|
|1223|app没有绑定产品|
|1301|一组设备id无效|
|1302|地域无效|
|1303|账号无效|
|1304|项目id无效|
|1305|relativeUri无效|
|1306|超时时长无效|
|1307|protocol无效|
|1308|method无效|
|1309|taskId无效|
|1310|设备id无效|
|1401|时间无效|
|1501|文件不存在|
|1502|文件格式出错|
|1503|创建ota包出错|
|1504|ota版本出错|
|1505|ota状态出错|


# 2 集成准备 ##
## 2.1  Android Studio集成
aar下载地址：
http://open.duer.baidu.com/iot/download/duer-light-android-sdk-v1.0.0-release.aar

复制duer-light-android-sdk-v1.0.0-release.aarr到项目/app/src/aar/目录 
在gradle文件中添加依赖
```
repositories{
    flatDir {
        dirs 'libs'
    }
}

dependencies {
    compile(name: 'duer-light-android-sdk-{$version}-release', ext: 'aar')
    compile 'com.google.code.gson:gson:2.4'
}
```
注意:上述{$version}指的是SDK版本,请根据下载的SDK进行导入
## 2.2  AndroidManifest.xml配置
在AndroidManifest.xml添加如下配置
```
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

<application
    android:icon="@drawable/ic_launcher"
    android:label="@string/app_name">
    <!-- begin: duer iot sdk -->
    <meta-data
        android:name="com.baidu.dueriot.APP_KEY"
        android:value="your app key" />
    <meta-data
        android:name="com.baidu.dueriot.SECRET_KEY"
        android:value="your api secret" />
    <!-- end : duer iot sdk -->

    <!-- begin: duer sdk -->
    <meta-data
        android:name="com.baidu.duersdk.APP_KEY"
        android:value="your app key" />
    <meta-data
        android:name="com.baidu.duersdk.SECRET_KEY"
        android:value="your api secret" />
    <!-- end : duer sdk -->

    <!-- begin: baidu speech sdk -->
    <meta-data
        android:name="com.baidu.speech.API_KEY"
        android:value="your api key" />
    <meta-data
        android:name="com.baidu.speech.SECRET_KEY"
        android:value="your api secret" />
    <service
        android:name="com.baidu.speech.VoiceRecognitionService"
        android:exported="false" />
    <!-- end : baidu speech sdk -->
</application>
```
# 3 SDK功能 ##
## 3.1  整体结构介绍
-IoTSDKManager:SDK API管理类,通过此类初始化SDK、获取各类API访问实例、获取用户信息、操作账号等功能 
-IoTRequestListener各类网络请求接口回调类 
-IAMWebView用于用户登录授权时封装的WebView 
-HttpStatus网络请求返回的状态信息,包含HttpCode、ResponseCode以及对应的Message 含义参照名词解释
## 3.2  接口说明
### 3.2.1  SDK初始化
#### IoTSDKManager.initSDK
**接口功能描述**
初始化SDK,在自定义Application或者MainActivity中调用如下方法,保证在调用IoT SDK接口前完成初始化,此方法可以在主线中调用。
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|Context|context|当前Context对象|

**回调函数** 
无
**实例**
```
IoTSDKManager.getInstance().initSDK(context);
```
### 3.2.2 登陆授权（百度账号有效）
#### IAMWebView.login
**接口功能描述**
登陆百度账号
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|LoginCallback|callback|登陆回调|

**回调函数** 
回调参数将在回调方法LoginCallback.onFinish中传递,返回登陆状态
**实例**
```
IAMWebView webView = (IAMWebView) findViewById(R.id.webview);
webView.login(new IAMWebView.LoginCallback() {
    @Override
    public void onFinish(boolean isSuccess) {
        if (isSuccess) {
           // login success
       }
    }
});
```
#### IoTSDKManager.getAccessToken
**接口功能描述**
获取用户AccessToken,包含token以及以及有效时间。AccessToken可以和开发者自身的账号体系进行对接。
**前置条件** 
无
**传入参数**
无
**回调函数** 
无
**实例**
```
IoTSDKManager.getInstance().getAccessToken();
```
#### IoTSDKManager.getUserInfo
**接口功能描述**
获取用户UserInfo,UserInfo中包含用户名、账号类型等信息
**前置条件** 
无
**传入参数**
无
**回调函数** 
无
**实例**
```
IoTSDKManager.getInstance().getUserInfo();
```
#### IoTSDKManager.logout
**接口功能描述**
退出当前用户账号
**前置条件** 
无
**传入参数**
无
**回调函数** 
无
**实例**
```
IoTSDKManager.getInstance().logout();
```
#### IoTSDKManager.setAccessToken
**接口功能描述**
设置第三方登陆后的AccessToken
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|AccessToken|accessToken|表示用户身份的Token|

**回调函数** 
无
**实例**
```
IoTSDKManager.getInstance().setAccessToken(accessToken);
```
### 3.2.3 设备相关接口
和设备相关的接口使用DeviceAPI调用,通过IoTSDKManager.createDeviceAPI()获取DeviceAPI实例。
#### DeviceAPI.bindDevice
**接口功能描述**
设备绑定需要设备的deviceUuid以及设备的token,开发者可以使用二维码扫描的方式扫描设备上二维码获取到,也可以通过用户手动输入获取。此处返回的DeviceInfo并非全信息
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String |deviceUuid|设备uuid|
|String |deviceToken|设备bind token|
|IoTRequestListener<DeviceInfo>|listener|请求回调|

**回调函数** 
回调参数DeviceInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().bindDevice(uuid, token, new IoTRequestListener<DeviceInfo>() {
    @Override
    public void onSuccess(HttpStatus code, DeviceInfo obj, PageInfo info) {
        // 绑定成功
    }

    @Override
    public void onFailed(HttpStatus code) {
        if (code.getResponseCode() == 1210){
            //该设备已被其他账号绑定
        } else if (code.getResponseCode() == 1223) {
            //App没有绑定该产品
        } else {
            //绑定失败
        }
    }

    @Override
    public void onError(IoTException error) {
        //绑定异常
    }
});
```
#### DeviceAPI.unBindDevice
**接口功能描述**
根据设备uuid解除设备绑定,解除绑定后无法再控制设备与获取设备信息
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String |deviceUuid|设备uuid|
|IoTRequestListener<DeviceInfo>|listener|请求回调|

**回调函数** 
回调参数DeviceInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().unBindDevice(uuid, new IoTRequestListener<DeviceInfo>() {
    @Override
    public void onSuccess(HttpStatus code, DeviceInfo obj, PageInfo info) {
        //解绑成功
    }

    @Override
    public void onFailed(HttpStatus code) {
        //解绑失败
    }

    @Override
    public void onError(IoTException error) {
        //解绑异常
    }
});
```
#### DeviceAPI.getDeviceInfo
**接口功能描述**
根据deviceUuid获取到单个设备的信息,异步返回DeviceInfo
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String |deviceUuid|设备uuid|
|IoTRequestListener<DeviceInfo>|listener|请求回调|

**回调函数** 
回调参数DeviceInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().getDeviceInfo(uuid, new IoTRequestListener<DeviceInfo>() {
    @Override
    public void onSuccess(HttpStatus code, DeviceInfo obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
#### DeviceAPI.getUnbindDeviceInfo
**接口功能描述**
根据deviceUuid获取到单个未绑定的设备的信息,异步返回DeviceInfo
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String |deviceUuid|设备uuid|
|String |bindToken|设备bindToken|
|IoTRequestListener<DeviceInfo>|listener|请求回调|

**回调函数** 
回调参数DeviceInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().getUnbindDeviceInfo(uuid, bindToken, new IoTRequestListener<DeviceInfo>() {
    @Override
    public void onSuccess(HttpStatus code, DeviceInfo obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
#### DeviceAPI.getUserAllDevices
**接口功能描述**
获取当前登录用户的所有设备信息,异步返回设备列表。支持分页,通过PageInfo设置start和limit,如果PageInfo为空则默认查询所有
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|IoTRequestListener<DeviceInfo>|listener|请求回调|
|PageInfo |pageInfo|分页信息|

**回调函数** 
回调参数DeviceInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().getUserAllDevices(new IoTRequestListener<List<DeviceInfo>>() {

    @Override
    public void onSuccess(HttpStatus code, List<DeviceInfo> obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
}, null);
```
#### DeviceAPI.setDeviceName
**接口功能描述**
通过设备deviceUuid修改设备别名,异步返回设备新信息
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String |deviceUuid|设备uuid|
|String |name|新设备名称|
|IoTRequestListener<DeviceInfo>|listener|请求回调|

**回调函数** 
回调参数DeviceInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().setDeviceName(uuid, "New Name", new IoTRequestListener<DeviceInfo>() {
    @Override
    public void onSuccess(HttpStatus code, DeviceInfo obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
#### DeviceAPI.getDeviceResource
**接口功能描述**
根据设备deviceUuid获取设备的自描述资源
设备自描述资源是指设备支持的相关属性,通过DeviceResource对这些属性进行描述。开发者可以使用这些属性对设备进行控制,读取设备的信息
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String |deviceUuid|设备uuid|
|IoTRequestListener<DeviceResource>|listener|请求回调|

**回调函数** 
回调参数DeviceResource将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().getDeviceResource(uuid, new IoTRequestListener<DeviceResource>() {
    @Override
    public void onSuccess(HttpStatus code, DeviceResource obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
#### DeviceAPI.controlDevice
**接口功能描述**
根据自描述资源中描述的参数类型,范围传入参数控制设备。成功调用后会返回对应的任务id。
支持对批量设备进行控制。
**前置条件** 
根据DeviceAPI.getDeviceResource获取到的设备自描述资源控制设备。
**传入参数**

|类型|名称|描述|
|-|-|-|
|String|device|设备uuid|
|DeviceResource.Resource|resource|设备自描述资源|
|RequestMethod|protocol|method|
|Object|value|参数值|
|IoTRequestListener|listener|请求回调|


**回调函数** 
回调参数ControlResult将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().controlDevices(device, resource, method, value, new IoTRequestListener<ControlResult>() {
    @Override
    public void onSuccess(HttpStatus code, ControlResult obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
}); 
```
#### DeviceAPI.getDeviceHistory
**接口功能描述**
根据设备的uuid,查询某个时间段内某个数据点上报的历史数据,支持分页查询
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String|deviceUuid|设备uuid|
|String|propertyKey|数据点名称|
|String|startTime|起始时间 格式"yyyy-MM-dd HH:mm:ss"|
|String|endTime|结束时间 格式"yyyy-MM-dd HH:mm:ss"|
|IoTRequestListener|listener|请求回调|
|PageInfo|info|分页信息|

**回调函数** 
回调参数List PropertyData IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().getDeviceHistory(uuid, propertyKey, startTime, endTime, new IoTRequestListener
    @Override
    public void onSuccess(HttpStatus code, List<PropertyData> obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
}, null);
```
#### DeviceAPI.getDeviceHistory
**接口功能描述**
根据设备的uuid,查询某个时间段内某个数据类型（控制数据点、上报数据点）上报的历史数据,支持分页查询
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String|deviceUuid|设备uuid|
|PropertyType|propertyType|数据点类型|
|String|startTime|起始时间 格式"yyyy-MM-dd HH:mm:ss"|
|String|endTime|结束时间 格式"yyyy-MM-dd HH:mm:ss"|
|IoTRequestListener|listener|请求回调|
|PageInfo|info|分页信息|

**回调函数** 
回调参数List PropertyData IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().getDeviceHistory(uuid, propertyKey, startTime, endTime, new IoTRequestListener
    @Override
    public void onSuccess(HttpStatus code, List<PropertyData> obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
}, null);
```
#### DeviceAPI.getDevicesOnlineStatus
**接口功能描述**
查询一组设备的uuid在线状态
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String[]|deviceUuids|一组设备uuid|
|IoTRequestListener|listener|请求回调|

**回调函数** 
回调参数List DeviceOnlineStatus Listener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createDeviceAPI().getDevicesOnlineStatus(deviceUuids, new IoTRequestListener
    @Override
    public void onSuccess(HttpStatus code, List<DeviceOnlineStatus> obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
### 3.2.4 OTA相关接口
和OTA升级相关的接口使用UpdateAPI调用,通过IoTSDKManager.createUpdateAPI()获取UpdateAPI实例。
#### UpdateAPI.checkOta
**接口功能描述**
根据设备uuid查询该设备OTA信息
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String|deviceUuid|设备uuid|
|IoTRequestListener|listener|请求回调|

**回调函数** 
回调参数List<DeviceOtaInfo>将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**

```
IoTSDKManager.getInstance().createUpdateAPI().checkOta(uuid, new IoTRequestListener<DeviceOtaInfo>() {
    @Override
    public void onSuccess(HttpStatus code, DeviceOtaInfo otaInfo, PageInfo info) {
        if (otaInfo.otaTaskId > 0) {
            //正在更新固件
        } else if (otaInfo.otaTaskId == 0) {
            //可更新到固件版本：otaInfo.newOsVersion
        } else {
            //当前是最新版本
       }
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
#### UpdateAPI.getOtaPackageInfo
**接口功能描述**
根据OTA packageId查询OTA升级包信息
**前置条件** 
根据UpdateAPI.checkOta获取到的OTA packageId
**传入参数**

|类型|名称|描述|
|-|-|-|
|String|packageId|OTA包id|
|IoTRequestListener|listener|请求回调|

**回调函数** 
回调参数OtaPakageInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createUpdateAPI().getOtaPackageInfo(packageId, new IoTRequestListener<OtaPackageInfo>() {
    @Override
    public void onSuccess(HttpStatus code, OtaPackageInfo obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```

#### UpdateAPI.createOtaTask
**接口功能描述**
根据DeviceOtaInfo传入参数执行设备升级。成功调用后会返回OTA任务是否创建成功。
**前置条件** 
UpdateAPI.checkOta获取到的DeviceOtaInfo
**传入参数**

|类型|名称|描述|
|-|-|-|
|DeviceOtaInfo|otaInfo|OTA信息实体|
|IoTRequestListener|listener|请求回调|

**回调函数** 
回调参数DeviceOtaInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createUpdateAPI().createOtaTask(otaInfo, new IoTRequestListener<Boolean>() {
    @Override
    public void onSuccess(HttpStatus code, Boolean obj, PageInfo info) {
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
#### UpdateAPI.getOtaTaskStatus
**接口功能描述**
根据设备uuid传入参数查询OTA状态
**前置条件** 
无
**传入参数**

|类型|名称|描述|
|-|-|-|
|String|deviceUuid|设备uuid|
|IoTRequestListener|listener|请求回调|

**回调函数** 
回调参数OtaTaskInfo将在回调方法IoTRequestListener.onSuccess中传递,返回data
**实例**
```
IoTSDKManager.getInstance().createUpdateAPI().getOtaTaskStatus(deviceUuid, new IoTRequestListener<OtaTaskInfo>() {
    @Override
    public void onSuccess(HttpStatus code, OtaTaskInfo obj, PageInfo info) {
        if (obj.status == OtaTaskInfo.OtaStatus.SUCCESS) {
            //升级成功
        } else if (obj.status == OtaTaskInfo.OtaStatus.FAILED) {
            //升级失败
        }
    }

    @Override
    public void onFailed(HttpStatus code) {
    }

    @Override
    public void onError(IoTException error) {
    }
});
```
