<script setup lang="ts">
import { computed, ref } from 'vue';

const threshold = 180;
const maxIndicatorHeight = 60; // 최대 인디케이터 높이 설정
const pullDistance = ref(0);
const startY = ref(0);
const refreshState = ref('idle');

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

    setTimeout(() => {
      refreshState.value = 'idle';
    }, 1000);
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

const indicatorHeight = computed(() => {
  return Math.min(Math.max(pullDistance.value / 4, 0), maxIndicatorHeight);
});

const contentStyle = computed(() => ({
  transform: `translateY(${refreshState.value === 'refreshing' ? `${maxIndicatorHeight}px` : '0'})`,
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
    <div
      class="pull-to-refresh__indicator"
      :style="{
        height: `${indicatorHeight}px`,
        opacity: indicatorVisible ? 1 : 0
      }"
    >
      <div class="pull-to-refresh__indicator-content">
        <i class="fa-solid fa-arrow-up" :style="rotationStyle"></i>
        {{ indicatorText }}
      </div>
    </div>
    <div class="pull-to-refresh__content" :style="contentStyle">
      <!-- <NavBar /> -->
      <RouterView />
    </div>
  </div>
</template>

<style scoped>
.pull-to-refresh {
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
  height: 100vh;
}

.pull-to-refresh__indicator {
  position: absolute;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: flex-end;
  overflow: hidden;
  transition:
    height 0.3s ease,
    opacity 0.3s ease;
  z-index: 999;
  top: 0;
}

.pull-to-refresh__indicator-content {
  padding-bottom: 10px;
}

.pull-to-refresh__content {
  min-height: 100%;
  padding-top: 1px;
  margin-top: -1px;
}

i {
  margin-right: 4px;
}
</style>
