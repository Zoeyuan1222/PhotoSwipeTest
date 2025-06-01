<template>
  <ImageViewer galleryID="my-dynamic-gallery" :images="dynamicImagesData" />
  <button @click="openGallery">Open Dynamic Gallery</button>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import ImageViewer from "@/components/ImageViewer.vue"; // 确保路径正确

// 定义 ImageViewer 所需的 ImageItem 类型 (如果 ImageViewer.vue 导出了它，可以导入)
interface ImageItem {
  largeURL: string;
  thumbnailURL: string;
  width: number;
  height: number;
  captionText?: string;
  captionHTML?: string;
}

const dynamicImagesData = ref<ImageItem[]>([]);
const numberOfImages = 10; // 你想要动态生成的图片数量

const generateImageData = () => {
  const images: ImageItem[] = [];
  for (let i = 0; i < numberOfImages; i++) {
    const size = 100 + Math.floor(Math.random() * 200); // 随机尺寸示例
    images.push({
      largeURL: `https://dummyimage.com/${size}x${size}/555/fff/?text=${i + 1}`,
      thumbnailURL: `https://dummyimage.com/150x150/555/fff/?text=Thumb ${i + 1}`, // 缩略图也可以动态生成
      width: size,
      height: size,
      captionText: `Dynamic Image ${i + 1}`,
    });
  }
  dynamicImagesData.value = images;
};

// 在组件挂载时生成初始数据
onMounted(() => {
  generateImageData();
});

// 假设我们只是想显示动态生成的缩略图，点击缩略图打开
const openGallery = () => {
  console.log("Dynamic images are loaded. Click a thumbnail to open the gallery.");
};

</script>
