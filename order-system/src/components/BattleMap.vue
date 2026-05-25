<template>
  <div class="battle-map">
    <div v-if="viewMode === 'map' && allCleared" class="victory-screen pixel-box-alert">
      <h2>🎉 傳說中的勇者，你成功討伐了所有魔物！ 🎉</h2>
      <p>現在你可以自由選擇任何關卡進行修練與農積分了！</p>
    </div>

    <div v-if="viewMode === 'map'" class="map-view pixel-box">
      <h2 class="map-title">📜 討伐路線圖</h2>
      <p class="map-desc" v-if="!allCleared">請依序擊敗魔王解鎖下一個區域！</p>
      <div class="route-container">
        <div v-for="(stage, idx) in stages" :key="idx" class="route-node">
          <button class="node-btn pixel-btn" :class="{ 'cleared': idx < progressLevel, 'locked': idx > progressLevel, 'current': idx === progressLevel }" @click="enterStage(idx)" @mouseenter="hoverIndex = idx" @mouseleave="hoverIndex = null" :disabled="idx > progressLevel">
            <span v-if="idx < progressLevel">✅</span><span v-else-if="idx > progressLevel">🔒</span><span v-else>❗</span>
          </button>
          <div class="node-name">{{ stage.name }}</div>
          <div class="tooltip pixel-box" v-if="hoverIndex === idx">
            <div class="tooltip-title">{{ stage.enemyName }} {{ stage.enemy }}</div>
            <div class="tooltip-info">血量：{{ stage.questionsCount }} HP</div>
            <div class="tooltip-info">難度：{{ stage.difficulty }}</div>
            <div class="tooltip-status" v-if="idx > progressLevel">⚠️ 尚未解鎖</div><div class="tooltip-status" v-else-if="idx < progressLevel">✨ 已討伐</div>
          </div>
        </div>
      </div>
    </div>

    <div class="battle-arena" v-if="viewMode === 'battle'">
      <div class="battle-header">
        <button class="pixel-btn back-btn" @click="retreatToMap">🔙 撤退回地圖</button>
        <div class="stage-info">
          <h3>📍 戰區：{{ currentStageInfo.name }}</h3>
          <div class="progress-txt">魔王血量：{{ currentHp }} / {{ maxHp }}</div>
          <div class="hp-bar-container"><div class="hp-bar" :style="{ width: (currentHp / maxHp) * 100 + '%' }"></div></div>
        </div>
      </div>
      
      <div class="battle-scene">
        <div class="player"><div class="sprite">🧙‍♂️</div><div class="name-tag">勇者</div></div>
        <div class="effects" v-if="showAttack">💥</div>
        <div class="enemy" :class="{ 'shake-anim': isShaking, 'hurt-anim': showAttack }">
          <div class="sprite">{{ currentStageInfo.enemy }}</div><div class="name-tag">{{ currentStageInfo.enemyName }}</div>
        </div>
      </div>

      <div v-if="stageCleared" class="stage-clear-overlay"><h2>✨ 戰役勝利 ✨</h2><p>贏得 1 積分！已記錄於地圖中...</p></div>
      
      <div class="quiz-section" v-if="!stageCleared">
        <div class="question-header"><div class="question-img">{{ currentQuestion.img }}</div><p class="question-hint">請辨識圖片，點擊對應的魔法單字：</p></div>
        <div class="options-grid">
          <button v-for="(opt, idx) in currentQuestion.options" :key="idx" class="pixel-btn option-btn" @click="checkAnswer(opt)" @mouseenter="playOptionHoverSound">{{ opt }}</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  globalBgmPlaying: Boolean
})

const emit = defineEmits(['gain-point', 'enter-battle', 'leave-battle', 'stage-cleared'])

const attackAudio = new Audio('/attack.mp3') 
const optionHoverAudio = new Audio('/option-hover.mp3') 
const enterAudio = new Audio('/enter.mp3') 

const battleBgm1 = new Audio('/battle_bgm1.mp3')
const battleBgm2 = new Audio('/battle_bgm2.mp3')
const battleBgm3 = new Audio('/battle_bgm3.mp3')
const battleBgms = [battleBgm1, battleBgm2, battleBgm3]

battleBgms.forEach(bgm => {
  bgm.loop = true
  bgm.volume = 0.35
})

let currentBattleBgm = null 

watch(() => props.globalBgmPlaying, (isPlaying) => {
  if (viewMode.value === 'battle' && currentBattleBgm) {
    isPlaying ? currentBattleBgm.play().catch(()=>{}) : currentBattleBgm.pause()
  }
})

const playAttackSound = () => { attackAudio.currentTime = 0; attackAudio.volume = 0.8; attackAudio.play().catch(()=>{}) }
const playOptionHoverSound = () => { optionHoverAudio.currentTime = 0; optionHoverAudio.volume = 0.4; optionHoverAudio.play().catch(()=>{}) }
const playEnterSound = () => { enterAudio.currentTime = 0; enterAudio.volume = 0.7; enterAudio.play().catch(()=>{}) }

const stages = [
  { name: '迷霧森林', enemy: '🐺', enemyName: '森林野狼', questionsCount: 6, difficulty: '⭐', level: 1 },
  { name: '死亡沙漠', enemy: '🦂', enemyName: '巨型毒蠍', questionsCount: 8, difficulty: '⭐⭐', level: 2 },
  { name: '惡龍城堡', enemy: '🐲', enemyName: '黑暗惡龍', questionsCount: 10, difficulty: '⭐⭐⭐', level: 3 }
]

const allWords = [
  { img: '🍎', en: 'Apple', level: 1 }, { img: '🐶', en: 'Dog', level: 1 }, { img: '🐱', en: 'Cat', level: 1 }, { img: '📖', en: 'Book', level: 1 },
  { img: '🖊️', en: 'Pen', level: 1 }, { img: '☀️', en: 'Sun', level: 1 }, { img: '🌙', en: 'Moon', level: 1 }, { img: '⭐', en: 'Star', level: 1 },
  { img: '💧', en: 'Water', level: 1 }, { img: '🌳', en: 'Tree', level: 1 }, { img: '🌸', en: 'Flower', level: 1 }, { img: '🐦', en: 'Bird', level: 1 },
  { img: '🐟', en: 'Fish', level: 1 }, { img: '🚗', en: 'Car', level: 1 }, { img: '🚌', en: 'Bus', level: 1 }, { img: '🎒', en: 'Bag', level: 1 },
  { img: '😊', en: 'Happy', level: 2 }, { img: '😠', en: 'Angry', level: 2 }, { img: '⚡', en: 'Fast', level: 2 }, { img: '🐢', en: 'Slow', level: 2 },
  { img: '❄️', en: 'Cold', level: 2 }, { img: '🔥', en: 'Hot', level: 2 }, { img: '🤖', en: 'Robot', level: 2 }, { img: '🚀', en: 'Rocket', level: 2 },
  { img: '✨', en: 'Magic', level: 2 }, { img: '🚆', en: 'Train', level: 2 }, { img: '⏰', en: 'Clock', level: 2 }, { img: '💰', en: 'Money', level: 2 },
  { img: '🏠', en: 'House', level: 2 }, { img: '☁️', en: 'Cloud', level: 2 }, { img: '🌧️', en: 'Rain', level: 2 }, { img: '⛄', en: 'Snow', level: 2 },
  { img: '🐉', en: 'Dragon', level: 3 }, { img: '⚔️', en: 'Sword', level: 3 }, { img: '🛡️', en: 'Shield', level: 3 }, { img: '🏰', en: 'Castle', level: 3 },
  { img: '🌲', en: 'Forest', level: 3 }, { img: '🌵', en: 'Desert', level: 3 }, { img: '🧪', en: 'Potion', level: 3 }, { img: '👻', en: 'Ghost', level: 3 },
  { img: '👑', en: 'Crown', level: 3 }, { img: '💎', en: 'Diamond', level: 3 }, { img: '👾', en: 'Monster', level: 3 }, { img: '🪙', en: 'Treasure', level: 3 },
  { img: '🏝️', en: 'Island', level: 3 }, { img: '⛰️', en: 'Mountain', level: 3 }, { img: '🌊', en: 'Ocean', level: 3 }, { img: '🧙‍♂️', en: 'Wizard', level: 3 }
]

const viewMode = ref('map') 
const progressLevel = ref(0)
const hoverIndex = ref(null) 
const allCleared = computed(() => progressLevel.value >= stages.length)
const currentStageIndex = ref(0)
const currentHp = ref(0)
const stageCleared = ref(false)
const showAttack = ref(false)
const isShaking = ref(false)
const currentQuestion = ref({})
const questionPool = ref([])

const currentStageInfo = computed(() => stages[currentStageIndex.value])
const maxHp = computed(() => stages[currentStageIndex.value].questionsCount)

const initStage = (idx) => {
  const stageLevel = stages[idx].level
  let availableWords = allWords.filter(w => w.level === stageLevel)
  availableWords.sort(() => Math.random() - 0.5)
  questionPool.value = availableWords.slice(0, stages[idx].questionsCount)
  currentStageIndex.value = idx
  currentHp.value = stages[idx].questionsCount
  currentQuestion.value = generateNextQuestion()
  viewMode.value = 'battle'
}

const enterStage = (idx) => { 
  if (idx > progressLevel.value) return
  playEnterSound()
  if (currentBattleBgm) currentBattleBgm.pause()
  currentBattleBgm = battleBgms[idx]
  currentBattleBgm.currentTime = 0
  emit('enter-battle')
  if (props.globalBgmPlaying) currentBattleBgm.play().catch(()=>{})
  initStage(idx) 
}

const retreatToMap = () => {
  if (currentBattleBgm) currentBattleBgm.pause()
  viewMode.value = 'map'
  emit('leave-battle') 
}

function generateNextQuestion() {
  if (questionPool.value.length === 0) return null
  const correctItem = questionPool.value.pop()
  let options = [correctItem.en]
  let distractors = allWords.filter(w => w.en !== correctItem.en)
  distractors.sort(() => Math.random() - 0.5)
  for (let i = 0; i < 3; i++) { options.push(distractors[i].en) }
  options.sort(() => Math.random() - 0.5)
  return { img: correctItem.img, correctWord: correctItem.en, options: options }
}

const checkAnswer = (selectedWord) => {
  if (selectedWord === currentQuestion.value.correctWord) {
    playAttackSound()
    showAttack.value = true
    currentHp.value -= 1
    setTimeout(() => { showAttack.value = false }, 500)
    if (currentHp.value <= 0) { handleStageClear() } else { currentQuestion.value = generateNextQuestion() }
  } else {
    isShaking.value = true
    setTimeout(() => { isShaking.value = false }, 500)
  }
}

// 💡 確保只有一個 handleStageClear 定義
const handleStageClear = () => {
  stageCleared.value = true
  emit('gain-point')
  emit('stage-cleared') // 確保 emit 正確觸發
  
  if (currentBattleBgm) currentBattleBgm.pause()
  
  setTimeout(() => {
    stageCleared.value = false
    viewMode.value = 'map'
    emit('leave-battle') 
    
    if (currentStageIndex.value === progressLevel.value) { 
      progressLevel.value += 1 
    }
  }, 2000)
}
</script>

<style scoped>
.battle-map { width: 100%; }
.map-view { background-color: #FAF4F2; padding: 40px; text-align: center; }
.map-title { color: #6F1D1B; font-size: 32px; margin-top: 0; }
.map-desc { color: #7f8c8d; margin-bottom: 40px; }
.route-container { display: flex; justify-content: center; align-items: center; gap: 80px; margin-top: 60px; position: relative; }
.route-node { position: relative; display: flex; flex-direction: column; align-items: center; }
.route-node:not(:last-child)::after { content: ''; position: absolute; top: 35px; left: 70px; width: 80px; height: 8px; background-color: #d69f9d; border-top: 2px solid #6F1D1B; border-bottom: 2px solid #6F1D1B; z-index: 1; }
.route-node.cleared:not(:last-child)::after { background-color: #8c2624; }
.node-btn { width: 70px; height: 70px; border-radius: 50%; font-size: 28px; display: flex; align-items: center; justify-content: center; z-index: 2; padding: 0; transition: transform 0.2s, box-shadow 0.2s; }
.node-btn.locked { background-color: #95a5a6; border-color: #7f8c8d; cursor: not-allowed; box-shadow: none; transform: none; }
.node-btn.cleared { background-color: #2ecc71; border-color: #27ae60; }
.node-btn.current { background-color: #f1c40f; border-color: #f39c12; color: #6F1D1B; animation: pulse 1.5s infinite; }
@keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(241, 196, 15, 0.7); } 70% { box-shadow: 0 0 0 15px rgba(241, 196, 15, 0); } 100% { box-shadow: 0 0 0 0 rgba(241, 196, 15, 0); } }
.node-name { margin-top: 15px; font-weight: bold; color: #6F1D1B; font-size: 18px; }
.tooltip { position: absolute; top: -110px; width: 170px; background-color: #6F1D1B; color: #fff; padding: 10px; z-index: 10; text-align: left; border: 3px solid #f1c40f; animation: floatUp 0.2s ease-out; }
.tooltip::after { content: ''; position: absolute; bottom: -10px; left: 50%; transform: translateX(-50%); border-width: 10px 10px 0; border-style: solid; border-color: #f1c40f transparent transparent transparent; }
@keyframes floatUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
.tooltip-title { font-weight: bold; color: #f1c40f; margin-bottom: 5px; font-size: 16px; border-bottom: 1px dashed #d69f9d; padding-bottom: 5px;}
.tooltip-info { font-size: 14px; margin-bottom: 3px; }
.tooltip-status { font-size: 14px; margin-top: 5px; color: #f39c12; }
.victory-screen { background-color: #6F1D1B; border: 6px solid #4a1211; padding: 20px; text-align: center; margin-bottom: 20px; }
.victory-screen h2 { color: #f1c40f; margin: 0 0 10px 0; }
.victory-screen p { color: #fff; margin: 0; }
.battle-header { display: flex; align-items: center; gap: 20px; margin-bottom: 20px; }
.back-btn { background-color: #e74c3c; border-color: #c0392b; color: #fff; }
.back-btn:hover { background-color: #c0392b; }
.stage-info { flex: 1; background-color: #6F1D1B; padding: 15px 20px; border: 4px solid #4a1211; box-shadow: 4px 4px 0 #000; }
.stage-info h3 { margin: 0 0 5px 0; color: #f1c40f; font-size: 20px; }
.progress-txt { font-size: 16px; margin-bottom: 5px; color: #fff; }
.hp-bar-container { height: 20px; background-color: #4a1211; border: 3px solid #fff; }
.hp-bar { height: 100%; background-color: #2ecc71; transition: width 0.3s ease; }
.battle-scene { display: flex; justify-content: space-around; align-items: flex-end; height: 260px; background: linear-gradient(to bottom, #d69f9d 60%, #6F1D1B 60%); border: 4px solid #4a1211; margin-bottom: 20px; position: relative; box-shadow: 4px 4px 0 #000; }
.sprite { font-size: 90px; line-height: 1; }
.name-tag { background-color: #000; color: #fff; padding: 6px 14px; border: 2px solid #fff; text-align: center; margin-top: 10px; font-size: 16px; }
.effects { position: absolute; font-size: 120px; left: 60%; top: 30px; animation: explode 0.5s ease-out forwards; z-index: 5; }
.quiz-section { background-color: #ffffff; border: 4px solid #6F1D1B; padding: 30px; box-shadow: 4px 4px 0 #000; }
.question-header { display: flex; align-items: center; gap: 20px; margin-bottom: 20px; }
.question-img { font-size: 70px; background-color: #FAF4F2; border: 3px solid #6F1D1B; padding: 15px 25px; }
.question-hint { font-size: 22px; color: #6F1D1B; font-weight: bold; }
.options-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; }
@media (max-width: 900px) { .options-grid { grid-template-columns: 1fr 1fr; } }
.option-btn { font-size: 20px; background-color: #8c2624; border-color: #fff; color: #fff; }
.option-btn:hover { background-color: #6F1D1B; }
.stage-clear-overlay { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background-color: #f1c40f; color: #6F1D1B; padding: 40px 60px; border: 6px solid #4a1211; text-align: center; font-size: 24px; z-index: 100; box-shadow: 8px 8px 0px #000; }
@keyframes explode { 0% { transform: scale(0.5); opacity: 1; } 100% { transform: scale(1.6); opacity: 0; } }
.shake-anim { animation: shake 0.4s; }
@keyframes shake { 0%, 100% { transform: translateX(0); } 25% { transform: translateX(-15px); } 50% { transform: translateX(15px); } 75% { transform: translateX(-15px); } }
.hurt-anim { filter: brightness(0.5) sepia(1) saturate(10) hue-rotate(330deg); }
</style>