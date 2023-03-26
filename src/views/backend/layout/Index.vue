<template>
  <el-container class="layout-container">
    <el-aside style="width: 10vw">
      <Menu/>
    </el-aside>

    <el-container>
      <el-header><Header/></el-header>
      <el-main class="main-content"><router-view/></el-main>
      <el-footer><Footer/></el-footer>
    </el-container>

  </el-container>
</template>

<script>
import Header from "@/views/backend/layout/Header.vue";
import Menu  from "@/views/backend/layout/Menu.vue";
import Footer from "@/views/backend/layout/Footer.vue";

export default {
  name: "Layout",
  components: {Header, Menu, Footer}
}
</script>

<style scoped>
.layout-container {
  width: 100vw;
  height: 100vh;
}

.main-content {
  background: #e8eaed;
}
</style>