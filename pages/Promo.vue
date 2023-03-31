<template>
  <div id="promo">
    <LoadWhenVisible>
      <Slider />
    </LoadWhenVisible>
  </div>
</template>

<script lang="ts" type="module">
import {
  defineComponent,
  ref,
  onMounted,
  useFetch,
} from '@nuxtjs/composition-api';
import { useCache, CacheTagPrefix } from '@vue-storefront/cache';
import { CmsPage } from '~/modules/GraphQL/types';
import Slider from '~/components/Slider.vue';
import { getMetaInfo } from '~/helpers/getMetaInfo';
import { useContent } from '~/composables';
import LoadWhenVisible from '~/components/utils/LoadWhenVisible.vue';

export default defineComponent({
  name: 'Promo',
  components: {
    Slider,

    LoadWhenVisible,
  },
  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  setup() {
    const { addTags } = useCache();
    const { loadPage } = useContent();

    const page = (ref < CmsPage) | (null > null);

    useFetch(async () => {
      page.value = await loadPage({ identifier: 'promo' });
    });

    onMounted(() => {
      addTags([{ prefix: CacheTagPrefix.View, value: 'promo' }]);
    });

    // @ts-ignore
    return {
      page,
    };
  },
  head() {
    return getMetaInfo(this.page);
  },
});
</script>
