<template>
  <div class="backend-container">
    <layout/>
  </div>
</template>

<script>
import Layout from './layout/Index.vue'
export default {
  name: "Backend",
  components: {Layout},
}
</script>

<style scoped>
.backend-container {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 9;
  width: 100vw;
  height: 100vh;
}
</style>