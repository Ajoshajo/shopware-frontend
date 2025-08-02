<script setup lang="ts">
import type {
  CmsElementProductListing,
  CmsElementSidebarFilter,
} from "@shopware/composables";
import { useCmsTranslations } from "@shopware/composables";
import { onClickOutside } from "@vueuse/core";
import { defu } from "defu";
import { computed, provide, reactive, ref, useTemplateRef } from "vue";
import type { ComputedRef, UnwrapNestedRefs } from "vue";
import { type LocationQueryRaw, useRoute, useRouter } from "vue-router";
import { useCategoryListing } from "#imports";
import type { Schemas, operations } from "#shopware";
import SwProductListingFilter from "./SwProductListingFilter.vue";

const props = defineProps<{
  content: CmsElementProductListing | CmsElementSidebarFilter;
  listingType?: string;
}>();

type Translations = {
  listing: {
    filters: string;
    sort: string;
    resetFilters: string;
    view: string;
  };
};

let translations: Translations = {
  listing: {
    filters: "Filters",
    sort: "Sort",
    resetFilters: "Reset filters",
    view: "View",
  },
};

translations = defu(useCmsTranslations(), translations) as Translations;

const route = useRoute();
const router = useRouter();

const isSortMenuOpen = ref(false);
const isFilterMenuOpen = ref(false);

const {
  changeCurrentSortingOrder,
  filtersToQuery,
  getCurrentFilters,
  getCurrentSortingOrder,
  getInitialFilters,
  getSortingOrders,
  getTotal,
  search,
} = useCategoryListing();

const sidebarSelectedFilters: UnwrapNestedRefs<{
  [key: string]: Set<string> | string | number | boolean | undefined;
}> = reactive<{
  manufacturer: Set<string>;
  properties: Set<string>;
  "min-price": string | number | undefined;
  "max-price": string | number | undefined;
  rating: string | number | undefined;
  "shipping-free": boolean | undefined;
}>({
  manufacturer: new Set(),
  properties: new Set(),
  "min-price": undefined,
  "max-price": undefined,
  rating: undefined,
  "shipping-free": undefined,
});

const showResetFiltersButton = computed<boolean>(() => {
  if (
    selectedOptionIds.value.length !== 0 ||
    sidebarSelectedFilters["max-price"] ||
    sidebarSelectedFilters["min-price"] ||
    sidebarSelectedFilters.rating ||
    sidebarSelectedFilters["shipping-free"]
  ) {
    return true;
  }

  return false;
});

const searchCriteriaForRequest: ComputedRef<Schemas["ProductListingCriteria"]> =
  computed(() => ({
    manufacturer: [
      ...(sidebarSelectedFilters.manufacturer as Set<string>),
    ]?.join("|"),
    properties: [...(sidebarSelectedFilters.properties as Set<string>)]?.join(
      "|"
    ),
    "min-price": sidebarSelectedFilters["min-price"] as number,
    "max-price": sidebarSelectedFilters["max-price"] as number,
    order: getCurrentSortingOrder.value as string,
    "shipping-free": sidebarSelectedFilters["shipping-free"] as boolean,
    rating: sidebarSelectedFilters.rating as number,
    search: "",
    limit: route.query.limit ? Number(route.query.limit) : 15,
  }));

for (const param in route.query) {
  if (Object.prototype.hasOwnProperty.call(sidebarSelectedFilters, param)) {
    if (
      sidebarSelectedFilters[param] &&
      typeof sidebarSelectedFilters[param] === "object"
    ) {
      const elements = (route.query[param] as unknown as string).split("|");
      for (const element of elements) {
        sidebarSelectedFilters[param].add(element);
      }
    } else {
      const queryValue = route.query[param];
      if (queryValue !== null && !Array.isArray(queryValue)) {
        sidebarSelectedFilters[param] = queryValue;
      }
    }
  }
}

const onOptionSelectToggle = async ({
  code,
  value,
}: {
  code: string;
  value: string | number | boolean;
}) => {
  if (!["properties", "manufacturer"].includes(code)) {
    sidebarSelectedFilters[code] = value;
  } else {
    const filterSet = sidebarSelectedFilters[code] as Set<string>;
    const stringValue = String(value);
    if (filterSet?.has(stringValue)) {
      filterSet.delete(stringValue);
    } else {
      filterSet?.add(stringValue);
    }
  }

  await executeSearch();
};

const executeSearch = async () => {
  await search(searchCriteriaForRequest.value);
  const query = filtersToQuery(searchCriteriaForRequest.value);
  const { limit: _, ...queryWithoutLimit } = query; // remove limit from query
  await router.push({
    query: queryWithoutLimit as LocationQueryRaw,
  });
};

const clearFilters = () => {
  (sidebarSelectedFilters.manufacturer as Set<string>).clear();
  (sidebarSelectedFilters.properties as Set<string>).clear();
  sidebarSelectedFilters["min-price"] = undefined;
  sidebarSelectedFilters["max-price"] = undefined;
  sidebarSelectedFilters.rating = undefined;
  sidebarSelectedFilters["shipping-free"] = undefined;
};

const currentSortingOrder = computed({
  get: (): string => getCurrentSortingOrder.value || "",
  set: async (order: string): Promise<void> => {
    await router.push({
      query: {
        ...route.query,
        order,
      },
    });

    await changeCurrentSortingOrder(order, {
      ...(route.query as unknown as operations["searchPage post /search"]["body"]),
      limit: route.query.limit ? Number(route.query.limit) : 15,
    });
  },
});

const selectedOptionIds = computed(() => [
  ...(sidebarSelectedFilters.properties as Set<string>),
  ...(sidebarSelectedFilters.manufacturer as Set<string>),
]);
provide("selectedOptionIds", selectedOptionIds);

async function invokeCleanFilters() {
  clearFilters();
  await executeSearch();
}
const isDefaultSidebarFilter =
  props.content.type === "sidebar-filter" &&
  props.content.config?.boxLayout?.value === "standard";

const filterController = useFilterModal();

const dropdownElement = useTemplateRef("dropdownElement");
onClickOutside(dropdownElement, () => {
  isSortMenuOpen.value = false;
});

const handleSortingClick = (key: string) => {
  currentSortingOrder.value = key;
  isSortMenuOpen.value = false;
};

const toggFilter = () => {
  filterController.isOpen.value
    ? filterController.close()
    : filterController.open();
};
</script>
<template>
  <div class="bg-white flex justify-between px-2 md:px-4 lg:px-0">
    <div ref="dropdownElement" class="flex justify-between text-left">
      <div>
        <button
          type="button"
          @click="isSortMenuOpen = !isSortMenuOpen"
          class="group inline-flex justify-center items-center bg-transparent text-base font-medium text-gray-700 hover:text-gray-900"
          id="menu-button"
          aria-expanded="false"
          aria-haspopup="true"
        >
          {{ translations.listing.sort }}
          <img
            src="~/assets/icons/down.svg"
            class="h-5 w-5 ml-1 transition duration-300"
            :class="{ 'rotate-180': isSortMenuOpen }"
          />
        </button>
      </div>
      <div
        :class="[isSortMenuOpen ? 'absolute' : 'hidden']"
        class="origin-top-left left-0 lg:origin-top-left lg:left-auto mt-8 w-40 rounded-md bg-white ring-1 ring-black ring-opacity-5 focus:outline-none z-10"
        role="menu"
        aria-orientation="vertical"
        aria-labelledby="menu-button"
        tabindex="-1"
      >
        <div class="py-1" role="none">
          <button
            v-for="sorting in getSortingOrders"
            :key="sorting.key"
            @click="handleSortingClick(sorting.key)"
            :class="[
              sorting.key === getCurrentSortingOrder
                ? 'font-medium text-gray-900'
                : 'text-gray-500',
            ]"
            class="block px-4 py-2 text-sm bg-transparent"
            role="menuitem"
            tabindex="-1"
          >
            {{ sorting.label }}
          </button>
        </div>
      </div>
    </div>
    <div class="m-0" :class="{ 'px-0': isDefaultSidebarFilter }">
      <ClientOnly>
        <div class="relative flex items-baseline justify-between">
          <button
            @click="toggFilter"
            class="text-base flex items-center cursor-pointer font-medium tracking-tight text-gray-900"
          >
            {{ translations.listing.filters }}
            <img
              src="~/assets/icons/filter.svg"
              class="h-5 w-5 ml-1 transition duration-300"
              :class="{ 'rotate-180': isFilterMenuOpen }"
            />
          </button>
        </div>

        <div
          :class="[isFilterMenuOpen ? 'absolute' : 'hidden']"
          class="origin-top-left right-0 lg:origin-top-left lg:left-auto mt-8 w-40 rounded-md bg-white ring-1 ring-black ring-opacity-5 focus:outline-none z-10 mx-4"
          v-if="getInitialFilters.length"
        >
          <LayoutSidebar :controller="filterController" side="right">
            <aside class="py-2 px-4 h-full flex flex-col">
              <div class="flex justify-between mt-2 px-5">
                <div
                  class="flex flex-grow justify-center text-xl font-semibold uppercase"
                >
                  Filter
                </div>
                <button @click="filterController.close">
                  <img src="~/assets/icons/close.svg" class="w-5 h-5" alt="" />
                </button>
              </div>
              <div class="mt-4 overflow-x-auto flex gap-y-2 flex-col flex-grow">
                <div
                  v-for="filter in getInitialFilters"
                  :key="`${filter?.id || filter?.code}`"
                  class="w-full"
                >
                  <SwProductListingFilter
                    @select-filter-value="onOptionSelectToggle"
                    :selected-filters="getCurrentFilters"
                    :filter="filter"
                    class="relative"
                  />
                </div>
              </div>

              <div class="flex gap-2">
                <div class="mx-auto mt-4 mb-2 flex-1">
                  <button
                    :disabled="!showResetFiltersButton"
                    class="w-full justify-center py-4 px-6 border shadow-sm text-sm font-medium rounded-md text-black bg-white border-black focus:outline-none uppercase rounded-br-xl rounded-tl-xl flex items-center"
                    :class="{
                      'opacity-50 cursor-not-allowed border-gray-500':
                        !showResetFiltersButton,
                    }"
                    @click="invokeCleanFilters"
                    type="button"
                  >
                    {{ translations.listing.resetFilters }}
                  </button>
                </div>
                <div class="mx-auto mt-4 mb-2 flex-1">
                  <button
                    class="w-full justify-center py-4 px-6 border shadow-sm text-sm font-medium rounded-md text-white bg-black focus:outline-noneuppercase rounded-br-xl rounded-tl-xl flex items-center"
                    @click="filterController.close"
                    type="button"
                  >
                    {{ translations.listing.view }} [ {{ getTotal }} ]
                  </button>
                </div>
              </div>
            </aside>
          </LayoutSidebar>
        </div>
      </ClientOnly>
    </div>
  </div>
</template>
