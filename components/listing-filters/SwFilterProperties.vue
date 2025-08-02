<script
  setup
  lang="ts"
  generic="
    ListingFilter extends {
      code: string;
      label: string;
      name: string;
      options: Array<Schemas['PropertyGroupOption']>;
      entities: Array<Schemas['ProductManufacturer']>;
    }
  "
>
import { getTranslatedProperty } from "@shopware/helpers";
import { inject, ref } from "vue";
import type { Schemas } from "#shopware";

const props = defineProps<{
  filter: ListingFilter;
}>();

const emits =
  defineEmits<
    (e: "select-value", value: { code: string; value: unknown }) => void
  >();
const selectedOptionIds = inject<string[]>("selectedOptionIds");
const isFilterVisible = ref<boolean>(false);
const toggle = () => {
  isFilterVisible.value = !isFilterVisible.value;
};
</script>

<template>
  <div class="py-3 px-5">
    <h2 class="!p-0 flow-root">
      <button
        type="button"
        class="flex w-full items-center justify-between bg-white text-base text-gray-400 hover:text-gray-500"
        @click="toggle"
      >
        <span
          class="font-medium text-gray-900 text-left uppercase"
          :class="{ 'font-semibold': isFilterVisible }"
          >{{ props.filter.label }}
        </span>
        <span class="ml-6 flex items-center">
          <img
            src="~/assets/icons/down.svg"
            alt="arrow-down"
            class="w-6 h-6 transition-all duration-300"
            :class="[!isFilterVisible ? 'rotate-0' : 'rotate-180']"
          />
        </span>
      </button>
    </h2>
    <transition name="fade" mode="out-in">
      <div v-show="isFilterVisible" :id="props.filter.code" class="pt-1">
        <fieldset class="space-y-4">
          <legend class="sr-only">{{ props.filter.name }}</legend>
          <div
            v-for="option in props.filter.options || props.filter.entities"
            :key="`${option.id}-${selectedOptionIds?.includes(option.id)}`"
            class="flex items-center"
          >
            <label
              class="flex items-center cursor-pointer relative"
              :for="`filter-mobile-${props.filter.code}-${option.id}`"
            >
              <input
                :id="`filter-mobile-${props.filter.code}-${option.id}`"
                :checked="selectedOptionIds?.includes(option.id)"
                :name="props.filter.name"
                :value="option.name"
                :aria-label="`${option.name} filter`"
                type="checkbox"
                class="peer h-5 w-5 cursor-pointer transition-all appearance-none rounded shadow hover:shadow-md border border-slate-300 checked:bg-slate-800 checked:border-slate-800"
                @change="
                  emits('select-value', {
                    code: props.filter.code,
                    value: option.id,
                  })
                "
              />
              <span
                class="absolute text-white opacity-0 peer-checked:opacity-100 top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 pointer-events-none"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-3.5 w-3.5"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                  stroke="currentColor"
                  stroke-width="1"
                >
                  <path
                    fill-rule="evenodd"
                    d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                    clip-rule="evenodd"
                  ></path>
                </svg>
              </span>
            </label>
            <div v-if="option.media?.url">
              <img
                loading="lazy"
                class="ml-2 h-4 w-4"
                :src="option.media.url"
                :alt="option.media.translated.alt || ''"
                :class="{
                  'border-blue border-2': selectedOptionIds?.includes(
                    option.id
                  ),
                }"
              />
            </div>
            <div
              v-else-if="option.colorHexCode"
              class="ml-2 h-4 w-4"
              :style="`background-color: ${option.colorHexCode}`"
              :class="{
                'border-blue border-2': selectedOptionIds?.includes(option.id),
              }"
            />
            <label
              :for="`filter-mobile-${props.filter.code}-${option.id}`"
              class="ml-3 text-gray-600 cursor-pointer w-full text-left"
            >
              {{ getTranslatedProperty(option, "name") }}
            </label>
          </div>
        </fieldset>
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
