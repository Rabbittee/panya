<template>
  <ToastMessage/>
  <PageLoading/>
  <router-view/>
</template>

<script>
export default {
  data() {
    return {};
  },
  watch: {
    $route(to) {
      const nowTitle = to.meta.title || '';
      document.title = `${nowTitle} PANYA`;
    },
  },
};
</script>

<style lang="sass">
@import '@/assets/css/style'
</style>
