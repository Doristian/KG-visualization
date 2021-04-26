<template>
  <div id="app">
    <el-container>

        <el-menu
            :default-active="activeIndex"
            class="el-menu-demo"
            mode="horizontal"
            @select="handleSelect"
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#ffd04b"
            router>
          <el-menu-item index="/" >首页</el-menu-item>
          <el-menu-item index="/kg" >知识图谱</el-menu-item>
          <el-menu-item index="3" >消息中心</el-menu-item>
          <el-menu-item index="4">个人中心</el-menu-item>
        </el-menu>


      <el-main>
        <router-view></router-view>
      </el-main>
      <el-footer>知识图谱可视化</el-footer>
    </el-container>

  </div>
</template>

<script>
export default {
  data() {
    return {
      activeIndex: '/',
    };
  },
  methods: {
    handleSelect(key, keyPath) {
      console.log(key, keyPath);
    }
  }
}
</script>

<style>

.el-footer {
  background-color: #545c64;
  color: #333;
  text-align: center;
  line-height: 60px;
}
.el-main {
  background-color: #0b2838;
  color: #333;
  text-align: center;
  line-height: 160px;
}
body > .el-container {
  margin-bottom: 40px;
}
</style>
