<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import Title from './components/Title.vue'
</script>

<template>
  <header class="flex">
    <div class="flex flex-col w-full justify-center">
      <a class="text-center" href="https://photoswipe.com">Photoswipe官方网站</a>
      <Title msg="PhotoSwipe Test" />
      <div class="flex px-20 justify-between">
        <RouterLink to="/test">测试用例</RouterLink>
        <RouterLink to="/">Home</RouterLink>
      </div>
    </div>
  </header>
  <RouterView />
</template>
