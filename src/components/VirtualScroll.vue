<script setup lang="ts">
import type { CSSProperties } from 'vue'

type ItemData = Record<string, any>

interface Props {
  list: ItemData[]
  itemSize?: number
}

const props = withDefaults(defineProps<Props>(), {
  itemSize: 50,
})

const infiniteContainerRef = ref<HTMLDivElement>()

const scrollHeight = computed(() => props.list.length * props.itemSize)

const screenHeight = ref(0)

const scrollTop = ref(0)

const startIndex = ref(0)

const endIndex = ref(0)

const visibleCount = computed(() =>
  Math.ceil(screenHeight.value / props.itemSize),
)

const scrollOffset = computed(
  () => scrollTop.value - (scrollTop.value % props.itemSize),
)

const visibleList = computed(() =>
  props.list.slice(startIndex.value, Math.min(endIndex.value, props.list.length)),
)

const phantomStyle = computed<CSSProperties>(() => ({
  height: `${scrollHeight.value}px`,
}))

const listStyle = computed<CSSProperties>(() => ({
  transform: `translate3d(0,${scrollOffset.value}px,0)`,
}))

function handleScroll() {
  scrollTop.value = infiniteContainerRef.value!.scrollTop
  startIndex.value = Math.floor(scrollTop.value / props.itemSize)
  endIndex.value = startIndex.value + visibleCount.value
}

function init() {
  screenHeight.value = infiniteContainerRef.value?.clientHeight || 0
  startIndex.value = 0
  endIndex.value = startIndex.value + visibleCount.value
}

onMounted(() => init())
</script>

<template>
  <div
    ref="infiniteContainerRef"
    class="infinite-container"
    @scroll="handleScroll"
  >
    <div class="infinite-phantom" :style="phantomStyle" />
    <div
      class="infinite-list"
      :style="listStyle"
    >
      <div
        v-for="(data, index) in visibleList"
        :key="data.key"
        class="infinite-list-item"
      >
        <slot :data="data" :index="index" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.infinite-container {
  height: 100%;
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}

.infinite-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  z-index: -1;
}

.infinite-list {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
}

.infinite-list-item {
  padding: 10px;
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
}
</style>
