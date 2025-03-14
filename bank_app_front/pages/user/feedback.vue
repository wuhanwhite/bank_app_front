<template>
  <view class="feedback-container">
    <view class="header">
      <text class="title">意见反馈</text>
    </view>
    
    <view class="form-section">
      <view class="form-card">
        <view class="form-header">
          <text class="form-title">提交反馈</text>
          <text class="form-subtitle">您的反馈将帮助我们不断改进产品</text>
        </view>
        
        <view class="form-content">
          <view class="form-item">
            <text class="form-label">反馈类型</text>
            <picker 
              :value="typeIndex" 
              :range="feedbackTypes" 
              @change="handleTypeChange"
            >
              <view class="picker-view">
                <view class="picker-value">{{ feedbackTypes[typeIndex] }}</view>
                <text class="iconfont icon-arrow-down"></text>
              </view>
            </picker>
          </view>
          
          <view class="form-item">
            <text class="form-label">反馈内容</text>
            <textarea 
              class="form-textarea" 
              v-model="feedbackContent" 
              placeholder="请详细描述您遇到的问题或建议..."
              maxlength="500"
            ></textarea>
            <view class="textarea-counter">
              <text>{{ feedbackContent.length }}/500</text>
            </view>
          </view>
          
          <view class="form-item">
            <text class="form-label">上传截图（选填）</text>
            <view class="upload-section">
              <view 
                class="upload-item" 
                v-for="(item, index) in uploadedImages" 
                :key="index"
              >
                <image class="upload-image" :src="item" mode="aspectFill"></image>
                <view class="delete-btn" @click="handleDeleteImage(index)">
                  <text class="iconfont icon-close"></text>
                </view>
              </view>
              
              <view class="upload-btn" @click="handleChooseImage" v-if="uploadedImages.length < 3">
                <text class="iconfont icon-plus"></text>
                <text class="upload-text">上传图片</text>
              </view>
            </view>
            <text class="upload-tip">最多上传3张图片，每张不超过5MB</text>
          </view>
          
          <view class="form-item">
            <text class="form-label">联系方式（选填）</text>
            <input 
              class="form-input" 
              type="text" 
              v-model="contactInfo" 
              placeholder="请留下您的手机号或邮箱，方便我们联系您"
            />
          </view>
        </view>
        
        <view class="form-actions">
          <button 
            class="btn btn-primary" 
            :loading="isSubmitting"
            @click="handleSubmit"
          >
            {{ isSubmitting ? '提交中...' : '提交反馈' }}
          </button>
        </view>
      </view>
    </view>
    
    <view class="feedback-history">
      <view class="history-card">
        <view class="history-header">
          <text class="history-title">我的反馈</text>
        </view>
        <view class="history-content">
          <view class="empty-state" v-if="!feedbackHistory.length">
            <text class="empty-text">暂无反馈记录</text>
          </view>
          <view class="history-list" v-else>
            <view 
              class="history-item" 
              v-for="(item, index) in feedbackHistory" 
              :key="index"
              @click="handleViewFeedback(item)"
            >
              <view class="history-top">
                <text class="feedback-type">{{ item.type }}</text>
                <text class="feedback-status" :class="getStatusClass(item.status)">
                  {{ getStatusText(item.status) }}
                </text>
              </view>
              <text class="feedback-content">{{ item.content }}</text>
              <view class="history-bottom">
                <text class="feedback-time">{{ formatDateTime(item.create_time) }}</text>
                <text class="view-detail">查看详情 ></text>
              </view>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useUserStore } from '../../store/user';

const userStore = useUserStore();
const isSubmitting = ref(false);

// 反馈类型
const feedbackTypes = ['功能建议', '问题反馈', '体验问题', '其他'];
const typeIndex = ref(0);

// 表单数据
const feedbackContent = ref('');
const contactInfo = ref('');
const uploadedImages = ref([]);

// 反馈历史
const feedbackHistory = ref([]);

// 处理类型选择
const handleTypeChange = (e) => {
  typeIndex.value = e.detail.value;
};

// 选择图片
const handleChooseImage = () => {
  if (uploadedImages.value.length >= 3) {
    uni.showToast({
      title: '最多上传3张图片',
      icon: 'none'
    });
    return;
  }
  
  uni.chooseImage({
    count: 3 - uploadedImages.value.length,
    sizeType: ['compressed'],
    sourceType: ['album', 'camera'],
    success: (res) => {
      // 检查图片大小
      const tempFilePaths = res.tempFilePaths;
      const tempFiles = res.tempFiles;
      
      for (let i = 0; i < tempFiles.length; i++) {
        if (tempFiles[i].size > 5 * 1024 * 1024) {
          uni.showToast({
            title: '图片大小不能超过5MB',
            icon: 'none'
          });
          return;
        }
      }
      
      // 添加到已上传图片列表
      uploadedImages.value = [...uploadedImages.value, ...tempFilePaths];
    }
  });
};

// 删除图片
const handleDeleteImage = (index) => {
  uploadedImages.value.splice(index, 1);
};

// 提交反馈
const handleSubmit = async () => {
  // 表单验证
  if (!feedbackContent.value.trim()) {
    uni.showToast({
      title: '请输入反馈内容',
      icon: 'none'
    });
    return;
  }
  
  isSubmitting.value = true;
  
  try {
    // 上传图片
    let imageUrls = [];
    
    if (uploadedImages.value.length > 0) {
      uni.showLoading({
        title: '上传图片中...'
      });
      
      for (let i = 0; i < uploadedImages.value.length; i++) {
        const result = await uploadImage(uploadedImages.value[i]);
        if (result && result.url) {
          imageUrls.push(result.url);
        }
      }
      
      uni.hideLoading();
    }
    
    // 提交反馈
    await userStore.submitFeedback({
      type: feedbackTypes[typeIndex.value],
      content: feedbackContent.value,
      contact: contactInfo.value,
      images: imageUrls
    });
    
    uni.showToast({
      title: '反馈提交成功',
      icon: 'success'
    });
    
    // 重置表单
    feedbackContent.value = '';
    contactInfo.value = '';
    uploadedImages.value = [];
    typeIndex.value = 0;
    
    // 刷新反馈历史
    fetchFeedbackHistory();
  } catch (error) {
    uni.showToast({
      title: error.message || '提交失败',
      icon: 'none'
    });
  } finally {
    isSubmitting.value = false;
  }
};

// 上传图片
const uploadImage = (filePath) => {
  return new Promise((resolve, reject) => {
    uni.uploadFile({
      url: userStore.getApiBaseUrl() + '/api/upload/image',
      filePath: filePath,
      name: 'file',
      header: {
        'Authorization': 'Bearer ' + userStore.getToken()
      },
      success: (res) => {
        if (res.statusCode === 200) {
          const data = JSON.parse(res.data);
          resolve(data.data);
        } else {
          reject(new Error('上传失败'));
        }
      },
      fail: (err) => {
        reject(err);
      }
    });
  });
};

// 获取反馈历史
const fetchFeedbackHistory = async () => {
  try {
    const history = await userStore.fetchFeedbackHistory();
    feedbackHistory.value = history || [];
  } catch (error) {
    console.error('获取反馈历史失败', error);
  }
};

// 查看反馈详情
const handleViewFeedback = (item) => {
  uni.navigateTo({
    url: `/pages/user/feedback-detail?id=${item.id}`
  });
};

// 获取状态样式类
const getStatusClass = (status) => {
  switch (status) {
    case 'pending':
      return 'status-pending';
    case 'processing':
      return 'status-processing';
    case 'resolved':
      return 'status-resolved';
    case 'rejected':
      return 'status-rejected';
    default:
      return '';
  }
};

// 获取状态文本
const getStatusText = (status) => {
  switch (status) {
    case 'pending':
      return '待处理';
    case 'processing':
      return '处理中';
    case 'resolved':
      return '已解决';
    case 'rejected':
      return '已驳回';
    default:
      return '未知状态';
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
  fetchFeedbackHistory();
});
</script>

<style lang="scss">
.feedback-container {
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

.form-section, .feedback-history {
  padding: 0 30rpx;
  margin-top: 30rpx;
  
  .form-card, .history-card {
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
      margin-bottom: 10rpx;
      display: block;
    }
    
    .form-subtitle {
      font-size: 26rpx;
      color: #666;
    }
  }
  
  .form-content {
    padding: 30rpx;
    
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
      }
      
      .form-textarea {
        width: 100%;
        height: 240rpx;
        border: 1px solid #e5e5e5;
        border-radius: 8rpx;
        padding: 20rpx;
        font-size: 28rpx;
        box-sizing: border-box;
      }
      
      .textarea-counter {
        text-align: right;
        font-size: 24rpx;
        color: #999;
        margin-top: 10rpx;
      }
      
      .picker-view {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 100%;
        height: 90rpx;
        border: 1px solid #e5e5e5;
        border-radius: 8rpx;
        padding: 0 20rpx;
        box-sizing: border-box;
        
        .picker-value {
          font-size: 28rpx;
          color: #333;
        }
        
        .icon-arrow-down {
          font-size: 24rpx;
          color: #999;
        }
      }
      
      .upload-section {
        display: flex;
        flex-wrap: wrap;
        margin-bottom: 15rpx;
        
        .upload-item, .upload-btn {
          width: 180rpx;
          height: 180rpx;
          margin-right: 20rpx;
          margin-bottom: 20rpx;
          border-radius: 8rpx;
          overflow: hidden;
        }
        
        .upload-item {
          position: relative;
          
          .upload-image {
            width: 100%;
            height: 100%;
          }
          
          .delete-btn {
            position: absolute;
            top: 10rpx;
            right: 10rpx;
            width: 40rpx;
            height: 40rpx;
            background-color: rgba(0, 0, 0, 0.5);
            border-radius: 20rpx;
            display: flex;
            justify-content: center;
            align-items: center;
            
            .icon-close {
              color: #fff;
              font-size: 24rpx;
            }
          }
        }
        
        .upload-btn {
          border: 1px dashed #ccc;
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          
          .icon-plus {
            font-size: 48rpx;
            color: #999;
            margin-bottom: 10rpx;
          }
          
          .upload-text {
            font-size: 26rpx;
            color: #999;
          }
        }
      }
      
      .upload-tip {
        font-size: 24rpx;
        color: #999;
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
      padding: 60rpx 0;
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
        padding: 30rpx;
        border-bottom: 1px solid #f0f0f0;
        
        &:last-child {
          border-bottom: none;
        }
        
        .history-top {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 15rpx;
          
          .feedback-type {
            font-size: 30rpx;
            font-weight: bold;
            color: #333;
          }
          
          .feedback-status {
            font-size: 26rpx;
            padding: 4rpx 16rpx;
            border-radius: 20rpx;
            
            &.status-pending {
              background-color: #f0f0f0;
              color: #666;
            }
            
            &.status-processing {
              background-color: #e6f7ff;
              color: #1890ff;
            }
            
            &.status-resolved {
              background-color: #f6ffed;
              color: #52c41a;
            }
            
            &.status-rejected {
              background-color: #fff2f0;
              color: #ff4d4f;
            }
          }
        }
        
        .feedback-content {
          font-size: 28rpx;
          color: #666;
          line-height: 1.5;
          margin-bottom: 15rpx;
          display: -webkit-box;
          -webkit-box-orient: vertical;
          -webkit-line-clamp: 2;
          overflow: hidden;
          text-overflow: ellipsis;
        }
        
        .history-bottom {
          display: flex;
          justify-content: space-between;
          align-items: center;
          
          .feedback-time {
            font-size: 26rpx;
            color: #999;
          }
          
          .view-detail {
            font-size: 26rpx;
            color: #3c9cff;
          }
        }
      }
    }
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

.icon-arrow-down:before {
  content: "\e6b9";
}

.icon-plus:before {
  content: "\e6ba";
}

.icon-close:before {
  content: "\e6bb";
}
</style>
