###前言
微信小程序是一种不需要下载安装即可使用的应用，它实现了应用“触手可及”的梦想，用户扫一扫或者搜一下即可打开应用。也体现了“用完即走”的理念，用户不用关心是否安装太多应用的问题。微信“小程序”可以为开发者提供基于微信的表单、导航、地图、媒体和位置等开发组件，让他们在微信的网页里构建一个HTML 5应用。同时微信还开放了登录和微信支付等接口，让这个“小程序”可以和用户的微信账号打通。微信将“小程序”定义为“一种新的应用形态”。微信方面强调，小程序、订阅号、服务号、企业号目前是并行的体系。
###目录结构

文件结构

小程序包含一个描述整体程序的 app 和多个描述各自页面的 page。

一个小程序主体部分由三个文件组成，必须放在项目的根目录，如下：
| 文件 | 必填 | 作用 |
| -------- |:-------:| -----:|
|app.js |是 |小程序逻辑|
|app.json|是 |小程序公共设置|
|app.wxss|否 |小程序公共样式表|

一个小程序页面由四个文件组成，分别是：
| 文件类型 | 必填 | 作用 |
| ------------- |:-------------:| -----:|
|js 	|是 	|页面逻辑|
|wxml 	|是 	|页面结构|
|wxss 	|否 	|页面样式表|
|json 	|否 	|页面配置|

注意：为了方便开发者减少配置项，我们规定描述页面的这四个文件必须具有相同的路径与文件名。
###配置
我们使用app.json文件来对微信小程序进行全局配置，决定页面文件的路径、窗口表现、设置网络超时时间、设置多 tab 等。

以下是我们天气小程序的简单配置app.json ：
```json
{
  "pages": [
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window": {
    "backgroundTextStyle": "light",
    "navigationBarBackgroundColor": "#2f3b53",
    "navigationBarTitleText": "Weather",
    "navigationBarTextStyle": "white"
  },
  "tabBar": {
    "selectedColor": "#73d0f4",
    "list": [
      {
        "pagePath": "pages/index/index",
        "iconPath": "assets/images/home_n.png",
        "selectedIconPath": "assets/images/home.png",
        "text": "首页"
      },
      {
        "pagePath": "pages/logs/logs",
        "iconPath": "assets/images/box_n.png",
        "selectedIconPath": "assets/images/box.png",
        "text": "日志"
      }
    ]
  }
}
```
然后是app.wxss
```css
/**app.wxss**/
.container {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding: 200rpx 0;
  box-sizing: border-box;
} 
```
然后是我们的首页结构,这里我们发现首页的结构和react,angularJS,vue等MVC,MVVC框架很像,一样的使用了双大括号来绑定数据
```html

<!--index.wxml-->
<view class="container">
  <view bindtap="bindViewTap" class="userinfo">
    <image class="userinfo-avatar" src="{{userInfo.avatarUrl}}" background-size="cover"></image>
    <text class="userinfo-nickname">{{userInfo.nickName}}</text>
  </view>
  <view wx:if="{{weather.location}}" class="weather">
    <text class="city">所在地 {{weather.location.name}}</text>
    <text class="condition">{{weather.now.text}} {{weather.now.temperature}} ℃</text>
  </view>
</view>
```
然后是index的样式,这里微信使用了rpx作为像素单位
```css
/**index.wxss**/

.userinfo {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.userinfo-avatar {
  width: 128rpx;
  height: 128rpx;
  margin: 20rpx;
  border-radius: 50%;
}

.userinfo-nickname {
  color: #aaa;
}

.usermotto {
  margin-top: 200px;
}

.weather {
  margin-top: 50px;
  display: flex;
  flex-direction: column;
}

.city, .condition, .temp {
  width: 100%;
  margin-top: 10px;
}

```
接下来就是重要的index.js了
```javascript
//index.js
//获取应用实例
var app = getApp()
Page({
  data: {
    userInfo: {},
    weather: {}
  },
  //事件处理函数
  bindViewTap: function () {
    this.getLocation() 
  },
  getLocation: function () {
    var that = this
    wx.getLocation({ //调用wxAPI获取当前位置
      type: 'wgs84',
      success: function (res) {
        var latitude = res.latitude
        var longitude = res.longitude
        wx.request({
          //设置为自己的key
          url: 'https://api.thinkpage.cn/v3/weather/now.json?key=xxxxxxxxxxxxxxxx&location=' + latitude + ':' + longitude,
          success: function (res) {
            console.log(res.data.results[0])
            that.setData({
              weather: res.data.results[0] //返回值为JSON数据可以直接调用
            })
          }
        })
      }
    })
  },
  onLoad: function () {
    console.log('onLoad')
    //var that = this
    //调用应用实例的方法获取全局数据
    app.getUserInfo((userInfo) => {
      //更新数据
      this.setData({
        userInfo: userInfo
      })
    })
    //that.getLocation()
  }
})
```

这里使用的是心知天气,是一个免费的天气查询api注册一个免费的帐号将上面的key换成你得到的key,
心知API返回JSON参照:
```json
{
  "results": [{
  "location": {
      "id": "C23NB62W20TF",
      "name": "西雅图",
      "country": "US",
      "timezone": "America/Los_Angeles",
      "timezone_offset": "-07:00"
  },
  "now": {
      "text": "多云", //天气现象文字
      "code": "4", //天气现象代码
      "temperature": "14", //温度，单位为c摄氏度或f华氏度
      "feels_like": "14", //体感温度，单位为c摄氏度或f华氏度
      "pressure": "1018", //气压，单位为mb百帕或in英寸
      "humidity": "76", //相对湿度，0~100，单位为百分比
      "visibility": "16.09", //能见度，单位为km公里或mi英里
      "wind_direction": "西北", //风向文字
      "wind_direction_degree": "340", //风向角度，范围0~360，0为正北，90为正东，180为正南，270为正西
      "wind_speed": "8.05", //风速，单位为km/h公里每小时或mph英里每小时
      "wind_scale": "2", //风力等级，请参考：http://baike.baidu.com/view/465076.htm
      "clouds": "90", //云量，范围0~100，天空被云覆盖的百分比 #目前不支持中国城市#
      "dew_point": "-12" //露点温度，请参考：http://baike.baidu.com/view/118348.htm #目前不支持中国城市#
  },
  "last_update": "2015-09-25T22:45:00-07:00" //数据更新时间（该城市的本地时间）
  }]
}
```
###完成
这样我们登录之后点击自己的头像就能在下方显示当前位置的天气了

![这里写图片描述](http://img.blog.csdn.net/20170203135620795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ2hhdGlvbl85OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

完整代码 https://github.com/chation/wechat-weather
