<template>
  <div class="header-container">
    <div class="back-btn" @click="back">返回商城</div>
  </div>
</template>

<script>
export default {
  name: "Header",
  methods: {
    back() {
      this.$router.push('/')
    }
  }
}
</script>

<style scoped>
.header-container {
  display: flex;
  width: 100%;
  height: 100%;
  box-shadow: 0 3px 5px 1px rgba(0,0,0,.1);
  line-height: 60px;
  border-bottom: 1px solid #eff1f2;
  background: #fff;
  font-size: 14px;
}

.back-btn {
  margin-left: auto;
  margin-right: 50px;
  cursor: pointer;
}
</style>