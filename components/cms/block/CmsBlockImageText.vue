<script setup lang="ts">
import type { CmsBlockImageText } from "@shopware/composables";
import { useCmsBlock } from "#imports";
import { getCmsLayoutConfiguration } from "@shopware/helpers";

const props = defineProps<{
  content: CmsBlockImageText;
}>();

const { getSlotContent } = useCmsBlock(props.content);

const leftContent = getSlotContent("left");
const rightContent = getSlotContent("right");
const { cssClasses, layoutStyles } = getCmsLayoutConfiguration(props.content);
</script>
<template>
  <div class="grid md:grid-cols-2 gap-10 !my-0">
    <CmsGenericElement :content="leftContent" />
    <CmsGenericElement :content="rightContent" />
  </div>
</template>
