<template>
	<view class="container">
		<view class="header">
			<view class="back-button" @click="navigateBack">
				<text class="back-icon">←</text>
			</view>
			<text class="title">账户详情</text>
			<view class="more-button" @click="showActionSheet">
				<text class="more-icon">⋮</text>
			</view>
		</view>
		
		<view class="account-card">
			<view class="bank-info">
				<image class="bank-logo" :src="account.bankLogo || '/static/images/bank-default.png'" mode="aspectFit"></image>
				<view class="bank-details">
					<text class="bank-name">{{ account.bankName }}</text>
					<text class="account-type">{{ account.accountType === 'debit' ? '储蓄卡' : '信用卡' }}</text>
				</view>
			</view>
			
			<view class="account-number">
				<text>{{ formatAccountNumber(account.accountNumber) }}</text>
				<view class="copy-button" @click="copyAccountNumber">
					<text>复制</text>
				</view>
			</view>
			
			<view class="balance-info">
				<view class="balance-item">
					<text class="balance-label">{{ account.accountType === 'debit' ? '账户余额' : '可用额度' }}</text>
					<text class="balance-value">{{ account.balance.toFixed(2) }}</text>
				</view>
				<view class="balance-item" v-if="account.accountType === 'credit'">
					<text class="balance-label">信用额度</text>
					<text class="balance-value">{{ account.creditLimit.toFixed(2) }}</text>
				</view>
			</view>
		</view>
		
		<view class="transaction-section">
			<view class="section-header">
				<text class="section-title">近期交易</text>
				<view class="view-all" @click="navigateTo('/pages/account/transactions?id=' + account.id)">
					<text>查看全部</text>
				</view>
			</view>
			
			<view v-if="transactions.length === 0" class="empty-transactions">
				<image class="empty-image" src="/static/images/empty-transaction.png" mode="aspectFit"></image>
				<text class="empty-text">暂无交易记录</text>
			</view>
			
			<view v-else class="transaction-list">
				<view v-for="(transaction, index) in transactions" :key="index" class="transaction-item">
					<view class="transaction-icon" :class="[transaction.type === 'income' ? 'income-icon' : 'expense-icon']">
						<text>{{ transaction.type === 'income' ? '+' : '-' }}</text>
					</view>
					<view class="transaction-info">
						<text class="transaction-title">{{ transaction.title }}</text>
						<text class="transaction-time">{{ transaction.time }}</text>
					</view>
					<text class="transaction-amount" :class="[transaction.type === 'income' ? 'income-amount' : 'expense-amount']">
						{{ (transaction.type === 'income' ? '+' : '-') + transaction.amount.toFixed(2) }}
					</text>
				</view>
			</view>
		</view>
		
		<view class="action-buttons">
			<view class="action-button transfer" @click="navigateTo('/pages/account/transfer?id=' + account.id)">
				<text class="action-icon">↑</text>
				<text class="action-text">转账</text>
			</view>
			<view class="action-button recharge" @click="navigateTo('/pages/account/recharge?id=' + account.id)">
				<text class="action-icon">+</text>
				<text class="action-text">充值</text>
			</view>
			<view class="action-button withdraw" @click="navigateTo('/pages/account/withdraw?id=' + account.id)">
				<text class="action-icon">↓</text>
				<text class="action-text">提现</text>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, onMounted } from 'vue';

// 获取路由参数
const accountId = ref(null);

// 账户信息
const account = ref({
  id: 1,
  bankName: '中国银行',
  bankLogo: '/static/images/bank-boc.png',
  accountType: 'debit',
  accountNumber: '6222020200123456789',
  balance: 10000.00,
  creditLimit: 0
});

// 交易记录
const transactions = ref([
  {
    id: 1,
    title: '工资收入',
    type: 'income',
    amount: 5000.00,
    time: '2023-05-15 09:30'
  },
  {
    id: 2,
    title: '超市购物',
    type: 'expense',
    amount: 328.50,
    time: '2023-05-14 16:45'
  },
  {
    id: 3,
    title: '餐饮消费',
    type: 'expense',
    amount: 108.00,
    time: '2023-05-13 12:30'
  }
]);

// 页面加载时获取数据
onMounted(() => {
  // 获取路由参数
  const params = uni.getSystemInfoSync().platform === 'devtools' ? 
    { id: 1 } : uni.getLaunchOptionsSync().query;
  
  accountId.value = params.id;
  
  // 这里可以调用API获取账户详情
  // fetchAccountDetail(accountId.value);
  
  // 这里可以调用API获取交易记录
  // fetchTransactions(accountId.value);
});

// 格式化账号
const formatAccountNumber = (accountNumber) => {
  if (!accountNumber) return '';
  return accountNumber.replace(/(\d{4})(?=\d)/g, '$1 ');
};

// 复制账号
const copyAccountNumber = () => {
  uni.setClipboardData({
    data: account.value.accountNumber,
    success: () => {
      uni.showToast({
        title: '复制成功',
        icon: 'success'
      });
    }
  });
};

// 显示操作菜单
const showActionSheet = () => {
  uni.showActionSheet({
    itemList: ['编辑账户', '删除账户'],
    success: (res) => {
      if (res.tapIndex === 0) {
        // 编辑账户
        navigateTo(`/pages/account/edit?id=${account.value.id}`);
      } else if (res.tapIndex === 1) {
        // 删除账户
        showDeleteConfirm();
      }
    }
  });
};

// 显示删除确认框
const showDeleteConfirm = () => {
  uni.showModal({
    title: '提示',
    content: '确定要删除该账户吗？',
    success: (res) => {
      if (res.confirm) {
        // 这里可以调用API删除账户
        // deleteAccount(account.value.id);
        
        // 模拟删除成功
        uni.showToast({
          title: '删除成功',
          icon: 'success'
        });
        
        // 返回账户列表页
        setTimeout(() => {
          uni.navigateBack();
        }, 1500);
      }
    }
  });
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
  background-color: #f8f8f8;
}

.header {
  display: flex;
  align-items: center;
  height: 100rpx;
  background-color: #fff;
  position: relative;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
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

.more-button {
  position: absolute;
  right: 0;
  top: 0;
  height: 100rpx;
  width: 100rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.more-icon {
  font-size: 40rpx;
  color: #333;
}

.account-card {
  margin: 30rpx;
  background: linear-gradient(to right, #3c9cff, #5dabff);
  border-radius: 20rpx;
  padding: 40rpx 30rpx;
  color: #fff;
  box-shadow: 0 10rpx 20rpx rgba(60, 156, 255, 0.2);
}

.bank-info {
  display: flex;
  align-items: center;
  margin-bottom: 30rpx;
}

.bank-logo {
  width: 80rpx;
  height: 80rpx;
  background-color: #fff;
  border-radius: 40rpx;
  padding: 10rpx;
  margin-right: 20rpx;
}

.bank-details {
  display: flex;
  flex-direction: column;
}

.bank-name {
  font-size: 36rpx;
  font-weight: bold;
  margin-bottom: 10rpx;
}

.account-type {
  font-size: 28rpx;
  opacity: 0.9;
}

.account-number {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 36rpx;
  letter-spacing: 2rpx;
  margin-bottom: 40rpx;
}

.copy-button {
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 30rpx;
  padding: 10rpx 30rpx;
  font-size: 24rpx;
}

.balance-info {
  display: flex;
  justify-content: space-between;
}

.balance-item {
  display: flex;
  flex-direction: column;
}

.balance-label {
  font-size: 28rpx;
  opacity: 0.9;
  margin-bottom: 10rpx;
}

.balance-value {
  font-size: 48rpx;
  font-weight: bold;
}

.transaction-section {
  margin: 0 30rpx 30rpx;
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

.view-all {
  font-size: 28rpx;
  color: #3c9cff;
}

.empty-transactions {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 60rpx 0;
}

.empty-image {
  width: 200rpx;
  height: 200rpx;
  margin-bottom: 30rpx;
}

.empty-text {
  font-size: 28rpx;
  color: #999;
}

.transaction-list {
  display: flex;
  flex-direction: column;
}

.transaction-item {
  display: flex;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1px solid #f5f5f5;
}

.transaction-item:last-child {
  border-bottom: none;
}

.transaction-icon {
  width: 80rpx;
  height: 80rpx;
  border-radius: 40rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 36rpx;
  font-weight: bold;
  margin-right: 20rpx;
}

.income-icon {
  background-color: rgba(52, 199, 89, 0.1);
  color: #34c759;
}

.expense-icon {
  background-color: rgba(255, 59, 48, 0.1);
  color: #ff3b30;
}

.transaction-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.transaction-title {
  font-size: 32rpx;
  color: #333;
  margin-bottom: 10rpx;
}

.transaction-time {
  font-size: 24rpx;
  color: #999;
}

.transaction-amount {
  font-size: 32rpx;
  font-weight: bold;
}

.income-amount {
  color: #34c759;
}

.expense-amount {
  color: #ff3b30;
}

.action-buttons {
  display: flex;
  justify-content: space-between;
  margin: 0 30rpx 30rpx;
}

.action-button {
  flex: 1;
  height: 180rpx;
  border-radius: 20rpx;
  margin: 0 10rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #fff;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.1);
}

.action-button:first-child {
  margin-left: 0;
}

.action-button:last-child {
  margin-right: 0;
}

.transfer {
  background: linear-gradient(to bottom, #ff9500, #ff7800);
}

.recharge {
  background: linear-gradient(to bottom, #34c759, #28a745);
}

.withdraw {
  background: linear-gradient(to bottom, #5856d6, #4745b3);
}

.action-icon {
  font-size: 48rpx;
  margin-bottom: 20rpx;
}

.action-text {
  font-size: 28rpx;
}
</style>
