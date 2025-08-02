<script
  setup
  lang="ts"
  generic="
    ListingFilter extends {
      code: string;
      label: string;
    }
  "
>
import { computed, ref } from "vue";
import type { Schemas } from "#shopware";

const emits =
  defineEmits<
    (e: "select-value", value: { code: string; value: unknown }) => void
  >();

const props = defineProps<{
  filter: ListingFilter;
  selectedFilters: Schemas["ProductListingResult"]["currentFilters"];
}>();
const isHoverActive = ref(false);
const hoveredIndex = ref(0);
const displayedScore = computed(() =>
  isHoverActive.value ? hoveredIndex.value : props.selectedFilters?.rating || 0
);

const hoverRating = (key: number) => {
  hoveredIndex.value = key;
  isHoverActive.value = true;
};
const onChangeRating = () => {
  const newValue =
    props.selectedFilters?.rating !== hoveredIndex.value
      ? hoveredIndex.value
      : undefined;
  emits("select-value", { code: props.filter?.code, value: newValue });
};

const isFilterVisible = ref<boolean>(false);
const toggle = () => {
  isFilterVisible.value = !isFilterVisible.value;
};
</script>

<template>
  <div class="py-3 px-5">
    <h3 class="!p-0 flow-root">
      <button
        type="button"
        class="flex w-full items-center justify-between bg-white text-base text-gray-400 hover:text-gray-500"
        @click="toggle"
      >
        <span
          class="font-medium text-gray-900 text-left uppercase"
          :class="{ 'font-semibold': isFilterVisible }"
          >{{ props.filter.label }}</span
        >
        <span class="ml-6 flex items-center">
          <img
            src="~/assets/icons/down.svg"
            alt="arrow-down"
            class="w-6 h-6 transition-all duration-300"
            :class="[!isFilterVisible ? 'rotate-0' : 'rotate-180']"
          />
        </span>
      </button>
    </h3>
    <transition name="fade" mode="out-in">
      <div v-show="isFilterVisible">
        <div class="space-y-6 mt-4">
          <div class="flex gap-4">
            <div
              v-for="i in 5"
              :key="i"
              class="fa-xl c-yellow-500"
              :class="{
                'fa fa-star': displayedScore >= i,
                'far fa-star': displayedScore < i,
              }"
              @mouseleave="isHoverActive = false"
              @click="onChangeRating()"
              @mouseover="hoverRating(i)"
            />
          </div>
        </div>
      </div>
    </transition>
  </div>
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
