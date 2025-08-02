<script setup lang="ts">
import type { CmsElementSidebarFilter } from "@shopware/composables";
import SwProductListingFilters from "~/components/listing-filters/SwProductListingFilters.vue";

defineProps<{
  content: CmsElementSidebarFilter;
}>();
</script>
<template>
  <div class="mx-auto">
    <SwProductListingFilters :content="content" />
  </div>
</template>
