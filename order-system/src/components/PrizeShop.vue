<template>
  <div class="prize-shop">
    <h2>🎁 冒險者現實美食兌換處</h2>
    <p class="desc">努力背單字打倒惡龍！累積的積分可以在這裡找老師兌換真正的夜市美食與珍奶喔！</p>
    <div class="prizes-grid">
      <div v-for="prize in prizes" :key="prize.id" class="prize-card">
        <div class="prize-icon">{{ prize.icon }}</div>
        <h3>{{ prize.name }}</h3>
        <p class="prize-desc">{{ prize.description }}</p>
        <p class="cost">💎 {{ prize.cost }} 積分</p>
        
        <button class="pixel-btn buy-btn" :class="{ disabled: points < prize.cost }" @click="$emit('spend-points', prize.cost)">
          {{ points >= prize.cost ? '立即兌換' : '積分不足' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({ points: { type: Number, required: true } })
defineEmits(['spend-points'])

const prizes = [
  { id: 1, name: '養樂多一瓶', icon: '🍼', cost: 1, description: '冰涼酸甜的基礎體力回復劑' },
  { id: 2, name: '科學麵一包', icon: '🍜', cost: 2, description: '嘎嘣脆的捏碎鹹香充飢點心' },
  { id: 3, name: '經典珍珠奶茶 (小杯)', icon: '🧋', cost: 4, description: '精緻小巧的療癒系快樂水' },
  { id: 4, name: '經典珍珠奶茶 (大杯)', icon: '🥤', cost: 6, description: '特大容量！雙倍Q彈珍珠的絕對滿足' },
  { id: 5, name: '現炸黃金雞排 (半張)', icon: '🍗', cost: 9, description: '外脆內嫩，肉汁滿滿的戰力補給' },
  { id: 6, name: '豪華套餐：雞排 + 大杯珍奶', icon: '🏆', cost: 12, description: '傳說級終極美食！滿分勇者的無上榮耀' }
]
</script>

<style scoped>
.prize-shop { width: 100%; background-color: #6F1D1B; padding: 40px; border: 6px solid #4a1211; box-shadow: 6px 6px 0px #000; text-align: center; }
h2 { color: #f1c40f; margin-top: 0; font-size: 28px; text-shadow: 2px 2px 0 #000; }
.desc { background-color: rgba(0,0,0,0.25); padding: 12px; border-radius: 4px; margin-bottom: 30px; color: #fceae9; }
.prizes-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 25px; }
.prize-card { background-color: #ffffff; border: 4px solid #4a1211; padding: 25px 15px; display: flex; flex-direction: column; align-items: center; box-shadow: 4px 4px 0 #000; }
.prize-icon { font-size: 55px; margin-bottom: 12px; background-color: #FAF4F2; border: 3px solid #6F1D1B; border-radius: 50%; width: 90px; height: 90px; display: flex; align-items: center; justify-content: center; box-shadow: 3px 3px 0px #000; }
h3 { margin: 10px 0 5px 0; font-size: 20px; color: #6F1D1B; line-height: 1.3; }
.prize-desc { font-size: 13px; color: #7f8c8d; margin: 5px 0 15px 0; min-height: 36px; }
.cost { color: #fff; font-weight: bold; font-size: 18px; margin: auto 0 15px 0; background: #8c2624; padding: 4px 12px; border: 2px solid #6F1D1B; }
.buy-btn { width: 100%; background-color: #6F1D1B; border-color: #4a1211; }
.buy-btn:hover { background-color: #8c2624; }
.buy-btn.disabled { background-color: #bdc3c7; color: #7f8c8d; cursor: not-allowed; box-shadow: none; transform: none; border-color: #95a5a6; }
</style>