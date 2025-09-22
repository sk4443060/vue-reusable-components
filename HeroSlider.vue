<template>
  <div
    class="relative w-full overflow-hidden"
    @mouseenter="pauseSlider"
    @mouseleave="startSlider"
    @touchstart="onTouchStart"
    @touchmove="onTouchMove"
    @touchend="onTouchEnd"
  >
    <!-- Slides -->
    <div
      class="flex transition-transform duration-500"
      :style="{ transform: `translateX(-${activeIndex * 100}%)` }"
    >
      <div
        v-for="(slide, index) in slides"
        :key="index"
        class="w-full flex-shrink-0 relative"
      >
        <a v-if="slide.link" :href="slide.link">
          <picture>
            <source :srcset="slide.mobile" media="(max-width: 767px)" />
            <source :srcset="slide.tablet" media="(max-width: 1023px)" />
            <source :srcset="slide.desktop" media="(min-width: 1024px)" />
            <img :src="slide.desktop" alt="slide image" class="w-full object-contain" />
          </picture>
        </a>
        <picture v-else>
          <source :srcset="slide.mobile" media="(max-width: 767px)" />
          <source :srcset="slide.tablet" media="(max-width: 1023px)" />
          <source :srcset="slide.desktop" media="(min-width: 1024px)" />
          <img :src="slide.desktop" alt="slide image" class="w-full object-contain" />
        </picture>
      </div>
    </div>

    <!-- Indicators + Controls -->
    <div class="absolute bottom-2 left-1/2 -translate-x-1/2 flex items-center gap-2">
      <button @click="prevSlide" class="bg-white/70 px-2 rounded-sm shadow-md hover:bg-white/90">‹</button>
      <div class="flex gap-2">
        <button
          v-for="(_, index) in slides"
          :key="index"
          @click="goToSlide(index)"
          class="w-3 h-3 rounded-full"
          :class="activeIndex === index ? 'bg-green-600' : 'bg-gray-300'"
        ></button>
      </div>
      <button @click="nextSlide" class="bg-white/70 px-2 rounded-sm shadow-md hover:bg-white/90">›</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const slides = ref([
  {
    desktop: "assets/sliders-images/desktop-slider250922.webp",
    tablet: "assets/sliders-images/tab-slider250922.webp",
    mobile: "assets/sliders-images/mobile-slider250922.webp",
    link: "/"
  },
  {
    desktop: "assets/sliders-images/desktop-slider2509221.webp",
    tablet: "assets/sliders-images/tab-slider2509221.webp",
    mobile: "assets/sliders-images/mobile-slider2509221.webp",
    link: "/"
  },
  {
    desktop: "assets/sliders-images/desktop-slider2509222.webp",
    tablet: "assets/sliders-images/tab-slider2509222.webp",
    mobile: "assets/sliders-images/mobile-slider2509222.webp",
    link: "/"
  },
]);

const activeIndex = ref(0);
let timer = null;
const speed = 3000;

const nextSlide = () => (activeIndex.value = (activeIndex.value + 1) % slides.value.length);
const prevSlide = () => (activeIndex.value = (activeIndex.value - 1 + slides.value.length) % slides.value.length);
const goToSlide = (i) => (activeIndex.value = i);

const startSlider = () => {
  stopSlider();
  timer = setInterval(nextSlide, speed);
};
const stopSlider = () => timer && clearInterval(timer);
const pauseSlider = () => stopSlider();

onMounted(startSlider);
onUnmounted(stopSlider);

// -------------------------
// Touch/Swipe functionality
// -------------------------
const touchStartX = ref(0);
const touchEndX = ref(0);

const onTouchStart = (e) => {
  touchStartX.value = e.changedTouches[0].screenX;
  pauseSlider(); // pause while swiping
};

const onTouchMove = (e) => {
  touchEndX.value = e.changedTouches[0].screenX;
};

const onTouchEnd = () => {
  const swipeDistance = touchEndX.value - touchStartX.value;
  const minSwipe = 50; // minimum swipe distance to trigger slide
  if (swipeDistance > minSwipe) {
    prevSlide();
  } else if (swipeDistance < -minSwipe) {
    nextSlide();
  }
  startSlider(); // resume slider
};
</script>
