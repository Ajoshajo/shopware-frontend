<script setup lang="ts">
import { getCategoryRoute, getTranslatedProperty } from "@shopware/helpers";

const { navigationElements } = useNavigation({ type: "footer-navigation" });
const localePath = useLocalePath();
const { formatLink } = useInternationalization(localePath);

const gridColumns = computed<number>(() =>
  navigationElements.value
    ? Object.keys(navigationElements.value).length + 2
    : 2
);
</script>

<template>
  <footer class="container mx-auto px-4 py-4">
    <menu class="flex flex-wrap">
      <div class="md:flex-1 mb-4">
        <div class="text-2xl font-semibold">Stay in the know</div>
        <p class="text-sm mt-3">
          Lorem ipsum dolor sit amet consectetur adipisicing elit.
        </p>
        <div class="mt-10 flex gap-4">
          <input
            type="text"
            class="lg:w-1/2 px-4 py-3 bg-gray-50 rounded-br-xl rounded-tl-xl border border-gray-200 focus:outline-none"
            placeholder="Enter your email"
          />
          <button
            class="px-4 py-3 bg-black text-white rounded-br-xl rounded-tl-xl"
          >
            <i class="fa fa-arrow-right"></i>
          </button>
        </div>
      </div>
      <div class="w-full md:flex-1 flex flex-wrap gap-x-4 md:gap-16">
        <div
          v-for="navigationElement in navigationElements"
          :key="navigationElement.id"
          class=""
        >
          <h3 class="mb-5 font-semibold">
            {{ getTranslatedProperty(navigationElement, "name") }}
          </h3>
          <template v-if="navigationElement.childCount > 0">
            <ul class="list-none p-0 mb-5">
              <li
                v-for="navigationChild in navigationElement.children"
                :key="navigationChild.id"
                class="pb-3 md:pb-1"
              >
                <NuxtLink
                  :target="
                    navigationChild.externalLink || navigationChild.linkNewTab
                      ? '_blank'
                      : ''
                  "
                  :to="formatLink(getCategoryRoute(navigationChild))"
                  class="text-base font-normal text-secondary-500 hover:text-secondary-900"
                >
                  {{ getTranslatedProperty(navigationChild, "name") }}
                </NuxtLink>
              </li>
            </ul>
          </template>
        </div>
      </div>
    </menu>
    <div class="mt-10 flex gap-6 fa-xl">
      <i class="fab fa-facebook"></i>
      <i class="fab fa-twitter"></i>
      <i class="fab fa-instagram"></i>
      <i class="fab fa-youtube"></i>
    </div>
    <div class="flex sm:justify-between mt-10 flex-wrap justify-center gap-x-4">
      <div class="flex items-center gap-4">
        <img src="~/assets/images/logo.png" class="h-14" alt="" />
        <p>@2025 Ajosh Ajo</p>
      </div>
      <div class="flex items-center justify-center gap-4 text-gray-500">
        <div>Terms</div>
        <div>Privacy</div>
        <div>Cookie Policy</div>
      </div>
    </div>
  </footer>
</template>
