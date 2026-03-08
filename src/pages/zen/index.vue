<template>
  <view class="zen-page">
    <!-- 背景八卦装饰 -->
    <view class="bagua-bg">
      <text class="bagua-symbol">☯</text>
    </view>

    <!-- 整屏滚动区域 -->
    <scroll-view scroll-y class="scroll-container">
      <!-- 顶部安全间距 -->
      <view class="status-bar"></view>

      <!-- 顶部牌匾 -->
      <view class="header-section">
        <view class="plaque animate-fade-in">
          <text class="plaque-text">修真渡劫院</text>
        </view>
        <text class="sub-plaque">「 贫道夜观天象，今日诸事不宜 」</text>
      </view>

      <!-- 主道场 -->
      <view class="main-content">
        <!-- 1. 电子木鱼 -->
        <view class="talisman-card muyu-section" @tap="handleMuyuTap">
          <view class="corner tl"></view>
          <view class="corner tr"></view>
          <view class="corner bl"></view>
          <view class="corner br"></view>

          <view class="muyu-container">
            <text class="muyu-icon" :class="{ 'muyu-press': isPressing }">🐟</text>
            <!-- 功德飘字粒子 -->
            <view
              v-for="(particle, index) in meritParticles"
              :key="particle.id"
              class="merit-particle"
              :style="{ left: particle.x + 'px', top: particle.y + 'px' }"
            >
              功德+1
            </view>
          </view>

          <text class="merit-counter">功德：{{ merit }}</text>
          <text class="muyu-tip">点击敲击 · 积攒阳气</text>
        </view>

        <!-- 2. 历劫推演 -->
        <view class="talisman-card divination-box animate-fade-in-up" style="animation-delay: 0.1s">
          <view class="section-title">
            <text class="title-icon">☯</text>
            <text class="title-text">九九归一 · 历劫推演</text>
            <text class="title-icon">☯</text>
          </view>

          <view class="result-row">
            <view class="result-item">
              <text class="result-label">当前劫数</text>
              <text class="val-green">-{{ lossValue }}%</text>
            </view>
            <view class="result-item">
              <text class="result-label">飞升需涨</text>
              <text class="val-red">{{ gainDisplay }}</text>
            </view>
          </view>

          <slider
            class="zen-slider"
            :min="1"
            :max="99"
            :value="lossValue"
            :block-size="28"
            active-color="#D4AF37"
            background-color="#2B2B2B"
            @changing="onSliderChange"
          />

          <view class="oracle-box">
            <text class="oracle-title">{{ currentDivination.gua }}</text>
            <text class="oracle-content">{{ currentDivination.desc }}</text>
          </view>
        </view>

        <!-- 3. 灵签求运 -->
        <view class="talisman-card fortune-box animate-fade-in-up" style="animation-delay: 0.2s">
          <view class="corner tl"></view>
          <view class="corner tr"></view>
          <view class="corner bl"></view>
          <view class="corner br"></view>

          <view class="section-title">
            <text class="title-icon">🧧</text>
            <text class="title-text">灵签问吉凶</text>
            <text class="title-icon">🧧</text>
          </view>

          <view class="fortune-container">
            <!-- 签筒（未求签时显示） -->
            <view
              v-if="!showFortune"
              class="qiantong"
              :class="{ 'shake-anim': isShaking, 'disabled': fortuneUsed }"
              @tap="handleFortuneTap"
            >
              <text class="qiantong-icon">🎋</text>
            </view>
            <text class="qiantong-tip">{{ fortuneTip }}</text>

            <!-- 签纸 -->
            <view v-if="showFortune" class="fortune-paper animate-slide-down">
              <view class="fortune-header">
                <text class="fortune-level" :class="fortuneLevelClass">{{ currentFortune.level }}</text>
                <view class="fortune-info">
                  <text class="fortune-poem">{{ currentFortune.poem }}</text>
                  <text class="fortune-desc">{{ currentFortune.desc }}</text>
                </view>
              </view>
            </view>
          </view>
        </view>

        <!-- 底部间距 -->
        <view class="bottom-spacer"></view>
      </view>
    </scroll-view>

    <!-- 底部经文滚动 -->
    <view class="sutra-scroll">
      <view class="sutra-content" :style="{ animationPlayState: sutraPlayState }">
        <text>🙏 股市有风险，入市需谨慎 🙏 色即是空，空即是色，跌即是涨，涨即是跌 🙏 施主，回头是岸，苦海无边 🙏</text>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

// ============ 存储常量 ============
const STORAGE_KEY_MERIT = 'zen_merit_count';
const STORAGE_KEY_FORTUNE = 'zen_daily_fortune';

// 获取今天的日期字符串 YYYY-MM-DD
const getTodayStr = () => {
  const now = new Date();
  return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`;
};

// ============ 木鱼模块 ============
const merit = ref(0);
const isPressing = ref(false);
const meritParticles = ref<{ id: number; x: number; y: number }[]>([]);
let particleId = 0;

// 加载存储的功德值
const loadMerit = () => {
  try {
    const saved = uni.getStorageSync(STORAGE_KEY_MERIT);
    if (saved && !isNaN(Number(saved))) {
      merit.value = Number(saved);
    }
  } catch (e) {
    console.error('加载功德失败', e);
  }
};

// 保存功德值
const saveMerit = () => {
  try {
    uni.setStorageSync(STORAGE_KEY_MERIT, merit.value);
  } catch (e) {
    console.error('保存功德失败', e);
  }
};

const handleMuyuTap = () => {
  merit.value++;

  // 保存到本地存储
  saveMerit();

  // 视觉缩放
  isPressing.value = true;
  setTimeout(() => {
    isPressing.value = false;
  }, 80);

  // 触觉震动
  uni.vibrateShort({ type: 'light' });

  // 创建飘字粒子
  const randomX = (Math.random() - 0.5) * 60;
  const newParticle = {
    id: particleId++,
    x: 150 + randomX,
    y: 40
  };
  meritParticles.value.push(newParticle);

  // 移除粒子
  setTimeout(() => {
    meritParticles.value = meritParticles.value.filter(p => p.id !== newParticle.id);
  }, 800);
};

// ============ 历劫推演模块 ============
const lossValue = ref(10);

const divinations = [
  { limit: 10, gua: "卦象：小耗", desc: "皮外伤，少喝两杯奶茶可解。" },
  { limit: 20, gua: "卦象：坎卦", desc: "行路难，切忌妄动，闭关锁仓。" },
  { limit: 30, gua: "卦象：困卦", desc: "深陷泥潭，唯有装死一法可保全。" },
  { limit: 40, gua: "卦象：剥卦", desc: "剥皮削骨，不如去深山老林喊两嗓子。" },
  { limit: 50, gua: "卦象：大过", desc: "天劫腰斩，想要回本难如登天。" },
  { limit: 60, gua: "卦象：损卦", desc: "损下益上，你的钱都补了庄家的天。" },
  { limit: 80, gua: "卦象：明夷", desc: "暗无天日，否极泰来在下个世纪。" },
  { limit: 99, gua: "卦象：涅槃", desc: "九九归一，建议销户重修，立地成佛。" }
];

const gainDisplay = computed(() => {
  const gain = (lossValue.value / (100 - lossValue.value)) * 100;
  return gain > 9900 ? "∞" : Math.round(gain) + "%";
});

const currentDivination = computed(() => {
  let current = divinations[divinations.length - 1];
  for (const item of divinations) {
    if (lossValue.value <= item.limit) {
      current = item;
      break;
    }
  }
  return current;
});

const onSliderChange = (e: any) => {
  lossValue.value = e.detail.value;
};

// ============ 灵签模块 ============
const isShaking = ref(false);
const showFortune = ref(false);
const fortuneTip = ref("点击签筒，摇动求签");
const fortuneUsed = ref(false); // 今天是否已求签
const currentFortune = ref({ level: "", poem: "", desc: "" });

const fortuneData = [
  { level: "上上签", poem: "枯木逢春", desc: "枯木逢春犹再发，施主手中废纸或将变废为宝，切记拿住，莫要下车。" },
  { level: "上吉", poem: "紫气东来", desc: "主力资金隐隐若现，似有抬轿之意，这波能不能飞升，全看造化。" },
  { level: "中平", poem: "静观其变", desc: "敌不动我不动，敌若动我乱动。目前局势不明，建议施主回家睡觉。" },
  { level: "下下签", poem: "天雷滚滚", desc: "大凶之兆！印堂发黑，近日忌看盘，忌补仓，宜卸载软件保平安。" },
  { level: "下签", poem: "关灯吃面", desc: "今日大概率又是随礼的一天，记得多放葱花，少放眼泪。" },
  { level: "中吉", poem: "否极泰来", desc: "跌无可跌，也就是涨的开始。虽然现在很难受，但万一反弹了呢？" },
  { level: "神签", poem: "相信国运", desc: "别问，问就是相信光。只要你活得够久，总能等到回本的那一天。" }
];

const fortuneLevelClass = computed(() => {
  const level = currentFortune.value.level;
  if (level === '上上签' || level === '神签') return 'level-best';
  if (level === '上吉' || level === '中吉') return 'level-good';
  if (level === '中平') return 'level-medium';
  return 'level-bad';
});

// 加载今日签文
const loadTodayFortune = () => {
  try {
    const saved = uni.getStorageSync(STORAGE_KEY_FORTUNE);
    if (saved) {
      const data = JSON.parse(saved);
      const today = getTodayStr();

      // 检查是否是今天的签
      if (data.date === today && data.fortune) {
        currentFortune.value = data.fortune;
        showFortune.value = true;
        fortuneUsed.value = true;
        fortuneTip.value = "今日已求签，明日再来";
      }
    }
  } catch (e) {
    console.error('加载签文失败', e);
  }
};

// 保存今日签文
const saveTodayFortune = (fortune: typeof currentFortune.value) => {
  try {
    const data = {
      date: getTodayStr(),
      fortune: fortune
    };
    uni.setStorageSync(STORAGE_KEY_FORTUNE, JSON.stringify(data));
  } catch (e) {
    console.error('保存签文失败', e);
  }
};

const handleFortuneTap = () => {
  // 今天已求签则不再允许
  if (fortuneUsed.value || isShaking.value) return;

  isShaking.value = true;
  fortuneTip.value = "诚心祷告，灵签摇动中...";

  // 震动反馈
  uni.vibrateShort({ type: 'medium' });

  setTimeout(() => {
    uni.vibrateShort({ type: 'medium' });
  }, 150);

  setTimeout(() => {
    uni.vibrateShort({ type: 'medium' });
  }, 300);

  setTimeout(() => {
    isShaking.value = false;
    const fortune = fortuneData[Math.floor(Math.random() * fortuneData.length)];
    currentFortune.value = fortune;
    showFortune.value = true;
    fortuneUsed.value = true;
    fortuneTip.value = "今日已求签，明日再来";

    // 保存到本地
    saveTodayFortune(fortune);

    uni.vibrateShort({ type: 'heavy' });
  }, 1500);
};

// 底部经文动画状态
const sutraPlayState = ref('running');

// 初始化
onMounted(() => {
  loadMerit();
  loadTodayFortune();
});
</script>

<style lang="scss" scoped>
.zen-page {
  min-height: 100vh;
  background-color: #F0E6D2;
  background-image: radial-gradient(#E6DBC4 1px, transparent 1px);
  background-size: 20rpx 20rpx;
  position: relative;
  overflow: hidden;
  font-family: 'KaiTi', 'STKaiti', 'SimSun', serif;
}

/* 背景八卦 */
.bagua-bg {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 600rpx;
  height: 600rpx;
  border: 40rpx solid rgba(0, 0, 0, 0.03);
  border-radius: 50%;
  z-index: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bagua-symbol {
  font-size: 300rpx;
  opacity: 0.05;
  animation: rotateBagua 60s linear infinite;
}

@keyframes rotateBagua {
  100% { transform: rotate(360deg); }
}

/* 顶部安全间距 */
.status-bar {
  height: 80rpx;
}

/* 顶部牌匾 */
.header-section {
  padding: 20rpx 40rpx;
  text-align: center;
  position: relative;
  z-index: 1;
}

.plaque {
  display: inline-block;
  padding: 16rpx 48rpx;
  background: linear-gradient(135deg, #C44238 0%, #A33A30 100%);
  border: 8rpx double #2B2B2B;
  box-shadow: 8rpx 8rpx 0 rgba(43, 43, 43, 0.2);
  border-radius: 8rpx;
  position: relative;
  overflow: hidden;

  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 215, 0, 0.3), transparent);
    animation: shimmerPlaque 3s ease-in-out infinite;
  }
}

@keyframes shimmerPlaque {
  0%, 100% { left: -100%; }
  50% { left: 100%; }
}

.plaque-text {
  font-size: 48rpx;
  font-weight: bold;
  color: #FFD700;
  letter-spacing: 12rpx;
  text-shadow: 2rpx 2rpx 4rpx rgba(0, 0, 0, 0.3);
}

.sub-plaque {
  display: block;
  margin-top: 20rpx;
  font-size: 28rpx;
  color: #2B2B2B;
  opacity: 0.7;
  font-style: italic;
}

/* 整屏滚动容器 */
.scroll-container {
  height: 100vh;
  position: relative;
  z-index: 1;
}

/* 主内容区 */
.main-content {
  padding: 40rpx;
  padding-bottom: 180rpx; /* 底部经文高度 */
}

/* 符箓卡片通用样式 */
.talisman-card {
  background: linear-gradient(145deg, #FFFBF0 0%, #FDF8E8 100%);
  border: 4rpx solid #2B2B2B;
  padding: 40rpx;
  position: relative;
  box-shadow: 8rpx 8rpx 0 rgba(43, 43, 43, 0.15);
  margin-bottom: 40rpx;
  border-radius: 8rpx;
}

/* 四角装饰 */
.corner {
  position: absolute;
  width: 20rpx;
  height: 20rpx;
  border: 4rpx solid #C44238;

  &.tl { top: 8rpx; left: 8rpx; border-right: none; border-bottom: none; }
  &.tr { top: 8rpx; right: 8rpx; border-left: none; border-bottom: none; }
  &.bl { bottom: 8rpx; left: 8rpx; border-right: none; border-top: none; }
  &.br { bottom: 8rpx; right: 8rpx; border-left: none; border-top: none; }
}

/* ============ 木鱼模块 ============ */
.muyu-section {
  text-align: center;
  padding: 60rpx 40rpx;
  background: radial-gradient(circle, #EFE5D0 0%, #FFFBF0 100%);
  cursor: pointer;
}

.muyu-container {
  position: relative;
  display: inline-block;
  height: 200rpx;
  width: 200rpx;
}

.muyu-icon {
  font-size: 160rpx;
  transition: transform 0.08s ease-out;
  filter: drop-shadow(8rpx 8rpx 0 rgba(0, 0, 0, 0.1));
}

.muyu-press {
  transform: scale(0.85);
}

.merit-particle {
  position: absolute;
  color: #C44238;
  font-weight: bold;
  font-size: 36rpx;
  pointer-events: none;
  animation: floatUpFade 0.8s ease-out forwards;
  z-index: 100;
  text-shadow: 0 2rpx 4rpx rgba(196, 66, 56, 0.3);
}

@keyframes floatUpFade {
  0% { transform: translate(-50%, 0) scale(0.5); opacity: 0; }
  20% { transform: translate(-50%, -40rpx) scale(1.2); opacity: 1; }
  100% { transform: translate(-50%, -160rpx) scale(1); opacity: 0; }
}

.merit-counter {
  display: block;
  font-size: 32rpx;
  margin-top: 20rpx;
  font-weight: bold;
  color: #C44238;
  border-bottom: 2rpx solid #C44238;
  padding-bottom: 8rpx;
}

.muyu-tip {
  display: block;
  font-size: 24rpx;
  margin-top: 16rpx;
  opacity: 0.6;
  color: #2B2B2B;
}

/* ============ 历劫推演模块 ============ */
.section-title {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16rpx;
  margin-bottom: 30rpx;
  padding-bottom: 16rpx;
  border-bottom: 2rpx dashed #2B2B2B;
}

.title-icon {
  font-size: 32rpx;
}

.title-text {
  font-size: 36rpx;
  font-weight: bold;
  letter-spacing: 4rpx;
  color: #2B2B2B;
}

.result-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30rpx;
}

.result-item {
  text-align: center;
}

.result-label {
  display: block;
  font-size: 24rpx;
  opacity: 0.6;
  margin-bottom: 8rpx;
  color: #2B2B2B;
}

.val-red {
  font-size: 40rpx;
  font-weight: bold;
  color: #C44238;
}

.val-green {
  font-size: 40rpx;
  font-weight: bold;
  color: #3E6B56;
}

.zen-slider {
  margin: 30rpx 0;
}

.oracle-box {
  background: linear-gradient(135deg, rgba(196, 66, 56, 0.05) 0%, rgba(196, 66, 56, 0.02) 100%);
  padding: 30rpx;
  border: 2rpx solid #C44238;
  border-radius: 8rpx;
  margin-top: 20rpx;
}

.oracle-title {
  display: block;
  font-size: 30rpx;
  font-weight: bold;
  color: #C44238;
  margin-bottom: 16rpx;
}

.oracle-content {
  font-size: 28rpx;
  color: #2B2B2B;
  line-height: 1.6;
}

/* ============ 灵签模块 ============ */
.fortune-container {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.qiantong {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 160rpx;
  height: 200rpx;
  cursor: pointer;
  transition: transform 0.2s, opacity 0.2s;

  &.disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }
}

.qiantong-icon {
  font-size: 160rpx;
}

.qiantong:active:not(.disabled) {
  transform: scale(0.95);
}

.shake-anim {
  animation: shakeHard 0.5s cubic-bezier(.36, .07, .19, .97) infinite;
}

@keyframes shakeHard {
  10%, 90% { transform: translate3d(-4rpx, 0, 0) rotate(-5deg); }
  20%, 80% { transform: translate3d(8rpx, 0, 0) rotate(5deg); }
  30%, 50%, 70% { transform: translate3d(-16rpx, 0, 0) rotate(-10deg); }
  40%, 60% { transform: translate3d(16rpx, 0, 0) rotate(10deg); }
}

.qiantong-tip {
  font-size: 24rpx;
  margin-top: 20rpx;
  margin-bottom: 20rpx;
  color: #666;
}

.fortune-paper {
  width: 100%;
  background: linear-gradient(145deg, #FFFFFF 0%, #FFF8E7 100%);
  border: 4rpx solid #C44238;
  box-shadow: 0 10rpx 30rpx rgba(0, 0, 0, 0.1);
  padding: 30rpx;
  border-radius: 8rpx;
  box-sizing: border-box;
}

.fortune-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20rpx;
}

.fortune-level {
  font-size: 44rpx;
  font-weight: bold;
  padding: 16rpx 40rpx;
  border: 4rpx double #C44238;
  border-radius: 8rpx;
  letter-spacing: 8rpx;
  text-align: center;

  &.level-best {
    color: #D4AF37;
    background: linear-gradient(135deg, rgba(212, 175, 55, 0.15) 0%, transparent 100%);
    border-color: #D4AF37;
  }
  &.level-good {
    color: #3E6B56;
    background: linear-gradient(135deg, rgba(62, 107, 86, 0.15) 0%, transparent 100%);
    border-color: #3E6B56;
  }
  &.level-medium {
    color: #8B7355;
    background: linear-gradient(135deg, rgba(139, 115, 85, 0.15) 0%, transparent 100%);
    border-color: #8B7355;
  }
  &.level-bad {
    color: #C44238;
    background: linear-gradient(135deg, rgba(196, 66, 56, 0.15) 0%, transparent 100%);
    border-color: #C44238;
  }
}

.fortune-info {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 16rpx;
}

.fortune-poem {
  font-size: 36rpx;
  font-weight: bold;
  color: #2B2B2B;
  text-align: center;
}

.fortune-desc {
  font-size: 26rpx;
  color: #666;
  line-height: 1.6;
  border-top: 2rpx dashed #ccc;
  padding-top: 20rpx;
  text-align: left;
}

/* ============ 底部经文 ============ */
.sutra-scroll {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  background: linear-gradient(135deg, #2B2B2B 0%, #1a1a1a 100%);
  padding: 16rpx 0;
  border-top: 4rpx solid #D4AF37;
  z-index: 100;
  overflow: hidden;
}

.sutra-content {
  display: inline-block;
  white-space: nowrap;
  animation: scrollText 20s linear infinite;
  color: #D4AF37;
  font-size: 24rpx;
}

@keyframes scrollText {
  0% { transform: translateX(100%); }
  100% { transform: translateX(-100%); }
}

.bottom-spacer {
  height: 40rpx;
}

/* ============ 动画效果 ============ */
.animate-fade-in {
  animation: fadeIn 0.6s ease-out forwards;
}

.animate-fade-in-up {
  animation: fadeInUp 0.6s ease-out forwards;
  opacity: 0;
}

.animate-slide-down {
  animation: slideDown 0.5s ease-out forwards;
}

@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}

@keyframes fadeInUp {
  0% { opacity: 0; transform: translateY(40rpx); }
  100% { opacity: 1; transform: translateY(0); }
}

@keyframes slideDown {
  from { opacity: 0; transform: translateY(-40rpx); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
