<template>
  <div>
    <SfLoader
      :class="{ loading }"
      :loading="loading"
    >
      <SfCarousel :settings="settings">
        <SfCarouselItem
          v-for="(product, i) in mappedProducts"
          :key="i"
        >
          <div class="carousel-product">
            <SfProductCard
              class="product"
              image-tag="nuxt-img"
              :title="productGetters.getName(product)"
              :image-width="imageSizes.productCard.width"
              :image-height="imageSizes.productCard.height"
              :image="
                getMagentoImage(
                  productGetters.getProductThumbnailImage(product)
                )
              "
              :nuxt-img-config="{
                fit: 'cover',
              }"
              :regular-price="$fc(productGetters.getPrice(product).regular)"
              :special-price="
                productGetters.getPrice(product).special &&
                  $fc(productGetters.getPrice(product).special)
              "
              :link="localePath(getProductPath(product))"
              :max-rating="5"
              :score-rating="productGetters.getAverageRating(product)"
              :reviews-count="productGetters.getTotalReviews(product)"
              :is-in-wishlist="isInWishlist({ product })"
              :is-added-to-cart="isInCart(product)"
              :wishlist-icon="isAuthenticated ? 'heart' : ''"
              :is-in-wishlist-icon="isAuthenticated ? 'heart_fill' : ''"
              @click:wishlist="addItemToWishlist(product)"
              @click:add-to-cart="addItemToCart({ product, quantity: 1 })"
            />
          </div>
        </SfCarouselItem>
        <template #next="{ go }">
          <SfButton
            size="sm"
            class="color-light"
            @click="go('next')"
          >
            <span class="sf-chevron--right sf-chevron">
              <span class="sf-chevron__bar sf-chevron__bar--left" />
              <span class="sf-chevron__bar sf-chevron__bar--right" />
            </span>
          </SfButton>
        </template>
        <template #prev="{ go }">
          <SfButton
            size="sm"
            class="color-light"
            @click="go('prev')"
          >
            <span class="sf-chevron--left sf-chevron">
              <span class="sf-chevron__bar sf-chevron__bar--right" />
              <span class="sf-chevron__bar sf-chevron__bar--left" />
            </span>
          </SfButton>
        </template>
      </SfCarousel>
    </SfLoader>
  </div>
</template>

<script lang="ts">
import {
  SfCarousel,
  SfProductCard,
  SfLoader,
  SfButton,
} from '@storefront-ui/vue';
import { useImage, useProduct } from '~/composables';
import { useUser } from '~/modules/customer/composables/useUser';
import useWishlist from '~/modules/wishlist/composables/useWishlist';
import { useAddToCart } from '~/helpers/cart/addToCart';
import productGetters from '~/modules/catalog/product/getters/productGetters';

import {
  defineComponent,
  onMounted,
  ref,
  computed,
} from '@nuxtjs/composition-api';

export default defineComponent({
  name: 'Slider',
  components: {
    SfCarousel,
    SfProductCard,
    SfLoader,
    SfButton,
  },

  setup() {
    const { isAuthenticated } = useUser();

    const products = ref([]);

    const { getProductList, loading, getProductPath } = useProduct();

    const { getMagentoImage, imageSizes } = useImage();

    const { isInWishlist, addOrRemoveItem } = useWishlist();

    const { addItemToCart, isInCart } = useAddToCart();

    const settings = {
      perView: 3,
    };

    const addItemToWishlist = async (product) => {
      await addOrRemoveItem({ product });
    };

    const mappedProducts = computed(() => products.value.map((product) => ({
      ...product,
      isInWishlist: isInWishlist({ product }),
    })));

    onMounted(async () => {
      const newestProducts = await getProductList({
        pageSize: 12,
      });

      if (newestProducts?.items?.length) {
        products.value = newestProducts.items;
      }
    });

    return {
      addItemToCart,
      addItemToWishlist,
      isAuthenticated,
      isInCart,
      isInWishlist,
      loading,
      mappedProducts,
      productGetters,
      getMagentoImage,
      imageSizes,
      getProductPath,
      settings,
    };
  },
});
</script>

<style scoped lang="scss">
.carousel-product {
  margin: 0.5rem 0;
}

.sf-button {
  border-radius: 100vh !important;
  padding: 1rem;
}

.products {
  display: flex;
  justify-content: space-between;

  @include for-mobile {
    flex-wrap: wrap;
  }

  .product {
    @include for-desktop {
      flex: 1 1 25%;
    }

    @include for-mobile {
      flex: 1 1 50%;
    }
  }
}
</style>
