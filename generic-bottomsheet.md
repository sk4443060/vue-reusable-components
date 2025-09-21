[01] Create GenericBottomSheet.vue in reusable location

      <template>
        <transition name="slide-up">
          <div
            v-if="isVisible"
            class="fixed inset-0 z-50 flex items-end justify-center bg-black/50"
            @click.self="closeSheet"
          >
            <div
              class="w-full max-w-md bg-white rounded-t-2xl shadow-lg p-4"
              :style="{ maxHeight: '80vh' }"
            >
              <!-- Close Button -->
              <button
                class="absolute top-2 right-4 text-gray-500 hover:text-black"
                @click="closeSheet"
              >
                ✕
              </button>
      
              <!-- Dynamic Content Slot -->
              <div class="mt-6">
                <slot />
              </div>
            </div>
          </div>
        </transition>
      </template>
      
      <script>
      export default {
        name: "GenericBottomSheet",
        props: {
          show: {
            type: Boolean,
            default: false,
          },
          autoShowDelay: {
            type: Number,
            default: null, // ms (example: 3000 for 3s delay)
          },
          mobileOnly: {
            type: Boolean,
            default: true,
          },
        },
        data() {
          return {
            isVisible: this.show,
          };
        },
        watch: {
          show(val) {
            this.isVisible = val;
          },
        },
        mounted() {
          if (this.mobileOnly && !this.isMobileDevice()) return;
      
          if (this.autoShowDelay) {
            setTimeout(() => {
              this.isVisible = true;
              this.$emit("update:show", true);
            }, this.autoShowDelay);
          }
        },
        methods: {
          closeSheet() {
            this.isVisible = false;
            this.$emit("update:show", false);
          },
          isMobileDevice() {
            return window.innerWidth <= 768;
          },
        },
      };
      </script>
      
      <style scoped>
      /* Slide-up transition */
      .slide-up-enter-active,
      .slide-up-leave-active {
        transition: all 0.3s ease;
      }
      .slide-up-enter {
        transform: translateY(100%);
        opacity: 0;
      }
      .slide-up-leave-to {
        transform: translateY(100%);
        opacity: 0;
      }
    </style>

[02] Head over to the file location where you want to use this bottom sheet.

        <script setup>
          import { ref } from "vue";                                                          <- Import this one
          import GenericBottomSheet from "@/Components/Shared/GenericBottomSheet.vue";        <- Import this too (Location where you have created GenericBottomSheet.vue)
        
          const showSheet = ref(false);                                                       <- Set initially false
        </script>

[03] Create rendering slot like or the methods you know (Keep this inside <template> tag in the same file where you imported in [02]): 

      <GenericBottomSheet
        v-model:show="showSheet"
        :autoShowDelay="3000"
        :mobileOnly="true"
      >                                                                                       <- Note this opening tag
        <!-- Customizable Slot Content -->
        <div class="text-center">
          <img src="/promo.png" alt="Offer" class="mx-auto w-24 mb-4" />
          <h2 class="text-xl font-bold">Get Our App</h2>
          <p class="text-gray-600 mb-4">
            Download now and enjoy live tracking, exclusive offers, and faster checkout.
          </p>
  
          <button class="w-full py-2 bg-green-600 text-white rounded mb-2">
            Download App
          </button>
          <button
            class="w-full py-2 bg-gray-200 rounded"
            @click="showSheet = false"
          >
            Continue on Web
          </button>
        </div>
      </GenericBottomSheet>

[04] You have successfully 🎉 integrated the plugin

📌 Notes
---------------------------------------------------------------------------------------------------
Ab aapko GenericBottomSheet har jagah import karne ki zarurat nahi hai.
showSheet state Layout me centralized hai.
Aap content ko slot ke through easily customize kar sakte ho (promo image, heading, buttons, etc.).
Future me agar multiple promos chahiye, aap v-if / v-for ke through multiple sheets bhi manage kar sakte ho.

📌 Notes
---------------------------------------------------------------------------------------------------
Yadi aap chahte ho ki har page par navigate kro aur na bottom sheet execute ho just update <script> of your [01]

😄 Your explanation about localStorage vs sessionStorage is spot on:
localStorage → persists across browser sessions. Sheet will never show again once the flag is set.

sessionStorage → only lasts for the current tab/session. Sheet will show again in the next browser session.
Now, for your GenericBottomSheet.vue, you can make it dual-option with a prop, e.g., storageType: { type: String, default: "local" } (accepts "local" or "session"). Here’s how you can adjust your code:

true  -> Execute each and every time during page to page navigation
false -> Executes once

      <script>
            export default {
              name: "GenericBottomSheet",
              props: {
                show: { type: Boolean, default: false },
                autoShowDelay: { type: Number, default: null }, // ms
                mobileOnly: { type: Boolean, default: true },
                storageType: { type: String, default: "local" }, // "local" or "session"
                storageKey: { type: String, default: "bottomSheetShown" },
              },
              data() {
                return {
                  isVisible: this.show,
                };
              },
              watch: {
                show(val) {
                  this.isVisible = val;
                },
              },
              mounted() {
                if (this.mobileOnly && !this.isMobileDevice()) return;
            
                const storage =
                  this.storageType === "session" ? sessionStorage : localStorage;
            
                if (storage.getItem(this.storageKey) === "true") return;
            
                if (this.autoShowDelay) {
                  setTimeout(() => {
                    this.isVisible = true;
                    this.$emit("update:show", true);
                    storage.setItem(this.storageKey, "true"); // ✅ flag set
                  }, this.autoShowDelay);
                }
              },
              methods: {
                closeSheet() {
                  this.isVisible = false;
                  this.$emit("update:show", false);
                },
                isMobileDevice() {
                  return window.innerWidth <= 768;
                },
              },
            };
       </script>

