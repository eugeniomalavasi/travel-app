<script>
  import axios from "axios"
  import AppNavBar from "./components/AppNavBar.vue"
  import AppFooter from "./components/AppFooter.vue"
  import Map from "./components/Map.vue"

  export default {
    components: { AppNavBar, AppFooter, Map },
  }
</script>

<template>
  <AppNavBar />
  <div class="">
    <Map />
  </div>
  <AppFooter />
</template>

<style scoped lang="scss">
    @import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400..700&display=swap');

</style>