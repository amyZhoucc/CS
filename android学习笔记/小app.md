# APP的开发记录

## 0. 天气接口

开发的是一个查询天气的app，需要用到外部的天气情况接口：

网址：https://dashboard.caiyunapp.com/——可以申请得到调用令牌

查询地址：输入名字，返回经纬度等具体地理信息

```
https://api.caiyunapp.com/v2/place?query=beijing&token={token}&lang=zh_CN
```

——可以快速获取指定位置的**经纬度信息**

Token：就是我们申请得到的调用令牌，填入即可

服务器会返回我们一段**JSON格式的数据**

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210609112148133.png" alt="image-20210609112148133" style="zoom:80%;" />

然后使用api接口，去根据经纬度返回具体的天气信息：——实时天气、预测天气

北京："lat":39.939268,"lng":116.357122，注意这边的经度是lng，纬度是lat，要反着来

```
https://api.caiyunapp.com/v2.5/{token}/经度,纬度/realtime.json	// 实时天气
https://api.caiyunapp.com/v2.5/{token}/经度,纬度/daily.json
```

```json
{"status":"ok","api_version":"v2.5","api_status":"active","lang":"zh_CN","unit":"metric","tzshift":28800,"timezone":"Asia\/Shanghai","server_time":1623834145,"location":[39.939268,116.357122],"result":{"realtime":{"status":"ok","temperature":20.79,"humidity":0.88,"cloudrate":0.9,"skycon":"CLOUDY","visibility":4.0,"dswrf":60.4,"wind":{"speed":3.96,"direction":93.0},"pressure":99528.14,"apparent_temperature":22.5,"precipitation":{"local":{"status":"ok","datasource":"radar","intensity":0.0},"nearest":{"status":"ok","distance":19.41,"intensity":0.1875}},"air_quality":{"pm25":23.0,"pm10":31.0,"o3":58.0,"so2":2.0,"no2":18.0,"co":0.8,"aqi":{"chn":33.0,"usa":74.0},"description":{"chn":"优","usa":"良"}},"life_index":{"ultraviolet":{"index":1.0,"desc":"很弱"},"comfort":{"index":4,"desc":"温暖"}}},"primary":0}}
```

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210609113702104.png" alt="image-20210609113702104" style="zoom:80%;" />

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210609113835556.png" alt="image-20210609113835556" style="zoom:80%;" />

## 1. 思想概念MVVM

MVVM架构可以将程序结构主要分成3部分：

- Model是数据模型部分
- View是界面展示部分
-  ViewModel是连接数据模型和页面展示的桥梁

——业务逻辑和界面展示分离

<img src="/Users/bytedance/Library/Application Support/typora-user-images/截屏2021-06-08 下午8.37.29.png" alt="截屏2021-06-08 下午8.37.29" style="zoom:100%;" />

- UI控制层包含了我们平时写的Activity、Fragment、布局文件等与**界面相关**的东西

  ——view层

- ViewModel层用于持有和UI元素相关的**数据**，以保证这些数据在屏幕旋转时不会丢失，并且还要提供接口给UI控制层调用以及和仓库层进行通信

  ——viewModel层

- 仓库层要做的主要工作是判断调用方请求的数据应该是从本地数据源中获取还是从网络数据源中获取，并将获取到的数据返回给调用方

  ——model层

  本地数据源可以使用数据库、 SharedPreferences等持久化技术来实现，而网络数据源则通常使用**Retrofit**访问服务器提供 的Webservice接口来实现。

## 2. 相关依赖

目前书上的部分已经不是最新的了，sync之后会有提示，可以进行升级（不知道是否存在兼容性问题）

```
implementation 'com.google.android.material:material:1.1.0'
```

由于引入了Material库，所以一定要记得将AppTheme的parent主题改成 MaterialComponents模式，也就是将原来的AppCompat部分改成MaterialComponents即可

==material默认已经加入了，不需要重复导入==

——定位：`AndroidManifest.xml`中，application中

```
<application
	.....
	android:theme="@style/Theme.Weather">		// 这边设定了主题
```

然后在res/values里面有一个`themes.xml`的文件，这个就是整个app的主题

默认是：

```
<style name="Theme.Weather" parent="Theme.AppCompat.Light.NoActionBar">
```

修改成：

```
<style name="Theme.Weather" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
```

然后就变成material风格的了

目前需要的依赖：

```
//    为了用retrofit，网络通信
implementation 'com.squareup.retrofit2:retrofit:2.6.1'
implementation 'com.squareup.retrofit2:converter-gson:2.6.1'

// 为了用协程
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.0"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1"

// lifecycle，在非activity中感知activity的，从而调用对应的代码
implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"

// 使用recyclerview
implementation 'androidx.recyclerview:recyclerview:1.2.1'
```

## 3. 整体逻辑

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20210615111604463.png" alt="image-20210615111604463" style="zoom:67%;" />

1. logic包用于存放业务逻辑相关的代码
   - dao：用于存放数据访问对象
   - Model：对象模型
   - Network：网络相关的代码
2. ui包用于存放界面展示相关的代码
   - place：页面1
   - weather：页面2

## Part1 搜索全球城市数据

