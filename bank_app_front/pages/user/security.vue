<template>
  <view class="security-container">
    <view class="header">
      <text class="title">账号安全</text>
    </view>
    
    <view class="form-section">
      <view class="form-card">
        <view class="form-header">
          <text class="form-title">修改密码</text>
        </view>
        <view class="form-content">
          <view class="form-item">
            <text class="form-label">当前密码</text>
            <input 
              class="form-input" 
              type="password" 
              v-model="passwordForm.oldPassword" 
              placeholder="请输入当前密码"
            />
          </view>
          
          <view class="form-item">
            <text class="form-label">新密码</text>
            <input 
              class="form-input" 
              type="password" 
              v-model="passwordForm.newPassword" 
              placeholder="请输入新密码"
            />
          </view>
          
          <view class="form-item">
            <text class="form-label">确认新密码</text>
            <input 
              class="form-input" 
              type="password" 
              v-model="passwordForm.confirmPassword" 
              placeholder="请再次输入新密码"
            />
          </view>
          
          <view class="password-tips">
            <text class="tip-text">* 密码长度至少为8位</text>
            <text class="tip-text">* 密码需包含字母、数字和特殊字符</text>
          </view>
        </view>
        <view class="form-actions">
          <button 
            class="btn btn-primary" 
            :loading="isSubmitting"
            @click="handleChangePassword"
          >
            {{ isSubmitting ? '提交中...' : '修改密码' }}
          </button>
        </view>
      </view>
    </view>
    
    <view class="security-options">
      <view class="option-card">
        <view class="option-header">
          <text class="option-title">安全设置</text>
        </view>
        <view class="option-list">
          <view class="option-item">
            <view class="option-left">
              <text class="option-label">手机验证</text>
              <text class="option-status" :class="{'verified': userInfo.phone_verified}">
                {{ userInfo.phone_verified ? '已验证' : '未验证' }}
              </text>
            </view>
            <button class="option-btn" @click="handleVerifyPhone">
              {{ userInfo.phone_verified ? '更换手机' : '立即验证' }}
            </button>
          </view>
          
          <view class="option-item">
            <view class="option-left">
              <text class="option-label">邮箱验证</text>
              <text class="option-status" :class="{'verified': userInfo.email_verified}">
                {{ userInfo.email_verified ? '已验证' : '未验证' }}
              </text>
            </view>
            <button class="option-btn" @click="handleVerifyEmail">
              {{ userInfo.email_verified ? '更换邮箱' : '立即验证' }}
            </button>
          </view>
          
          <view class="option-item">
            <view class="option-left">
              <text class="option-label">两步验证</text>
              <text class="option-status" :class="{'verified': userInfo.two_factor_enabled}">
                {{ userInfo.two_factor_enabled ? '已开启' : '未开启' }}
              </text>
            </view>
            <switch 
              :checked="userInfo.two_factor_enabled" 
              @change="handleToggleTwoFactor"
              color="#3c9cff"
            />
          </view>
        </view>
      </view>
    </view>
    
    <view class="login-history">
      <view class="history-card">
        <view class="history-header">
          <text class="history-title">登录历史</text>
        </view>
        <view class="history-content">
          <view class="empty-state" v-if="!loginHistory.length">
            <text class="empty-text">暂无登录记录</text>
          </view>
          <view class="history-list" v-else>
            <view 
              class="history-item" 
              v-for="(item, index) in loginHistory" 
              :key="index"
            >
              <view class="history-left">
                <text class="device-name">{{ item.device }}</text>
                <text class="login-time">{{ formatDateTime(item.login_time) }}</text>
              </view>
              <text class="login-location">{{ item.location }}</text>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';
import { useUserStore } from '../../store/user';

const userStore = useUserStore();
const isSubmitting = ref(false);
const userInfo = ref({
  phone_verified: false,
  email_verified: false,
  two_factor_enabled: false
});
const loginHistory = ref([]);

// 密码表单数据
const passwordForm = reactive({
  oldPassword: '',
  newPassword: '',
  confirmPassword: ''
});

// 获取用户安全信息
const fetchSecurityInfo = async () => {
  try {
    // 获取用户信息
    const info = await userStore.fetchUserInfo();
    userInfo.value = info;
    
    // 获取登录历史
    const history = await userStore.fetchLoginHistory();
    loginHistory.value = history || [];
  } catch (error) {
    console.error('获取安全信息失败', error);
    uni.showToast({
      title: '获取安全信息失败',
      icon: 'none'
    });
  }
};

// 处理修改密码
const handleChangePassword = async () => {
  // 表单验证
  if (!passwordForm.oldPassword) {
    uni.showToast({
      title: '请输入当前密码',
      icon: 'none'
    });
    return;
  }
  
  if (!passwordForm.newPassword) {
    uni.showToast({
      title: '请输入新密码',
      icon: 'none'
    });
    return;
  }
  
  if (passwordForm.newPassword !== passwordForm.confirmPassword) {
    uni.showToast({
      title: '两次输入的密码不一致',
      icon: 'none'
    });
    return;
  }
  
  if (!validatePassword(passwordForm.newPassword)) {
    uni.showToast({
      title: '密码不符合安全要求',
      icon: 'none'
    });
    return;
  }
  
  isSubmitting.value = true;
  
  try {
    await userStore.changePassword({
      old_password: passwordForm.oldPassword,
      new_password: passwordForm.newPassword
    });
    
    uni.showToast({
      title: '密码修改成功',
      icon: 'success'
    });
    
    // 清空表单
    passwordForm.oldPassword = '';
    passwordForm.newPassword = '';
    passwordForm.confirmPassword = '';
  } catch (error) {
    uni.showToast({
      title: error.message || '密码修改失败',
      icon: 'none'
    });
  } finally {
    isSubmitting.value = false;
  }
};

// 验证密码格式
const validatePassword = (password) => {
  // 至少8位，包含字母、数字和特殊字符
  const regex = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$/;
  return regex.test(password);
};

// 处理手机验证
const handleVerifyPhone = () => {
  uni.navigateTo({
    url: '/pages/user/verify-phone'
  });
};

// 处理邮箱验证
const handleVerifyEmail = () => {
  uni.navigateTo({
    url: '/pages/user/verify-email'
  });
};

// 处理两步验证开关
const handleToggleTwoFactor = async (e) => {
  const isChecked = e.detail.value;
  
  try {
    if (isChecked) {
      // 开启两步验证，跳转到设置页面
      uni.navigateTo({
        url: '/pages/user/two-factor'
      });
    } else {
      // 关闭两步验证
      await userStore.disableTwoFactor();
      
      uni.showToast({
        title: '已关闭两步验证',
        icon: 'success'
      });
      
      // 刷新安全信息
      fetchSecurityInfo();
    }
  } catch (error) {
    uni.showToast({
      title: error.message || '操作失败',
      icon: 'none'
    });
  }
};

// 格式化日期时间
const formatDateTime = (dateTimeStr) => {
  if (!dateTimeStr) return '';
  
  const date = new Date(dateTimeStr);
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0');
  
  return `${year}-${month}-${day} ${hours}:${minutes}`;
};

// 页面加载
onMounted(() => {
  fetchSecurityInfo();
  
  // 监听安全信息更新事件
  uni.$on('securityInfoUpdated', () => {
    fetchSecurityInfo();
  });
});
</script>

<style lang="scss">
.security-container {
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

.form-section, .security-options, .login-history {
  padding: 0 30rpx;
  margin-top: 30rpx;
  
  .form-card, .option-card, .history-card {
    background-color: #fff;
    border-radius: 12rpx;
    box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
  }
}

.form-card {
  .form-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .form-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .form-content {
    padding: 30rpx;
    
    .form-item {
      margin-bottom: 25rpx;
      
      .form-label {
        font-size: 28rpx;
        color: #333;
        margin-bottom: 15rpx;
        display: block;
      }
      
      .form-input {
        width: 100%;
        height: 90rpx;
        border: 1px solid #e5e5e5;
        border-radius: 8rpx;
        padding: 0 20rpx;
        font-size: 28rpx;
        box-sizing: border-box;
      }
    }
    
    .password-tips {
      margin-top: 20rpx;
      
      .tip-text {
        font-size: 24rpx;
        color: #999;
        display: block;
        margin-bottom: 10rpx;
      }
    }
  }
  
  .form-actions {
    padding: 0 30rpx 30rpx;
    
    .btn {
      width: 100%;
      height: 90rpx;
      line-height: 90rpx;
      font-size: 32rpx;
      border-radius: 8rpx;
    }
    
    .btn-primary {
      background-color: #3c9cff;
      color: #fff;
    }
  }
}

.option-card {
  .option-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .option-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .option-list {
    .option-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 30rpx;
      border-bottom: 1px solid #f0f0f0;
      
      &:last-child {
        border-bottom: none;
      }
      
      .option-left {
        .option-label {
          font-size: 30rpx;
          color: #333;
          margin-bottom: 8rpx;
          display: block;
        }
        
        .option-status {
          font-size: 26rpx;
          color: #fa3534;
          
          &.verified {
            color: #07c160;
          }
        }
      }
      
      .option-btn {
        font-size: 26rpx;
        color: #3c9cff;
        background-color: transparent;
        padding: 0;
        margin: 0;
        line-height: 1.5;
        
        &::after {
          border: none;
        }
      }
    }
  }
}

.history-card {
  .history-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .history-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .history-content {
    .empty-state {
      padding: 40rpx 0;
      display: flex;
      justify-content: center;
      align-items: center;
      
      .empty-text {
        font-size: 28rpx;
        color: #999;
      }
    }
    
    .history-list {
      .history-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 30rpx;
        border-bottom: 1px solid #f0f0f0;
        
        &:last-child {
          border-bottom: none;
        }
        
        .history-left {
          .device-name {
            font-size: 30rpx;
            color: #333;
            margin-bottom: 8rpx;
            display: block;
          }
          
          .login-time {
            font-size: 26rpx;
            color: #666;
          }
        }
        
        .login-location {
          font-size: 28rpx;
          color: #666;
        }
      }
    }
  }
}
</style>
