<script setup lang="ts">
import { computed, ref } from 'vue';

const threshold = 180; // 새로고침을 트리거하는 당김 거리 (픽셀)
const pullDistance = ref(0);
const startY = ref(0);
const randomNumber = ref(Math.floor(Math.random() * 1000)); // 초기 랜덤 숫자 생성
const isRefreshing = ref(false);

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
    startRefresh();
  } else {
    pullDistance.value = 0;
  }
};

const startRefresh = () => {
  isRefreshing.value = true;
  setTimeout(() => {
    refreshContent();
    isRefreshing.value = false;
    pullDistance.value = 0;
  }, 1000);
};

const refreshContent = () => {
  // 새로운 랜덤 숫자 생성
  randomNumber.value = Math.floor(Math.random() * 1000);
  emit('refresh');
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
    <div class="pull-to-refresh__indicator" :style="{ height: `${pullDistance / 1.6}px` }">
      <template v-if="!isRefreshing">
        <i class="fa-solid fa-arrow-up" :style="rotationStyle"></i>
        {{ pullDistance > threshold ? '놓아서 새로고침' : '당겨서 새로고침' }}
      </template>
      <template v-else>
        새로고침 중
      </template>
    </div>
    <div class="content" :style="{ marginTop: isRefreshing ? '40px' : '0' }">
      <!-- <NavBar /> -->
      <RouterView />
    </div>
    <div class="random-number">랜덤 숫자: {{ randomNumber }}</div>
  </div>
</template>

<style scoped>
.pull-to-refresh {
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
  height: 100vh;
}

.pull-to-refresh__indicator {
  position: fixed;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 0;
  overflow: hidden;
  transition: height 0.1s ease;
  z-index: 999;
  top: 0 !important;
}

.content {
  transition: margin-top 0.3s ease;
}

i {
  margin-right: 4px;
}

.random-number {
  position: fixed;
  bottom: 20px;
  left: 20px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 10px;
  border-radius: 5px;
  z-index: 999;
}
</style>