<template>
	<view class="container">
		<view class="register-header">
			<view class="back-button" @click="navigateBack">
				<text class="back-icon">←</text>
			</view>
			<text class="title">注册账号</text>
		</view>
		
		<view class="register-form">
			<view class="form-item">
				<text class="form-label">手机号码</text>
				<input class="form-input" type="number" v-model="form.phone" placeholder="请输入手机号码" maxlength="11" />
			</view>
			
			<view class="form-item">
				<text class="form-label">验证码</text>
				<view class="verification-input">
					<input class="form-input" type="number" v-model="form.verificationCode" placeholder="请输入验证码" maxlength="6" />
					<button class="verification-button" :disabled="countdown > 0" @click="getVerificationCode">
						{{ countdown > 0 ? `${countdown}秒后重试` : '获取验证码' }}
					</button>
				</view>
			</view>
			
			<view class="form-item">
				<text class="form-label">设置密码</text>
				<view class="password-input">
					<input class="form-input" :type="showPassword ? 'text' : 'password'" v-model="form.password" placeholder="请设置8-20位密码，包含字母和数字" />
					<text class="password-toggle" @click="togglePasswordVisibility">{{ showPassword ? '隐藏' : '显示' }}</text>
				</view>
			</view>
			
			<view class="form-item">
				<text class="form-label">确认密码</text>
				<input class="form-input" :type="showPassword ? 'text' : 'password'" v-model="form.confirmPassword" placeholder="请再次输入密码" />
			</view>
			
			<view class="form-options">
				<view class="agreement">
					<checkbox :checked="agreeTerms" @click="agreeTerms = !agreeTerms" color="#3c9cff" />
					<text>我已阅读并同意</text>
					<text class="link" @click="navigateTo('/pages/agreement/user')">《用户协议》</text>
					<text>和</text>
					<text class="link" @click="navigateTo('/pages/agreement/privacy')">《隐私政策》</text>
				</view>
			</view>
			
			<button class="register-button" :disabled="!agreeTerms" :class="{ 'button-disabled': !agreeTerms }" @click="handleRegister">注册</button>
			
			<view class="login-link">
				<text>已有账号？</text>
				<text class="link" @click="navigateTo('/pages/login/login')">立即登录</text>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref } from 'vue';

// 表单数据
const form = ref({
  phone: '',
  verificationCode: '',
  password: '',
  confirmPassword: ''
});

// 是否显示密码
const showPassword = ref(false);

// 是否同意协议
const agreeTerms = ref(false);

// 验证码倒计时
const countdown = ref(0);

// 切换密码显示状态
const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value;
};

// 获取验证码
const getVerificationCode = () => {
  // 验证手机号
  if (!form.value.phone) {
    uni.showToast({
      title: '请输入手机号码',
      icon: 'none'
    });
    return;
  }
  
  if (!/^1\d{10}$/.test(form.value.phone)) {
    uni.showToast({
      title: '请输入正确的手机号码',
      icon: 'none'
    });
    return;
  }
  
  // 开始倒计时
  countdown.value = 60;
  const timer = setInterval(() => {
    countdown.value--;
    if (countdown.value <= 0) {
      clearInterval(timer);
    }
  }, 1000);
  
  // 模拟发送验证码
  uni.showToast({
    title: '验证码已发送',
    icon: 'success'
  });
};

// 处理注册
const handleRegister = () => {
  // 表单验证
  if (!form.value.phone) {
    uni.showToast({
      title: '请输入手机号码',
      icon: 'none'
    });
    return;
  }
  
  if (!/^1\d{10}$/.test(form.value.phone)) {
    uni.showToast({
      title: '请输入正确的手机号码',
      icon: 'none'
    });
    return;
  }
  
  if (!form.value.verificationCode) {
    uni.showToast({
      title: '请输入验证码',
      icon: 'none'
    });
    return;
  }
  
  if (!form.value.password) {
    uni.showToast({
      title: '请设置密码',
      icon: 'none'
    });
    return;
  }
  
  if (!/^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,20}$/.test(form.value.password)) {
    uni.showToast({
      title: '密码必须包含字母和数字，长度8-20位',
      icon: 'none'
    });
    return;
  }
  
  if (form.value.password !== form.value.confirmPassword) {
    uni.showToast({
      title: '两次输入的密码不一致',
      icon: 'none'
    });
    return;
  }
  
  if (!agreeTerms.value) {
    uni.showToast({
      title: '请阅读并同意用户协议和隐私政策',
      icon: 'none'
    });
    return;
  }
  
  // 模拟注册请求
  uni.showLoading({
    title: '注册中...'
  });
  
  setTimeout(() => {
    uni.hideLoading();
    
    // 模拟注册成功
    uni.showToast({
      title: '注册成功',
      icon: 'success'
    });
    
    // 跳转到登录页
    setTimeout(() => {
      uni.navigateTo({
        url: '/pages/login/login'
      });
    }, 1500);
  }, 2000);
};

// 页面导航
const navigateTo = (url) => {
  uni.navigateTo({
    url
  });
};

// 返回上一页
const navigateBack = () => {
  uni.navigateBack();
};
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  padding: 0 40rpx 60rpx;
  background-color: #f8f8f8;
}

.register-header {
  display: flex;
  align-items: center;
  height: 100rpx;
  margin-bottom: 60rpx;
  position: relative;
}

.back-button {
  position: absolute;
  left: 0;
  top: 0;
  height: 100rpx;
  width: 100rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.back-icon {
  font-size: 40rpx;
  color: #333;
}

.title {
  width: 100%;
  text-align: center;
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.register-form {
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

.verification-input {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.verification-input .form-input {
  flex: 1;
  margin-right: 20rpx;
}

.verification-button {
  width: 220rpx;
  height: 80rpx;
  background: #3c9cff;
  color: #fff;
  border-radius: 40rpx;
  font-size: 28rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}

.verification-button[disabled] {
  background: #cccccc;
  color: #ffffff;
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
  margin-bottom: 60rpx;
}

.agreement {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
}

.agreement text {
  font-size: 28rpx;
  color: #666;
  margin-left: 10rpx;
}

.link {
  color: #3c9cff;
}

.register-button {
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

.button-disabled {
  background: #cccccc;
}

.login-link {
  display: flex;
  justify-content: center;
  font-size: 28rpx;
  color: #666;
}
</style>
