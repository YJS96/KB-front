<script setup lang="ts">
import { computed, ref } from 'vue';

// import NavBar from '@/components/NavBar.vue'

const threshold = 180; // 새로고침을 트리거하는 당김 거리 (픽셀)
const pullDistance = ref(0);
const startY = ref(0);
const refreshState = ref('idle'); // 'idle', 'pulling', 'refreshing'

const emit = defineEmits(['refresh']);

const onTouchStart = (e: TouchEvent) => {
  if (refreshState.value === 'idle') {
    startY.value = e.touches[0].clientY;
    refreshState.value = 'pulling';
  }
};

const onTouchMove = (e: TouchEvent) => {
  if (refreshState.value === 'pulling') {
    const currentY = e.touches[0].clientY;
    pullDistance.value = Math.max(0, currentY - startY.value);
  }
};

const onTouchEnd = () => {
  if (pullDistance.value > threshold && refreshState.value === 'pulling') {
    refreshState.value = 'refreshing';
    emit('refresh');

    // 새로고침 상태를 시뮬레이션합니다
    setTimeout(() => {
      refreshState.value = 'idle';
    }, 1000); // '새로고침 중' 상태를 2초 동안 유지
  } else if (refreshState.value === 'pulling') {
    refreshState.value = 'idle';
  }
  pullDistance.value = 0;
};

const isOverThreshold = computed(() => pullDistance.value > threshold);
const rotationStyle = computed(() => ({
  transform: `rotate(${isOverThreshold.value ? 180 : 0}deg)`,
  transition: 'transform 0.2s ease'
}));

const indicatorText = computed(() => {
  switch (refreshState.value) {
    case 'refreshing':
      return '새로고침 중';
    case 'pulling':
      return pullDistance.value > threshold ? '놓아서 새로고침' : '당겨서 새로고침';
    default:
      return '';
  }
});

const indicatorVisible = computed(() => refreshState.value !== 'idle' || pullDistance.value > 0);
</script>

<template>
  <div
    class="pull-to-refresh"
    @touchstart="onTouchStart"
    @touchmove="onTouchMove"
    @touchend="onTouchEnd"
  >
    <div
      class="pull-to-refresh__indicator"
      :style="{
        height: indicatorVisible ? `${Math.max(pullDistance / 1.6, 40)}px` : '0px',
        opacity: indicatorVisible ? 1 : 0
      }"
    >
      <i class="fa-solid fa-arrow-up" :style="rotationStyle"></i>
      {{ indicatorText }}
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
  position: fixed;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  transition:
    height 0.3s ease,
    opacity 0.3s ease;
  z-index: 999;
  top: 0 !important;
}

i {
  margin-right: 4px;
}
</style>
