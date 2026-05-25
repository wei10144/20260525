<template>
  <div class="modal-overlay">
    <div class="modal-box pixel-box">
      <h2>勇者登記處</h2>
      <p>請輸入座號與姓名，以利登記排行榜成績：</p>
      <input type="text" v-model="seat" placeholder="座號 (例如: 01)" class="pixel-input">
      <input type="text" v-model="name" placeholder="姓名" class="pixel-input">
      <button class="pixel-btn" @click="register">開始冒險</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
const seat = ref('')
const name = ref('')
const emit = defineEmits(['registered'])

const register = () => {
  if (!seat.value || !name.value) return alert('請填寫完整資訊！')
  
  // 登記資料
  const userInfo = { seat: seat.value, name: name.value, clears: 0 }
  localStorage.setItem('hero_info', JSON.stringify(userInfo))
  
  // 初始化排行榜 (如果不存在)
  let board = JSON.parse(localStorage.getItem('class_leaderboard') || '[]')
  
  // 檢查是否已存在，若無則加入
  if (!board.find(b => b.seat === seat.value)) {
    board.push(userInfo)
    localStorage.setItem('class_leaderboard', JSON.stringify(board))
  }
  
  emit('registered', userInfo)
}
</script>

<style scoped>
/* 💡 修正這裡：height 改為 100vh 確保覆蓋整個視窗，Flex 確保內容永遠在正中央 */
.modal-overlay { 
  position: fixed; top: 0; left: 0; 
  width: 100vw; height: 100vh; 
  background: rgba(0,0,0,0.9); 
  z-index: 9999; 
  display: flex; 
  justify-content: center; 
  align-items: center; 
}

.modal-box { 
  padding: 40px; 
  background: #FAF4F2; 
  width: 300px; 
  text-align: center; 
  border: 6px solid #6F1D1B;
  box-shadow: 8px 8px 0px #000;
  font-family: 'LanaPixel', sans-serif;
}

.pixel-input { 
  display: block; 
  width: 100%; 
  margin: 15px 0; 
  padding: 10px; 
  border: 3px solid #6F1D1B; 
  font-family: 'LanaPixel', sans-serif;
  background: #fff;
}

.pixel-btn {
  font-family: 'LanaPixel', sans-serif;
  background-color: #6F1D1B;
  color: white;
  border: 3px solid #fff;
  padding: 10px 20px;
  cursor: pointer;
  margin-top: 10px;
}
</style>