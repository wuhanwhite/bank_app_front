<template>
	<view class="container">
		<view class="login-header">
			<image class="logo" src="/static/images/logo.png" mode="aspectFit"></image>
			<text class="title">多银行资产管理系统</text>
			<text class="subtitle">安全便捷地管理您的多家银行资产</text>
		</view>
		
		<view class="login-form">
			<view class="form-item">
				<text class="form-label">手机号码</text>
				<input class="form-input" type="number" v-model="form.phone" placeholder="请输入手机号码" maxlength="11" />
			</view>
			
			<view class="form-item">
				<text class="form-label">密码</text>
				<view class="password-input">
					<input class="form-input" :type="showPassword ? 'text' : 'password'" v-model="form.password" placeholder="请输入密码" />
					<text class="password-toggle" @click="togglePasswordVisibility">{{ showPassword ? '隐藏' : '显示' }}</text>
				</view>
			</view>
			
			<view class="form-options">
				<view class="remember-password">
					<checkbox :checked="rememberPassword" @click="rememberPassword = !rememberPassword" color="#3c9cff" />
					<text>记住密码</text>
				</view>
				<text class="forgot-password" @click="navigateTo('/pages/login/forgot-password')">忘记密码?</text>
			</view>
			
			<button class="login-button" @click="handleLogin">登录</button>
			
			<view class="register-link">
				<text>还没有账号？</text>
				<text class="link" @click="navigateTo('/pages/register/register')">立即注册</text>
			</view>
		</view>
		
		<view class="login-footer">
			<text class="footer-text">登录即表示您同意</text>
			<text class="link" @click="navigateTo('/pages/agreement/user')">《用户协议》</text>
			<text class="footer-text">和</text>
			<text class="link" @click="navigateTo('/pages/agreement/privacy')">《隐私政策》</text>
		</view>
	</view>
</template>

<script setup>
import { ref } from 'vue';

// 表单数据
const form = ref({
  phone: '',
  password: ''
});

// 是否显示密码
const showPassword = ref(false);

// 是否记住密码
const rememberPassword = ref(false);

// 切换密码显示状态
const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value;
};

// 处理登录
const handleLogin = () => {
  // 表单验证
  if (!form.value.phone) {
    uni.showToast({
      title: '请输入手机号码',
      icon: 'none'
    });
    return;
  }
  
  if (!form.value.password) {
    uni.showToast({
      title: '请输入密码',
      icon: 'none'
    });
    return;
  }
  
  // 模拟登录请求
  uni.showLoading({
    title: '登录中...'
  });
  
  setTimeout(() => {
    uni.hideLoading();
    
    // 模拟登录成功
    uni.setStorageSync('token', 'mock_token');
    uni.setStorageSync('userInfo', {
      id: 1,
      nickname: '张三',
      phone: form.value.phone
    });
    
    // 跳转到首页
    uni.switchTab({
      url: '/pages/index/index'
    });
  }, 1500);
};

// 页面导航
const navigateTo = (url) => {
  uni.navigateTo({
    url
  });
};
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  padding: 60rpx 40rpx;
  background-color: #f8f8f8;
}

.login-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 80rpx;
}

.logo {
  width: 160rpx;
  height: 160rpx;
  margin-bottom: 30rpx;
}

.title {
  font-size: 40rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 20rpx;
}

.subtitle {
  font-size: 28rpx;
  color: #999;
}

.login-form {
  background-color: #fff;
  border-radius: 20rpx;
  padding: 40rpx;
  margin-bottom: 40rpx;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.05);
}

.form-item {
  margin-bottom: 40rpx;
}

.form-label {
  font-size: 28rpx;
  color: #666;
  margin-bottom: 20rpx;
  display: block;
}

.form-input {
  height: 90rpx;
  border-bottom: 1px solid #eee;
  font-size: 32rpx;
  color: #333;
  width: 100%;
}

.password-input {
  position: relative;
}

.password-toggle {
  position: absolute;
  right: 0;
  top: 30rpx;
  font-size: 28rpx;
  color: #3c9cff;
}

.form-options {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 60rpx;
}

.remember-password {
  display: flex;
  align-items: center;
}

.remember-password text {
  font-size: 28rpx;
  color: #666;
  margin-left: 10rpx;
}

.forgot-password {
  font-size: 28rpx;
  color: #3c9cff;
}

.login-button {
  width: 100%;
  height: 90rpx;
  background: #3c9cff;
  color: #fff;
  border-radius: 45rpx;
  font-size: 32rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 40rpx;
}

.register-link {
  display: flex;
  justify-content: center;
  font-size: 28rpx;
  color: #666;
}

.link {
  color: #3c9cff;
  margin-left: 10rpx;
}

.login-footer {
  display: flex;
  justify-content: center;
  font-size: 24rpx;
  color: #999;
  margin-top: auto;
  padding: 20rpx 0;
  flex-wrap: wrap;
}

.footer-text {
  margin-right: 5rpx;
}
</style>
