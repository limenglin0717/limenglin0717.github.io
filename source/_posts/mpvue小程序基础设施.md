---
title: mpvue小程序基础设施
date: 2018-09-27 21:36:15
tags: mpvue
categories: 小程序
---

真正着手从美团开源的mpvue开始：

图片上传CDN，打上MD5戳
-------------

开发环境加入两个命令dev 和 test； 且加入一个新的环境变量process.env.CURENV

* dev：所有图片通过webpack打成bash64

  缺点：当图片过多时，文件过打时，无法进行收集预览

* test：所有图片路径通过webpack替换为测试环境服务器地址

  缺点：需要写相应配置，rsync图片到公司的测试机上

* build：所有图片路径通过webpack替换为正式环境CDN地址，并且加上md5

  缺点：需要每次上线执行推图片的脚本【增量覆盖】

---

对webpack的大概修改：

```bash
"scripts": {
    "dev": "node build/dev-server.js local",
    "test": "node build/dev-server.js test",
    "build": "node build/build.js",
},
```

webpack.base.conf.js

```bash
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader: 'url-loader',
    options: {
        limit: process.env.CURENV !== 'local' ? 1 : 999999999,
        name: utils.assetsPathForYOU('img/[name].[hash:7].[ext]')
    }
}
```

utils.js

```bash
// 针对图片音视频
exports.assetsPathForYOU = function (_path) {
  if (process.env.CURENV === 'local') {
    return path.posix.join(config.dev.assetsSubDirectory, _path)
  }
  if (process.env.CURENV === 'test') {
    return path.posix.join(config.dev.assetsSubDirectory, _path.replace('.[hash:7]', ''))
  }
  return path.posix.join(config.build.assetsSubDirectory, _path)
}
```

> 其中的config根据个人配置写就行

wx.request封装
-------------

* 用promise进行封装

* 对所有接口错误请求进行日志打点上传

```bash
export function callApi (urlPath, method = 'GET', data = {}, header = { 'content-type': 'application/json' }) {
  return new Promise((resolve, reject) => {
    // 对于接口认证处理
    const AUTH = wx.getStorageSync('token')
    if (AUTH) {
      header = {
        'Authorization': AUTH,
        ...header
      }
    }
    wx.request({
      url: urlPath,
      method: method.toUpperCase(),
      data,
      header,
      success (res) {
        // statusCode 为 http状态码
        if (res.statusCode === 200) {
          resolve(res.data)
        } else if (token过期) {
          // token过期，或者不存在, 会清除storage
          wx.clearStorageSync()
          wx.redirectTo({ url: '*****' })
        } else {
          // 业务异常qos打点日志上传
          // 可以做接口错误的统一处理
          reject(res.data)
        }
      },
      fail (e) {
        // 请求超时等
        reject(e)
      }
    })
  })
}
```

app.vue 的基础全局配置
---------------

```bash
created () {
  const updateManager = wx.getUpdateManager()
  // 强制更新，建议这样做
  updateManager.onUpdateReady(function () {
    updateManager.applyUpdate()
  })

  // 无网络提示（对弱网不是特别准确）
  wx.onNetworkStatusChange(function (res) {
    const { isConnected, networkType } = res
    if (!isConnected || networkType === '2g') {
      // 长时间显示提示且禁止操作
      wx.showToast({
        title: '网络不好',
        icon: 'none',
        duration: 3600000,
        mask: true
      })
    } else {
      wx.hideToast()
    }
  })
},
```

> 注意:
>
> 对ipad记得关注点，特别是锁屏的页面，可能出现显示不全；
>
> 用wx.getSystemInfo查看设备像素，进行相应的scale处理