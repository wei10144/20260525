<template>
  <div class="particles-container">
    <div 
      v-for="n in 20" 
      :key="n" 
      class="particle" 
      :style="getParticleStyle(n)"
    >
      {{ getRandomEmoji() }}
    </div>
  </div>
</template>

<script setup>
const emojis = ['✨', '💎', '⚔️', '🛡️', '✦', '✧', '⭐']

const getRandomEmoji = () => emojis[Math.floor(Math.random() * emojis.length)]

const getParticleStyle = (n) => {
  return {
    top: `${Math.random() * 100}%`,
    left: `${Math.random() * 100}%`,
    animationDelay: `${Math.random() * 10}s`,
    animationDuration: `${10 + Math.random() * 20}s`,
    fontSize: `${10 + Math.random() * 20}px`
  }
}
</script>

<style scoped>
.particles-container {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
  pointer-events: none; z-index: 0; overflow: hidden;
}
.particle {
  position: absolute; opacity: 0.4;
  animation: float linear infinite;
}
@keyframes float {
  0% { transform: translate(0, 0); }
  33% { transform: translate(10vw, 20vh); }
  66% { transform: translate(-10vw, -20vh); }
  100% { transform: translate(0, 0); }
}
</style>