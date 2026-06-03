<template>
  <view class="app-page">
    <!-- 八卦背景装饰 -->
    <view class="bagua-bg">
      <text class="bagua-icon">☯</text>
    </view>

    <!-- 主内容区 - 整屏滚动 -->
    <scroll-view scroll-y class="main-scroll">
      <!-- 顶部牌匾 -->
      <view class="header">
        <view class="plaque">修真渡劫院</view>
        <text class="sub-plaque">「 贫道夜观天象，今日诸事不宜 」</text>
      </view>
      <!-- 1. 功德木鱼 -->
      <view class="talisman-card muyu-section" @tap="handleMuyuTap">
        <view class="corner tl"></view>
        <view class="corner tr"></view>
        <view class="corner bl"></view>
        <view class="corner br"></view>

        <view class="merit-header">
          <view class="merit-info">
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
        <view v-for="p in meritParticles" :key="p.id" class="merit-particle" :style="{ left: p.x + 'px', top: p.y + 'px' }">
          功德+1
        </view>
        <text class="tap-hint">点击敲击 · 积攒阳气</text>
      </view>

      <!-- 2. 历劫推演 (回本计算器) -->
      <view class="talisman-card">
        <view class="card-title">☯ 九九归一 · 历劫推演 ☯</view>

        <view class="result-row">
          <view class="result-item">
            <text class="result-label">当前劫数</text>
            <text class="result-val val-green">-{{ lossValue }}%</text>
          </view>
          <view class="result-item">
            <text class="result-label">飞升需涨</text>
            <text class="result-val val-red">{{ gainDisplay }}</text>
          </view>
        </view>

        <view class="slider-wrap">
          <slider
            :min="1"
            :max="99"
            :value="lossValue"
            :block-size="24"
            active-color="#D4AF37"
            background-color="#2B2B2B"
            @changing="onSliderChange"
          />
        </view>

        <view class="oracle-box">
          <text class="oracle-title">卦象：{{ currentDivination.gua }}</text>
          <text class="oracle-content">{{ currentDivination.desc }}</text>
        </view>
      </view>

      <!-- 3. 收益波动称号 -->
      <view class="talisman-card">
        <view class="card-title">🏆 收益波动称号</view>

        <view class="profit-labels">
          <text class="profit-label val-green">亏损</text>
          <text class="profit-label val-red">盈利</text>
        </view>

        <view class="slider-wrap">
          <slider
            :min="-99"
            :max="99"
            :value="profitValue"
            :block-size="24"
            active-color="#D4AF37"
            background-color="#2B2B2B"
            @changing="onProfitSliderChange"
          />
        </view>

        <view class="title-result" :class="'level-' + currentTitle.level">
          <view class="title-badge">
            <text class="title-emoji">{{ currentTitle.emoji }}</text>
            <text class="title-name">{{ currentTitle.name }}</text>
          </view>
          <text class="title-desc">{{ currentTitle.desc }}</text>
        </view>
      </view>

      <!-- 4. 灵签问吉凶 -->
      <view class="talisman-card">
        <view class="corner tl"></view>
        <view class="corner tr"></view>
        <view class="corner bl"></view>
        <view class="corner br"></view>

        <view class="card-title">🧧 灵签问吉凶 🧧</view>

        <view class="fortune-container">
          <!-- 签筒 -->
          <view
            v-if="!showFortune"
            class="qian-area"
            :class="{ 'shaking': isShaking, 'disabled': fortuneUsed }"
            @tap="handleFortuneTap"
          >
            <text class="qiantong">🎋</text>
            <text class="qiantong-tip">{{ isShaking ? '诚心祷告，灵签摇动中...' : (fortuneUsed ? '今日已求签' : '点击签筒，摇动求签') }}</text>
          </view>

          <!-- 签文 -->
          <view v-if="showFortune" class="fortune-paper">
            <view class="fortune-level" :class="fortuneLevelClass">{{ currentFortune.level }}</view>
            <view class="fortune-content">
              <text class="fortune-poem">{{ currentFortune.poem }}</text>
              <text class="fortune-desc">{{ currentFortune.desc }}</text>
            </view>
          </view>
        </view>
      </view>

      <!-- 底部留白 -->
      <view class="bottom-space"></view>
    </scroll-view>

    <!-- 底部经文 -->
    <view class="sutra-scroll">
      <text class="sutra-content">🙏 股市有风险，入市需谨慎 🙏 色即是空，空即是色，跌即是涨，涨即是跌 🙏 施主，回头是岸，苦海无边 🙏</text>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { onShareAppMessage, onShareTimeline } from '@dcloudio/uni-app';

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
    x: 160 + (Math.random() - 0.5) * 80,
    y: 80
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
  { level: "上上签", poem: "枯木逢春", desc: "枯木逢春犹再发，施主手中废纸或将变废为宝，切记拿住，莫要下车。" },
  { level: "上吉", poem: "紫气东来", desc: "主力资金隐隐若现，似有抬轿之意，这波能不能飞升，全看造化。" },
  { level: "中吉", poem: "否极泰来", desc: "跌无可跌，也就是涨的开始。虽然现在很难受，但万一反弹了呢？" },
  { level: "中平", poem: "静观其变", desc: "敌不动我不动，敌若动我乱动。目前局势不明，建议施主回家睡觉。" },
  { level: "下签", poem: "关灯吃面", desc: "今日大概率又是随礼的一天，记得多放葱花，少放眼泪。" },
  { level: "下下签", poem: "天雷滚滚", desc: "大凶之兆！印堂发黑，近日忌看盘，忌补仓，宜卸载软件保平安。" },
  { level: "神签", poem: "相信国运", desc: "别问，问就是相信光。只要你活得够久，总能等到回本的那一天。" }
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
  }, 1500);
};

onMounted(() => {
  loadMerit();
  loadTodayFortune();
});

// ============ 分享配置 ============
// 分享给朋友
onShareAppMessage(() => {
  return {
    title: '修真渡劫院 - 今日诸事不宜',
    path: '/pages/zen/index',
    imageUrl: '/static/logo.png'
  };
});

// 分享到朋友圈
onShareTimeline(() => {
  return {
    title: '修真渡劫院 - 贫道夜观天象，今日诸事不宜',
    query: '',
    imageUrl: '/static/logo.png'
  };
});
</script>

<style lang="scss" scoped>
/* 玄学色板 */
$paper-bg: #F0E6D2;
$paper-texture: #E6DBC4;
$cinnabar: #C44238;
$ink-black: #2B2B2B;
$tao-gold: #D4AF37;
$jade-green: #3E6B56;

.app-page {
  min-height: 100vh;
  height: 100vh;
  background-color: $paper-bg;
  background-image: radial-gradient($paper-texture 1px, transparent 1px);
  background-size: 20rpx 20rpx;
  position: relative;
  font-family: "KaiTi", "STKaiti", "SimSun", serif;
}

/* 八卦背景装饰 */
.bagua-bg {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 600rpx;
  height: 600rpx;
  border: 40rpx solid rgba(0,0,0,0.03);
  border-radius: 50%;
  z-index: 0;
  pointer-events: none;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bagua-icon {
  font-size: 300rpx;
  opacity: 0.05;
  animation: rotate 60s linear infinite;
}

@keyframes rotate {
  100% { transform: rotate(360deg); }
}

/* 顶部牌匾 */
.header {
  padding: 120rpx 40rpx 40rpx;
  text-align: center;
  position: relative;
  z-index: 1;
}

.plaque {
  display: inline-block;
  border: 8rpx double $ink-black;
  padding: 16rpx 48rpx;
  background: $cinnabar;
  color: #FFD700;
  font-size: 48rpx;
  font-weight: bold;
  box-shadow: 8rpx 8rpx 0 rgba(43, 43, 43, 0.2);
  letter-spacing: 8rpx;
  border-radius: 8rpx;
}

.sub-plaque {
  display: block;
  margin-top: 20rpx;
  font-size: 26rpx;
  color: $ink-black;
  opacity: 0.7;
  font-style: italic;
}

/* 滚动区 */
.main-scroll {
  height: calc(100vh - 80rpx);
  position: relative;
  z-index: 1;
}

/* 通用符箓卡片 */
.talisman-card {
  background: #FFFBF0;
  border: 4rpx solid $ink-black;
  padding: 40rpx;
  margin: 32rpx;
  position: relative;
  box-shadow: 8rpx 8rpx 0 rgba(43, 43, 43, 0.2);
}

/* 四角装饰 */
.corner {
  position: absolute;
  width: 20rpx;
  height: 20rpx;
  border: 4rpx solid $cinnabar;
}
.tl { top: 8rpx; left: 8rpx; border-right: none; border-bottom: none; }
.tr { top: 8rpx; right: 8rpx; border-left: none; border-bottom: none; }
.bl { bottom: 8rpx; left: 8rpx; border-right: none; border-top: none; }
.br { bottom: 8rpx; right: 8rpx; border-left: none; border-top: none; }

.card-title {
  font-size: 36rpx;
  text-align: center;
  border-bottom: 2rpx dashed $ink-black;
  padding-bottom: 16rpx;
  margin-bottom: 32rpx;
  letter-spacing: 4rpx;
  font-weight: bold;
}

/* 功德木鱼 */
.muyu-section {
  text-align: center;
  padding: 40rpx;
  background: radial-gradient(circle, #EFE5D0 0%, #FFFBF0 100%);
  cursor: pointer;
  overflow: hidden;
}

.merit-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: relative;
  z-index: 1;
}

.merit-info {
  flex: 1;
  text-align: left;
}

.merit-label {
  font-size: 28rpx;
  color: $ink-black;
  opacity: 0.7;
  display: block;
  margin-bottom: 8rpx;
}

.merit-value-row {
  display: flex;
  align-items: baseline;
  gap: 8rpx;
}

.merit-value {
  font-size: 72rpx;
  font-weight: bold;
  color: $cinnabar;
  font-family: "Arial", sans-serif;
}

.merit-unit {
  font-size: 28rpx;
  color: $cinnabar;
}

.cat-muyu-img {
  width: 180rpx;
  height: 180rpx;
  transition: transform 0.08s;

  &.pressing {
    transform: scale(0.9);
  }
}

/* 功德+1 粒子动画 */
.merit-particle {
  position: absolute;
  color: $cinnabar;
  font-weight: bold;
  font-size: 40rpx;
  pointer-events: none;
  animation: floatUpFade 0.8s ease-out forwards;
  z-index: 100;
}

@keyframes floatUpFade {
  0% { transform: translate(-50%, 0) scale(0.5); opacity: 0; }
  20% { transform: translate(-50%, -40rpx) scale(1.2); opacity: 1; }
  100% { transform: translate(-50%, -160rpx) scale(1); opacity: 0; }
}

.tap-hint {
  display: block;
  font-size: 24rpx;
  margin-top: 24rpx;
  opacity: 0.6;
  color: $ink-black;
}

/* 历劫推演 */
.result-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 24rpx;
}

.result-item {
  text-align: center;
}

.result-label {
  display: block;
  font-size: 24rpx;
  opacity: 0.6;
  margin-bottom: 8rpx;
  color: $ink-black;
}

.result-val {
  font-size: 40rpx;
  font-weight: bold;
  font-family: "Arial", sans-serif;
}

.val-red { color: $cinnabar; }
.val-green { color: $jade-green; }

.slider-wrap {
  margin: 24rpx 0;
}

.oracle-box {
  background: rgba(196, 66, 56, 0.05);
  border: 2rpx solid $cinnabar;
  padding: 24rpx;
  margin-top: 24rpx;
}

.oracle-title {
  display: block;
  font-size: 30rpx;
  font-weight: bold;
  color: $cinnabar;
  margin-bottom: 12rpx;
}

.oracle-content {
  font-size: 28rpx;
  color: $ink-black;
  line-height: 1.6;
}

/* 收益波动称号 */
.profit-labels {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8rpx;
}

.profit-label {
  font-size: 26rpx;
  font-weight: bold;
}

.title-result {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 32rpx;
  padding: 32rpx;
  border-radius: 12rpx;
  border: 2rpx solid;

  &.level-purple {
    background: linear-gradient(135deg, #9b59b6 0%, #8e44ad 100%);
    border-color: #8e44ad;
    .title-emoji { font-size: 72rpx; }
    .title-name { color: #fff; font-size: 44rpx; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-red {
    background: linear-gradient(135deg, #e64340 0%, #c9382e 100%);
    border-color: #c9382e;
    .title-emoji { font-size: 68rpx; }
    .title-name { color: #fff; font-size: 42rpx; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-orange {
    background: linear-gradient(135deg, #ff9500 0%, #ff6b00 100%);
    border-color: #ff6b00;
    .title-emoji { font-size: 64rpx; }
    .title-name { color: #fff; font-size: 40rpx; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-yellow {
    background: linear-gradient(135deg, #fbbf24 0%, #f59e0b 100%);
    border-color: #f59e0b;
    .title-emoji { font-size: 60rpx; }
    .title-name { color: #333; font-size: 38rpx; }
    .title-desc { color: #666; }
  }
  &.level-green {
    background: linear-gradient(135deg, #07c160 0%, #059a41 100%);
    border-color: #059a41;
    .title-emoji { font-size: 56rpx; }
    .title-name { color: #fff; font-size: 36rpx; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-lightgreen {
    background: linear-gradient(135deg, #52c41a 0%, #27ae60 100%);
    border-color: #27ae60;
    .title-emoji { font-size: 60rpx; }
    .title-name { color: #fff; font-size: 38rpx; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-cyan {
    background: linear-gradient(135deg, #00bcd4 0%, #0097a7 100%);
    border-color: #0097a7;
    .title-emoji { font-size: 64rpx; }
    .title-name { color: #fff; font-size: 40rpx; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-blue {
    background: linear-gradient(135deg, #2196f3 0%, #1976d2 100%);
    border-color: #1976d2;
    .title-emoji { font-size: 68rpx; }
    .title-name { color: #fff; font-size: 42rpx; }
    .title-desc { color: rgba(255,255,255,0.85); }
  }
  &.level-gold {
    background: linear-gradient(135deg, #ffc107 0%, #ff9800 100%);
    border-color: #ff9800;
    box-shadow: 0 8rpx 24rpx rgba(255, 193, 7, 0.3);
    .title-emoji { font-size: 72rpx; }
    .title-name { color: #fff; font-size: 46rpx; text-shadow: 0 2rpx 4rpx rgba(0,0,0,0.2); }
    .title-desc { color: rgba(255,255,255,0.9); }
  }
}

.title-badge {
  display: flex;
  align-items: center;
  gap: 16rpx;
}

.title-emoji {
  line-height: 1;
}

.title-name {
  font-weight: bold;
}

.title-desc {
  font-size: 24rpx;
  margin-top: 16rpx;
  text-align: center;
}

/* 灵签 */
.fortune-container {
  min-height: 280rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.qian-area {
  display: flex;
  flex-direction: column;
  align-items: center;

  &.disabled { opacity: 0.5; }
}

.qiantong {
  font-size: 160rpx;
  transition: transform 0.2s;
}

.qian-area:not(.disabled):active .qiantong {
  transform: scale(0.9);
}

.shaking .qiantong {
  animation: shakeHard 0.3s ease-in-out infinite;
}

@keyframes shakeHard {
  10%, 90% { transform: translateX(-4rpx) rotate(-5deg); }
  20%, 80% { transform: translateX(8rpx) rotate(5deg); }
  30%, 50%, 70% { transform: translateX(-16rpx) rotate(-10deg); }
  40%, 60% { transform: translateX(16rpx) rotate(10deg); }
}

.qiantong-tip {
  margin-top: 16rpx;
  font-size: 24rpx;
  color: #666;
}

.fortune-paper {
  width: 100%;
  background: #FFF;
  border: 4rpx solid $cinnabar;
  box-shadow: 0 10rpx 30rpx rgba(0,0,0,0.1);
  padding: 24rpx;
  animation: slideDown 0.5s ease-out;
  display: flex;
  flex-direction: row;
}

@keyframes slideDown {
  from { opacity: 0; transform: translateY(-40rpx); }
  to { opacity: 1; transform: translateY(0); }
}

.fortune-level {
  writing-mode: vertical-rl;
  font-size: 36rpx;
  font-weight: bold;
  border-left: 4rpx double $cinnabar;
  padding-left: 16rpx;
  height: 160rpx;
  margin-left: 20rpx;
  letter-spacing: 8rpx;

  &.level-best { color: #ff6b00; }
  &.level-good { color: $cinnabar; }
  &.level-medium { color: #ff9500; }
  &.level-bad { color: $jade-green; }
}

.fortune-content {
  flex: 1;
}

.fortune-poem {
  display: block;
  font-size: 32rpx;
  font-weight: bold;
  color: $ink-black;
  margin-bottom: 16rpx;
}

.fortune-desc {
  font-size: 26rpx;
  color: #666;
  line-height: 1.6;
  border-top: 2rpx dashed #ccc;
  padding-top: 16rpx;
}

/* 底部 */
.bottom-space {
  height: 200rpx;
}

/* 底部经文 */
.sutra-scroll {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  background: $ink-black;
  color: $tao-gold;
  padding: 16rpx 0;
  font-size: 24rpx;
  white-space: nowrap;
  overflow: hidden;
  border-top: 4rpx solid $tao-gold;
  z-index: 10;
}

.sutra-content {
  display: inline-block;
  animation: scrollText 20s linear infinite;
}

@keyframes scrollText {
  0% { transform: translateX(100%); }
  100% { transform: translateX(-100%); }
}
</style>
