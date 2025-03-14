<template>
  <view class="edit-profile-container">
    <view class="header">
      <text class="title">编辑个人资料</text>
    </view>
    
    <view class="loading-container" v-if="isLoading">
      <uni-load-more status="loading" :contentText="loadingText"></uni-load-more>
    </view>
    
    <block v-else>
      <view class="form-section">
        <view class="avatar-section">
          <view class="avatar-container">
            <image class="avatar-image" :src="userForm.avatar || '/static/images/default-avatar.png'" mode="aspectFill"></image>
            <view class="avatar-upload" @click="chooseAvatar">
              <text class="iconfont icon-camera"></text>
            </view>
          </view>
          <text class="avatar-tip">点击更换头像</text>
        </view>
        
        <view class="form-content">
          <view class="form-item">
            <text class="form-label">用户名</text>
            <input 
              class="form-input" 
              type="text" 
              v-model="userForm.username" 
              placeholder="请输入用户名"
              disabled
            />
          </view>
          
          <view class="form-item">
            <text class="form-label">昵称</text>
            <input 
              class="form-input" 
              type="text" 
              v-model="userForm.nickname" 
              placeholder="请输入昵称"
            />
          </view>
          
          <view class="form-item">
            <text class="form-label">邮箱</text>
            <input 
              class="form-input" 
              type="text" 
              v-model="userForm.email" 
              placeholder="请输入邮箱"
            />
          </view>
          
          <view class="form-item">
            <text class="form-label">手机号</text>
            <input 
              class="form-input" 
              type="number" 
              v-model="userForm.phone" 
              placeholder="请输入手机号"
            />
          </view>
          
          <view class="form-item">
            <text class="form-label">个人简介</text>
            <textarea 
              class="form-textarea" 
              v-model="userForm.bio" 
              placeholder="请输入个人简介（选填）"
            ></textarea>
          </view>
        </view>
      </view>
      
      <view class="form-actions">
        <button 
          class="btn btn-primary" 
          :loading="isSubmitting"
          @click="handleSubmit"
        >
          {{ isSubmitting ? '保存中...' : '保存修改' }}
        </button>
        <button class="btn btn-default" @click="handleCancel">取消</button>
      </view>
    </block>
  </view>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';
import { useUserStore } from '../../store/user';

const userStore = useUserStore();
const isLoading = ref(true);
const isSubmitting = ref(false);

// 加载文本
const loadingText = {
  loading: '加载中...',
  loadmore: '点击加载更多',
  nomore: '没有更多数据了'
};

// 用户表单数据
const userForm = reactive({
  username: '',
  nickname: '',
  email: '',
  phone: '',
  bio: '',
  avatar: ''
});

// 获取用户信息
const fetchUserInfo = async () => {
  isLoading.value = true;
  
  try {
    const info = await userStore.fetchUserInfo();
    
    userForm.username = info.username || '';
    userForm.nickname = info.nickname || '';
    userForm.email = info.email || '';
    userForm.phone = info.phone || '';
    userForm.bio = info.bio || '';
    userForm.avatar = info.avatar || '';
  } catch (error) {
    console.error('获取用户信息失败', error);
    uni.showToast({
      title: '获取用户信息失败',
      icon: 'none'
    });
  } finally {
    isLoading.value = false;
  }
};

// 选择头像
const chooseAvatar = () => {
  uni.chooseImage({
    count: 1,
    sizeType: ['compressed'],
    sourceType: ['album', 'camera'],
    success: (res) => {
      const tempFilePath = res.tempFilePaths[0];
      
      // 预览选择的图片
      userForm.avatar = tempFilePath;
      
      // 上传头像
      uploadAvatar(tempFilePath);
    }
  });
};

// 上传头像
const uploadAvatar = async (filePath) => {
  uni.showLoading({
    title: '上传中...'
  });
  
  try {
    const result = await userStore.uploadAvatar(filePath);
    
    if (result && result.url) {
      userForm.avatar = result.url;
      
      uni.showToast({
        title: '头像上传成功',
        icon: 'success'
      });
    } else {
      throw new Error('上传失败');
    }
  } catch (error) {
    console.error('上传头像失败', error);
    uni.showToast({
      title: '上传头像失败',
      icon: 'none'
    });
    
    // 恢复原头像
    fetchUserInfo();
  } finally {
    uni.hideLoading();
  }
};

// 表单提交
const handleSubmit = async () => {
  // 表单验证
  if (!userForm.nickname) {
    uni.showToast({
      title: '请输入昵称',
      icon: 'none'
    });
    return;
  }
  
  if (userForm.email && !validateEmail(userForm.email)) {
    uni.showToast({
      title: '邮箱格式不正确',
      icon: 'none'
    });
    return;
  }
  
  if (userForm.phone && !validatePhone(userForm.phone)) {
    uni.showToast({
      title: '手机号格式不正确',
      icon: 'none'
    });
    return;
  }
  
  isSubmitting.value = true;
  
  try {
    // 更新用户信息
    await userStore.updateUserInfo({
      nickname: userForm.nickname,
      email: userForm.email,
      phone: userForm.phone,
      bio: userForm.bio
    });
    
    uni.showToast({
      title: '更新成功',
      icon: 'success'
    });
    
    // 触发用户信息更新事件
    uni.$emit('userInfoUpdated');
    
    // 更新成功后返回上一页
    setTimeout(() => {
      uni.navigateBack();
    }, 1500);
  } catch (error) {
    uni.showToast({
      title: error.message || '更新失败',
      icon: 'none'
    });
  } finally {
    isSubmitting.value = false;
  }
};

// 验证邮箱格式
const validateEmail = (email) => {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
};

// 验证手机号格式
const validatePhone = (phone) => {
  const regex = /^1[3-9]\d{9}$/;
  return regex.test(phone);
};

// 取消操作
const handleCancel = () => {
  uni.navigateBack();
};

// 页面加载
onMounted(() => {
  fetchUserInfo();
});
</script>

<style lang="scss">
.edit-profile-container {
  min-height: 100vh;
  background-color: #f8f8f8;
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

.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 400rpx;
}

.form-section {
  padding: 30rpx;
  
  .avatar-section {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-bottom: 40rpx;
    
    .avatar-container {
      position: relative;
      width: 180rpx;
      height: 180rpx;
      margin-bottom: 20rpx;
      
      .avatar-image {
        width: 100%;
        height: 100%;
        border-radius: 90rpx;
      }
      
      .avatar-upload {
        position: absolute;
        right: 0;
        bottom: 0;
        width: 60rpx;
        height: 60rpx;
        background-color: #3c9cff;
        border-radius: 30rpx;
        display: flex;
        justify-content: center;
        align-items: center;
        
        .iconfont {
          color: #fff;
          font-size: 32rpx;
        }
      }
    }
    
    .avatar-tip {
      font-size: 26rpx;
      color: #666;
    }
  }
  
  .form-content {
    background-color: #fff;
    border-radius: 12rpx;
    padding: 30rpx;
    box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
    
    .form-item {
      margin-bottom: 25rpx;
      
      &:last-child {
        margin-bottom: 0;
      }
      
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
        
        &[disabled] {
          background-color: #f5f5f5;
          color: #999;
        }
      }
      
      .form-textarea {
        width: 100%;
        height: 180rpx;
        border: 1px solid #e5e5e5;
        border-radius: 8rpx;
        padding: 20rpx;
        font-size: 28rpx;
        box-sizing: border-box;
      }
    }
  }
}

.form-actions {
  padding: 30rpx;
  
  .btn {
    width: 100%;
    height: 90rpx;
    line-height: 90rpx;
    font-size: 32rpx;
    border-radius: 8rpx;
    margin-bottom: 20rpx;
  }
  
  .btn-primary {
    background-color: #3c9cff;
    color: #fff;
  }
  
  .btn-default {
    background-color: #f5f5f5;
    color: #666;
    border: 1px solid #e5e5e5;
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

.icon-camera:before {
  content: "\e6b7";
}
</style>
