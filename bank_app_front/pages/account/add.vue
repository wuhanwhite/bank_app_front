<template>
	<view class="container">
		<view class="header">
			<view class="back-button" @click="navigateBack">
				<text class="back-icon">←</text>
			</view>
			<text class="title">添加银行账户</text>
		</view>
		
		<view class="form-container">
			<view class="form-group">
				<text class="form-label">选择银行</text>
				<view class="bank-selector" @click="showBankPicker = true">
					<view v-if="form.bankName" class="selected-bank">
						<image class="bank-logo" :src="getBankLogo(form.bankName)" mode="aspectFit"></image>
						<text class="bank-name">{{ form.bankName }}</text>
					</view>
					<view v-else class="bank-placeholder">
						<text>请选择银行</text>
					</view>
					<text class="selector-arrow">></text>
				</view>
			</view>
			
			<view class="form-group">
				<text class="form-label">账户类型</text>
				<view class="account-type-tabs">
					<view 
						v-for="(type, index) in accountTypes" 
						:key="index" 
						:class="['type-tab', { active: form.accountType === type.value }]"
						@click="form.accountType = type.value"
					>
						<text>{{ type.label }}</text>
					</view>
				</view>
			</view>
			
			<view class="form-group">
				<text class="form-label">账号</text>
				<input class="form-input" type="number" v-model="form.accountNumber" placeholder="请输入银行卡号" maxlength="19" />
			</view>
			
			<view class="form-group">
				<text class="form-label">户名</text>
				<input class="form-input" type="text" v-model="form.accountName" placeholder="请输入开户人姓名" />
			</view>
			
			<view class="form-group">
				<text class="form-label">余额</text>
				<view class="balance-input">
					<text class="currency-symbol">¥</text>
					<input class="form-input" type="digit" v-model="form.balance" placeholder="请输入账户余额" />
				</view>
			</view>
			
			<view class="form-group" v-if="form.accountType === 'credit'">
				<text class="form-label">信用额度</text>
				<view class="balance-input">
					<text class="currency-symbol">¥</text>
					<input class="form-input" type="digit" v-model="form.creditLimit" placeholder="请输入信用卡额度" />
				</view>
			</view>
			
			<view class="form-group">
				<text class="form-label">备注</text>
				<textarea class="form-textarea" v-model="form.remark" placeholder="请输入备注信息（选填）" />
			</view>
			
			<button class="submit-button" @click="handleSubmit">确认添加</button>
		</view>
		
		<!-- 银行选择弹窗 -->
		<view class="bank-picker-mask" v-if="showBankPicker" @click="showBankPicker = false"></view>
		<view class="bank-picker" v-if="showBankPicker">
			<view class="picker-header">
				<text class="picker-title">选择银行</text>
				<text class="picker-close" @click="showBankPicker = false">×</text>
			</view>
			<scroll-view class="bank-list" scroll-y>
				<view 
					v-for="(bank, index) in banks" 
					:key="index" 
					class="bank-item"
					@click="selectBank(bank)"
				>
					<image class="bank-logo" :src="bank.logo" mode="aspectFit"></image>
					<text class="bank-name">{{ bank.name }}</text>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script setup>
import { ref } from 'vue';

// 表单数据
const form = ref({
  bankName: '',
  bankLogo: '',
  accountType: 'debit', // debit: 储蓄卡, credit: 信用卡
  accountNumber: '',
  accountName: '',
  balance: '',
  creditLimit: '',
  remark: ''
});

// 账户类型
const accountTypes = [
  { label: '储蓄卡', value: 'debit' },
  { label: '信用卡', value: 'credit' }
];

// 银行列表
const banks = ref([
  { name: '中国银行', logo: '/static/images/bank-boc.png' },
  { name: '工商银行', logo: '/static/images/bank-icbc.png' },
  { name: '建设银行', logo: '/static/images/bank-ccb.png' },
  { name: '农业银行', logo: '/static/images/bank-abc.png' },
  { name: '交通银行', logo: '/static/images/bank-bocom.png' },
  { name: '招商银行', logo: '/static/images/bank-cmb.png' },
  { name: '邮储银行', logo: '/static/images/bank-psbc.png' },
  { name: '浦发银行', logo: '/static/images/bank-spdb.png' },
  { name: '中信银行', logo: '/static/images/bank-citic.png' },
  { name: '光大银行', logo: '/static/images/bank-ceb.png' },
  { name: '华夏银行', logo: '/static/images/bank-hxb.png' },
  { name: '民生银行', logo: '/static/images/bank-cmbc.png' }
]);

// 是否显示银行选择器
const showBankPicker = ref(false);

// 选择银行
const selectBank = (bank) => {
  form.value.bankName = bank.name;
  form.value.bankLogo = bank.logo;
  showBankPicker.value = false;
};

// 获取银行logo
const getBankLogo = (bankName) => {
  const bank = banks.value.find(item => item.name === bankName);
  return bank ? bank.logo : '/static/images/bank-default.png';
};

// 提交表单
const handleSubmit = () => {
  // 表单验证
  if (!form.value.bankName) {
    uni.showToast({
      title: '请选择银行',
      icon: 'none'
    });
    return;
  }
  
  if (!form.value.accountNumber) {
    uni.showToast({
      title: '请输入银行卡号',
      icon: 'none'
    });
    return;
  }
  
  if (!form.value.accountName) {
    uni.showToast({
      title: '请输入户名',
      icon: 'none'
    });
    return;
  }
  
  if (!form.value.balance) {
    uni.showToast({
      title: '请输入账户余额',
      icon: 'none'
    });
    return;
  }
  
  if (form.value.accountType === 'credit' && !form.value.creditLimit) {
    uni.showToast({
      title: '请输入信用额度',
      icon: 'none'
    });
    return;
  }
  
  // 模拟提交请求
  uni.showLoading({
    title: '添加中...'
  });
  
  setTimeout(() => {
    uni.hideLoading();
    
    // 模拟添加成功
    uni.showToast({
      title: '添加成功',
      icon: 'success'
    });
    
    // 返回账户列表页
    setTimeout(() => {
      uni.navigateBack();
    }, 1500);
  }, 2000);
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

.form-container {
  flex: 1;
  padding: 30rpx;
}

.form-group {
  margin-bottom: 40rpx;
}

.form-label {
  font-size: 28rpx;
  color: #666;
  margin-bottom: 20rpx;
  display: block;
}

.bank-selector {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 100rpx;
  background-color: #fff;
  border-radius: 10rpx;
  padding: 0 30rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.selected-bank {
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
  color: #333;
}

.bank-placeholder {
  font-size: 32rpx;
  color: #999;
}

.selector-arrow {
  font-size: 32rpx;
  color: #ccc;
}

.account-type-tabs {
  display: flex;
  border-radius: 10rpx;
  overflow: hidden;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.type-tab {
  flex: 1;
  height: 100rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #fff;
  font-size: 32rpx;
  color: #666;
}

.type-tab.active {
  background-color: #3c9cff;
  color: #fff;
}

.form-input {
  height: 100rpx;
  background-color: #fff;
  border-radius: 10rpx;
  padding: 0 30rpx;
  font-size: 32rpx;
  color: #333;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.balance-input {
  display: flex;
  align-items: center;
  background-color: #fff;
  border-radius: 10rpx;
  padding: 0 30rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.currency-symbol {
  font-size: 32rpx;
  color: #333;
  margin-right: 10rpx;
}

.balance-input .form-input {
  flex: 1;
  box-shadow: none;
  padding: 0;
}

.form-textarea {
  height: 200rpx;
  background-color: #fff;
  border-radius: 10rpx;
  padding: 20rpx 30rpx;
  font-size: 32rpx;
  color: #333;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.submit-button {
  width: 100%;
  height: 100rpx;
  background: #3c9cff;
  color: #fff;
  border-radius: 50rpx;
  font-size: 36rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 60rpx;
  box-shadow: 0 10rpx 20rpx rgba(60, 156, 255, 0.2);
}

.bank-picker-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 100;
}

.bank-picker {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  height: 70vh;
  background-color: #fff;
  border-radius: 30rpx 30rpx 0 0;
  z-index: 101;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.picker-header {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100rpx;
  position: relative;
  border-bottom: 1px solid #f5f5f5;
}

.picker-title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.picker-close {
  position: absolute;
  right: 30rpx;
  top: 0;
  height: 100rpx;
  width: 100rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40rpx;
  color: #999;
}

.bank-list {
  flex: 1;
  padding: 0 30rpx;
}

.bank-item {
  display: flex;
  align-items: center;
  padding: 30rpx 0;
  border-bottom: 1px solid #f5f5f5;
}

.bank-item:last-child {
  border-bottom: none;
}
</style>
