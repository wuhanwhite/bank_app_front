<template>
  <view class="user-container">
    <!-- 重定向到用户个人资料页面 -->
  </view>
</template>

<script setup>
import { onMounted } from 'vue';

// 页面加载时重定向到用户个人资料页面
onMounted(() => {
  // 使用重定向而不是导航，以避免在导航历史中留下多余的记录
  uni.redirectTo({
    url: '/pages/user/profile'
  });
});
</script>

<style lang="scss">
.user-container {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
