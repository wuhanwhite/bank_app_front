<template>
	<view class="container">
		<view class="user-header">
			<view class="user-info">
				<view class="avatar-container" @click="chooseAvatar">
					<image class="avatar" :src="userInfo.avatar || '/static/images/default-avatar.png'" mode="aspectFill"></image>
					<view class="avatar-edit">
						<text class="edit-icon">✏️</text>
					</view>
				</view>
				<view class="user-details">
					<text class="username">{{ userInfo.nickname || '未设置昵称' }}</text>
					<text class="user-id">账号: {{ maskPhone(userInfo.phone) }}</text>
				</view>
			</view>
		</view>
		
		<view class="menu-section">
			<view class="menu-item" @click="navigateTo('/pages/user/edit')">
				<view class="menu-icon">👤</view>
				<view class="menu-content">
					<text class="menu-title">个人资料</text>
				</view>
				<view class="menu-arrow">></view>
			</view>
			
			<view class="menu-item" @click="navigateTo('/pages/user/security')">
				<view class="menu-icon">🔒</view>
				<view class="menu-content">
					<text class="menu-title">安全设置</text>
				</view>
				<view class="menu-arrow">></view>
			</view>
			
			<view class="menu-item" @click="navigateTo('/pages/user/notification')">
				<view class="menu-icon">🔔</view>
				<view class="menu-content">
					<text class="menu-title">通知设置</text>
				</view>
				<view class="menu-arrow">></view>
			</view>
		</view>
		
		<view class="menu-section">
			<view class="menu-item" @click="navigateTo('/pages/user/feedback')">
				<view class="menu-icon">📝</view>
				<view class="menu-content">
					<text class="menu-title">意见反馈</text>
				</view>
				<view class="menu-arrow">></view>
			</view>
			
			<view class="menu-item" @click="navigateTo('/pages/user/about')">
				<view class="menu-icon">ℹ️</view>
				<view class="menu-content">
					<text class="menu-title">关于我们</text>
				</view>
				<view class="menu-arrow">></view>
			</view>
		</view>
		
		<view class="logout-button" @click="showLogoutConfirm">
			<text>退出登录</text>
		</view>
	</view>
</template>

<script setup>
import { ref, onMounted } from 'vue';

// 用户信息
const userInfo = ref({
  id: 1,
  nickname: '张三',
  phone: '13800138000',
  avatar: ''
});

// 页面加载时获取用户信息
onMounted(() => {
  // 从本地存储获取用户信息
  const storedUserInfo = uni.getStorageSync('userInfo');
  if (storedUserInfo) {
    userInfo.value = storedUserInfo;
  }
  
  // 这里可以调用API获取最新的用户信息
  // fetchUserInfo();
});

// 选择头像
const chooseAvatar = () => {
  uni.chooseImage({
    count: 1,
    sizeType: ['compressed'],
    sourceType: ['album', 'camera'],
    success: (res) => {
      const tempFilePath = res.tempFilePaths[0];
      
      // 更新本地头像
      userInfo.value.avatar = tempFilePath;
      
      // 这里可以调用API上传头像
      // uploadAvatar(tempFilePath);
      
      // 更新本地存储
      uni.setStorageSync('userInfo', userInfo.value);
    }
  });
};

// 显示退出登录确认框
const showLogoutConfirm = () => {
  uni.showModal({
    title: '提示',
    content: '确定要退出登录吗？',
    success: (res) => {
      if (res.confirm) {
        handleLogout();
      }
    }
  });
};

// 处理退出登录
const handleLogout = () => {
  // 清除本地存储
  uni.removeStorageSync('token');
  uni.removeStorageSync('userInfo');
  
  // 跳转到登录页
  uni.reLaunch({
    url: '/pages/login/login'
  });
};

// 页面导航
const navigateTo = (url) => {
  uni.navigateTo({
    url
  });
};

// 手机号脱敏
const maskPhone = (phone) => {
  if (!phone) return '';
  return phone.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2');
};
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #f8f8f8;
  padding-bottom: 40rpx;
}

.user-header {
  background: linear-gradient(to right, #3c9cff, #5dabff);
  padding: 60rpx 40rpx;
  color: #fff;
}

.user-info {
  display: flex;
  align-items: center;
}

.avatar-container {
  position: relative;
  margin-right: 30rpx;
}

.avatar {
  width: 150rpx;
  height: 150rpx;
  border-radius: 75rpx;
  border: 4rpx solid rgba(255, 255, 255, 0.6);
}

.avatar-edit {
  position: absolute;
  right: 0;
  bottom: 0;
  width: 50rpx;
  height: 50rpx;
  background-color: #fff;
  border-radius: 25rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.1);
}

.edit-icon {
  font-size: 24rpx;
}

.user-details {
  flex: 1;
}

.username {
  font-size: 40rpx;
  font-weight: bold;
  margin-bottom: 10rpx;
  display: block;
}

.user-id {
  font-size: 28rpx;
  opacity: 0.8;
}

.menu-section {
  background-color: #fff;
  border-radius: 20rpx;
  margin: 30rpx;
  padding: 10rpx 0;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.05);
}

.menu-item {
  display: flex;
  align-items: center;
  padding: 30rpx 40rpx;
  border-bottom: 1px solid #f5f5f5;
}

.menu-item:last-child {
  border-bottom: none;
}

.menu-icon {
  margin-right: 30rpx;
  font-size: 40rpx;
}

.menu-content {
  flex: 1;
}

.menu-title {
  font-size: 32rpx;
  color: #333;
}

.menu-subtitle {
  font-size: 24rpx;
  color: #999;
  margin-top: 10rpx;
}

.menu-arrow {
  color: #ccc;
  font-size: 32rpx;
}

.logout-button {
  margin: 60rpx 30rpx;
  height: 90rpx;
  background-color: #fff;
  border-radius: 45rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32rpx;
  color: #ff3b30;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.05);
}
</style>
