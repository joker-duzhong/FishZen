<template>
  <view class="phone-shell">
    <!-- 入口卡片 - 韭菜修真院 -->
    <view class="entry-card" @tap="navigateToZen">
      <view class="card-glow"></view>
      <view class="card-content">
        <view class="icon-wrapper">
          <text class="main-icon">☯</text>
          <view class="icon-ring"></view>
        </view>
        <view class="text-content">
          <text class="card-title">修真渡劫院</text>
          <text class="card-desc">贫道夜观天象，今日诸事不宜</text>
        </view>
        <view class="arrow-wrapper">
          <text class="arrow-icon">→</text>
        </view>
      </view>
      <view class="card-decoration">
        <text class="deco-text">🙏 功德无量 🙏</text>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { onShareAppMessage, onShareTimeline } from '@dcloudio/uni-app';

const navigateToZen = () => {
  uni.navigateTo({
    url: '/pages/zen/index'
  });
};
</script>

<style lang="scss" scoped>
.phone-shell {
  padding: 60rpx 40rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(180deg, #F0E6D2 0%, #E6DBC4 100%);
}

.entry-card {
  width: 100%;
  max-width: 650rpx;
  background: linear-gradient(145deg, #FFFBF0 0%, #FDF8E8 100%);
  border: 4rpx solid #2B2B2B;
  border-radius: 16rpx;
  padding: 48rpx;
  position: relative;
  overflow: hidden;
  box-shadow:
    12rpx 12rpx 0 rgba(43, 43, 43, 0.15),
    0 20rpx 60rpx rgba(196, 66, 56, 0.1);
  transition: all 0.3s ease;

  &:active {
    transform: scale(0.98);
    box-shadow:
      6rpx 6rpx 0 rgba(43, 43, 43, 0.15),
      0 10rpx 30rpx rgba(196, 66, 56, 0.15);
  }
}

.card-glow {
  position: absolute;
  top: -50%;
  right: -30%;
  width: 300rpx;
  height: 300rpx;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.2) 0%, transparent 70%);
  animation: pulseGlow 3s ease-in-out infinite;
}

@keyframes pulseGlow {
  0%, 100% { transform: scale(1); opacity: 0.5; }
  50% { transform: scale(1.2); opacity: 0.8; }
}

.card-content {
  display: flex;
  align-items: center;
  gap: 32rpx;
  position: relative;
  z-index: 1;
}

.icon-wrapper {
  position: relative;
  width: 120rpx;
  height: 120rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.main-icon {
  font-size: 80rpx;
  color: #2B2B2B;
  animation: rotateSlow 10s linear infinite;
  position: relative;
  z-index: 2;
}

@keyframes rotateSlow {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.icon-ring {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 3rpx solid rgba(196, 66, 56, 0.3);
  border-radius: 50%;
  animation: expandRing 2s ease-out infinite;
}

@keyframes expandRing {
  0% { transform: scale(0.8); opacity: 1; }
  100% { transform: scale(1.5); opacity: 0; }
}

.text-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8rpx;
}

.card-title {
  font-size: 40rpx;
  font-weight: bold;
  color: #2B2B2B;
  letter-spacing: 4rpx;
  font-family: 'KaiTi', 'STKaiti', serif;
}

.card-desc {
  font-size: 26rpx;
  color: #666;
  font-style: italic;
}

.arrow-wrapper {
  width: 60rpx;
  height: 60rpx;
  background: linear-gradient(135deg, #C44238 0%, #A33A30 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4rpx 12rpx rgba(196, 66, 56, 0.3);
}

.arrow-icon {
  color: #FFD700;
  font-size: 28rpx;
  font-weight: bold;
}

.card-decoration {
  margin-top: 32rpx;
  padding-top: 24rpx;
  border-top: 2rpx dashed rgba(43, 43, 43, 0.2);
  text-align: center;
  position: relative;
  z-index: 1;
}

.deco-text {
  font-size: 24rpx;
  color: #C44238;
  letter-spacing: 8rpx;
}
</style>
