<template>
  <view class="loading-page">
    <text class="loading-text">加载中...</text>
  </view>
</template>

<script setup lang="ts">
import { onMounted } from 'vue';

onMounted(() => {
  // 直接跳转到修真院，清除路由栈
  uni.reLaunch({
    url: '/pages/zen/index'
  });
});
</script>

<style lang="scss" scoped>
.loading-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: #1a1a2e;
}

.loading-text {
  color: #666;
  font-size: 28rpx;
}
</style>
