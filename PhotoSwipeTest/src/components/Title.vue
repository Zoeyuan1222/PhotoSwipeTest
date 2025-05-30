<script setup lang="ts">
defineProps<{
  msg: string
}>()
</script>

<template>
  <div class="flex flex-col w-full p-5 items-center">
    <div class="font-bold text-5xl w-full p-5 text-green-500 text-center">{{ msg }}</div>
    <div class="font-bold text-xl text-green-600 text-center">
      测试photoswipe可用性
    </div>
  </div>
</template>
