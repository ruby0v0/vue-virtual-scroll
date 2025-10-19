<script setup lang="ts">
import type { CSSProperties } from 'vue'

interface Position {
  index: number
  height: number
  top: number
  bottom: number
}

type ItemData = Record<string, any>

interface Props {
  /** 需要展示的数据 */
  list: ItemData[]
  /** 项目高度 */
  itemSize?: number
}

const props = withDefaults(defineProps<Props>(), {
  itemSize: 50,
})

const containerRef = ref<HTMLDivElement>()

const listRef = ref<HTMLDivElement>()

const itemRef = ref<HTMLDivElement[]>()

const scrollHeight = ref(0)

const positions = ref<Position[]>([])

const screenHeight = ref(0)

const scrollTop = ref(0)

const scrollOffset = ref(0)

const startIndex = ref(0)

const endIndex = ref(0)

const _list = computed(() =>
  props.list.map(item => ({
    ...item,
  })),
)

const visibleCount = computed(() =>
  Math.ceil(screenHeight.value / props.itemSize),
)

const visibleList = computed(() =>
  _list.value.slice(
    startIndex.value,
    Math.min(endIndex.value, props.list.length),
  ),
)

const phantomStyle = computed<CSSProperties>(() => ({
  height: `${scrollHeight.value}px`,
}))

const listStyle = computed<CSSProperties>(() => ({
  transform: `translate3d(0,${scrollOffset.value}px,0)`,
}))

function binarySearch(list: Position[], value: number) {
  let start = 0
  let end = list.length - 1
  let tempIndex: number | null = null

  while (start <= end) {
    const midIndex = Math.floor((start + end) / 2)
    const midValue = list[midIndex]!.bottom
    if (midValue === value) {
      tempIndex = midIndex
      break
    }
    else if (midValue < value) {
      start = midIndex + 1
    }
    else if (midValue > value) {
      if (tempIndex === null || tempIndex > midIndex) {
        tempIndex = midIndex
      }
    }
    end = midIndex - 1
  }
  return tempIndex as number
}

function getStartIndex(scrollTop: number = 0) {
  return binarySearch(positions.value, scrollTop)
}

function setScrollOffset() {
  scrollOffset.value
    = startIndex.value >= 1 ? positions.value[startIndex.value - 1]!.bottom : 0
}

function handleScroll() {
  scrollTop.value = containerRef.value!.scrollTop
  startIndex.value = getStartIndex(scrollTop.value)
  endIndex.value = startIndex.value + visibleCount.value
  setScrollOffset()
}

function updateItemSize() {
  itemRef.value?.forEach((node) => {
    const rect = node.getBoundingClientRect()
    const height = rect.height
    const index = +node.id.slice(1)
    const oldHeight = positions.value[index]!.height
    const diff = oldHeight - height
    if (diff) {
      positions.value[index]!.bottom -= diff
      positions.value[index]!.height = height
      for (let i = index + 1; i < positions.value.length; i++) {
        positions.value[i]!.top = positions.value[i - 1]!.bottom
        positions.value[i]!.bottom -= diff
      }
    }
  })
}

function initPositions() {
  positions.value = props.list.map((_, index) => ({
    index,
    height: props.itemSize,
    top: index * props.itemSize,
    bottom: (index + 1) * props.itemSize,
  }))
}

function init() {
  screenHeight.value = containerRef.value!.clientHeight
  startIndex.value = 0
  endIndex.value = startIndex.value + visibleCount.value
  initPositions()
}

onMounted(() => init())

onUpdated(() => {
  nextTick(() => {
    if (!itemRef.value || !itemRef.value.length)
      return
    scrollHeight.value = positions.value[positions.value.length - 1]!.bottom
    updateItemSize()
  })
})
</script>

<template>
  <div ref="containerRef" class="infinite-container" @scroll="handleScroll">
    <div class="infinite-phantom" :style="phantomStyle" />
    <div ref="listRef" class="infinite-list" :style="listStyle">
      <div
        v-for="(data, index) in visibleList"
        ref="itemRef"
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
  bottom: 0;
}

.infinite-list-item {
  padding: 5px;
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
}
</style>
