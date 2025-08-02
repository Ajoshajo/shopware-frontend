<script setup lang="ts">
const { count } = useCart();
const { count: wishlistCount } = useWishlist();
const localePath = useLocalePath();
const { formatLink } = useInternationalization(localePath);

const miniCartModal = useMiniCartModal();
</script>

<template>
  <!-- <LayoutHeaderTop /> -->
  <header>
    <div
      class="container mx-auto py-0 md:py-2 md:px-4 px-2 flex justify-between items-center transition-all duration-500 ease-in-out"
    >
      <div class="flex justify-start items-center min-w-10 lg:min-w-12 py-2">
        <div class="order-2 lg:order-1 ml-4 lg:ml-0">
          <NuxtLink :to="formatLink(`/`)">
            <span class="sr-only">Shopware</span>
            <img
              class="h-10 lg:h-12"
              src="~/assets/images/logo.png"
              alt="logo of the shop"
            />
          </NuxtLink>
        </div>
        <div class="order-1 lg:order-2 flex justify-center">
          <LayoutSideMenu />
        </div>
      </div>
      <div class="hidden md:flex gap-12 text-lg text-center flex-grow-[0.1]">
        <LayoutStoreSearch />
      </div>
      <div class="flex items-center gap-3 md:gap-6">
        <div class="flex flow-root">
          <NuxtLink
            class="group -m-2 p-2 flex items-center relative text-center"
            aria-label="wishlist"
            data-testid="wishlist-button"
            :to="formatLink(`/wishlist`)"
          >
            <img
              src="~/assets/icons/heart.svg"
              alt="wishlist icon"
              class="w-5 h-5"
            />
            <ClientOnly>
              <span
                v-if="wishlistCount"
                class="text-xs text-white absolute bg-black rounded-full min-w-5 min-h-4 top-0 right-0 leading-5"
              >
                {{ wishlistCount }}
              </span>
            </ClientOnly>
          </NuxtLink>
        </div>
        <!-- Cart -->
        <div class="flex flow-root">
          <button
            class="group bg-transparent -m-2 p-2 flex items-center relative"
            aria-label="Mini cart"
            data-testid="cart-button"
            @click="miniCartModal.open"
          >
            <!-- Heroicon name: outline/shopping-bag -->
            <img
              src="~/assets/icons/shopping-bag.svg"
              alt="cart icon"
              class="w-6 h-6"
            />
            <span
              v-if="count > 0"
              class="text-xs text-white absolute bg-black rounded-full min-w-5 min-h-5 top-0 right-0 leading-5"
            >
              {{ count || "" }}
            </span>
            <span class="sr-only"
              >{{ $t("cart.itemsInCart") }}, {{ $t("cart.viewCart") }}</span
            >
          </button>
        </div>
        <CheckoutSideCart :controller="miniCartModal" />
        <AccountMenu />
      </div>
    </div>
    <!-- <div class="border-b border-black-100 hidden lg:block"></div> -->
    <LayoutTopNavigation />
    <div class="border-b border-black-100 pb-2"></div>
  </header>
</template>
