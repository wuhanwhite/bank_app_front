<template>
  <view class="feedback-detail-container">
    <view class="header">
      <text class="title">反馈详情</text>
    </view>
    
    <view class="loading-container" v-if="isLoading">
      <uni-load-more status="loading" :contentText="loadingText"></uni-load-more>
    </view>
    
    <block v-else-if="feedbackDetail">
      <view class="detail-section">
        <view class="detail-card">
          <view class="detail-header">
            <view class="detail-top">
              <text class="feedback-type">{{ feedbackDetail.type }}</text>
              <text class="feedback-status" :class="getStatusClass(feedbackDetail.status)">
                {{ getStatusText(feedbackDetail.status) }}
              </text>
            </view>
            <text class="feedback-time">提交时间：{{ formatDateTime(feedbackDetail.create_time) }}</text>
          </view>
          
          <view class="detail-content">
            <view class="content-section">
              <text class="section-title">反馈内容</text>
              <text class="feedback-content">{{ feedbackDetail.content }}</text>
            </view>
            
            <view class="images-section" v-if="feedbackDetail.images && feedbackDetail.images.length > 0">
              <text class="section-title">附件图片</text>
              <view class="images-list">
                <image 
                  v-for="(image, index) in feedbackDetail.images" 
                  :key="index"
                  class="feedback-image"
                  :src="image"
                  mode="widthFix"
                  @click="previewImage(index)"
                ></image>
              </view>
            </view>
            
            <view class="contact-section" v-if="feedbackDetail.contact">
              <text class="section-title">联系方式</text>
              <text class="contact-info">{{ feedbackDetail.contact }}</text>
            </view>
          </view>
        </view>
      </view>
      
      <view class="reply-section" v-if="feedbackDetail.replies && feedbackDetail.replies.length > 0">
        <view class="reply-card">
          <view class="reply-header">
            <text class="reply-title">官方回复</text>
          </view>
          <view class="reply-list">
            <view 
              class="reply-item" 
              v-for="(reply, index) in feedbackDetail.replies" 
              :key="index"
            >
              <view class="reply-top">
                <text class="reply-from">客服 {{ reply.staff_name }}</text>
                <text class="reply-time">{{ formatDateTime(reply.reply_time) }}</text>
              </view>
              <text class="reply-content">{{ reply.content }}</text>
            </view>
          </view>
        </view>
      </view>
      
      <view class="additional-section" v-if="feedbackDetail.status !== 'resolved' && feedbackDetail.status !== 'rejected'">
        <view class="additional-card">
          <view class="additional-header">
            <text class="additional-title">补充说明</text>
          </view>
          <view class="additional-content">
            <textarea 
              class="additional-textarea" 
              v-model="additionalContent" 
              placeholder="如有需要，可以在这里补充说明..."
              maxlength="500"
            ></textarea>
            <view class="textarea-counter">
              <text>{{ additionalContent.length }}/500</text>
            </view>
            
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
            
            <button 
              class="submit-btn" 
              :loading="isSubmitting"
              @click="handleSubmitAdditional"
            >
              {{ isSubmitting ? '提交中...' : '提交补充说明' }}
            </button>
          </view>
        </view>
      </view>
    </block>
    
    <view class="error-container" v-else>
      <text class="error-text">加载失败，请返回重试</text>
      <button class="back-btn" @click="handleBack">返回</button>
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useUserStore } from '../../store/user';
import { onLoad } from '@dcloudio/uni-app';

const userStore = useUserStore();
const isLoading = ref(true);
const isSubmitting = ref(false);
const feedbackId = ref(null);
const feedbackDetail = ref(null);
const additionalContent = ref('');
const uploadedImages = ref([]);

// 加载文本
const loadingText = {
  loading: '加载中...',
  loadmore: '点击加载更多',
  nomore: '没有更多数据了'
};

// 获取反馈详情
const fetchFeedbackDetail = async () => {
  isLoading.value = true;
  
  try {
    const detail = await userStore.fetchFeedbackDetail(feedbackId.value);
    feedbackDetail.value = detail;
  } catch (error) {
    console.error('获取反馈详情失败', error);
    uni.showToast({
      title: '获取反馈详情失败',
      icon: 'none'
    });
    feedbackDetail.value = null;
  } finally {
    isLoading.value = false;
  }
};

// 预览图片
const previewImage = (index) => {
  if (feedbackDetail.value && feedbackDetail.value.images && feedbackDetail.value.images.length > 0) {
    uni.previewImage({
      current: index,
      urls: feedbackDetail.value.images
    });
  }
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

// 提交补充说明
const handleSubmitAdditional = async () => {
  // 表单验证
  if (!additionalContent.value.trim() && uploadedImages.value.length === 0) {
    uni.showToast({
      title: '请输入补充说明或上传图片',
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
    
    // 提交补充说明
    await userStore.submitFeedbackAdditional(feedbackId.value, {
      content: additionalContent.value,
      images: imageUrls
    });
    
    uni.showToast({
      title: '补充说明提交成功',
      icon: 'success'
    });
    
    // 重置表单
    additionalContent.value = '';
    uploadedImages.value = [];
    
    // 刷新反馈详情
    fetchFeedbackDetail();
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

// 返回上一页
const handleBack = () => {
  uni.navigateBack();
};

// 页面加载
onLoad((options) => {
  if (options.id) {
    feedbackId.value = options.id;
    fetchFeedbackDetail();
  } else {
    isLoading.value = false;
    uni.showToast({
      title: '反馈ID不能为空',
      icon: 'none'
    });
  }
});
</script>

<style lang="scss">
.feedback-detail-container {
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

.loading-container, .error-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 400rpx;
  
  .error-text {
    font-size: 30rpx;
    color: #666;
    margin-bottom: 30rpx;
  }
  
  .back-btn {
    font-size: 30rpx;
    color: #3c9cff;
    background-color: #fff;
    padding: 20rpx 60rpx;
    border-radius: 8rpx;
    border: 1px solid #3c9cff;
  }
}

.detail-section, .reply-section, .additional-section {
  padding: 0 30rpx;
  margin-top: 30rpx;
  
  .detail-card, .reply-card, .additional-card {
    background-color: #fff;
    border-radius: 12rpx;
    box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
  }
}

.detail-card {
  .detail-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .detail-top {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 15rpx;
      
      .feedback-type {
        font-size: 32rpx;
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
    
    .feedback-time {
      font-size: 26rpx;
      color: #666;
    }
  }
  
  .detail-content {
    padding: 30rpx;
    
    .content-section, .images-section, .contact-section {
      margin-bottom: 30rpx;
      
      &:last-child {
        margin-bottom: 0;
      }
      
      .section-title {
        font-size: 30rpx;
        font-weight: bold;
        color: #333;
        margin-bottom: 15rpx;
        display: block;
      }
      
      .feedback-content {
        font-size: 28rpx;
        color: #666;
        line-height: 1.6;
      }
      
      .images-list {
        display: flex;
        flex-direction: column;
        
        .feedback-image {
          width: 100%;
          border-radius: 8rpx;
          margin-bottom: 20rpx;
          
          &:last-child {
            margin-bottom: 0;
          }
        }
      }
      
      .contact-info {
        font-size: 28rpx;
        color: #666;
      }
    }
  }
}

.reply-card {
  .reply-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .reply-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .reply-list {
    padding: 0 30rpx;
    
    .reply-item {
      padding: 30rpx 0;
      border-bottom: 1px solid #f0f0f0;
      
      &:last-child {
        border-bottom: none;
      }
      
      .reply-top {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 15rpx;
        
        .reply-from {
          font-size: 30rpx;
          font-weight: bold;
          color: #333;
        }
        
        .reply-time {
          font-size: 26rpx;
          color: #999;
        }
      }
      
      .reply-content {
        font-size: 28rpx;
        color: #666;
        line-height: 1.6;
      }
    }
  }
}

.additional-card {
  .additional-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .additional-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .additional-content {
    padding: 30rpx;
    
    .additional-textarea {
      width: 100%;
      height: 240rpx;
      border: 1px solid #e5e5e5;
      border-radius: 8rpx;
      padding: 20rpx;
      font-size: 28rpx;
      box-sizing: border-box;
      margin-bottom: 15rpx;
    }
    
    .textarea-counter {
      text-align: right;
      font-size: 24rpx;
      color: #999;
      margin-bottom: 20rpx;
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
      margin-bottom: 30rpx;
      display: block;
    }
    
    .submit-btn {
      width: 100%;
      height: 90rpx;
      line-height: 90rpx;
      background-color: #3c9cff;
      color: #fff;
      font-size: 32rpx;
      border-radius: 8rpx;
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

.icon-plus:before {
  content: "\e6ba";
}

.icon-close:before {
  content: "\e6bb";
}
</style>
