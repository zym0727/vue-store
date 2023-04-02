<template>
  <div class="menu-container">
    <div class="shopping-box">
      <img src="~@/assets/imgs/logo.png" class="shopping-img" alt />
    </div>
    <el-menu
        :default-active="activeMenu"
        :router="true"
        background-color="#545c64"
        text-color="#fff"
        style="border: 0"
        active-text-color="#ffd04b">
      <el-menu-item index="/backend/user">
        <i class="el-icon-user"></i>
        <span slot="title">用户管理</span>
      </el-menu-item>
      <el-menu-item index="/backend/order">
        <i class="el-icon-mobile"></i>
        <span slot="title">订单管理</span>
      </el-menu-item>
      <el-menu-item index="/backend/product">
        <i class="el-icon-potato-strips"></i>
        <span slot="title">商品管理</span>
      </el-menu-item>
    </el-menu>
  </div>
</template>

<script>
export default {
  name: "Menu",
  data() {
    return {
      activeMenu: '/backend/user'
    }
  },
  watch: {
    '$route': {
      handler(newRoute) {
        this.activeMenu = newRoute.path
      },
      immediate: true
    }
  }
}
</script>

<style scoped>
.menu-container {
  height: 100vh;
  background: #3d3d3d;
}

.shopping-box {
  background: #2b2a36;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 60px;
}

.shopping-img {
  height: 30px;
}
</style>
