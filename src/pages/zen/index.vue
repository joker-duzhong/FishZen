<template>
  <view class="app-page">
    <!-- 顶部红色导航栏 -->
    <view class="header">
      <view class="header-top">
        <text class="header-title">韭菜修真院</text>
      </view>
    </view>

    <!-- 主内容区 -->
    <scroll-view scroll-y class="main-scroll">
      <!-- 1. 功德木鱼 - 点击区域扩大 -->
      <view class="section-card merit-section" @tap="handleMuyuTap">
        <view class="merit-header">
          <view class="merit-left">
            <text class="merit-label">我的功德</text>
            <view class="merit-value-row">
              <text class="merit-value">{{ merit }}</text>
              <text class="merit-unit">点</text>
            </view>
          </view>
          <!-- 小猫敲木鱼 -->
          <image
            class="cat-muyu-img"
            :class="{ 'pressing': isPressing }"
            src="/static/logo.png"
            mode="aspectFit"
          />
        </view>
        <!-- 大弹幕飘字 -->
        <view v-for="p in meritParticles" :key="p.id" class="big-float" :style="{ left: p.x + 'px', top: p.y + 'px' }">
          功德+1
        </view>
        <text class="tap-hint">点击卡片积攒功德</text>
      </view>

      <!-- 2. 回本计算器 - 类似行情数据展示 -->
      <view class="section-card">
        <view class="card-title-bar">
          <text class="card-title">📈 回本计算器</text>
        </view>

        <view class="calc-box">
          <view class="calc-row">
            <view class="calc-item">
              <text class="calc-label">当前亏损</text>
              <text class="calc-num down">-{{ lossValue }}%</text>
            </view>
            <view class="calc-arrow">
              <text class="arrow-line"></text>
              <text class="arrow-text">回本需涨</text>
              <text class="arrow-line"></text>
            </view>
            <view class="calc-item">
              <text class="calc-label">需要涨幅</text>
              <text class="calc-num up">{{ gainDisplay }}</text>
            </view>
          </view>

          <view class="slider-box">
            <slider
              :min="1"
              :max="99"
              :value="lossValue"
              :block-size="22"
              active-color="#e64340"
              background-color="#e5e5e5"
              @changing="onSliderChange"
            />
            <view class="slider-labels">
              <text>1%</text>
              <text>50%</text>
              <text>99%</text>
            </view>
          </view>

          <view class="result-box">
            <view class="result-tag">{{ currentDivination.gua }}</view>
            <text class="result-text">{{ currentDivination.desc }}</text>
          </view>
        </view>
      </view>

      <!-- 收益波动称号 -->
      <view class="section-card">
        <view class="card-title-bar">
          <text class="card-title">🏆 收益波动称号</text>
        </view>

        <view class="title-box">
          <view class="profit-slider-wrap">
            <view class="profit-labels">
              <text class="profit-label loss-label">亏损</text>
              <text class="profit-label gain-label">盈利</text>
            </view>
            <slider
              :min="-99"
              :max="99"
              :value="profitValue"
              :block-size="24"
              active-color="#e64340"
              background-color="#e5e5e5"
              @changing="onProfitSliderChange"
            />
          </view>

          <view class="title-result">
            <view class="title-badge" :class="'level-' + currentTitle.level">
              <text class="title-emoji">{{ currentTitle.emoji }}</text>
              <text class="title-name">{{ currentTitle.name }}</text>
            </view>
            <text class="title-desc">{{ currentTitle.desc }}</text>
          </view>
        </view>
      </view>

      <!-- 3. 今日运势 -->
      <view class="section-card">
        <view class="card-title-bar">
          <text class="card-title">🎋 今日运势</text>
          <text class="card-extra">{{ fortuneUsed ? '已求签' : '可求签' }}</text>
        </view>

        <view class="fortune-box">
          <!-- 签筒 -->
          <view
            v-if="!showFortune"
            class="qian-area"
            :class="{ 'shaking': isShaking, 'disabled': fortuneUsed }"
            @tap="handleFortuneTap"
          >
            <view class="qian-tong">
              <text class="qian-icon">🎋</text>
            </view>
            <text class="qian-text">{{ isShaking ? '求签中...' : '点击求签' }}</text>
          </view>

          <!-- 签文 -->
          <view v-if="showFortune" class="fortune-result">
            <view class="fortune-header">
              <text class="fortune-level" :class="fortuneLevelClass">{{ currentFortune.level }}</text>
              <text class="fortune-poem">{{ currentFortune.poem }}</text>
            </view>
            <text class="fortune-desc">{{ currentFortune.desc }}</text>
          </view>
        </view>
      </view>

      <!-- 底部留白 -->
      <view class="bottom-space"></view>
    </scroll-view>

    <!-- 底部Tab栏 - 暂时隐藏 -->
    <!-- <view class="tab-bar">
      <view class="tab-item active">
        <text class="tab-icon">🏠</text>
        <text class="tab-text">首页</text>
      </view>
      <view class="tab-item">
        <text class="tab-icon">📊</text>
        <text class="tab-text">行情</text>
      </view>
      <view class="tab-item">
        <text class="tab-icon">📝</text>
        <text class="tab-text">交易</text>
      </view>
      <view class="tab-item">
        <text class="tab-icon">👤</text>
        <text class="tab-text">我的</text>
      </view>
    </view> -->
  </view>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

// ============ 存储 ============
const STORAGE_KEY_MERIT = 'zen_merit_count';
const STORAGE_KEY_FORTUNE = 'zen_daily_fortune';

const getTodayStr = () => {
  const now = new Date();
  return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`;
};

// ============ 木鱼 ============
const merit = ref(0);
const isPressing = ref(false);
const meritParticles = ref<{ id: number; x: number; y: number }[]>([]);
let particleId = 0;

const loadMerit = () => {
  try {
    const saved = uni.getStorageSync(STORAGE_KEY_MERIT);
    if (saved && !isNaN(Number(saved))) merit.value = Number(saved);
  } catch (e) {}
};

const saveMerit = () => {
  try { uni.setStorageSync(STORAGE_KEY_MERIT, merit.value); } catch (e) {}
};

const handleMuyuTap = () => {
  merit.value++;
  saveMerit();
  isPressing.value = true;
  setTimeout(() => { isPressing.value = false; }, 80);
  uni.vibrateShort({ type: 'light' });

  const newParticle = {
    id: particleId++,
    x: 100 + (Math.random() - 0.5) * 80,
    y: 60
  };
  meritParticles.value.push(newParticle);
  setTimeout(() => {
    meritParticles.value = meritParticles.value.filter(p => p.id !== newParticle.id);
  }, 800);
};

// ============ 计算器 ============
const lossValue = ref(10);

const divinations = [
  { limit: 10, gua: "小耗", desc: "皮外伤，少喝两杯奶茶可解。" },
  { limit: 20, gua: "坎卦", desc: "行路难，切忌妄动，闭关锁仓。" },
  { limit: 30, gua: "困卦", desc: "深陷泥潭，唯有装死一法可保全。" },
  { limit: 40, gua: "剥卦", desc: "剥皮削骨，不如去深山老林喊两嗓子。" },
  { limit: 50, gua: "大过", desc: "天劫腰斩，想要回本难如登天。" },
  { limit: 60, gua: "损卦", desc: "损下益上，你的钱都补了庄家的天。" },
  { limit: 80, gua: "明夷", desc: "暗无天日，否极泰来在下个世纪。" },
  { limit: 99, gua: "涅槃", desc: "九九归一，建议销户重修，立地成佛。" }
];

const gainDisplay = computed(() => {
  const gain = (lossValue.value / (100 - lossValue.value)) * 100;
  return gain > 9900 ? "∞" : Math.round(gain) + "%";
});

const currentDivination = computed(() => {
  let current = divinations[divinations.length - 1];
  for (const item of divinations) {
    if (lossValue.value <= item.limit) { current = item; break; }
  }
  return current;
});

const onSliderChange = (e: any) => { lossValue.value = e.detail.value; };

// ============ 收益波动称号 ============
const profitValue = ref(-10);

const titles = [
  { min: -99, max: -70, emoji: "👑", name: "韭帝", desc: "九五之尊，割到极致", level: "purple" },
  { min: -70, max: -50, emoji: "🔥", name: "韭神", desc: "封神榜上第一人", level: "red" },
  { min: -50, max: -30, emoji: "💎", name: "韭皇", desc: "深藏功与名", level: "orange" },
  { min: -30, max: -10, emoji: "🥀", name: "老韭菜", desc: "久经沙场的老兵", level: "yellow" },
  { min: -10, max: 0, emoji: "🌱", name: "小韭菜", desc: "刚刚发芽还在成长", level: "green" },
  { min: 0, max: 10, emoji: "💰", name: "小赚", desc: "不贪不躁见好就收", level: "lightgreen" },
  { min: 10, max: 20, emoji: "🐂", name: "小牛", desc: "初露锋芒未来可期", level: "cyan" },
  { min: 20, max: 50, emoji: "🦁", name: "大牛", desc: "牛气冲天势不可挡", level: "blue" },
  { min: 50, max: 100, emoji: "🏆", name: "股神", desc: "封神演义你是主角", level: "gold" }
];

const currentTitle = computed(() => {
  for (const t of titles) {
    if (profitValue.value >= t.min && profitValue.value < t.max) {
      return t;
    }
  }
  return titles[0];
});

const onProfitSliderChange = (e: any) => { profitValue.value = e.detail.value; };

// ============ 灵签 ============
const isShaking = ref(false);
const showFortune = ref(false);
const fortuneUsed = ref(false);
const currentFortune = ref({ level: "", poem: "", desc: "" });

const fortuneData = [
  { level: "上上签", poem: "枯木逢春", desc: "枯木逢春犹再发，废纸或将变废为宝，切记拿住。" },
  { level: "上吉", poem: "紫气东来", desc: "主力资金隐隐若现，这波能不能飞升，全看造化。" },
  { level: "中吉", poem: "否极泰来", desc: "跌无可跌就是涨的开始，万一反弹了呢？" },
  { level: "中平", poem: "静观其变", desc: "敌不动我不动，目前局势不明，建议回家睡觉。" },
  { level: "下签", poem: "关灯吃面", desc: "今日大概率又是随礼的一天，记得多放葱花。" },
  { level: "下下签", poem: "天雷滚滚", desc: "大凶！近日忌看盘，忌补仓，宜卸载软件保平安。" },
  { level: "神签", poem: "相信国运", desc: "别问，问就是相信光，总能等到回本的那天。" }
];

const fortuneLevelClass = computed(() => {
  const level = currentFortune.value.level;
  if (level === '上上签' || level === '神签') return 'level-best';
  if (level === '上吉' || level === '中吉') return 'level-good';
  if (level === '中平') return 'level-medium';
  return 'level-bad';
});

const loadTodayFortune = () => {
  try {
    const saved = uni.getStorageSync(STORAGE_KEY_FORTUNE);
    if (saved) {
      const data = JSON.parse(saved);
      if (data.date === getTodayStr() && data.fortune) {
        currentFortune.value = data.fortune;
        showFortune.value = true;
        fortuneUsed.value = true;
      }
    }
  } catch (e) {}
};

const saveTodayFortune = (fortune: typeof currentFortune.value) => {
  try {
    uni.setStorageSync(STORAGE_KEY_FORTUNE, JSON.stringify({ date: getTodayStr(), fortune }));
  } catch (e) {}
};

const handleFortuneTap = () => {
  if (fortuneUsed.value || isShaking.value) return;
  isShaking.value = true;
  uni.vibrateShort({ type: 'medium' });
  setTimeout(() => uni.vibrateShort({ type: 'medium' }), 150);
  setTimeout(() => uni.vibrateShort({ type: 'medium' }), 300);
  setTimeout(() => {
    isShaking.value = false;
    const fortune = fortuneData[Math.floor(Math.random() * fortuneData.length)];
    currentFortune.value = fortune;
    showFortune.value = true;
    fortuneUsed.value = true;
    saveTodayFortune(fortune);
    uni.vibrateShort({ type: 'heavy' });
  }, 1200);
};

onMounted(() => {
  loadMerit();
  loadTodayFortune();
});
</script>

<style lang="scss" scoped>
.app-page {
  min-height: 100vh;
  height: 100vh;
  background: #f5f5f5;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 顶部导航 */
.header {
  background: linear-gradient(135deg, #e64340 0%, #c9382e 100%);
  padding-top: 88rpx;
  padding-bottom: 24rpx;
  padding-left: 32rpx;
  padding-right: 32rpx;
  flex-shrink: 0;
}

.header-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.header-title {
  font-size: 36rpx;
  font-weight: bold;
  color: #fff;
}

/* 滚动区 */
.main-scroll {
  flex: 1;
  height: 0;
}

/* 卡片 */
.section-card {
  background: #fff;
  margin: 20rpx 24rpx;
  border-radius: 16rpx;
  padding: 28rpx;
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.05);
}

.card-title-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 24rpx;
  padding-bottom: 20rpx;
  border-bottom: 1rpx solid #f0f0f0;
}

.card-title {
  font-size: 30rpx;
  font-weight: bold;
  color: #333;
}

.card-extra {
  font-size: 24rpx;
  color: #999;
}

/* 功德区域 */
.merit-section {
  background: linear-gradient(135deg, #fff5f5 0%, #fff 100%);
  position: relative;
  overflow: hidden;
  cursor: pointer;
}

.merit-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: relative;
  z-index: 1;
}

.merit-left {
  flex: 1;
}

.merit-label {
  font-size: 26rpx;
  color: #666;
  margin-bottom: 8rpx;
  display: block;
}

.merit-value-row {
  display: flex;
  align-items: baseline;
  gap: 8rpx;
}

.merit-value {
  font-size: 64rpx;
  font-weight: bold;
  color: #e64340;
  font-family: 'DIN Alternate', 'Arial', sans-serif;
}

.merit-unit {
  font-size: 26rpx;
  color: #e64340;
}

.merit-btn {
  width: 140rpx;
  height: 140rpx;
  border-radius: 50%;
  background: linear-gradient(145deg, #ff6b6b 0%, #e64340 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8rpx 24rpx rgba(230, 67, 64, 0.3);
  position: relative;
  transition: transform 0.08s;

  &.pressing {
    transform: scale(0.9);
  }
}

/* 小猫敲木鱼 */
.cat-muyu-img {
  width: 160rpx;
  height: 160rpx;
  transition: transform 0.08s;

  &.pressing {
    transform: scale(0.9);
  }
}

/* 大弹幕飘字 */
.big-float {
  position: absolute;
  color: #e64340;
  font-size: 48rpx;
  font-weight: bold;
  pointer-events: none;
  animation: bigFloatUp 0.8s ease-out forwards;
  z-index: 100;
  text-shadow: 0 2rpx 8rpx rgba(230, 67, 64, 0.3);
}

@keyframes bigFloatUp {
  0% { transform: translate(-50%, 0) scale(0.5); opacity: 0; }
  20% { transform: translate(-50%, -30rpx) scale(1.3); opacity: 1; }
  100% { transform: translate(-50%, -150rpx) scale(1); opacity: 0; }
}

.tap-hint {
  display: block;
  text-align: center;
  font-size: 24rpx;
  color: #999;
  margin-top: 20rpx;
}

/* 计算器 */
.calc-box {
  padding: 0 8rpx;
}

.calc-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20rpx 0;
}

.calc-item {
  flex: 1;
  text-align: center;
}

.calc-label {
  display: block;
  font-size: 24rpx;
  color: #999;
  margin-bottom: 12rpx;
}

.calc-num {
  font-size: 48rpx;
  font-weight: bold;
  font-family: 'DIN Alternate', 'Arial', sans-serif;

  &.down { color: #07c160; }
  &.up { color: #e64340; }
}

.calc-arrow {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8rpx;
  padding: 0 16rpx;
}

.arrow-line {
  width: 60rpx;
  height: 1rpx;
  background: #ddd;
}

.arrow-text {
  font-size: 22rpx;
  color: #999;
  white-space: nowrap;
}

.slider-box {
  margin: 24rpx 0;
}

.slider-labels {
  display: flex;
  justify-content: space-between;
  font-size: 22rpx;
  color: #999;
  margin-top: 8rpx;
}

.result-box {
  background: #fff8f8;
  border-radius: 12rpx;
  padding: 20rpx 24rpx;
  margin-top: 16rpx;
  border-left: 6rpx solid #e64340;
}

.result-tag {
  display: inline-block;
  background: #e64340;
  color: #fff;
  font-size: 22rpx;
  padding: 6rpx 16rpx;
  border-radius: 4rpx;
  margin-bottom: 12rpx;
}

.result-text {
  display: block;
  font-size: 26rpx;
  color: #666;
  line-height: 1.6;
}

/* 收益波动称号 */
.title-box {
  padding: 0 8rpx;
}

.profit-slider-wrap {
  margin: 16rpx 0;
}

.profit-labels {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12rpx;
}

.profit-label {
  font-size: 24rpx;
  font-weight: bold;

  &.loss-label { color: #07c160; }
  &.gain-label { color: #e64340; }
}

.title-result {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 32rpx;
  padding: 32rpx;
  background: #fafafa;
  border-radius: 16rpx;
}

.title-badge {
  display: flex;
  align-items: center;
  gap: 16rpx;
  padding: 20rpx 32rpx;
  border-radius: 16rpx;

  &.level-purple {
    background: linear-gradient(135deg, #9b59b6 0%, #8e44ad 100%);
    .title-emoji { font-size: 64rpx; }
    .title-name { color: #fff; font-size: 40rpx; font-weight: bold; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-red {
    background: linear-gradient(135deg, #e64340 0%, #c9382e 100%);
    .title-emoji { font-size: 60rpx; }
    .title-name { color: #fff; font-size: 38rpx; font-weight: bold; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-orange {
    background: linear-gradient(135deg, #ff9500 0%, #ff6b00 100%);
    .title-emoji { font-size: 56rpx; }
    .title-name { color: #fff; font-size: 36rpx; font-weight: bold; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-yellow {
    background: linear-gradient(135deg, #fbbf24 0%, #f59e0b 100%);
    .title-emoji { font-size: 52rpx; }
    .title-name { color: #333; font-size: 34rpx; font-weight: bold; }
    .title-desc { color: #666; }
  }
  &.level-green {
    background: linear-gradient(135deg, #07c160 0%, #059a41 100%);
    .title-emoji { font-size: 48rpx; }
    .title-name { color: #fff; font-size: 32rpx; font-weight: bold; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-lightgreen {
    background: linear-gradient(135deg, #52c41a 0%, #27ae60 100%);
    .title-emoji { font-size: 52rpx; }
    .title-name { color: #fff; font-size: 34rpx; font-weight: bold; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-cyan {
    background: linear-gradient(135deg, #00bcd4 0%, #0097a7 100%);
    .title-emoji { font-size: 56rpx; }
    .title-name { color: #fff; font-size: 36rpx; font-weight: bold; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-blue {
    background: linear-gradient(135deg, #2196f3 0%, #1976d2 100%);
    .title-emoji { font-size: 60rpx; }
    .title-name { color: #fff; font-size: 38rpx; font-weight: bold; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-gold {
    background: linear-gradient(135deg, #ffc107 0%, #ff9800 100%);
    box-shadow: 0 8rpx 24rpx rgba(255, 193, 7, 0.3);
    .title-emoji { font-size: 64rpx; }
    .title-name { color: #fff; font-size: 42rpx; font-weight: bold; text-shadow: 0 2rpx 4rpx rgba(0,0,0,0.2); }
    .title-desc { color: rgba(255,255,255,0.9); }
  }
}

.title-emoji {
  line-height: 1;
}

.title-name {
  display: block;
}

.title-desc {
  font-size: 24rpx;
  margin-top: 16rpx;
  text-align: center;
  display: block;
}

/* 灵签 */
.fortune-box {
  min-height: 200rpx;
}

.qian-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40rpx 0;
  transition: opacity 0.2s;

  &.disabled { opacity: 0.5; }
}

.qian-tong {
  width: 120rpx;
  height: 160rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s;
}

.qian-icon {
  font-size: 100rpx;
}

.qian-area:not(.disabled):active .qian-icon {
  transform: scale(0.9);
}

.shaking .qian-icon {
  animation: shake 0.3s ease-in-out infinite;
}

@keyframes shake {
  0%, 100% { transform: rotate(-10deg); }
  50% { transform: rotate(10deg); }
}

.qian-text {
  margin-top: 16rpx;
  font-size: 26rpx;
  color: #666;
}

.fortune-result {
  background: linear-gradient(135deg, #fffbf0 0%, #fff 100%);
  border-radius: 12rpx;
  padding: 28rpx;
  border: 1rpx solid #ffe0b0;
  animation: fadeIn 0.4s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(16rpx); }
  to { opacity: 1; transform: translateY(0); }
}

.fortune-header {
  display: flex;
  align-items: center;
  gap: 20rpx;
  margin-bottom: 16rpx;
}

.fortune-level {
  font-size: 28rpx;
  font-weight: bold;
  padding: 8rpx 20rpx;
  border-radius: 8rpx;

  &.level-best {
    color: #ff6b00;
    background: rgba(255, 107, 0, 0.1);
  }
  &.level-good {
    color: #e64340;
    background: rgba(230, 67, 64, 0.1);
  }
  &.level-medium {
    color: #ff9500;
    background: rgba(255, 149, 0, 0.1);
  }
  &.level-bad {
    color: #07c160;
    background: rgba(7, 193, 96, 0.1);
  }
}

.fortune-poem {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.fortune-desc {
  font-size: 26rpx;
  color: #666;
  line-height: 1.6;
}

/* 底部 */
.bottom-space {
  height: 120rpx;
}

.tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100rpx;
  background: #fff;
  display: flex;
  border-top: 1rpx solid #eee;
  padding-bottom: env(safe-area-inset-bottom);
}

.tab-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6rpx;

  &.active .tab-text {
    color: #e64340;
  }
}

.tab-icon {
  font-size: 40rpx;
}

.tab-text {
  font-size: 22rpx;
  color: #999;
}
</style>
