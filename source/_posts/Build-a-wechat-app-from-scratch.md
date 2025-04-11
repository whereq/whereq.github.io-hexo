---
title: Build a wechat app from scratch
date: 2025-04-11 17:33:52
categories:
- WeChat
tags:
- WeChat
---

## 1. 项目结构

```
miniprogram/
├── pages/
│   └── index/
│       ├── index.js
│       ├── index.json
│       ├── index.wxml
│       └── index.wxss
├── app.js
├── app.json
├── app.wxss
└── utils/
    └── api.js
```

## 2. 代码实现

### 2.1 app.json 配置

```json
{
  "pages": [
    "pages/index/index"
  ],
  "window": {
    "navigationBarTitleText": "图片风格转换",
    "navigationBarBackgroundColor": "#ffffff",
    "navigationBarTextStyle": "black"
  },
  "style": "v2",
  "sitemapLocation": "sitemap.json"
}
```

### 2.2 页面布局 (index.wxml)

```xml
<view class="container">
  <!-- 上部：图片输入区域 -->
  <view class="top-section">
    <view class="image-preview" wx:if="{{originalImage}}">
      <image src="{{originalImage}}" mode="widthFix"></image>
    </view>
    <view class="button-group">
      <button bindtap="chooseImage" class="btn">选择图片</button>
      <button bindtap="transformImage" class="btn transform-btn" disabled="{{!canTransform}}">转换</button>
    </view>
  </view>
  
  <!-- 中部：风格选择区域 -->
  <view class="middle-section">
    <view class="title">选择转换风格</view>
    <view class="style-buttons">
      <block wx:for="{{styles}}" wx:key="id">
        <button 
          class="style-btn {{selectedStyle === item.id ? 'active' : ''}}" 
          bindtap="selectStyle" 
          data-id="{{item.id}}"
        >
          {{item.name}}
        </button>
      </block>
    </view>
  </view>
  
  <!-- 下部：结果展示区域 -->
  <view class="bottom-section">
    <view wx:if="{{isLoading}}" class="loading">
      <image src="/images/loading.gif" mode="aspectFit"></image>
      <text>正在转换中，请稍候...</text>
    </view>
    <view wx:if="{{transformedImage && !isLoading}}" class="result-image">
      <image src="{{transformedImage}}" mode="widthFix"></image>
    </view>
  </view>
</view>
```

### 2.3 页面样式 (index.wxss)

```css
.container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  padding: 20rpx;
  box-sizing: border-box;
}

.top-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-bottom: 1px solid #eee;
  padding-bottom: 20rpx;
}

.image-preview {
  width: 90%;
  margin-bottom: 20rpx;
}

.image-preview image {
  width: 100%;
  border-radius: 10rpx;
}

.button-group {
  display: flex;
  width: 90%;
  justify-content: space-between;
}

.btn {
  width: 48%;
  height: 80rpx;
  line-height: 80rpx;
  font-size: 32rpx;
}

.transform-btn {
  background-color: #07c160;
  color: white;
}

.transform-btn[disabled] {
  background-color: #cccccc;
  color: #666666;
}

.middle-section {
  flex: 1;
  padding: 20rpx 0;
  border-bottom: 1px solid #eee;
}

.title {
  text-align: center;
  font-size: 36rpx;
  margin-bottom: 30rpx;
  color: #333;
}

.style-buttons {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.style-btn {
  width: 45%;
  margin-bottom: 20rpx;
  font-size: 28rpx;
  background-color: #f5f5f5;
  color: #333;
}

.style-btn.active {
  background-color: #07c160;
  color: white;
}

.bottom-section {
  flex: 2;
  display: flex;
  justify-content: center;
  align-items: center;
  padding-top: 20rpx;
}

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.loading image {
  width: 100rpx;
  height: 100rpx;
  margin-bottom: 20rpx;
}

.loading text {
  font-size: 28rpx;
  color: #666;
}

.result-image {
  width: 90%;
}

.result-image image {
  width: 100%;
  border-radius: 10rpx;
}
```

### 2.4 页面逻辑 (index.js)

```javascript
const api = require('../../utils/api.js')

Page({
  data: {
    originalImage: null,
    transformedImage: null,
    selectedStyle: null,
    isLoading: false,
    canTransform: false,
    styles: [
      { id: 'anime', name: '动漫' },
      { id: 'vintage', name: '怀旧' },
      { id: 'cool', name: '酷帅' },
      { id: 'fresh', name: '清新' }
    ]
  },

  // 选择图片
  chooseImage() {
    wx.chooseImage({
      count: 1,
      sizeType: ['compressed'],
      sourceType: ['album', 'camera'],
      success: (res) => {
        this.setData({
          originalImage: res.tempFilePaths[0],
          transformedImage: null
        })
        this.checkTransformStatus()
      }
    })
  },

  // 选择风格
  selectStyle(e) {
    const styleId = e.currentTarget.dataset.id
    this.setData({
      selectedStyle: styleId
    })
    this.checkTransformStatus()
  },

  // 检查是否可以转换
  checkTransformStatus() {
    this.setData({
      canTransform: !!this.data.originalImage && !!this.data.selectedStyle
    })
  },

  // 转换图片
  transformImage() {
    if (!this.data.originalImage || !this.data.selectedStyle) {
      wx.showToast({
        title: '请选择图片和风格',
        icon: 'none'
      })
      return
    }

    this.setData({
      isLoading: true,
      transformedImage: null
    })

    // 上传图片并转换
    wx.uploadFile({
      url: 'https://www.catobigato.com/api/v1/imageTransform',
      filePath: this.data.originalImage,
      name: 'image',
      formData: {
        style: this.data.selectedStyle
      },
      success: (res) => {
        if (res.statusCode === 200) {
          const data = JSON.parse(res.data)
          if (data.success && data.resultUrl) {
            this.setData({
              transformedImage: data.resultUrl
            })
          } else {
            wx.showToast({
              title: data.message || '转换失败',
              icon: 'none'
            })
          }
        } else {
          wx.showToast({
            title: '服务器错误',
            icon: 'none'
          })
        }
      },
      fail: (err) => {
        console.error(err)
        wx.showToast({
          title: '网络错误',
          icon: 'none'
        })
      },
      complete: () => {
        this.setData({
          isLoading: false
        })
      }
    })
  }
})
```

### 2.5 API工具类 (utils/api.js)

```javascript
const BASE_URL = 'https://www.catobigato.com/api/v1'

const request = (url, method, data) => {
  return new Promise((resolve, reject) => {
    wx.request({
      url: BASE_URL + url,
      method: method,
      data: data,
      header: {
        'Content-Type': 'application/json'
      },
      success: (res) => {
        if (res.statusCode === 200) {
          resolve(res.data)
        } else {
          reject(res.data)
        }
      },
      fail: (err) => {
        reject(err)
      }
    })
  })
}

module.exports = {
  request
}
```

## 3. 功能说明

1. **图片选择**：
   - 用户点击"选择图片"按钮，可以从相册或相机获取图片
   - 选择的图片会显示在上部区域

2. **风格选择**：
   - 中部区域提供4种风格选项（动漫、怀旧、酷帅、清新）
   - 用户点击风格按钮后，按钮会高亮显示

3. **转换功能**：
   - 只有当用户选择了图片和风格后，"转换"按钮才会启用
   - 点击"转换"按钮后，小程序将图片和风格参数上传到后端API
   - 转换过程中显示加载动画

4. **结果展示**：
   - 转换完成后，结果图片显示在下部区域
   - 如果转换失败，会显示错误提示

## 4. 注意事项

1. **域名配置**：
   - 在微信小程序后台配置`https://www.catobigato.com`为合法域名
   - 确保服务器支持HTTPS

2. **图片大小限制**：
   - 微信小程序对上传文件大小有限制（通常为10MB）
   - 建议在上传前对图片进行压缩处理

3. **加载动画**：
   - 准备一个loading.gif动画文件放在`/images/`目录下
   - 也可以使用微信自带的loading组件

4. **错误处理**：
   - 添加了网络错误和服务器错误的处理逻辑
   - 显示友好的错误提示

5. **性能优化**：
   - 对于大图片，可以考虑先压缩再上传
   - 可以添加取消上传的功能
