<script setup lang="ts">
import { getProductRoute } from "@shopware/helpers";

import { onClickOutside, useFocus, useMagicKeys } from "@vueuse/core";
import { useTemplateRef } from "vue";

withDefaults(
  defineProps<{
    displayTotal?: number;
  }>(),
  { displayTotal: 10 }
);

defineEmits<(e: "link-clicked") => void>();

const { searchTerm, search, getProducts, getTotal, loading } =
  useProductSearchSuggest();

// True when the search bar is active and user can type in the search field
const active = ref(false);

// Reference to the search container
const searchContainer = useTemplateRef("searchContainer");
const searchInput = useTemplateRef("searchInput");
const localePath = useLocalePath();
const { formatLink } = useInternationalization(localePath);

watch(active, (value) => {
  const { focused } = useFocus(searchInput);

  focused.value = value;
});

// String the user has typed in the search field
const typingQuery = ref("");

watch(typingQuery, (value) => {
  if (value.length >= 3) {
    performSuggestSearch(value);
  }
});

// Defer the search request to prevent the search from being triggered too after typing
const performSuggestSearch = useDebounceFn((value) => {
  searchTerm.value = value;
  search();
}, 300);

// Suggest results will only be shown, when the search bar is active and the user has typed more than three characters in the search field
const showSuggest = computed(() => {
  return typingQuery.value.length >= 3 && active.value;
});

if (import.meta.client) {
  onClickOutside(searchContainer, () => {
    active.value = false;
  });
}

// @ts-expect-error - wait for vueuse to adapt to templateRefs
const { enter } = useMagicKeys({ target: searchInput });
const { push } = useRouter();

const sideMenuModal = useSideMenuModal();

watch(enter, (value) => {
  if (!value) return;

  sideMenuModal.close();

  active.value = false;
  push(`/search?search=${typingQuery.value}`);
});
</script>

<template>
  <transition name="fade">
    <div
      v-if="sideMenuModal.isOpen"
      ref="searchContainer"
      class="relative group p-3 px-4 pr-0 rounded-br-2xl rounded-tl-2xl transition-all duration-300 inline-block w-full border-gray-200 border"
    >
      <div class="flex items-center">
        <img
          src="~/assets/icons/search-gray.svg"
          alt="search icon"
          class="sw-search-input w-6 h-6 flex-none cursor-pointer"
        />

        <transition name="fade" mode="out-in">
          <input
            @click="active = true"
            ref="searchInput"
            v-model="typingQuery"
            data-testid="layout-search-input"
            type="text"
            aria-label="Search for products"
            class="sw-search-input text-base text-secondary-600 placehotext-secondary-600y-600 focus:text-secondary-700 p-2 ml-2 lg:ml-0 xl:ml-2 grow h-6 transition-all duration-200 focus:outline-none w-56 lg:w-10/12"
            :placeholder="$t('form.searchPlaceholder')"
          />
        </transition>
      </div>
      <div
        v-if="showSuggest"
        data-testid="layout-search-result-box"
        class="absolute border-secondary-100 border-t-1 duration-300 left-0 mt-2 overflow-hidden right-0 rounded-b-md shadow-md transition-height w-auto z-1"
      >
        <NuxtLink
          v-for="product in getProducts?.slice(0, displayTotal)"
          :key="product.id"
          :to="formatLink(getProductRoute(product))"
          data-testid="layout-search-suggest-link"
          @click="[(active = false), $emit('link-clicked')]"
        >
          <ProductSuggestSearch :product="product" />
        </NuxtLink>

        <div
          class="h-11 text-sm rounded-b-md p-3 text-center transition"
          style="clip-path: inset(0% 0% 0% 0%)"
          :class="[loading ? ['bg-primary'] : ['bg-secondary-100']]"
        >
          <div
            v-if="loading"
            class="w-80 h-40 bg-light blur-2xl fixed animate-spin"
          />
          <div v-else>
            <NuxtLink
              v-if="getTotal > 0"
              data-testid="layout-search-result-box-more-link"
              :to="
                formatLink({ path: `/search`, query: { search: typingQuery } })
              "
              @click="[(active = false), $emit('link-clicked')]"
            >
              {{ $t("search.see") }}
              <span v-if="getTotal !== 1">{{ $t("search.all") }}</span>
              {{ getTotal }}
              <span>{{ $t("search.result", getTotal) }}</span>
            </NuxtLink>
            <div v-else data-testid="layout-search-result-box-no-result">
              {{ $t("search.noResults") }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>
<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: all 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
