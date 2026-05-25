<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="parchment pixel-box">
      <h2>📜 班級英雄榜</h2>
      <div class="board">
        <div class="row header">
          <span class="rank">排名</span>
          <span class="info">勇者</span>
          <span class="score">過關數</span>
        </div>
        <div v-for="(hero, index) in sortedBoard" :key="hero.seat" 
             class="row" :class="{ 'self': hero.seat === mySeat }">
          <span class="rank">{{ index + 1 }} {{ index === 0 ? '👑' : '' }}</span>
          <span class="info">{{ hero.seat }}號 {{ hero.name }}</span>
          <span class="score">{{ hero.clears || 0 }} 次</span>
        </div>
      </div>
      <div class="rewards">
        <p>🥇 前五名獎勵：免值日生！ (第1名: 5週, 第5名: 1週)</p>
      </div>
      <button class="pixel-btn" @click="$emit('close')">關閉</button>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps(['mySeat'])
defineEmits(['close'])

// 讀取本地排行榜資料
const board = JSON.parse(localStorage.getItem('class_leaderboard') || '[]')

// 依照過關次數(clears)由高到低排序
const sortedBoard = computed(() => {
  return [...board].sort((a, b) => (b.clears || 0) - (a.clears || 0))
})
</script>

<style scoped>
.modal-overlay { 
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; 
  background: rgba(0,0,0,0.8); z-index: 9999; display: flex; 
  justify-content: center; align-items: center; 
}
.parchment { 
  background: #f5d6a7; width: 400px; padding: 30px; text-align: center; 
  border: 8px solid #8c5624; box-shadow: 10px 10px 0px rgba(0,0,0,0.5);
  font-family: 'LanaPixel', sans-serif;
}
h2 { color: #6F1D1B; margin-top: 0; }
.board { margin: 20px 0; border: 2px solid #8c5624; background: rgba(255, 255, 255, 0.3); }
.row { display: flex; justify-content: space-between; padding: 10px; border-bottom: 1px solid #8c5624; }
.header { font-weight: bold; background: #8c5624; color: #fff; }
.self { background: rgba(241, 196, 15, 0.4); font-weight: bold; color: #6F1D1B; }
.rank { width: 60px; }
.info { flex: 1; text-align: left; }
.score { width: 60px; }
.rewards { margin-top: 20px; font-size: 14px; color: #6F1D1B; border-top: 2px solid #8c5624; padding-top: 10px;}
.pixel-btn { 
  background-color: #6F1D1B; color: #fff; border: 3px solid #fff; 
  padding: 8px 20px; cursor: pointer; font-family: 'LanaPixel', sans-serif; margin-top: 10px;
}
</style>