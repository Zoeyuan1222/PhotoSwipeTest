<template>
  <div :id="props.galleryID">
    <a
      v-for="(image, key) in imagesData"
      :key="key"
      :href="image.largeURL"
      :data-pswp-width="image.width"
      :data-pswp-height="image.height"
      :data-pswp-caption="image.captionText || ''"
      class="pswp-gallery__item"
      rel="noreferrer"
    >
    <img :src="image.thumbnailURL" :alt="image.captionText || `Image ${key + 1} thumbnail`" />
    </a>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import PhotoSwipeLightbox from 'photoswipe/lightbox';
import 'photoswipe/style.css';

// 定义图片对象的数据结构
interface ImageItem {
  largeURL: string; // 大图的 URL
  thumbnailURL: string; // 缩略图的 URL
  width: number; // 大图宽度
  height: number; // 大图高度
  captionText?: string; // 用于 alt 和备用标题
}

// 定义组件的props
interface Props {
  galleryID: string; // 画廊的唯一 ID (用于初始化 PhotoSwipe)
  images: ImageItem[]; // 图片数据数组
}
const props = defineProps<Props>()

const imagesData = ref(props.images)

// 监听传入的图片数据（props.images）是否变化，并同步更新本地响应式数据（imagesData）
// 以便组件内部逻辑（如 lightbox 显示）能响应这些变化。
watch(() => props.images, (newImages) => {
  imagesData.value = newImages
  initializeLightbox();
}, { deep: true })

// 创建一个名为 'lightbox' 的响应式引用，用于存储 PhotoSwipeLightbox 的实例。
// 它初始化为 'null'，在 PhotoSwipe 初始化后会被赋值为其实例。
const lightbox = ref<PhotoSwipeLightbox | null>(null);

// 负责 PhotoSwipe 灯箱的设置和初始化
function initializeLightbox() {
  // 销毁之前的 PhotoSwipe 灯箱，重新初始化
  if (lightbox.value) {
    lightbox.value.destroy();
    lightbox.value = null;
  }

  if (props.galleryID && imagesData.value.length > 0) {
    const options = {
      gallery: '#' + props.galleryID, // CSS 选择器，指向画廊的容器
      children: '.pswp-gallery__item', // CSS 选择器，指向单个画廊项目
      pswpModule: () => import('photoswipe'), // 动态导入 PhotoSwipe 核心模块，按需加载
    };

    // 使用指定的选项创建一个新的 PhotoSwipeLightbox 实例。
    const localLightbox = new PhotoSwipeLightbox(options);

    localLightbox.on('uiRegister', function() {
      if (!localLightbox.pswp) return;
      // --- 注册项目符号指示器 ---
      localLightbox.pswp.ui.registerElement({
        name: 'bulletsIndicator',
        className: 'pswp__bullets-indicator',
        appendTo: 'wrapper',
        onInit: (el, pswpInstance) => {
          const bullets: HTMLDivElement[] = [];
          let bullet: HTMLDivElement;
          let prevIndex = -1;
          el.innerHTML = '';
          for (let i = 0; i < pswpInstance.getNumItems(); i++) {
            bullet = document.createElement('div');
            bullet.className = 'pswp__bullet';
            bullet.onclick = (e) => {
              if (e.target instanceof HTMLDivElement) {
                pswpInstance.goTo(bullets.indexOf(e.target));
              }
            };
            el.appendChild(bullet);
            bullets.push(bullet);
          }
          pswpInstance.on('change', () => {
            if (prevIndex >= 0 && bullets[prevIndex]) {
              bullets[prevIndex].classList.remove('pswp__bullet--active');
            }
            if (bullets[pswpInstance.currIndex]) {
              bullets[pswpInstance.currIndex].classList.add('pswp__bullet--active');
            }
            prevIndex = pswpInstance.currIndex;
          });
          if (bullets.length > 0 && pswpInstance.currIndex >= 0 && bullets[pswpInstance.currIndex]) {
            bullets[pswpInstance.currIndex].classList.add('pswp__bullet--active');
            prevIndex = pswpInstance.currIndex;
          }
        }
      });

      // --- 注册自定义标题 UI ---
      localLightbox.pswp.ui.registerElement({
        name: 'custom-caption', // UI 元素名称
        order: 9,              // 显示顺序，可以调整
        isButton: false,       // 表明它不是一个按钮
        appendTo: 'root',      // 附加到 PhotoSwipe 的根元素 (pswp__bg 的兄弟)
                               // 或者 'wrapper' 如果想让它在图片内容区内
        html: '',             // 初始 HTML 内容，可以为空，后续动态更新
        onInit: (el, pswpInstance) => { // el 是标题的 DOM 元素
          // 监听图片切换事件
          pswpInstance.on('change', () => {
            // pswpInstance.currSlide 可能是 null，特别是在关闭过程中
            if (!pswpInstance.currSlide || !pswpInstance.currSlide.data) {
              el.innerHTML = ''; // 清空标题
              return;
            }

            const currSlideElement = pswpInstance.currSlide.data.element as HTMLElement; // 获取当前幻灯片对应的原始DOM元素
            let captionHTML = '';

            if (currSlideElement) {
              // 尝试从 class="hidden-caption-content" 的元素获取 HTML 标题
              const hiddenCaption = currSlideElement.querySelector('.hidden-caption-content');
              if (hiddenCaption) {
                captionHTML = hiddenCaption.innerHTML;
              } else {
                // 如果没有，则尝试从 <img> 标签的 alt 属性获取文本标题
                const imgElement = currSlideElement.querySelector('img');
                if (imgElement) {
                  captionHTML = imgElement.getAttribute('alt') || '';
                }
              }
            }
            el.innerHTML = captionHTML; // 更新标题元素的 HTML 内容
          });

          // 初始加载时也设置一次标题 (可选，因为 change 事件通常会立即触发一次)
          // 确保在 pswpInstance.currSlide 和其 data 存在时才执行
          if (pswpInstance.currSlide && pswpInstance.currSlide.data) {
            const initialSlideElement = pswpInstance.currSlide.data.element as HTMLElement;
            if (initialSlideElement) {
              const hiddenCaption = initialSlideElement.querySelector('.hidden-caption-content');
              if (hiddenCaption) {
                el.innerHTML = hiddenCaption.innerHTML;
              } else {
                const imgElement = initialSlideElement.querySelector('img');
                if (imgElement) {
                  el.innerHTML = imgElement.getAttribute('alt') || '';
                }
              }
            }
          }
        }
      });
    });

    localLightbox.init();
    lightbox.value = localLightbox;
  }
}

onMounted(() => {
  initializeLightbox();
});

onBeforeUnmount(() => {
  if (lightbox.value) {
    lightbox.value.destroy();
    lightbox.value = null;
  }
});

</script>

<style>
/* --- 项目符号指示器样式 (来自上一个回答) --- */
.pswp__bullets-indicator {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1550;
  padding: 5px;
}
.pswp__bullet {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.5);
  margin: 0 6px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
  box-shadow: 0 0 5px rgba(0,0,0,0.2);
}
.pswp__bullet:hover {
  background-color: rgba(255, 255, 255, 0.8);
}
.pswp__bullet--active {
  background-color: #4CAF50;
  transform: scale(1.2);
}

/* --- 自定义标题样式 --- */
.pswp__custom-caption { /* 类名与 registerElement 中的 name 对应 (PhotoSwipe会自动加前缀pswp__) */
  position: absolute;
  top: 20px; /* 定位到顶部 */
  left: 50%;
  transform: translateX(-50%);
  color: white;
  background-color: rgba(0, 0, 0, 0.5); /* 半透明背景 */
  padding: 8px 15px;
  border-radius: 4px;
  font-size: 16px;
  text-align: center;
  z-index: 1550; /* 确保在其他 UI 之上 */
  max-width: 80%; /* 防止标题过长 */
  box-sizing: border-box;
}

/* 如果标题内容可能包含 HTML，你可能需要为其中的特定标签设置样式 */
.pswp__custom-caption a {
  color: #8ab4f8; /* 示例链接颜色 */
  text-decoration: none;
}
.pswp__custom-caption a:hover {
  text-decoration: underline;
}
</style>
