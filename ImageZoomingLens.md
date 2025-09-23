Create a reusable file ImageZoomingLens.vue

    <template>
      <div
        ref="container"
        class="relative overflow-hidden"
        @mousemove="moveLens"
        @mouseenter="showLens"
        @mouseleave="hideLens"
    
        @touchstart.prevent="showLensTouch"
        @touchmove.prevent="moveLensTouch"
        @touchend="hideLens"
    
        :style="{ width: '100%', height: '85%', position: 'relative' }"
      >
        <!-- Original Image -->
        <img :src="src" :alt="alt" class="w-full h-full object-contain" />
    
        <!-- Lens (same container me zoom hoga) -->
        <div
          v-if="isZooming"
          ref="lens"
          class="absolute border-2 border-green-500 rounded-md pointer-events-none"
          :style="lensStyle"
        ></div>
      </div>
    </template>
    
    <script setup>
    import { ref } from "vue";
    
    const props = defineProps({
      src: String,
      alt: String,
      zoomLevel: { type: Number, default: 2 }, // zoom factor (2x, 3x, etc.)
    });
    
    const container = ref(null);
    const lens = ref(null);
    const isZooming = ref(false);
    const lensStyle = ref({});
    
    function showLens() {
      isZooming.value = true;
    }
    
    function hideLens() {
      isZooming.value = false;
    }
    
    function moveLens(e) {
      const containerRect = container.value.getBoundingClientRect();
      const lensSize = 250; // lens ka size
    
      // mouse ya touch dono ke liye clientX/clientY nikaalna
      const clientX = e.clientX ?? e.touches?.[0]?.clientX;
      const clientY = e.clientY ?? e.touches?.[0]?.clientY;
      if (!clientX || !clientY) return;
    
      const x = clientX - containerRect.left;
      const y = clientY - containerRect.top;
    
      // lens ka center calculation
      let lensX = x - lensSize / 2;
      let lensY = y - lensSize / 2;
    
      // container ke andar hi lens rahe
      if (lensX < 0) lensX = 0;
      if (lensY < 0) lensY = 0;
      if (lensX > containerRect.width - lensSize)
        lensX = containerRect.width - lensSize;
      if (lensY > containerRect.height - lensSize)
        lensY = containerRect.height - lensSize;
    
      // lens background position calculation
      const bgX = (x / containerRect.width) * 100;
      const bgY = (y / containerRect.height) * 100;
    
      lensStyle.value = {
        left: lensX + "px",
        top: lensY + "px",
        width: lensSize + "px",
        height: lensSize + "px",
        backgroundImage: `url(${props.src})`,
        backgroundRepeat: "no-repeat",
        backgroundSize: `${containerRect.width * props.zoomLevel}px ${
          containerRect.height * props.zoomLevel
        }px`,
        backgroundPosition: `${bgX}% ${bgY}%`,
      };
    }
    
    // 👇 Touch events
    function showLensTouch(e) {
      isZooming.value = true;
      moveLens(e); // start me hi lens position set kar do
    }
    
    function moveLensTouch(e) {
      moveLens(e);
    }
    </script>
    
    <style scoped>
    /* lens ko circular rakhna ho to: */
    .lens {
      border-radius: 50%;
    }
    </style>


Use this instead of simple image tag. First you need to import that ImageZoomingLens.vue like (
import ZoomImage from "@/Components/Shared/ZoomImage.vue";) and than use it

    <ZoomImage 
      :src="mainImage"
      :alt="product.name"
      :zoom-level="3"
    />
