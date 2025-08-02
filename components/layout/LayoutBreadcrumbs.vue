<script setup lang="ts">
const { breadcrumbs } = useBreadcrumbs();
const localePath = useLocalePath();
const { formatLink } = useInternationalization(localePath);
</script>
<template>
  <nav
    class="hidden lg:flex container px-2 md:px-4 mx-auto mt-8 mb-2 text-black"
    aria-label="Breadcrumb"
  >
    <ol class="inline-flex items-center space-x-1 md:space-x-3 text-gray-400">
      <li class="inline-flex items-center">
        <NuxtLink
          :to="formatLink(`/`)"
          class="inline-flex items-center text-sm"
        >
          {{ $t("home") }}
        </NuxtLink>
        <img src="~/assets/icons/slash.svg" class="w-4 h-4 ml-2 text-sm" />
      </li>
      <li
        v-for="(breadcrumb, index) in breadcrumbs"
        :key="breadcrumb.path"
        class="inline-flex items-center"
      >
        <NuxtLink
          v-if="breadcrumb.path && breadcrumbs?.length - 1 !== index"
          :to="formatLink(breadcrumb.path)"
          class="inline-flex items-center text-sm"
        >
          {{ breadcrumb.name }}
        </NuxtLink>
        <span v-else class="inline-flex items-center text-sm !text-black">
          {{ breadcrumb.name }}
        </span>
        <img
          v-if="index < breadcrumbs.length - 1"
          src="~/assets/icons/slash.svg"
          class="w-4 h-4 ml-2 text-sm"
        />
      </li>
    </ol>
  </nav>
</template>
