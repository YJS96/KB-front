<script setup lang="ts">
// import NavBar from '@/components/NavBar.vue'
const threshold = 120;
const pullDistance = ref(0);
const startY = ref(0);

const emit = defineEmits(['refresh']);

const onTouchStart = (e) => {
  startY.value = e.touches[0].clientY;
};

const onTouchMove = (e) => {
  const currentY = e.touches[0].clientY;
  pullDistance.value = Math.max(0, currentY - startY.value);
};

const onTouchEnd = () => {
  if (pullDistance.value > threshold) {
    emit('refresh');
  }
  pullDistance.value = 0;
};
</script>

<template>
  <div
    class="pull-to-refresh"
    @touchstart="onTouchStart"
    @touchmove="onTouchMove"
    @touchend="onTouchEnd"
  >
    <div class="pull-to-refresh__indicator" :style="{ height: `${pullDistance}px` }">
      {{ pullDistance > threshold ? '놓아서 새로고침' : '당겨서 새로고침' }}
    </div>
    <!-- <NavBar /> -->
    <RouterView />
  </div>
</template>

<style scoped></style>
