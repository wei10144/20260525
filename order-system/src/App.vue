<template>
  <div class="app-container" @click.once="initBGM">
    <FlyingDragons />
    
    <RegisterModal v-if="!isRegistered" @registered="handleRegister" />
    
    <Leaderboard v-if="showLeaderboard" @close="showLeaderboard = false" :my-seat="mySeat" />

    <div class="nav-trigger-zone" @mouseenter="isNavOpen = true"></div>

    <transition name="slide-nav">
      <nav v-if="isNavOpen" class="navbar" @mouseleave="isNavOpen = false">
        <div class="logo" @click="switchTab('home')" @mouseenter="playHoverSound">⚔️ 討龍英文趣</div>
        
        <div class="nav-links">
          <button class="nav-btn" :class="{ active: currentTab === 'home' }" @click="switchTab('home')" @mouseenter="playHoverSound">🏠 遊戲首頁</button>
          <button class="nav-btn" :class="{ active: currentTab === 'vocab' }" @click="switchTab('vocab')" @mouseenter="playHoverSound">📖 單字秘笈</button>
          <button class="nav-btn" :class="{ active: currentTab === 'battle', locked: !isVocabTrained }" @click="isVocabTrained ? switchTab('battle') : showLockedAlert()" @mouseenter="playHoverSound">
            🐉 龍之試驗 <span v-if="!isVocabTrained">🔒</span>
          </button>
          <button class="nav-btn" :class="{ active: currentTab === 'shop' }" @click="switchTab('shop')" @mouseenter="playHoverSound">🎁 兌換獎品</button>
          <button class="nav-btn rank-btn" @click="showLeaderboard = true" @mouseenter="playHoverSound">🏆 排行榜</button>
        </div>
      </nav>
    </transition>

    <div v-if="!isNavOpen" class="hover-tip-bar" @mouseenter="isNavOpen = true">
      <span>🔽 游標移至頂部展開選單 🔽</span>
    </div>

    <div v-if="!hasInteracted" class="interaction-overlay">
      <div class="interaction-msg pixel-box-full">
        <h2>⚔️ 歡迎來到討龍英文趣 ⚔️</h2>
        <p>點擊畫面開始你的英語冒險！</p>
        <p class="blink-text">>>> CLICK TO START <<<</p>
      </div>
    </div>

    <div class="status-bar-wrapper">
      <div class="status-bar">
        <span>目前累積積分：<span class="highlight">{{ pointsDisplay }}</span></span>
      </div>
    </div>

    <main class="content-area">
      <Home v-if="currentTab === 'home'" @go-vocab="switchTab('vocab')" />
      <VocabBook v-if="currentTab === 'vocab'" :is-trained="isVocabTrained" @page-flip="playFlipSound" @unlock-battle="unlockBattle" @go-to-battle="switchTab('battle')" />
      <BattleMap v-show="currentTab === 'battle'" :global-bgm-playing="isBgmPlaying" @gain-point="addPoint" @enter-battle="pauseMainBGM" @leave-battle="resumeMainBGM" @stage-cleared="incrementClears" />
      <PrizeShop v-if="currentTab === 'shop'" :points="points" @spend-points="deductPoints" />
    </main>

    <button 
      class="floating-bgm-btn" 
      @click="toggleBGM" 
      @mouseenter="playHoverSound"
      :title="isBgmPlaying ? '關閉背景音樂' : '開啟背景音樂'"
    >
      {{ isBgmPlaying ? '🔊' : '🔇' }}
    </button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import FlyingDragons from './components/FlyingDragons.vue'
import Home from './components/Home.vue'
import VocabBook from './components/VocabBook.vue'
import BattleMap from './components/BattleMap.vue'
import PrizeShop from './components/PrizeShop.vue'
import RegisterModal from './components/RegisterModal.vue'
import Leaderboard from './components/Leaderboard.vue'

// 音效與音樂
const hoverAudio = new Audio('/hover.mp3') 
const clickAudio = new Audio('https://assets.mixkit.co/active_storage/sfx/2568/2568-preview.mp3')
const successAudio = new Audio('/success.mp3')
const flipAudio = new Audio('/flip.mp3')
const mainBgmAudio = new Audio('/bgm.mp3')
mainBgmAudio.loop = true 
mainBgmAudio.volume = 0.3 

// 狀態管理
const isRegistered = ref(false)
const mySeat = ref('')
const showLeaderboard = ref(false)
const isBgmPlaying = ref(false)
const hasInteracted = ref(false) 
const isMainBgmPausedByBattle = ref(false) 
const isVocabTrained = ref(false)
const isNavOpen = ref(false)
const currentTab = ref('home') 
const points = ref(0)
const pointsDisplay = computed(() => `${points.value} 點`)

onMounted(() => {
  const info = localStorage.getItem('hero_info')
  if (info) {
    isRegistered.value = true
    mySeat.value = JSON.parse(info).seat
  }
})

const handleRegister = (info) => {
  isRegistered.value = true
  mySeat.value = info.seat
}

const incrementClears = () => {
  let info = JSON.parse(localStorage.getItem('hero_info'))
  info.clears += 1
  localStorage.setItem('hero_info', JSON.stringify(info))
  
  let board = JSON.parse(localStorage.getItem('class_leaderboard') || '[]')
  let myIdx = board.findIndex(b => b.seat === info.seat)
  if (myIdx > -1) {
    board[myIdx].clears += 1
    localStorage.setItem('class_leaderboard', JSON.stringify(board))
  }
}

const initBGM = () => {
  if (!hasInteracted.value) {
    hasInteracted.value = true
    mainBgmAudio.play().then(() => { isBgmPlaying.value = true }).catch(() => {})
  }
}

const toggleBGM = () => {
  isBgmPlaying.value = !isBgmPlaying.value
  if (!isMainBgmPausedByBattle.value) {
    isBgmPlaying.value ? mainBgmAudio.play() : mainBgmAudio.pause()
  }
}

const pauseMainBGM = () => { isMainBgmPausedByBattle.value = true; mainBgmAudio.pause() }
const resumeMainBGM = () => { isMainBgmPausedByBattle.value = false; if (isBgmPlaying.value) mainBgmAudio.play() }

const playHoverSound = () => { hoverAudio.currentTime = 0; hoverAudio.volume = 0.5; hoverAudio.play().catch(() => {}); }
const playFlipSound = () => { flipAudio.currentTime = 0; flipAudio.volume = 0.8; flipAudio.play().catch(() => {}); }

const switchTab = (tabName) => { 
  clickAudio.currentTime = 0; clickAudio.play().catch(() => {}); 
  currentTab.value = tabName; 
}

const showLockedAlert = () => { alert('⚠️ 勇者，你尚未完成【單字秘笈】的修練！請先熟讀秘笈並通過最後一頁的試驗！') }
const unlockBattle = () => { isVocabTrained.value = true }

const addPoint = () => { points.value += 1 }
const deductPoints = (cost) => { 
    if (points.value >= cost) {
        points.value -= cost
        successAudio.currentTime = 0; successAudio.volume = 0.8; successAudio.play().catch(() => {})
        setTimeout(() => { alert('🎉 兌換成功！') }, 300)
    } else { alert('❌ 積分不足！') }
}
</script>


<style scoped>
.app-container { min-height: 100vh; width: 100%; display: flex; flex-direction: column; position: relative; z-index: 1; font-family: 'LanaPixel', sans-serif; }
.nav-trigger-zone { position: fixed; top: 0; left: 0; width: 100%; height: 20px; z-index: 999; }

.navbar { 
  position: fixed; top: 0; left: 0; width: 100%; display: flex; 
  justify-content: space-between; align-items: center; padding: 15px 40px; 
  background-color: #6F1D1B; border-bottom: 6px solid #4a1211; 
  box-shadow: 0 6px 15px rgba(0,0,0,0.3); z-index: 1000; 
}

.logo { 
  font-size: 26px; font-weight: bold; color: #f1c40f; text-shadow: 2px 2px 0 #000; 
  cursor: pointer; transition: transform 0.2s ease;
}
.logo:hover { transform: scale(1.05); }

.nav-links { display: flex; gap: 15px; align-items: center; flex-wrap: nowrap; margin-right: 60px; }

.nav-btn { 
  font-family: 'LanaPixel', sans-serif; background-color: #4a1211; color: #fceae9; 
  padding: 10px 20px; font-size: 16px; cursor: pointer; border: 3px solid #fff; 
  box-shadow: 2px 2px 0px #000; transition: all 0.2s; white-space: nowrap; flex-shrink: 0; 
}
.nav-btn:hover { background-color: #8c2624; transform: scale(1.05); }
.nav-btn.active { background-color: #f1c40f; color: #6F1D1B; border-color: #fff; font-weight: bold; box-shadow: 4px 4px 0px #000; }
.nav-btn.locked { opacity: 0.6; cursor: not-allowed; border-color: #7f8c8d; }
.nav-btn.locked:hover { background-color: #4a1211; transform: none; }
.rank-btn { background-color: #27ae60; border-color: #2ecc71; }

.hover-tip-bar { position: fixed; top: 0; left: 50%; transform: translateX(-50%); background-color: #6F1D1B; color: #f1c40f; padding: 6px 20px; font-size: 14px; border-radius: 0 0 8px 8px; z-index: 998; animation: bounce 2s infinite; }
@keyframes bounce { 0%, 100% { transform: translateX(-50%) translateY(0); } 50% { transform: translateX(-50%) translateY(3px); } }
.slide-nav-enter-active, .slide-nav-leave-active { transition: transform 0.4s ease; }
.slide-nav-enter-from, .slide-nav-leave-to { transform: translateY(-100%); }

.status-bar-wrapper { width: 100%; padding: 0 40px; margin-top: 80px; }
.status-bar { background-color: #8c2624; color: #fff; padding: 12px 24px; font-size: 18px; display: flex; justify-content: flex-end; border: 4px solid #6F1D1B; box-shadow: 4px 4px 0px rgba(0,0,0,0.1); }
.highlight { color: #f1c40f; font-size: 22px; font-weight: bold; text-shadow: 1px 1px 0 #000; }
.content-area { flex: 1; width: 100%; padding: 20px 40px; display: flex; justify-content: center; position: relative; z-index: 10; }

.floating-bgm-btn {
  position: fixed; bottom: 30px; right: 30px; width: 60px; height: 60px;
  border-radius: 16px; background-color: #2c3e50; border: 4px solid #f1c40f;
  color: #f1c40f; font-size: 28px; display: flex; justify-content: center;
  align-items: center; cursor: pointer; z-index: 1500;
  box-shadow: 4px 4px 0px rgba(0, 0, 0, 0.5); transition: all 0.2s ease;
}
.floating-bgm-btn:hover { background-color: #34495e; transform: scale(1.1); }

.interaction-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background-color: rgba(0,0,0,0.85); z-index: 2000; display: flex; justify-content: center; align-items: center; cursor: pointer; }
.interaction-msg { background-color: #6F1D1B; border: 6px solid #f1c40f; padding: 40px 60px; text-align: center; color: white; }
.blink-text { color: #2ecc71; animation: blink 1s infinite; }
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
</style>