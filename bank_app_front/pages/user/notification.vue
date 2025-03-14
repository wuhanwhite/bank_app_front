<template>
  <view class="notification-container">
    <view class="header">
      <text class="title">消息通知</text>
    </view>
    
    <view class="loading-container" v-if="isLoading">
      <uni-load-more status="loading" :contentText="loadingText"></uni-load-more>
    </view>
    
    <block v-else>
      <view class="settings-section">
        <view class="settings-card">
          <view class="settings-header">
            <text class="settings-title">通知设置</text>
          </view>
          <view class="settings-list">
            <view class="settings-item">
              <view class="settings-left">
                <text class="settings-label">资产变动提醒</text>
                <text class="settings-desc">当资产发生变动时通知您</text>
              </view>
              <switch 
                :checked="notificationSettings.asset_change" 
                @change="(e) => handleToggleSetting('asset_change', e.detail.value)"
                color="#3c9cff"
              />
            </view>
            
            <view class="settings-item">
              <view class="settings-left">
                <text class="settings-label">收益提醒</text>
                <text class="settings-desc">当资产收益达到设定阈值时通知您</text>
              </view>
              <switch 
                :checked="notificationSettings.profit_alert" 
                @change="(e) => handleToggleSetting('profit_alert', e.detail.value)"
                color="#3c9cff"
              />
            </view>
            
            <view class="settings-item">
              <view class="settings-left">
                <text class="settings-label">定期报告</text>
                <text class="settings-desc">定期发送资产报告摘要</text>
              </view>
              <switch 
                :checked="notificationSettings.periodic_report" 
                @change="(e) => handleToggleSetting('periodic_report', e.detail.value)"
                color="#3c9cff"
              />
            </view>
            
            <view class="settings-item">
              <view class="settings-left">
                <text class="settings-label">安全提醒</text>
                <text class="settings-desc">账户安全相关的通知</text>
              </view>
              <switch 
                :checked="notificationSettings.security_alert" 
                @change="(e) => handleToggleSetting('security_alert', e.detail.value)"
                color="#3c9cff"
              />
            </view>
          </view>
        </view>
      </view>
      
      <view class="frequency-section" v-if="notificationSettings.periodic_report">
        <view class="frequency-card">
          <view class="frequency-header">
            <text class="frequency-title">报告频率</text>
          </view>
          <view class="frequency-content">
            <radio-group @change="handleFrequencyChange">
              <label class="frequency-item" v-for="(item, index) in frequencyOptions" :key="index">
                <view class="frequency-label">{{ item.label }}</view>
                <radio 
                  :value="item.value" 
                  :checked="reportFrequency === item.value"
                  color="#3c9cff"
                />
              </label>
            </radio-group>
          </view>
        </view>
      </view>
      
      <view class="threshold-section" v-if="notificationSettings.profit_alert">
        <view class="threshold-card">
          <view class="threshold-header">
            <text class="threshold-title">收益阈值设置</text>
          </view>
          <view class="threshold-content">
            <view class="threshold-item">
              <text class="threshold-label">收益率阈值 (%)</text>
              <view class="threshold-input-container">
                <input 
                  class="threshold-input" 
                  type="digit" 
                  v-model="profitThreshold" 
                  placeholder="请输入收益率阈值"
                />
                <text class="threshold-unit">%</text>
              </view>
            </view>
            <view class="threshold-desc">
              <text>当日收益率超过设定阈值时，系统将发送通知提醒</text>
            </view>
            <button class="save-threshold-btn" @click="handleSaveThreshold">保存阈值</button>
          </view>
        </view>
      </view>
      
      <view class="channel-section">
        <view class="channel-card">
          <view class="channel-header">
            <text class="channel-title">通知方式</text>
          </view>
          <view class="channel-list">
            <view class="channel-item">
              <view class="channel-left">
                <text class="channel-label">应用内通知</text>
                <text class="channel-desc">在应用内接收通知</text>
              </view>
              <switch 
                :checked="notificationChannels.app" 
                @change="(e) => handleToggleChannel('app', e.detail.value)"
                color="#3c9cff"
              />
            </view>
            
            <view class="channel-item">
              <view class="channel-left">
                <text class="channel-label">邮件通知</text>
                <text class="channel-desc">通过邮件接收通知</text>
              </view>
              <switch 
                :checked="notificationChannels.email" 
                @change="(e) => handleToggleChannel('email', e.detail.value)"
                color="#3c9cff"
              />
            </view>
            
            <view class="channel-item">
              <view class="channel-left">
                <text class="channel-label">短信通知</text>
                <text class="channel-desc">通过短信接收通知</text>
              </view>
              <switch 
                :checked="notificationChannels.sms" 
                @change="(e) => handleToggleChannel('sms', e.detail.value)"
                color="#3c9cff"
              />
            </view>
          </view>
        </view>
      </view>
    </block>
  </view>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';
import { useUserStore } from '../../store/user';

const userStore = useUserStore();
const isLoading = ref(true);
const isSaving = ref(false);

// 加载文本
const loadingText = {
  loading: '加载中...',
  loadmore: '点击加载更多',
  nomore: '没有更多数据了'
};

// 通知设置
const notificationSettings = reactive({
  asset_change: true,
  profit_alert: true,
  periodic_report: true,
  security_alert: true
});

// 通知渠道
const notificationChannels = reactive({
  app: true,
  email: false,
  sms: false
});

// 报告频率
const reportFrequency = ref('weekly');
const frequencyOptions = [
  { label: '每日', value: 'daily' },
  { label: '每周', value: 'weekly' },
  { label: '每月', value: 'monthly' }
];

// 收益阈值
const profitThreshold = ref('5.0');

// 获取通知设置
const fetchNotificationSettings = async () => {
  isLoading.value = true;
  
  try {
    const settings = await userStore.fetchNotificationSettings();
    
    // 更新通知设置
    if (settings) {
      notificationSettings.asset_change = settings.asset_change || false;
      notificationSettings.profit_alert = settings.profit_alert || false;
      notificationSettings.periodic_report = settings.periodic_report || false;
      notificationSettings.security_alert = settings.security_alert || false;
      
      // 更新通知渠道
      if (settings.channels) {
        notificationChannels.app = settings.channels.app || false;
        notificationChannels.email = settings.channels.email || false;
        notificationChannels.sms = settings.channels.sms || false;
      }
      
      // 更新报告频率
      if (settings.report_frequency) {
        reportFrequency.value = settings.report_frequency;
      }
      
      // 更新收益阈值
      if (settings.profit_threshold) {
        profitThreshold.value = settings.profit_threshold.toString();
      }
    }
  } catch (error) {
    console.error('获取通知设置失败', error);
    uni.showToast({
      title: '获取通知设置失败',
      icon: 'none'
    });
  } finally {
    isLoading.value = false;
  }
};

// 切换通知设置
const handleToggleSetting = async (key, value) => {
  notificationSettings[key] = value;
  
  try {
    await userStore.updateNotificationSettings({
      settings: notificationSettings,
      channels: notificationChannels,
      report_frequency: reportFrequency.value,
      profit_threshold: parseFloat(profitThreshold.value)
    });
  } catch (error) {
    console.error('更新通知设置失败', error);
    uni.showToast({
      title: '更新通知设置失败',
      icon: 'none'
    });
    
    // 恢复原值
    notificationSettings[key] = !value;
  }
};

// 切换通知渠道
const handleToggleChannel = async (key, value) => {
  notificationChannels[key] = value;
  
  try {
    await userStore.updateNotificationSettings({
      settings: notificationSettings,
      channels: notificationChannels,
      report_frequency: reportFrequency.value,
      profit_threshold: parseFloat(profitThreshold.value)
    });
  } catch (error) {
    console.error('更新通知渠道失败', error);
    uni.showToast({
      title: '更新通知渠道失败',
      icon: 'none'
    });
    
    // 恢复原值
    notificationChannels[key] = !value;
  }
};

// 处理频率变更
const handleFrequencyChange = async (e) => {
  const value = e.detail.value;
  reportFrequency.value = value;
  
  try {
    await userStore.updateNotificationSettings({
      settings: notificationSettings,
      channels: notificationChannels,
      report_frequency: value,
      profit_threshold: parseFloat(profitThreshold.value)
    });
    
    uni.showToast({
      title: '报告频率已更新',
      icon: 'success'
    });
  } catch (error) {
    console.error('更新报告频率失败', error);
    uni.showToast({
      title: '更新报告频率失败',
      icon: 'none'
    });
  }
};

// 保存收益阈值
const handleSaveThreshold = async () => {
  // 验证输入
  if (!profitThreshold.value || isNaN(parseFloat(profitThreshold.value))) {
    uni.showToast({
      title: '请输入有效的收益率阈值',
      icon: 'none'
    });
    return;
  }
  
  isSaving.value = true;
  
  try {
    await userStore.updateNotificationSettings({
      settings: notificationSettings,
      channels: notificationChannels,
      report_frequency: reportFrequency.value,
      profit_threshold: parseFloat(profitThreshold.value)
    });
    
    uni.showToast({
      title: '阈值已保存',
      icon: 'success'
    });
  } catch (error) {
    console.error('保存阈值失败', error);
    uni.showToast({
      title: '保存阈值失败',
      icon: 'none'
    });
  } finally {
    isSaving.value = false;
  }
};

// 页面加载
onMounted(() => {
  fetchNotificationSettings();
});
</script>

<style lang="scss">
.notification-container {
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

.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 400rpx;
}

.settings-section, .frequency-section, .threshold-section, .channel-section {
  padding: 0 30rpx;
  margin-top: 30rpx;
  
  .settings-card, .frequency-card, .threshold-card, .channel-card {
    background-color: #fff;
    border-radius: 12rpx;
    box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
  }
}

.settings-card, .channel-card {
  .settings-header, .channel-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .settings-title, .channel-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .settings-list, .channel-list {
    .settings-item, .channel-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 30rpx;
      border-bottom: 1px solid #f0f0f0;
      
      &:last-child {
        border-bottom: none;
      }
      
      .settings-left, .channel-left {
        .settings-label, .channel-label {
          font-size: 30rpx;
          color: #333;
          margin-bottom: 8rpx;
          display: block;
        }
        
        .settings-desc, .channel-desc {
          font-size: 26rpx;
          color: #666;
        }
      }
    }
  }
}

.frequency-card {
  .frequency-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .frequency-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .frequency-content {
    padding: 20rpx 0;
    
    .frequency-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 20rpx 30rpx;
      
      .frequency-label {
        font-size: 30rpx;
        color: #333;
      }
    }
  }
}

.threshold-card {
  .threshold-header {
    padding: 30rpx;
    border-bottom: 1px solid #f0f0f0;
    
    .threshold-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
    }
  }
  
  .threshold-content {
    padding: 30rpx;
    
    .threshold-item {
      margin-bottom: 20rpx;
      
      .threshold-label {
        font-size: 30rpx;
        color: #333;
        margin-bottom: 15rpx;
        display: block;
      }
      
      .threshold-input-container {
        display: flex;
        align-items: center;
        
        .threshold-input {
          flex: 1;
          height: 90rpx;
          border: 1px solid #e5e5e5;
          border-radius: 8rpx;
          padding: 0 20rpx;
          font-size: 28rpx;
          box-sizing: border-box;
        }
        
        .threshold-unit {
          font-size: 28rpx;
          color: #333;
          margin-left: 15rpx;
        }
      }
    }
    
    .threshold-desc {
      font-size: 26rpx;
      color: #666;
      margin-bottom: 30rpx;
    }
    
    .save-threshold-btn {
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
</style>
