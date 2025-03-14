<template>
	<view class="content">
		<view class="header">
			<view class="welcome">
				<text class="greeting">您好，{{ userInfo.nickname || '用户' }}</text>
				<text class="date">{{ currentDate }}</text>
			</view>
			<view class="balance-card">
				<view class="balance-title">总资产 (元)</view>
				<view class="balance-amount">{{ totalAssets.toFixed(2) }}</view>
				<view class="balance-actions">
					<view class="action-item" @click="navigateTo('/pages/account/add')">
						<text class="action-icon">+</text>
						<text class="action-text">添加账户</text>
					</view>
					<view class="action-item" @click="navigateTo('/pages/asset/statistics')">
						<text class="action-icon">📊</text>
						<text class="action-text">资产统计</text>
					</view>
				</view>
			</view>
		</view>
		
		<view class="section">
			<view class="section-header">
				<text class="section-title">我的账户</text>
				<text class="section-more" @click="navigateTo('/pages/account/list')">查看全部</text>
			</view>
			<view class="account-list">
				<view v-if="accounts.length === 0" class="empty-tip">
					<text>暂无账户，点击"添加账户"开始管理您的资产</text>
				</view>
				<view v-else class="account-item" v-for="(account, index) in accounts.slice(0, 3)" :key="index" @click="navigateTo(`/pages/account/detail?id=${account.id}`)">
					<view class="account-info">
						<text class="account-name">{{ account.bankName }}</text>
						<text class="account-number">{{ maskAccountNumber(account.accountNumber) }}</text>
					</view>
					<view class="account-balance">{{ account.balance.toFixed(2) }}</view>
				</view>
			</view>
		</view>
		
		<view class="section">
			<view class="section-header">
				<text class="section-title">快捷功能</text>
			</view>
			<view class="quick-actions">
				<view class="quick-action-item" @click="navigateTo('/pages/account/transfer')">
					<text class="quick-action-icon">💸</text>
					<text class="quick-action-text">转账</text>
				</view>
				<view class="quick-action-item" @click="navigateTo('/pages/account/payment')">
					<text class="quick-action-icon">💳</text>
					<text class="quick-action-text">支付</text>
				</view>
				<view class="quick-action-item" @click="navigateTo('/pages/user/feedback')">
					<text class="quick-action-icon">📝</text>
					<text class="quick-action-text">反馈</text>
				</view>
				<view class="quick-action-item" @click="navigateTo('/pages/user/profile')">
					<text class="quick-action-icon">👤</text>
					<text class="quick-action-text">我的</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

// 用户信息
const userInfo = ref({
  nickname: '张三',
  avatar: ''
});

// 账户列表
const accounts = ref([
  {
    id: 1,
    bankName: '中国银行',
    accountNumber: '6222020200123456789',
    balance: 10000.00
  },
  {
    id: 2,
    bankName: '工商银行',
    accountNumber: '6222020200987654321',
    balance: 5000.00
  },
  {
    id: 3,
    bankName: '建设银行',
    accountNumber: '6222020200123498765',
    balance: 8000.00
  }
]);

// 计算总资产
const totalAssets = computed(() => {
  return accounts.value.reduce((sum, account) => sum + account.balance, 0);
});

// 获取当前日期
const currentDate = computed(() => {
  const date = new Date();
  return `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日`;
});

// 页面加载时获取数据
onMounted(() => {
  // 这里可以调用API获取真实数据
  // fetchUserInfo();
  // fetchAccounts();
});

// 方法
// 导航到指定页面
const navigateTo = (url) => {
  uni.navigateTo({
    url
  });
};

// 隐藏账号中间部分
const maskAccountNumber = (accountNumber) => {
  if (!accountNumber) return '';
  const length = accountNumber.length;
  if (length <= 8) return accountNumber;
  return accountNumber.substring(0, 4) + '****' + accountNumber.substring(length - 4);
};
</script>

<style>
.content {
  padding: 30rpx;
}

.header {
  margin-bottom: 40rpx;
}

.welcome {
  margin-bottom: 20rpx;
}

.greeting {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
  display: block;
}

.date {
  font-size: 24rpx;
  color: #999;
  display: block;
  margin-top: 10rpx;
}

.balance-card {
  background: linear-gradient(to right, #3c9cff, #5dabff);
  border-radius: 20rpx;
  padding: 40rpx 30rpx;
  color: #fff;
  box-shadow: 0 10rpx 20rpx rgba(60, 156, 255, 0.2);
}

.balance-title {
  font-size: 28rpx;
  opacity: 0.9;
}

.balance-amount {
  font-size: 60rpx;
  font-weight: bold;
  margin: 20rpx 0 30rpx;
}

.balance-actions {
  display: flex;
  justify-content: space-around;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  padding-top: 30rpx;
}

.action-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.action-icon {
  font-size: 40rpx;
  margin-bottom: 10rpx;
}

.action-text {
  font-size: 24rpx;
}

.section {
  margin-bottom: 40rpx;
  background-color: #fff;
  border-radius: 20rpx;
  padding: 30rpx;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.05);
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.section-more {
  font-size: 24rpx;
  color: #3c9cff;
}

.account-list {
  margin-top: 20rpx;
}

.account-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1px solid #f5f5f5;
}

.account-item:last-child {
  border-bottom: none;
}

.account-info {
  display: flex;
  flex-direction: column;
}

.account-name {
  font-size: 28rpx;
  color: #333;
  margin-bottom: 10rpx;
}

.account-number {
  font-size: 24rpx;
  color: #999;
}

.account-balance {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
}

.empty-tip {
  text-align: center;
  color: #999;
  font-size: 28rpx;
  padding: 40rpx 0;
}

.quick-actions {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}

.quick-action-item {
  width: 22%;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20rpx;
}

.quick-action-icon {
  font-size: 50rpx;
  margin-bottom: 10rpx;
}

.quick-action-text {
  font-size: 24rpx;
  color: #666;
}
</style>
