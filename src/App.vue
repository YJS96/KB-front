<script setup lang="ts">
import { computed, ref } from 'vue';

// import NavBar from '@/components/NavBar.vue'

const threshold = 180; // 새로고침을 트리거하는 당김 거리 (픽셀)
const pullDistance = ref(0);
const startY = ref(0);

const emit = defineEmits(['refresh']);

const onTouchStart = (e: TouchEvent) => {
  startY.value = e.touches[0].clientY;
};

const onTouchMove = (e: TouchEvent) => {
  const currentY = e.touches[0].clientY;
  pullDistance.value = Math.max(0, currentY - startY.value);
};

const onTouchEnd = () => {
  if (pullDistance.value > threshold) {
    emit('refresh');
  }
  pullDistance.value = 0;
};

const isOverThreshold = computed(() => pullDistance.value > threshold);
const rotationStyle = computed(() => ({
  transform: `rotate(${isOverThreshold.value ? 180 : 0}deg)`,
  transition: 'transform 0.3s ease'
}));
</script>

<template>
  <div
    class="pull-to-refresh"
    @touchstart="onTouchStart"
    @touchmove="onTouchMove"
    @touchend="onTouchEnd"
  >
    <div class="pull-to-refresh__indicator" :style="{ height: `${pullDistance}px` }">
      <i class="fa-solid fa-arrow-up" :style="rotationStyle"></i>
      {{ pullDistance > threshold ? '놓아서 새로고침' : '당겨서 새로고침' }}
    </div>
    <!-- <NavBar /> -->
    <RouterView />
  </div>
</template>

<style scoped>
.pull-to-refresh {
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
}

.pull-to-refresh__indicator {
  position: absolute;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 0;
  overflow: hidden;
  transition: height 0.3s ease;
  z-index: 999;
  top: 0 !important;
}
</style>
