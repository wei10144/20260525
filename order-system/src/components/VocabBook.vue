<template>
  <div class="vocab-book pixel-box-full">
    <h2>📖 勇者單字秘笈</h2>
    
    <div class="book-container">
      <div class="page left-page">
        <ul>
          <li v-for="(word, i) in leftWords" :key="i" class="word-item" @click="openWordDetail(word)">
            <span class="emoji">{{ word.img }}</span> 
            <span class="en">{{ word.en }}</span>
            <span class="tw">({{ word.tw }})</span>
            <span class="click-hint">🔍</span>
          </li>
        </ul>
      </div>
      <div class="page right-page">
        <ul>
          <li v-for="(word, i) in rightWords" :key="i" class="word-item" @click="openWordDetail(word)">
            <span class="emoji">{{ word.img }}</span> 
            <span class="en">{{ word.en }}</span>
            <span class="tw">({{ word.tw }})</span>
            <span class="click-hint">🔍</span>
          </li>
        </ul>
      </div>
    </div>
    
    <div class="controls">
      <button class="pixel-btn accent-btn" @click="prevPage" :disabled="currentPage === 0">上一頁</button>
      <span class="page-counter">第 {{ currentPage + 1 }} 頁 / 共 {{ totalPages }} 頁</span>
      
      <button v-if="currentPage >= totalPages - 1 && !isTrained" class="pixel-btn training-btn" @click="showStartModal = true">進入試驗前小訓練！</button>
      <button v-else class="pixel-btn accent-btn" @click="nextPage" :disabled="currentPage >= totalPages - 1">下一頁</button>
    </div>

    <div v-if="showWordDetailModal" class="modal-overlay" @click.self="closeWordDetail">
      <div class="modal-box pixel-box detail-box">
        <div class="detail-header">
          <span class="detail-emoji">{{ selectedWord.img }}</span>
          <h3 class="detail-en">{{ selectedWord.en }}</h3>
          <button class="close-btn pixel-btn" @click="closeWordDetail">X</button>
        </div>
        <div class="detail-body">
          <p class="detail-info"><strong>KK音標：</strong> <span>{{ selectedWord.kk }}</span></p>
          <p class="detail-info"><strong>詞性：</strong> <span>{{ selectedWord.pos }}</span></p>
          <p class="detail-info"><strong>中文：</strong> <span>{{ selectedWord.tw }}</span></p>
          
          <div class="detail-example">
            <h4>📜 例句示範：</h4>
            <p class="ex-en">{{ selectedWord.exampleEn }}</p>
            <p class="ex-tw">{{ selectedWord.exampleTw }}</p>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showStartModal" class="modal-overlay">
      <div class="modal-box pixel-box">
        <h3>即將進入試驗前小訓練</h3>
        <p>勇者，你準備好測試你從秘笈中學到的知識了嗎？</p>
        <div class="modal-actions">
          <button class="pixel-btn yes-btn" @click="startTraining">是，放馬過來！</button>
          <button class="pixel-btn no-btn" @click="showStartModal = false">否，我再看一下</button>
        </div>
      </div>
    </div>

    <div v-if="isTraining" class="modal-overlay">
      <div class="modal-box training-box pixel-box">
        <h3>小考進行中 ({{ currentQIndex + 1 }} / {{ quizQuestions.length }})</h3>
        
        <div v-if="currentQuestion.type === 'EnToTw'" class="quiz-content">
          <div class="question-header">
            <span class="quiz-emoji">{{ currentQuestion.img }}</span>
            <span class="quiz-tw">{{ currentQuestion.tw }}</span>
          </div>
          <p class="instruction">請依序點擊下方字母拼出正確單字：</p>
          
          <div class="answer-slots">
            <div v-for="(slot, idx) in currentQuestion.targetWord.length" :key="'slot'+idx" class="slot pixel-box" @click="removeLetterFromAnswer(idx)">
              {{ currentAnswerLetters[idx] ? currentAnswerLetters[idx].char : '' }}
            </div>
          </div>
          
          <div class="letters-pool">
            <button v-for="letterObj in currentAvailableLetters" :key="letterObj.id" class="pixel-btn letter-btn" :class="{ hidden: letterObj.used }" @click="addLetterToAnswer(letterObj)">
              {{ letterObj.char }}
            </button>
          </div>
        </div>

        <div v-else-if="currentQuestion.type === 'TwToEn'" class="quiz-content">
          <div class="question-header">
            <span class="quiz-emoji">{{ currentQuestion.img }}</span>
            <span class="quiz-en">{{ currentQuestion.en }}</span>
          </div>
          <p class="instruction">請選擇對應的中文意思：</p>
          <div class="options-grid">
            <button v-for="(opt, idx) in currentQuestion.options" :key="idx" class="pixel-btn option-btn" @click="checkOptionAnswer(opt)">
              {{ opt }}
            </button>
          </div>
        </div>

        <div v-if="feedbackMsg" class="feedback-msg" :class="feedbackStatus">
          {{ feedbackMsg }}
        </div>
      </div>
    </div>

    <div v-if="showFinishModal" class="modal-overlay">
      <div class="modal-box pixel-box finish-box">
        <h2>🎉 恭喜完成訓練！ 🎉</h2>
        <p>你已經完全掌握了單字秘笈的精髓，龍之試驗的大門已為你敞開！</p>
        <div class="modal-actions">
          <button class="pixel-btn yes-btn" @click="goToBattle">來試驗吧！</button>
          <button class="pixel-btn no-btn" @click="showFinishModal = false">還想再逛逛</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({ isTrained: Boolean })
const emit = defineEmits(['page-flip', 'unlock-battle', 'go-to-battle'])

const currentPage = ref(0)
const itemsPerPage = 8

// ================= 單字字典擴充區 =================
const vocabList = [
  // Level 1: 基礎單字
  { img: '🍎', en: 'Apple', tw: '蘋果', level: 1, kk: '[ˋæp!]', pos: '名詞 (n.)', exampleEn: 'I have a red apple.', exampleTw: '我有一顆紅蘋果。' },
  { img: '🐶', en: 'Dog', tw: '狗', level: 1, kk: '[dɔg]', pos: '名詞 (n.)', exampleEn: 'The dog is barking.', exampleTw: '狗在叫。' },
  { img: '🐱', en: 'Cat', tw: '貓', level: 1, kk: '[kæt]', pos: '名詞 (n.)', exampleEn: 'The cat is sleeping.', exampleTw: '貓在睡覺。' },
  { img: '📖', en: 'Book', tw: '書本', level: 1, kk: '[bʊk]', pos: '名詞 (n.)', exampleEn: 'I read a book.', exampleTw: '我讀一本書。' },
  { img: '🖊️', en: 'Pen', tw: '筆', level: 1, kk: '[pɛn]', pos: '名詞 (n.)', exampleEn: 'I write with a pen.', exampleTw: '我用筆寫字。' },
  { img: '☀️', en: 'Sun', tw: '太陽', level: 1, kk: '[sʌn]', pos: '名詞 (n.)', exampleEn: 'The sun is bright.', exampleTw: '太陽很亮。' },
  { img: '🌙', en: 'Moon', tw: '月亮', level: 1, kk: '[mun]', pos: '名詞 (n.)', exampleEn: 'The moon is beautiful.', exampleTw: '月亮很美。' },
  { img: '⭐', en: 'Star', tw: '星星', level: 1, kk: '[stɑr]', pos: '名詞 (n.)', exampleEn: 'Look at the star!', exampleTw: '看那顆星星！' },
  { img: '💧', en: 'Water', tw: '水', level: 1, kk: '[ˋwɔtɚ]', pos: '名詞 (n.)', exampleEn: 'Drink some water.', exampleTw: '喝點水。' },
  { img: '🌳', en: 'Tree', tw: '樹', level: 1, kk: '[tri]', pos: '名詞 (n.)', exampleEn: 'The tree is tall.', exampleTw: '這棵樹很高。' },
  { img: '🌸', en: 'Flower', tw: '花', level: 1, kk: '[ˋflaʊɚ]', pos: '名詞 (n.)', exampleEn: 'I love this flower.', exampleTw: '我喜歡這朵花。' },
  { img: '🐦', en: 'Bird', tw: '鳥', level: 1, kk: '[bɝd]', pos: '名詞 (n.)', exampleEn: 'The bird can fly.', exampleTw: '鳥會飛。' },
  { img: '🐟', en: 'Fish', tw: '魚', level: 1, kk: '[fɪʃ]', pos: '名詞 (n.)', exampleEn: 'The fish swims fast.', exampleTw: '魚游得很快。' },
  { img: '🚗', en: 'Car', tw: '汽車', level: 1, kk: '[kɑr]', pos: '名詞 (n.)', exampleEn: 'He drives a car.', exampleTw: '他開一輛車。' },
  { img: '🚌', en: 'Bus', tw: '公車', level: 1, kk: '[bʌs]', pos: '名詞 (n.)', exampleEn: 'I take the bus to school.', exampleTw: '我搭公車上學。' },
  { img: '🎒', en: 'Bag', tw: '背包', level: 1, kk: '[bæg]', pos: '名詞 (n.)', exampleEn: 'My bag is heavy.', exampleTw: '我的背包很重。' },
  
  // Level 2: 狀態形容詞與進階單字
  { img: '😊', en: 'Happy', tw: '快樂的', level: 2, kk: '[ˋhæpɪ]', pos: '形容詞 (adj.)', exampleEn: 'I feel happy today.', exampleTw: '我今天覺得很快樂。' },
  { img: '😠', en: 'Angry', tw: '生氣的', level: 2, kk: '[ˋæŋgrɪ]', pos: '形容詞 (adj.)', exampleEn: 'She is very angry.', exampleTw: '她非常生氣。' },
  { img: '⚡', en: 'Fast', tw: '快的', level: 2, kk: '[fæst]', pos: '形容詞 (adj.)', exampleEn: 'The train is fast.', exampleTw: '這火車很快。' },
  { img: '🐢', en: 'Slow', tw: '慢的', level: 2, kk: '[slo]', pos: '形容詞 (adj.)', exampleEn: 'The turtle is slow.', exampleTw: '烏龜很慢。' },
  { img: '❄️', en: 'Cold', tw: '冷的', level: 2, kk: '[kold]', pos: '形容詞 (adj.)', exampleEn: 'It is cold in winter.', exampleTw: '冬天很冷。' },
  { img: '🔥', en: 'Hot', tw: '熱的', level: 2, kk: '[hɑt]', pos: '形容詞 (adj.)', exampleEn: 'The soup is hot.', exampleTw: '這湯很熱。' },
  { img: '🤖', en: 'Robot', tw: '機器人', level: 2, kk: '[ˋrobot]', pos: '名詞 (n.)', exampleEn: 'The robot can walk.', exampleTw: '這機器人會走路。' },
  { img: '🚀', en: 'Rocket', tw: '火箭', level: 2, kk: '[ˋrɑkɪt]', pos: '名詞 (n.)', exampleEn: 'The rocket goes to space.', exampleTw: '火箭飛向太空。' },
  { img: '✨', en: 'Magic', tw: '魔法', level: 2, kk: '[ˋmædʒɪk]', pos: '名詞 (n.)', exampleEn: 'Do you believe in magic?', exampleTw: '你相信魔法嗎？' },
  { img: '🚆', en: 'Train', tw: '火車', level: 2, kk: '[tren]', pos: '名詞 (n.)', exampleEn: 'The train is long.', exampleTw: '這列火車很長。' },
  { img: '⏰', en: 'Clock', tw: '時鐘', level: 2, kk: '[klɑk]', pos: '名詞 (n.)', exampleEn: 'Look at the clock.', exampleTw: '看一下時鐘。' },
  { img: '💰', en: 'Money', tw: '錢', level: 2, kk: '[ˋmʌnɪ]', pos: '名詞 (n.)', exampleEn: 'I need some money.', exampleTw: '我需要一些錢。' },
  { img: '🏠', en: 'House', tw: '房子', level: 2, kk: '[haʊs]', pos: '名詞 (n.)', exampleEn: 'This is my house.', exampleTw: '這是我的房子。' },
  { img: '☁️', en: 'Cloud', tw: '雲', level: 2, kk: '[klaʊd]', pos: '名詞 (n.)', exampleEn: 'The cloud is white.', exampleTw: '雲是白色的。' },
  { img: '🌧️', en: 'Rain', tw: '雨', level: 2, kk: '[ren]', pos: '名詞 (n.)', exampleEn: 'The rain is falling.', exampleTw: '雨正在下。' },
  { img: '⛄', en: 'Snow', tw: '雪', level: 2, kk: '[sno]', pos: '名詞 (n.)', exampleEn: 'I play in the snow.', exampleTw: '我在雪中玩耍。' },

  // Level 3: 奇幻冒險詞彙
  { img: '🐉', en: 'Dragon', tw: '龍', level: 3, kk: '[ˋdrægən]', pos: '名詞 (n.)', exampleEn: 'The dragon breathes fire.', exampleTw: '龍會吐火。' },
  { img: '⚔️', en: 'Sword', tw: '劍', level: 3, kk: '[sord]', pos: '名詞 (n.)', exampleEn: 'The knight has a sword.', exampleTw: '騎士有一把劍。' },
  { img: '🛡️', en: 'Shield', tw: '盾牌', level: 3, kk: '[ʃild]', pos: '名詞 (n.)', exampleEn: 'The shield protects him.', exampleTw: '盾牌保護了他。' },
  { img: '🏰', en: 'Castle', tw: '城堡', level: 3, kk: '[ˋkæs!]', pos: '名詞 (n.)', exampleEn: 'The king lives in a castle.', exampleTw: '國王住在城堡裡。' },
  { img: '🌲', en: 'Forest', tw: '森林', level: 3, kk: '[ˋfɔrɪst]', pos: '名詞 (n.)', exampleEn: 'We walk in the forest.', exampleTw: '我們在森林裡散步。' },
  { img: '🌵', en: 'Desert', tw: '沙漠', level: 3, kk: '[ˋdɛzɚt]', pos: '名詞 (n.)', exampleEn: 'The desert is dry.', exampleTw: '沙漠很乾燥。' },
  { img: '🧪', en: 'Potion', tw: '藥水', level: 3, kk: '[ˋpoʃən]', pos: '名詞 (n.)', exampleEn: 'Drink the magic potion.', exampleTw: '喝下這魔法藥水。' },
  { img: '👻', en: 'Ghost', tw: '幽靈', level: 3, kk: '[gost]', pos: '名詞 (n.)', exampleEn: 'A ghost appears at night.', exampleTw: '幽靈在夜晚出現。' },
  { img: '👑', en: 'Crown', tw: '皇冠', level: 3, kk: '[kraʊn]', pos: '名詞 (n.)', exampleEn: 'The queen wears a crown.', exampleTw: '女王戴著皇冠。' },
  { img: '💎', en: 'Diamond', tw: '鑽石', level: 3, kk: '[ˋdaɪəmənd]', pos: '名詞 (n.)', exampleEn: 'The diamond is shiny.', exampleTw: '這顆鑽石很閃亮。' },
  { img: '👾', en: 'Monster', tw: '怪物', level: 3, kk: '[ˋmɑnstɚ]', pos: '名詞 (n.)', exampleEn: 'The monster is scary.', exampleTw: '這怪物很可怕。' },
  { img: '🪙', en: 'Treasure', tw: '寶藏', level: 3, kk: '[ˋtrɛʒɚ]', pos: '名詞 (n.)', exampleEn: 'We found the treasure.', exampleTw: '我們找到了寶藏。' },
  { img: '🏝️', en: 'Island', tw: '島嶼', level: 3, kk: '[ˋaɪlənd]', pos: '名詞 (n.)', exampleEn: 'They live on an island.', exampleTw: '他們住在一個島上。' },
  { img: '⛰️', en: 'Mountain', tw: '山', level: 3, kk: '[ˋmaʊntn]', pos: '名詞 (n.)', exampleEn: 'The mountain is high.', exampleTw: '這座山很高。' },
  { img: '🌊', en: 'Ocean', tw: '海洋', level: 3, kk: '[ˋoʃən]', pos: '名詞 (n.)', exampleEn: 'The ocean is deep blue.', exampleTw: '海洋是深藍色的。' },
  { img: '🧙‍♂️', en: 'Wizard', tw: '巫師', level: 3, kk: '[ˋwɪzɚd]', pos: '名詞 (n.)', exampleEn: 'The wizard casts a spell.', exampleTw: '巫師施展了一個咒語。' }
]

const totalPages = computed(() => Math.ceil(vocabList.length / itemsPerPage))
const currentWords = computed(() => vocabList.slice(currentPage.value * itemsPerPage, (currentPage.value + 1) * itemsPerPage))
const leftWords = computed(() => currentWords.value.slice(0, 4))
const rightWords = computed(() => currentWords.value.slice(4, 8))

const nextPage = () => { if (currentPage.value < totalPages.value - 1) { currentPage.value++; emit('page-flip'); } }
const prevPage = () => { if (currentPage.value > 0) { currentPage.value--; emit('page-flip'); } }

// ================= 單字詳情卡片邏輯 =================
const showWordDetailModal = ref(false)
const selectedWord = ref({})

const openWordDetail = (word) => {
  selectedWord.value = word
  showWordDetailModal.value = true
}

const closeWordDetail = () => {
  showWordDetailModal.value = false
}

// ================= 試驗前小訓練邏輯 =================
const showStartModal = ref(false)
const isTraining = ref(false)
const showFinishModal = ref(false)

const quizQuestions = ref([])
const currentQIndex = ref(0)
const feedbackMsg = ref('')
const feedbackStatus = ref('') 

const currentAvailableLetters = ref([])
const currentAnswerLetters = ref([])

const currentQuestion = computed(() => quizQuestions.value[currentQIndex.value] || {})

const startTraining = () => {
  showStartModal.value = false
  generateQuiz()
  isTraining.value = true
}

const generateQuiz = () => {
  let shuffledVocab = [...vocabList].sort(() => Math.random() - 0.5)
  let selectedVocab = shuffledVocab.slice(0, 5) 
  
  quizQuestions.value = selectedVocab.map(word => {
    const isEnToTw = Math.random() > 0.5 
    if (isEnToTw) {
      return { type: 'EnToTw', img: word.img, tw: word.tw, targetWord: word.en.toUpperCase() }
    } else {
      let options = [word.tw]
      let distractors = vocabList.filter(w => w.tw !== word.tw).sort(() => Math.random() - 0.5)
      for(let i=0; i<3; i++) options.push(distractors[i].tw)
      options.sort(() => Math.random() - 0.5)
      return { type: 'TwToEn', img: word.img, en: word.en, options: options, targetAnswer: word.tw }
    }
  })
  currentQIndex.value = 0
  setupCurrentQuestion()
}

const setupCurrentQuestion = () => {
  feedbackMsg.value = ''
  if (currentQuestion.value.type === 'EnToTw') {
    let lettersArray = currentQuestion.value.targetWord.split('').map((char, index) => {
      return { id: index, char: char, used: false }
    })
    currentAvailableLetters.value = lettersArray.sort(() => Math.random() - 0.5)
    currentAnswerLetters.value = [] 
  }
}

const addLetterToAnswer = (letterObj) => {
  if (letterObj.used) return
  letterObj.used = true
  currentAnswerLetters.value.push(letterObj)
  checkSpellingAnswer()
}

const removeLetterFromAnswer = (index) => {
  if (!currentAnswerLetters.value[index]) return
  let removed = currentAnswerLetters.value.splice(index, 1)[0]
  let originalObj = currentAvailableLetters.value.find(l => l.id === removed.id)
  if (originalObj) originalObj.used = false
}

const checkSpellingAnswer = () => {
  const target = currentQuestion.value.targetWord
  if (currentAnswerLetters.value.length === target.length) {
    const currentWord = currentAnswerLetters.value.map(l => l.char).join('')
    if (currentWord === target) { showCorrectAndNext() } 
    else { feedbackStatus.value = 'wrong'; feedbackMsg.value = '拼錯了!請點擊框框退回字母再試一次。' }
  }
}

const checkOptionAnswer = (opt) => {
  if (opt === currentQuestion.value.targetAnswer) { showCorrectAndNext() } 
  else { feedbackStatus.value = 'wrong'; feedbackMsg.value = '答錯了!請再試一次!' }
}

const showCorrectAndNext = () => {
  feedbackStatus.value = 'correct'
  feedbackMsg.value = '恭喜答對!'
  setTimeout(() => {
    if (currentQIndex.value < quizQuestions.value.length - 1) {
      currentQIndex.value++; setupCurrentQuestion()
    } else {
      isTraining.value = false; showFinishModal.value = true; emit('unlock-battle') 
    }
  }, 1500)
}

const goToBattle = () => { showFinishModal.value = false; emit('go-to-battle') }
</script>

<style scoped>
.vocab-book { width: 100%; background-color: #6F1D1B; padding: 40px; border: 6px solid #4a1211; box-shadow: 6px 6px 0px #000; position: relative; }
h2 { color: #f1c40f; margin-top: 0; font-size: 28px; text-shadow: 2px 2px 0px #000; text-align: center; }
.book-container { display: flex; background-color: #f5d6a7; padding: 15px; border: 4px solid #000; margin: 25px 0; gap: 10px; }
.page { flex: 1; background-color: #fff; color: #2c3e50; padding: 30px; min-height: 400px; border: 3px solid #6F1D1B; }
.left-page { border-right: 2px dashed #6F1D1B; }
.right-page { border-left: 2px dashed #6F1D1B; }

/* 💡 將單字列設定為可點擊互動的樣式 */
ul { list-style: none; padding: 0; margin: 0; }
.word-item { 
  font-size: 22px; margin-bottom: 20px; border-bottom: 2px dashed #f2d1cf; padding-bottom: 10px; 
  display: flex; align-items: center; cursor: pointer; transition: all 0.2s ease;
  border-radius: 8px;
}
.word-item:hover { background-color: #fae5d3; transform: translateX(5px); }
.emoji { font-size: 32px; margin-right: 15px; }
.en { font-weight: bold; color: #6F1D1B; margin-right: 15px; }
.tw { color: #7f8c8d; font-size: 18px; flex: 1; }
.click-hint { font-size: 16px; opacity: 0; transition: opacity 0.2s; }
.word-item:hover .click-hint { opacity: 1; }

.controls { display: flex; justify-content: space-between; align-items: center; color: white; font-size: 20px; padding: 0 10px; }
.page-counter { color: #f1c40f; font-weight: bold; }
.accent-btn { background-color: #4a1211; border-color: #fff; color: #fff;}
.accent-btn:hover { background-color: #8c2624; }
button:disabled { opacity: 0.4; cursor: not-allowed; box-shadow: none; transform: none; }
.training-btn { background-color: #f39c12; color: #6F1D1B; border-color: #fff; font-weight: bold; animation: pulse 1.5s infinite; }
@keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(243, 156, 18, 0.7); } 70% { box-shadow: 0 0 0 15px rgba(243, 156, 18, 0); } 100% { box-shadow: 0 0 0 0 rgba(243, 156, 18, 0); } }

/* ================= 彈窗與測驗樣式 ================= */
.modal-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background-color: rgba(0,0,0,0.8); z-index: 2000; display: flex; justify-content: center; align-items: center; }
.modal-box { background-color: #FAF4F2; border: 6px solid #6F1D1B; padding: 40px; text-align: center; max-width: 500px; box-shadow: 8px 8px 0px #000; }
.modal-box h3 { color: #6F1D1B; font-size: 24px; margin-top: 0; }
.modal-box p { color: #333; font-size: 18px; margin-bottom: 30px; line-height: 1.5; }
.modal-actions { display: flex; justify-content: center; gap: 20px; }
.yes-btn { background-color: #27ae60; color: white; }
.no-btn { background-color: #7f8c8d; color: white; }
.finish-box h2 { color: #d35400; font-size: 32px; margin-top: 0; }

.training-box { max-width: 600px; width: 90%; }
.question-header { background-color: #6F1D1B; color: #f1c40f; padding: 20px; font-size: 36px; border-radius: 8px; margin-bottom: 20px; display: flex; align-items: center; justify-content: center; gap: 15px; }
.quiz-emoji { font-size: 50px; background-color: #fff; padding: 10px; border-radius: 50%; }
.instruction { color: #2c3e50; font-weight: bold; margin-bottom: 15px; }

.answer-slots { display: flex; justify-content: center; gap: 10px; margin-bottom: 25px; flex-wrap: wrap; }
.slot { width: 50px; height: 50px; border: 3px dashed #7f8c8d; display: flex; align-items: center; justify-content: center; font-size: 24px; font-weight: bold; color: #6F1D1B; cursor: pointer; background-color: #f1c40f; box-shadow: none; }
.slot:hover { background-color: #e67e22; }

.letters-pool { display: flex; justify-content: center; gap: 15px; flex-wrap: wrap; }
.letter-btn { padding: 10px 20px; font-size: 24px; background-color: #34495e; color: #fff; }
.letter-btn.hidden { opacity: 0; pointer-events: none; }

.options-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
.option-btn { font-size: 20px; background-color: #2980b9; color: white; }
.option-btn:hover { background-color: #1abc9c; }

.feedback-msg { margin-top: 25px; padding: 15px; font-size: 20px; font-weight: bold; border-radius: 4px; }
.feedback-msg.correct { background-color: #d4edda; color: #27ae60; border: 2px solid #27ae60; }
.feedback-msg.wrong { background-color: #f8d7da; color: #c0392b; border: 2px solid #c0392b; }

/* ================= 單字詳情卡片樣式 ================= */
.detail-box { padding: 30px; text-align: left; }
.detail-header { display: flex; align-items: center; border-bottom: 4px solid #6F1D1B; padding-bottom: 15px; margin-bottom: 20px; position: relative; }
.detail-emoji { font-size: 48px; margin-right: 20px; background: #fff; border: 3px solid #6F1D1B; border-radius: 50%; padding: 10px; }
.detail-en { font-size: 32px; color: #6F1D1B; margin: 0; }
.close-btn { position: absolute; right: 0; top: 0; background-color: #e74c3c; padding: 5px 12px; font-size: 16px; border-color: #c0392b; color: white; }
.close-btn:hover { background-color: #c0392b; }

.detail-body p { margin: 10px 0; font-size: 18px; color: #2c3e50; }
.detail-info strong { color: #d35400; display: inline-block; width: 90px; }

.detail-example { margin-top: 25px; background-color: #fcf3cf; padding: 15px; border-left: 5px solid #f1c40f; border-radius: 4px; }
.detail-example h4 { margin: 0 0 10px 0; color: #b7950b; font-size: 18px; }
.ex-en { font-weight: bold; color: #2c3e50; margin-bottom: 5px !important; }
.ex-tw { color: #7f8c8d; margin-top: 0 !important; }
</style>