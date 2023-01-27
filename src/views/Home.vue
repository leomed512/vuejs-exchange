<template>
  <div>
    <bounce-loader :loading="isLoading" :color="`#68d391`" :size="200" />
    <px-assets-table v-if="!isLoading" :assets="assets" />
  </div>
</template>

<script>
import api from "@/api";
import PxAssetsTable from "@/components/PxAssetsTable.vue";

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Home",
  components: { PxAssetsTable },

  data() {
    return {
      assets: [],
      isLoading: false,
    };
  },
  created() {
    this.isLoading = true;
    api
      .getAssets()
      .then((assets) => (this.assets = assets))
      .finally(() => (this.isLoading = false));
  },
};
</script>
