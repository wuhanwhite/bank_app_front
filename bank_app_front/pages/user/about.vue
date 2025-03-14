<template>
  <view class="about-container">
    <view class="header">
      <text class="title">关于我们</text>
    </view>
    
    <view class="app-info-section">
      <view class="app-logo">
        <image class="logo-image" src="/static/images/app-logo.png" mode="aspectFit"></image>
      </view>
      <text class="app-name">多银行资产管理</text>
      <text class="app-version">版本 1.0.0</text>
    </view>
    
    <view class="content-section">
      <view class="info-card">
        <view class="card-header">
          <text class="card-title">应用介绍</text>
        </view>
        <view class="card-content">
          <text class="content-text">
            多银行资产管理是一款专为个人用户设计的资产管理工具，支持多家银行账户的统一管理，帮助用户实时掌握资产状况，分析收益变化，提高资产管理效率。
          </text>
        </view>
      </view>
      
      <view class="info-card">
        <view class="card-header">
          <text class="card-title">主要功能</text>
        </view>
        <view class="card-content">
          <view class="feature-item">
            <text class="feature-icon iconfont icon-bank"></text>
            <view class="feature-info">
              <text class="feature-title">多银行账户管理</text>
              <text class="feature-desc">支持添加多家银行的多个账户，统一管理</text>
            </view>
          </view>
          
          <view class="feature-item">
            <text class="feature-icon iconfont icon-asset"></text>
            <view class="feature-info">
              <text class="feature-title">资产记录与分析</text>
              <text class="feature-desc">记录每日资产变化，分析收益情况</text>
            </view>
          </view>
          
          <view class="feature-item">
            <text class="feature-icon iconfont icon-upload"></text>
            <view class="feature-info">
              <text class="feature-title">银行截图识别</text>
              <text class="feature-desc">上传银行截图，自动识别资产信息</text>
            </view>
          </view>
          
          <view class="feature-item">
            <text class="feature-icon iconfont icon-statistics"></text>
            <view class="feature-info">
              <text class="feature-title">资产统计与报表</text>
              <text class="feature-desc">多维度统计资产分布，生成收益报表</text>
            </view>
          </view>
        </view>
      </view>
      
      <view class="info-card">
        <view class="card-header">
          <text class="card-title">联系我们</text>
        </view>
        <view class="card-content">
          <view class="contact-item">
            <text class="contact-label">官方网站：</text>
            <text class="contact-value">www.bankapp.com</text>
          </view>
          
          <view class="contact-item">
            <text class="contact-label">客服邮箱：</text>
            <text class="contact-value">support@bankapp.com</text>
          </view>
          
          <view class="contact-item">
            <text class="contact-label">客服电话：</text>
            <text class="contact-value">400-123-4567</text>
          </view>
        </view>
      </view>
    </view>
    
    <view class="action-section">
      <button class="action-btn" @click="handleCheckUpdate">检查更新</button>
      <button class="action-btn" @click="handleFeedback">意见反馈</button>
      <button class="action-btn" @click="handlePrivacyPolicy">隐私政策</button>
      <button class="action-btn" @click="handleUserAgreement">用户协议</button>
    </view>
    
    <view class="copyright-section">
      <text class="copyright-text">© 2023 BankApp All Rights Reserved</text>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue';

// 检查更新
const handleCheckUpdate = () => {
  uni.showLoading({
    title: '检查更新中...'
  });
  
  setTimeout(() => {
    uni.hideLoading();
    
    uni.showModal({
      title: '检查更新',
      content: '当前已是最新版本',
      showCancel: false
    });
  }, 1500);
};

// 意见反馈
const handleFeedback = () => {
  uni.navigateTo({
    url: '/pages/user/feedback'
  });
};

// 隐私政策
const handlePrivacyPolicy = () => {
  uni.navigateTo({
    url: '/pages/user/privacy-policy'
  });
};

// 用户协议
const handleUserAgreement = () => {
  uni.navigateTo({
    url: '/pages/user/user-agreement'
  });
};
</script>

<style lang="scss">
.about-container {
  min-height: 100vh;
  background-color: #f8f8f8;
  padding-bottom: 40rpx;
}

.header {
  background-color: #3c9cff;
  padding: 30rpx;
  color: #fff;
  
  .title {
    font-size: 36rpx;
    font-weight: bold;
  }
}

.app-info-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40rpx 0;
  
  .app-logo {
    width: 160rpx;
    height: 160rpx;
    margin-bottom: 20rpx;
    
    .logo-image {
      width: 100%;
      height: 100%;
      border-radius: 30rpx;
    }
  }
  
  .app-name {
    font-size: 36rpx;
    font-weight: bold;
    color: #333;
    margin-bottom: 10rpx;
  }
  
  .app-version {
    font-size: 28rpx;
    color: #666;
  }
}

.content-section {
  padding: 0 30rpx;
  
  .info-card {
    background-color: #fff;
    border-radius: 12rpx;
    margin-bottom: 30rpx;
    box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
    
    .card-header {
      padding: 30rpx;
      border-bottom: 1px solid #f0f0f0;
      
      .card-title {
        font-size: 32rpx;
        font-weight: bold;
        color: #333;
      }
    }
    
    .card-content {
      padding: 30rpx;
      
      .content-text {
        font-size: 28rpx;
        color: #666;
        line-height: 1.6;
      }
      
      .feature-item {
        display: flex;
        margin-bottom: 30rpx;
        
        &:last-child {
          margin-bottom: 0;
        }
        
        .feature-icon {
          font-size: 48rpx;
          color: #3c9cff;
          margin-right: 20rpx;
        }
        
        .feature-info {
          flex: 1;
          
          .feature-title {
            font-size: 30rpx;
            font-weight: bold;
            color: #333;
            margin-bottom: 8rpx;
            display: block;
          }
          
          .feature-desc {
            font-size: 26rpx;
            color: #666;
          }
        }
      }
      
      .contact-item {
        display: flex;
        margin-bottom: 20rpx;
        
        &:last-child {
          margin-bottom: 0;
        }
        
        .contact-label {
          font-size: 28rpx;
          color: #333;
          width: 180rpx;
        }
        
        .contact-value {
          font-size: 28rpx;
          color: #666;
          flex: 1;
        }
      }
    }
  }
}

.action-section {
  padding: 0 30rpx;
  
  .action-btn {
    width: 100%;
    height: 90rpx;
    line-height: 90rpx;
    background-color: #fff;
    color: #333;
    font-size: 30rpx;
    border-radius: 8rpx;
    margin-bottom: 20rpx;
    box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
    
    &:last-child {
      margin-bottom: 0;
    }
    
    &::after {
      border: none;
    }
  }
}

.copyright-section {
  padding: 40rpx 0;
  text-align: center;
  
  .copyright-text {
    font-size: 26rpx;
    color: #999;
  }
}

// 为了兼容图标，这里使用一个简单的占位符
// 实际使用时需要引入iconfont或其他图标库
.iconfont {
  font-family: "iconfont";
  font-size: 28rpx;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-bank:before {
  content: "\e6b0";
}

.icon-asset:before {
  content: "\e6b1";
}

.icon-upload:before {
  content: "\e6b8";
}

.icon-statistics:before {
  content: "\e6b2";
}
</style>
