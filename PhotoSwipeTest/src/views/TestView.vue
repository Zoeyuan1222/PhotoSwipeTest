<template>
  <ImageViewer galleryID="my-test-gallery" :images="imagesData" />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import ImageViewer from "@/components/ImageViewer.vue";

const imagesData = ref([
  {
    largeURL:
      'https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-2500.jpg',
    thumbnailURL:
      'https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-200.jpg',
    width: 1875,
    height: 2500,
    captionText: 'Caption for Image 1',
  },
  {
    largeURL:
      'https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-2500.jpg',
    thumbnailURL:
      'https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-200.jpg',
    width: 1669,
    height: 2500,
  },
  {
    largeURL:
      'https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-2500.jpg',
    thumbnailURL:
      'https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-200.jpg',
    width: 2500,
    height: 1666,
  },
]);
</script>
