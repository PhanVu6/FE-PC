<template>
  <el-container class="layout-container-demo" style="height: 100vh">
    <el-aside width="200px">
      <el-scrollbar>
        <el-menu :default-openeds="['1']">
          <el-sub-menu index="1">
            <template #title>
              <p>
                <el-icon>
                  <icon-menu/>
                </el-icon>
                Menu
              </p>
            </template>
            <!--            <el-menu-item-->
            <!--                index="1-1"-->
            <!--                @click="viewProduct"-->
            <!--                :style="-->
            <!--                isViewProductTab-->
            <!--                  ? 'background: var(&#45;&#45;el-menu-hover-bg-color); color: cornflowerblue;'-->
            <!--                  : ''-->
            <!--              "-->
            <!--            >-->
            <!--              <router-link to=""> Manager Product</router-link>-->
            <!--            </el-menu-item>-->
          </el-sub-menu>
        </el-menu>
      </el-scrollbar>
    </el-aside>

    <el-main>
      <router-view></router-view>
    </el-main>
  </el-container>
</template>

<script setup lang="ts">

</script>

<style scoped>
.layout-container-demo .el-aside,
.el-menu .el-sub-menu {
  background: #494455;
}

.el-menu .el-sub-menu p {
  color: #dddddd;
}

.el-menu .el-sub-menu:hover p {
  color: cornflowerblue;
}

.layout-container-demo .el-aside .el-menu-item {
  background: #625c70;
  color: #dddddd;
}

.demo-tabs > .el-tabs__content {
  color: #6b778c;
  font-size: 32px;
  font-weight: 600;
}
</style>
