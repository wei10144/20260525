<template>
  <div class="home-page">
    <div class="pixel-box top-panel">
      <h2>🎉 歡迎來到討龍英文趣 🎉</h2>
      <p>準備好展開一場充滿魔法與美食的英語冒險了嗎？</p>
    </div>

    <div class="main-content">
      <div class="pixel-box left-panel">
        <h3>📜 遊玩規則</h3>
        <ol>
          <li>先前往<strong>【📖 單字秘笈】</strong>熟讀單字，並完成試驗前小訓練。</li>
          <li>小訓練全數通關後，即可解鎖<strong>【🐉 龍之試驗】</strong>。</li>
          <li>在龍之試驗中挑戰魔物，答對施放魔法，打倒魔王可獲得 <strong>1 積分</strong>。</li>
          <li>帶著你的積分前往<strong>【🎁 兌換獎品】</strong>，找老師換取真實的美味獎勵！</li>
        </ol>
      </div>

      <div class="pixel-box right-panel">
        <h3>🎯 教學目標</h3>
        <ul>
          <li><strong>遊戲化學習：</strong>將單字記憶與打怪結合，提升學習動機與專注力。</li>
          <li><strong>多感官刺激：</strong>結合圖像、發音與拼字互動，強化大腦對英文的直覺反應。</li>
          <li><strong>實質回饋：</strong>透過虛擬積分兌換實體獎勵，讓學習成就感具象化，英文不再枯燥！</li>
        </ul>
        <div class="action-area">
          <button class="pixel-btn start-btn" @click="goToVocab">前往單字秘笈 ➡️</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const emit = defineEmits(['go-vocab'])

// 點擊前往單字秘笈，順便播放點擊音效(音效在 App.vue 會處理)
const goToVocab = () => {
  emit('go-vocab')
}
</script>

<style scoped>
.home-page {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.pixel-box {
  background-color: #fff;
  border: 5px solid #6F1D1B;
  box-shadow: 6px 6px 0px #000;
  padding: 30px;
}

.top-panel {
  text-align: center;
  background-color: #f5d6a7;
}

.top-panel h2 {
  color: #d35400;
  font-size: 32px;
  margin-top: 0;
  margin-bottom: 10px;
  text-shadow: 1px 1px 0px #fff;
}

.top-panel p {
  font-size: 20px;
  color: #6F1D1B;
  font-weight: bold;
  margin: 0;
}

.main-content {
  display: flex;
  gap: 20px;
  flex: 1;
}

.left-panel, .right-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
}

h3 {
  color: #f1c40f;
  background-color: #6F1D1B;
  padding: 10px 15px;
  margin-top: 0;
  border: 3px solid #4a1211;
  box-shadow: 3px 3px 0 #000;
  text-align: center;
  font-size: 24px;
}

ol, ul {
  font-size: 18px;
  line-height: 1.8;
  color: #2c3e50;
  padding-left: 25px;
  margin-bottom: 20px;
}

li {
  margin-bottom: 15px;
  border-bottom: 2px dashed #f2d1cf;
  padding-bottom: 10px;
}

strong {
  color: #c0392b;
}

.action-area {
  margin-top: auto; /* 將按鈕推到最下方 */
  display: flex;
  justify-content: flex-end; /* 靠右對齊 */
}

.start-btn {
  background-color: #27ae60;
  font-size: 20px;
  animation: pulse 1.5s infinite;
}

.start-btn:hover {
  background-color: #2ecc71;
  transform: scale(1.05);
}

@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(39, 174, 96, 0.7); }
  70% { box-shadow: 0 0 0 15px rgba(39, 174, 96, 0); }
  100% { box-shadow: 0 0 0 0 rgba(39, 174, 96, 0); }
}

@media (max-width: 900px) {
  .main-content {
    flex-direction: column;
  }
}
</style>