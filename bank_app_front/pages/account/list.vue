<template>
	<view class="container">
		<view class="header">
			<text class="title">我的账户</text>
			<view class="add-button" @click="navigateTo('/pages/account/add')">
				<text class="add-icon">+</text>
				<text class="add-text">添加账户</text>
			</view>
		</view>
		
		<view class="total-assets">
			<view class="total-label">总资产 (元)</view>
			<view class="total-amount">{{ totalAssets.toFixed(2) }}</view>
		</view>
		
		<view class="account-list">
			<view v-if="loading" class="loading-container">
				<view class="loading-spinner"></view>
				<text class="loading-text">加载中...</text>
			</view>
			
			<view v-else-if="accounts.length === 0" class="empty-container">
				<image class="empty-image" src="/static/images/empty-account.png" mode="aspectFit"></image>
				<text class="empty-text">暂无账户，点击"添加账户"开始管理您的资产</text>
				<button class="add-account-button" @click="navigateTo('/pages/account/add')">添加账户</button>
			</view>
			
			<view v-else>
				<view v-for="(bank, bankIndex) in groupedAccounts" :key="bankIndex" class="bank-group">
					<view class="bank-header">
						<view class="bank-info">
							<image class="bank-logo" :src="bank.logo || '/static/images/bank-default.png'" mode="aspectFit"></image>
							<text class="bank-name">{{ bank.name }}</text>
						</view>
						<text class="bank-total">{{ bank.totalAmount.toFixed(2) }}</text>
					</view>
					
					<view class="account-items">
						<view v-for="(account, accountIndex) in bank.accounts" :key="accountIndex" class="account-item" @click="navigateTo(`/pages/account/detail?id=${account.id}`)">
							<view class="account-info">
								<text class="account-type">{{ account.accountType }}</text>
								<text class="account-number">{{ maskAccountNumber(account.accountNumber) }}</text>
							</view>
							<view class="account-balance">{{ account.balance.toFixed(2) }}</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

// 加载状态
const loading = ref(true);

// 账户列表
const accounts = ref([
  {
    id: 1,
    bankName: '中国银行',
    bankLogo: '/static/images/bank-boc.png',
    accountType: '储蓄卡',
    accountNumber: '6222020200123456789',
    balance: 10000.00
  },
  {
    id: 2,
    bankName: '中国银行',
    bankLogo: '/static/images/bank-boc.png',
    accountType: '信用卡',
    accountNumber: '6222020200123456790',
    balance: 5000.00
  },
  {
    id: 3,
    bankName: '工商银行',
    bankLogo: '/static/images/bank-icbc.png',
    accountType: '储蓄卡',
    accountNumber: '6222020200987654321',
    balance: 8000.00
  },
  {
    id: 4,
    bankName: '建设银行',
    bankLogo: '/static/images/bank-ccb.png',
    accountType: '储蓄卡',
    accountNumber: '6222020200123498765',
    balance: 12000.00
  }
]);

// 按银行分组账户
const groupedAccounts = computed(() => {
  const groups = {};
  
  accounts.value.forEach(account => {
    if (!groups[account.bankName]) {
      groups[account.bankName] = {
        name: account.bankName,
        logo: account.bankLogo,
        accounts: [],
        totalAmount: 0
      };
    }
    
    groups[account.bankName].accounts.push(account);
    groups[account.bankName].totalAmount += account.balance;
  });
  
  return Object.values(groups);
});

// 计算总资产
const totalAssets = computed(() => {
  return accounts.value.reduce((sum, account) => sum + account.balance, 0);
});

// 页面加载时获取数据
onMounted(() => {
  // 模拟加载数据
  setTimeout(() => {
    loading.value = false;
  }, 1000);
  
  // 这里可以调用API获取真实数据
  // fetchAccounts();
});

// 页面导航
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
.container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #f8f8f8;
  padding: 30rpx;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.add-button {
  display: flex;
  align-items: center;
  background-color: #3c9cff;
  padding: 10rpx 20rpx;
  border-radius: 30rpx;
  color: #fff;
}

.add-icon {
  font-size: 32rpx;
  margin-right: 10rpx;
}

.add-text {
  font-size: 28rpx;
}

.total-assets {
  background: linear-gradient(to right, #3c9cff, #5dabff);
  border-radius: 20rpx;
  padding: 40rpx 30rpx;
  color: #fff;
  margin-bottom: 30rpx;
  box-shadow: 0 10rpx 20rpx rgba(60, 156, 255, 0.2);
}

.total-label {
  font-size: 28rpx;
  opacity: 0.9;
}

.total-amount {
  font-size: 60rpx;
  font-weight: bold;
  margin-top: 20rpx;
}

.account-list {
  flex: 1;
}

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 100rpx 0;
}

.loading-spinner {
  width: 60rpx;
  height: 60rpx;
  border: 6rpx solid #f3f3f3;
  border-top: 6rpx solid #3c9cff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 20rpx;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-text {
  font-size: 28rpx;
  color: #999;
}

.empty-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 100rpx 0;
}

.empty-image {
  width: 200rpx;
  height: 200rpx;
  margin-bottom: 30rpx;
}

.empty-text {
  font-size: 28rpx;
  color: #999;
  text-align: center;
  margin-bottom: 40rpx;
}

.add-account-button {
  background-color: #3c9cff;
  color: #fff;
  font-size: 28rpx;
  padding: 20rpx 60rpx;
  border-radius: 40rpx;
}

.bank-group {
  background-color: #fff;
  border-radius: 20rpx;
  margin-bottom: 30rpx;
  overflow: hidden;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.05);
}

.bank-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30rpx;
  border-bottom: 1px solid #f5f5f5;
}

.bank-info {
  display: flex;
  align-items: center;
}

.bank-logo {
  width: 60rpx;
  height: 60rpx;
  margin-right: 20rpx;
}

.bank-name {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.bank-total {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
}

.account-items {
  padding: 0 30rpx;
}

.account-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30rpx 0;
  border-bottom: 1px solid #f5f5f5;
}

.account-item:last-child {
  border-bottom: none;
}

.account-info {
  display: flex;
  flex-direction: column;
}

.account-type {
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
</style>
