<script>
  import axios from "axios"
  import AppNavBar from "./components/AppNavBar.vue"
  import AppFooter from "./components/AppFooter.vue"
  import MainMap from "./components/MainMap.vue"

  export default {
    components: { AppNavBar, AppFooter, MainMap },
  }
</script>

<template>
  <AppNavBar />
  <div class="container">
    <MainMap />
  </div>
  <AppFooter />
</template>

<style scoped lang="scss">

</style>