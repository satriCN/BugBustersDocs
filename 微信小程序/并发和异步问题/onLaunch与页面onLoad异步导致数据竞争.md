
# 解决微信小程序onLaunch与页面onLoad（常见的为index.js）异步冲突问题。

## 问题描述

在app.js的onLaunch()中进行wx.login()与wx.request()并赋值给this.globalData，由于微信小程序的所有网络请求均为默认异步进行，因此小程序全局初始状态下，在任何页面（常见为首页）的onLoad()中均无法有效获取全局数据。

## 解决办法

使用Promise封装onLaunch()网络请求：

```
//app.js核心代码 
onLaunch() {
  launch();
}, 
globalData: {
  openid: '',
  userid: '',
  account: ''
}, 
launch() {
  var that = this;
  return new Promise((resolve, reject) => {
    // 登录 
    wx.login({
      success: (res) => {
        wx.showLoading({
          title: '加载中...',
          mask: true
        });
        console.log(res.code);
        wx.request({
          url: 'https://official.satri.cn/[去敏处理]',
          data: {
            js_code: res.code,
          },
          success: (res) => {
            //更新数据 
            console.log(res) console.log(that.globalData) that.globalData.account = res.data.account;
            that.globalData.openid = res.data.openid;
            that.globalData.userid = res.data.userid;
            wx.hideLoading();
            resolve(res.data);
          }
        })
      },
    });
  })
}
```

```
//index.js核心代码
onLoad(options) {
  wx.stopPullDownRefresh() //刷新完成后停止下拉刷新动效
  var that = this;
  app.launch().then(res => {
    wx.showShareMenu({
      withShareTicket: true,
      menus: ['shareAppMessage', 'shareTimeline']
    });
    wx.request({
      url: 'https://official.satri.cn/api/[去敏处理]',
      data: {
        userid: app.globalData.userid,
      },
      success: (res) => {
        console.log(res);
        //赋值
        that.setData({
          commonnews: res.data.commonnews,
          diynews: res.data.diynews,
          carouselImgUrls: res.data.carouselImgUrls,
          anli: res.data.anli
        })
        wx.hideLoading();
      }
    })
  })
}
```
