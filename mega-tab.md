Create A file with name CategoryTabs.vue in laravel file directory resources/js/Components/Shared/

    <template>
      <div class="relative bg-white shadow-md border-b">
        <!-- Tabs (scrollable top) -->
        <div class="overflow-x-auto no-scrollbar px-4 py-2 ">
          <div class="flex space-x-4 w-max">
            <button
              v-for="cat in categories"
              :key="cat.name"
              :aria-expanded="activeCategory?.name === cat.name ? 'true' : 'false'"
              class="flex items-center gap-2 px-3 py-2 rounded-lg whitespace-nowrap transition border"
              :class="activeCategory?.name === cat.name
                      ? 'bg-[#65a30d] text-white'
                      : 'hover:bg-[#facc15] text-gray-800'"
              @click="onCategoryClick($event, cat)"
            >
              <!-- If you only have icon component -->
              <!-- <component
                :is="cat.icon"
                class="w-5 h-5"
                :class="activeCategory?.name === cat.name ? 'text-white' : 'text-[#65a30d]'"
              /> -->
    
              <!-- OR -->
    
              <!-- You want to show either image URL or Vue icon component -->
              <!-- Show Image if icon is URL -->
              <template v-if="typeof cat.icon === 'string'">
                <img
                  :src="cat.icon"
                  alt=""
                  class="w-5 h-5 rounded object-contain"
                />
              </template>
    
              <!-- AND Also use this block-->
              <!-- Otherwise show Vue icon component -->
              <template v-else>
                <component
                  :is="cat.icon"
                  class="w-5 h-5"
                  :class="activeCategory?.name === cat.name ? 'text-white' : 'text-[#65a30d]'"
                />
              </template>
    
    
              <span class="font-medium text-sm">{{ cat.name }}</span>
            </button>
          </div>
        </div>
    
        <!-- Dropdown via Teleport -->
        <teleport to="body">
          <div
            v-if="activeCategory"
            ref="dropdown"
            class="absolute left-0 right-0 w-full flex justify-center z-50 bg-white"
            :style="{ top: dropdownStyle.top }"
          >
            <div
              class="w-full max-w-screen-xl mx-4 md:mx-auto bg-white rounded-lg border shadow-lg p-4"
              role="dialog"
              aria-modal="true"
            >
              <div class="flex gap-4">
                <!-- Left Column (Subcategories) -->
                <div class="flex-shrink-0 w-20 sm:w-36 md:w-56 max-h-[60vh] overflow-y-auto pr-2">
                  <div class="grid grid-cols-1 gap-3">
                    <template v-for="(sub, i) in activeCategory.children || []" :key="i">
                      <button
                        v-if="!sub.href && !sub.waLink"
                        @click="selectSubCategory(sub)"
                        :class="[ 
                          'aspect-square flex flex-col items-center justify-center p-2 rounded-lg border transition',
                          selectedSubCategory === sub
                            ? 'bg-[#f0f7e6] font-semibold border-[#d7eacb]'
                            : 'hover:bg-[#f7fbef]'
                        ]"
                      >
                        <div class="w-full h-full sm:w-full sm:h-full bg-gray-100 rounded flex items-center justify-center mb-2 overflow-hidden">
                          <img
                            v-if="sub.image"
                            :src="sub.image"
                            alt=""
                            class="w-full h-full object-cover"
                          />
                          <span v-else class="text-xs text-gray-600">
                            {{ typeof sub === 'string' ? sub[0] : (sub?.name?.[0] || '?') }}
                          </span>
                        </div>
                        <span class="text-[10px] sm:text-xs text-center px-1 leading-tight break-words">
                          {{ typeof sub === 'string' ? sub : (sub.name || sub.label) }}
                        </span>
                      </button>
    
                      <button
                        v-else
                        class="aspect-square flex flex-col items-center justify-center p-2 rounded-lg border transition hover:bg-[#f7fbef]"
                        @click="handleSubCategoryLink(sub)"
                      >
                        <div class="w-full h-full bg-gray-100 rounded flex items-center justify-center mb-2 overflow-hidden">
                          <img
                            v-if="sub.image"
                            :src="sub.image"
                            alt=""
                            class="w-full h-full object-cover"
                          />
                          <span class="text-xs text-gray-600">
                            {{ sub.name?.[0] || '?' }}
                          </span>
                        </div>
                        <span class="text-[10px] sm:text-xs text-center px-1 leading-tight break-words">
                          {{ sub.name }}
                        </span>
                      </button>
                    </template>
                  </div>
                </div>
    
                <!-- Right Column (Sub-subcategories grid) -->
                <div class="flex-1 max-h-[62vh] overflow-y-auto">
                  <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4">
                    <template v-if="selectedSubCategory && subSubCategoriesFor(selectedSubCategory).length">
                      <button
                        v-for="(item, idx) in subSubCategoriesFor(selectedSubCategory)"
                        :key="idx"
                        class="aspect-square flex flex-col items-center justify-center border rounded-lg p-3 hover:shadow-md transition text-center"
                        @click="handleSubSubClick(item)"
                      >
                        <img v-if="item.image" :src="item.image" alt="" class="w-full h-full object-cover mb-2" />
                        <div class="text-xs font-medium">{{ item.name || item.label }}</div>
                      </button>
                    </template>
                    <template v-else>
                      <div class="col-span-full text-center text-gray-500 py-8">
                        Select a subcategory to view items
                      </div>
                    </template>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </teleport>
      </div>
    </template>
    
    <script setup>
    import { ref, reactive, nextTick, watch, onMounted, onBeforeUnmount } from 'vue'
    import { Home, Tag, CreditCard, Printer, PenTool, FileText, Layers, BookOpen, ShoppingCart } from 'lucide-vue-next'
    import { router } from '@inertiajs/vue3'
    
    /* Main Categories Tabs */
    const categories = [
    
      
      // If there is sub-categories in category than use like this
      // NOTE: You need to map sub-sub category or Reactive Sub-sub category mapping
      { name: "Grocery", icon: "/assets/gif/hot-deal-fire.gif", children: [
      // { name: "Grocery", icon: ShoppingCart, children: [
          { name: "Foodgrains, Oil & Masala", image: "https://placehold.co/400" },
          { name: "Fruits & Vegetables", image: "https://placehold.co/400" },
          { name: "Exotic Fruits & Veggies", image: "https://placehold.co/400" },
          
          // This has no sub-sub categories
          { name: "Dairy & Bakery", href: "/bikes/accessories", image: "https://placehold.co/400" },
        ]
      },
    
      { name: "Printing Services", icon: Printer, href: "/print" },
      { name: "Printout & Scanning", icon: FileText, waLink: "https://wa.me/9888034024?text=I'm%20inquiring%20about%20Photocopy%20%26%20Scanning" },
      { name: "Binding & Lamination", icon: Layers, waLink: "https://wa.me/9888034024?text=I'm%20inquiring%20about%20Binding%20%26%20Lamination" },
      { name: "Business Card", icon: CreditCard, waLink: "https://wa.me/9888034024?text=I'm%20inquiring%20about%20Business%20Card" },
      { name: "Design Services", icon: PenTool, waLink: "https://wa.me/9888034024?text=I'm%20inquiring%20about%20Design%20Services" },
      { name: "Custom Orders", icon: BookOpen, waLink: "https://wa.me/9888034024?text=I'm%20inquiring%20about%20Custom%20Orders" },
    
      // { name: "Home", icon: Home, href: "/" },
      { name: "Deals", icon: Tag, href: "/deals" },
    ]
    
    /* Sub-sub category mapping */
    const subSubCategories = {
      // "Foodgrains, Oil & Masala": Array.from({ length: 12 }, (_, i) => ({ name: `Food Grain ${i+1}`, image: "https://placehold.co/400", href: `/bikes/sports/${i+1}` })),
      
      // This statically serves the sub-sub categories for "Foodgrains, Oil & Masala"
      // And do not delete below code for future reference
      "Foodgrains, Oil & Masala": [
        { name: "Atta & Flours", image: "https://placehold.co/400", href: "/grocery/foodgrains/atta-flours" },
        { name: "Rice & Rice Products", image: "https://placehold.co/400", href: "/grocery/foodgrains/rice" },
        { name: "Pulses", image: "https://placehold.co/400", href: "/grocery/foodgrains/pulses" },
        { name: "Edible Oils & Ghee", image: "https://placehold.co/400", href: "/grocery/foodgrains/oils-ghee" },
        { name: "Masala & Spices", image: "https://placehold.co/400", href: "/grocery/foodgrains/spices" },
        { name: "Salt, Sugar & Jaggery", image: "https://placehold.co/400", href: "/grocery/foodgrains/salt-sugar" },
        { name: "Poha & Flattened Rice", image: "https://placehold.co/400", href: "/grocery/foodgrains/poha" },
        { name: "Sooji & Rava", image: "https://placehold.co/400", href: "/grocery/foodgrains/sooji" },
        { name: "Besan & Gram Flour", image: "https://placehold.co/400", href: "/grocery/foodgrains/besan" },
        { name: "Jaggery Products", image: "https://placehold.co/400", href: "/grocery/foodgrains/jaggery" },
        { name: "Organic Staples", image: "https://placehold.co/400", href: "/grocery/foodgrains/organic" },
        { name: "Ready Mixes (Dosa, Idli)", image: "https://placehold.co/400", href: "/grocery/foodgrains/ready-mix" }
      ],
    
      // This dynamically serves the sub-sub categories for "Foodgrains, Oil & Masala"
      // "Foodgrains, Oil & Masala": Array.from({ length: 12 }, (_, i) => ({ name: `Food Grain ${i+1}`, image: "https://placehold.co/400", href: `/grocery/foodgrains-Oil-&-Masala/${i+1}` })),
    
      "Fruits & Vegetables": Array.from({ length: 8 }, (_, i) => ({ name: `Fruits & Vegetables ${i+1}`, image: "https://placehold.co/400", href: `/bikes/cruiser/${i+1}` })),
      "Exotic Fruits & Veggies": Array.from({ length: 10 }, (_, i) => ({ name: `Exotic Fruits & Veggies ${i+1}`, image: "https://placehold.co/400", href: `/cars/suv/${i+1}` })),
      "Dairy & Bakery": Array.from({ length: 10 }, (_, i) => ({ name: `SUV Model ${i+1}`, image: "https://placehold.co/400", href: `/cars/suv/${i+1}` })),
    }
    
    // OR
    /* Reactive Sub-sub category mapping */
    // const subSubCategories = reactive({
    //   "Sports": [
    //     { name: "Sports Bike 1", image: "https://placehold.co/400", href: "https://wa.me/988YYYYY?text=I'm%20inquiring%20about%20Sports%20Bike%201" },
    //     { name: "Sports Bike 2", image: "https://placehold.co/400", href: "/bikes/sports/2" },
    //     { name: "Sports Bike 3", image: "https://placehold.co/400", href: "/bikes/sports/3" },
    //   ],
    //   "Cruiser": [
    //     { name: "Cruiser 1", image: "https://placehold.co/400", href: "/bikes/cruiser/1" },
    //     { name: "Cruiser 2", image: "https://placehold.co/400", href: "/bikes/cruiser/2" },
    //   ],
    // })
    
    /* State */
    const activeCategory = ref(null)
    const selectedSubCategory = ref(null)
    const dropdownStyle = reactive({ top: '0px' })
    const dropdown = ref(null)
    const activeButtonEl = ref(null)
    
    /* Helpers */
    const subSubCategoriesFor = (sub) => {
      if (!sub) return []
      const key = typeof sub === 'string' ? sub : (sub?.name || sub?.label)
      return subSubCategories[key] || []
    }
    
    /* Category click */
    const onCategoryClick = async (event, cat) => {
      if(cat.waLink) { window.open(cat.waLink, "_blank"); return }
      if (cat.href && !(cat.children && cat.children.length)) { closeDropdown(); router.visit(cat.href); return }
      if (activeCategory.value?.name === cat.name) { closeDropdown(); return }
    
      activeCategory.value = cat
      activeButtonEl.value = event.currentTarget
      selectedSubCategory.value = cat.children?.[0] || null
      await nextTick()
      const rect = event.currentTarget.getBoundingClientRect()
      dropdownStyle.top = `${rect.bottom + window.scrollY + 8}px`
    }
    
    /* Subcategory link click */
    const handleSubCategoryLink = (sub) => {
      const href = sub.href || sub.waLink
      if(!href) return
      if(href.startsWith('https://wa.me/')) window.open(href, "_blank")
      else router.visit(href)
      closeDropdown()
    }
    
    /* Sub-sub category click */
    const handleSubSubClick = (item) => {
      const href = item.href || item.waLink
      if(!href) return
      if(href.startsWith('https://wa.me/')) window.open(href, "_blank")
      else router.visit(href)
      closeDropdown()
    }
    
    /* Select subcategory */
    const selectSubCategory = (sub) => selectedSubCategory.value = sub
    
    /* Close dropdown */
    function closeDropdown() {
      activeCategory.value = null
      selectedSubCategory.value = null
      activeButtonEl.value = null
    }
    
    /* Outside click detection */
    const handleClickOutside = (e) => {
      const target = e.target
      if (!activeCategory.value) return
      if (dropdown.value?.contains(target)) return
      if (activeButtonEl.value?.contains(target)) return
      closeDropdown()
    }
    
    /* Scroll lock */
    watch(activeCategory, (val) => { document.body.style.overflow = val ? 'hidden' : '' })
    onMounted(() => document.addEventListener('click', handleClickOutside))
    onBeforeUnmount(() => { document.removeEventListener('click', handleClickOutside); document.body.style.overflow = '' })
    </script>
    
    <style>
    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>


Now navigate into the main AppLayout.vue file (For now: resources/js/Components/Layouts.vue)
Import this file and use it as:

    <template>
      <Navbar />
      <!-- <CategoryTabs /> -->
      <CategoryTabs class="sticky top-[100px] md:top-[160px] z-20"/>   <--- Notice Here//////////////////
      <!-- Dynamic Page Content Will Be Rendered Here -->
      <slot />
      <Footer />
  
      <!-- Bottom Nav (only visible on <=1439px) -->
      <BottomNav />
  
    </template>
    
    <script setup>
    import Navbar from '@/Components/Shared/Navbar.vue';
    import Footer from '@/Components/Shared/Footer.vue';
    import BottomNav from '@/Components/Shared/BottomNav.vue'
    import CategoryTabs from '@/Components/Shared/CategoryTabs.vue'   <--- Notice Here//////////////////
  
    </script>

If you want that the link should work for walink than you can use it instead of above code.

    <template>
      <div class="relative bg-white shadow-md border-b">
        <!-- Tabs (scrollable top) -->
        <div class="flex overflow-x-auto no-scrollbar px-4 py-2 space-x-4">
          <button
            v-for="cat in categories"
            :key="cat.name"
            :aria-expanded="activeCategory?.name === cat.name ? 'true' : 'false'"
            class="flex items-center gap-2 px-3 py-2 rounded-lg whitespace-nowrap transition"
            :class="activeCategory?.name === cat.name
                     ? 'bg-[#65a30d] text-white'
                     : 'hover:bg-[#facc15] text-gray-800'"
            @click="onCategoryClick($event, cat)"
          >
            <component
              :is="cat.icon"
              class="w-5 h-5"
              :class="activeCategory?.name === cat.name ? 'text-white' : 'text-[#65a30d]'"
            />
            <span class="font-medium text-sm">{{ cat.name }}</span>
          </button>
        </div>
    
        <!-- Dropdown via Teleport -->
        <teleport to="body">
          <div
            v-if="activeCategory"
            ref="dropdown"
            class="absolute left-0 right-0 w-full flex justify-center z-50"
            :style="{ top: dropdownStyle.top }"
          >
            <div
              class="w-full max-w-screen-xl mx-4 md:mx-auto bg-white rounded-lg border shadow-lg p-4"
              role="dialog"
              aria-modal="true"
            >
              <div class="flex gap-4">
                <!-- Left Column (Subcategories) -->
                <div class="flex-shrink-0 w-28 sm:w-36 md:w-56 max-h-[60vh] overflow-y-auto pr-2">
                  <div class="grid grid-cols-1 gap-3">
                    <template v-for="(sub, i) in activeCategory.children || []" :key="i">
                      <button
                        v-if="!sub.href && !sub.waLink"
                        @click="selectSubCategory(sub)"
                        :class="[ 
                          'aspect-square flex flex-col items-center justify-center p-2 rounded-lg border transition',
                          selectedSubCategory === sub
                            ? 'bg-[#f0f7e6] font-semibold border-[#d7eacb]'
                            : 'hover:bg-[#f7fbef]'
                        ]"
                      >
                        <div class="w-full h-full sm:w-full sm:h-full bg-gray-100 rounded flex items-center justify-center mb-2 overflow-hidden">
                          <img
                            v-if="sub.image"
                            :src="sub.image"
                            alt=""
                            class="w-full h-full object-cover"
                          />
                          <span v-else class="text-xs text-gray-600">
                            {{ typeof sub === 'string' ? sub[0] : (sub?.name?.[0] || '?') }}
                          </span>
                        </div>
                        <span class="text-[10px] sm:text-xs text-center px-1 leading-tight break-words">
                          {{ typeof sub === 'string' ? sub : (sub.name || sub.label) }}
                        </span>
                      </button>
    
                      <button
                        v-else
                        class="aspect-square flex flex-col items-center justify-center p-2 rounded-lg border transition hover:bg-[#f7fbef]"
                        @click="handleSubCategoryLink(sub)"
                      >
                        <div class="w-full h-full bg-gray-100 rounded flex items-center justify-center mb-2 overflow-hidden">
                          <img
                            v-if="sub.image"
                            :src="sub.image"
                            alt=""
                            class="w-full h-full object-cover"
                          />
                          <span class="text-xs text-gray-600">
                            {{ sub.name?.[0] || '?' }}
                          </span>
                        </div>
                        <span class="text-[10px] sm:text-xs text-center px-1 leading-tight break-words">
                          {{ sub.name }}
                        </span>
                      </button>
                    </template>
                  </div>
                </div>
    
                <!-- Right Column (Sub-subcategories grid) -->
                <div class="flex-1 max-h-[60vh] overflow-y-auto">
                  <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4">
                    <template v-if="selectedSubCategory && subSubCategoriesFor(selectedSubCategory).length">
                      <button
                        v-for="(item, idx) in subSubCategoriesFor(selectedSubCategory)"
                        :key="idx"
                        class="aspect-square flex flex-col items-center justify-center border rounded-lg p-3 hover:shadow-md transition text-center"
                        @click="handleSubSubClick(item)"
                      >
                        <img v-if="item.image" :src="item.image" alt="" class="w-full h-full object-cover mb-2" />
                        <div class="text-xs font-medium">{{ item.name || item.label }}</div>
                      </button>
                    </template>
                    <template v-else>
                      <div class="col-span-full text-center text-gray-500 py-8">
                        Select a subcategory to view items
                      </div>
                    </template>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </teleport>
      </div>
    </template>
    
    <script setup>
    import { ref, reactive, nextTick, watch, onMounted, onBeforeUnmount } from 'vue'
    import { Home, CreditCard, Printer, PenTool, FileText, Layers, BookOpen } from 'lucide-vue-next'
    import { router } from '@inertiajs/vue3'
    
    /* Main Categories Tabs */
    const categories = [
      { name: "Home", icon: Home, href: "/" },
      { name: "Printing Services", icon: Printer, href: "/print" },
    
      // If there is sub-categories in category than use like this
      // { name: "Bike", icon: Printer, children: [
      //     { name: "Sports", image: "https://placehold.co/400" },
      //     { name: "Cruiser", image: "https://placehold.co/400" },
      //     { name: "Electric", image: "https://placehold.co/400" },
      //     { name: "Accessories", href: "/bikes/accessories", image: "https://placehold.co/400" },
      //   ]
      // },
      // NOTE: You need to map sub-sub category or Reactive Sub-sub category mapping
    
      { name: "Photocopy & Scanning", icon: FileText, waLink: "https://wa.me/988XXXXX?text=I'm%20inquiring%20about%20Photocopy%20%26%20Scanning" },
      { name: "Binding & Lamination", icon: Layers, waLink: "https://wa.me/988XXXXX?text=I'm%20inquiring%20about%20Binding%20%26%20Lamination" },
      { name: "Business Card", icon: CreditCard, waLink: "https://wa.me/988XXXXX?text=I'm%20inquiring%20about%20Business%20Card" },
      { name: "Design Services", icon: PenTool, waLink: "https://wa.me/988XXXXX?text=I'm%20inquiring%20about%20Design%20Services" },
      { name: "Custom Orders", icon: BookOpen, waLink: "https://wa.me/988XXXXX?text=I'm%20inquiring%20about%20Custom%20Orders" }
    ]
    
    /* Sub-sub category mapping */
    // const subSubCategories = {
    //   "Sports": Array.from({ length: 12 }, (_, i) => ({ name: `Sports Bike ${i+1}`, image: "https://placehold.co/400", href: `/bikes/sports/${i+1}` })),
    //   "Cruiser": Array.from({ length: 8 }, (_, i) => ({ name: `Cruiser ${i+1}`, image: "https://placehold.co/400", href: `/bikes/cruiser/${i+1}` })),
    //   "SUV": Array.from({ length: 10 }, (_, i) => ({ name: `SUV Model ${i+1}`, image: "https://placehold.co/400", href: `/cars/suv/${i+1}` })),
    // }
    // OR
    /* Reactive Sub-sub category mapping */
    // const subSubCategories = reactive({
    //   "Sports": [
    //     { name: "Sports Bike 1", image: "https://placehold.co/400", href: "https://wa.me/988YYYYY?text=I'm%20inquiring%20about%20Sports%20Bike%201" },
    //     { name: "Sports Bike 2", image: "https://placehold.co/400", href: "/bikes/sports/2" },
    //     { name: "Sports Bike 3", image: "https://placehold.co/400", href: "/bikes/sports/3" },
    //   ],
    //   "Cruiser": [
    //     { name: "Cruiser 1", image: "https://placehold.co/400", href: "/bikes/cruiser/1" },
    //     { name: "Cruiser 2", image: "https://placehold.co/400", href: "/bikes/cruiser/2" },
    //   ],
    // })
    
    /* State */
    const activeCategory = ref(null)
    const selectedSubCategory = ref(null)
    const dropdownStyle = reactive({ top: '0px' })
    const dropdown = ref(null)
    const activeButtonEl = ref(null)
    
    /* Helpers */
    const subSubCategoriesFor = (sub) => {
      if (!sub) return []
      const key = typeof sub === 'string' ? sub : (sub?.name || sub?.label)
      return subSubCategories[key] || []
    }
    
    /* Category click */
    const onCategoryClick = async (event, cat) => {
      if(cat.waLink) { window.open(cat.waLink, "_blank"); return }
      if (cat.href && !(cat.children && cat.children.length)) { closeDropdown(); router.visit(cat.href); return }
      if (activeCategory.value?.name === cat.name) { closeDropdown(); return }
    
      activeCategory.value = cat
      activeButtonEl.value = event.currentTarget
      selectedSubCategory.value = cat.children?.[0] || null
      await nextTick()
      const rect = event.currentTarget.getBoundingClientRect()
      dropdownStyle.top = `${rect.bottom + window.scrollY + 8}px`
    }
    
    /* Subcategory link click */
    const handleSubCategoryLink = (sub) => {
      const href = sub.href || sub.waLink
      if(!href) return
      if(href.startsWith('https://wa.me/')) window.open(href, "_blank")
      else router.visit(href)
      closeDropdown()
    }
    
    /* Sub-sub category click */
    const handleSubSubClick = (item) => {
      const href = item.href || item.waLink
      if(!href) return
      if(href.startsWith('https://wa.me/')) window.open(href, "_blank")
      else router.visit(href)
      closeDropdown()
    }
    
    /* Select subcategory */
    const selectSubCategory = (sub) => selectedSubCategory.value = sub
    
    /* Close dropdown */
    function closeDropdown() {
      activeCategory.value = null
      selectedSubCategory.value = null
      activeButtonEl.value = null
    }
    
    /* Outside click detection */
    const handleClickOutside = (e) => {
      const target = e.target
      if (!activeCategory.value) return
      if (dropdown.value?.contains(target)) return
      if (activeButtonEl.value?.contains(target)) return
      closeDropdown()
    }
    
    /* Scroll lock */
    watch(activeCategory, (val) => { document.body.style.overflow = val ? 'hidden' : '' })
    onMounted(() => document.addEventListener('click', handleClickOutside))
    onBeforeUnmount(() => { document.removeEventListener('click', handleClickOutside); document.body.style.overflow = '' })
    </script>
    
    <style>
    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
