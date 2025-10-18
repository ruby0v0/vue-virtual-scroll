<script setup lang="ts">
const list = Array.from({ length: 10000 }, (_, i) => ({
  key: `${i}`,
  value: i,
  message: `item ${i}`,
}))
</script>

<template>
  <div>
    <VirtualScroll style="min-height: 240px" :list="list">
      <template #default="{ data }">
        <div class="flex items-center justify-center" :style="{ height: '50px' }">
          {{ data.message }}
        </div>
      </template>
    </VirtualScroll>
  </div>
</template>

<style scoped>

</style>
