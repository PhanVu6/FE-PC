<script setup lang="ts">
import {Menu as IconMenu} from '@element-plus/icons-vue'
import TableProduct from "@/components/TableProduct.vue";
import {ref} from "vue";
import CreateForm from "@/components/CreateForm.vue";
import UpdateForm from "@/components/UpdateForm.vue";

const idProduct = ref();
const isViewProduct = ref(true);
const isCreate = ref(false);
const isUpdate = ref(false);
const updateTable = ref(false);

const injectUpdate = (id: number) => {
  isUpdate.value = true;
  idProduct.value = id;
}
const viewProduct = () => {
  isViewProduct.value = !isViewProduct.value;
}
const viewCreate = () => {
  isCreate.value = false;
}
const openCreate = () => {
  isCreate.value = true
}

const loadTable = () => {
  // updateTable giá trị true hay false đều update lại table hết, đây chỉ là hàm để TableProduct nhận biết có tác động
  updateTable.value = !updateTable.value;
}

const viewUpate = () => {
  isUpdate.value = false;
}

</script>

<template>

  <el-container class="layout-container-demo">
    <el-aside width="200px" style="height: 97vh;">
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
            <el-menu-item index="1-1" @click="viewProduct"
                          :style="isViewProduct?
                          'background: var(--el-menu-hover-bg-color); color: cornflowerblue;'
                          :''">
              Manager Product
            </el-menu-item>
            <!--            <el-menu-item index="1-2" @click="viewCourse">Table Course</el-menu-item>-->
          </el-sub-menu>
        </el-menu>
      </el-scrollbar>
    </el-aside>

    <el-container>
      <el-header style="text-align: left; font-size: 40px;">
        <div class="toolbar">
          <el-button @click="openCreate" color="#626aef" plain>Create Product</el-button>
          <el-button @click="viewProduct" color="#626aef" plain
                     :style="isViewProduct?'background-color: var(--el-button-hover-bg-color);'+
                        'border-color: var(--el-button-hover-border-color);'+
                        'color: var(--el-button-hover-text-color);'+
                        'outline: none;':''">
            Manager Product
          </el-button>
        </div>
      </el-header>

      <el-main>
        <div style="overflow: auto; height: 84vh" v-if="isViewProduct">
          <TableProduct :loadTable="updateTable" @update="injectUpdate"/>
        </div>
      </el-main>
    </el-container>


    <!-- Dialog Create -->
    <el-dialog v-model="isCreate" title="Create Product" style="width: 80vw; top:-13%;">
      <CreateForm v-if="isCreate" @close-create="viewCreate"/>
    </el-dialog>

    <!-- Dialog Update -->
    <el-dialog v-model="isUpdate" title="Update Product" style="width: 80vw; top:-13%;">
      <UpdateForm v-if="isUpdate" @close-update="viewUpate" :idProduct="idProduct" @load-table="loadTable"/>
    </el-dialog>
  </el-container>
</template>

<style scoped>
.layout-container-demo .el-header {
  border-bottom: 1px solid var(--el-border-color-darker);
  border-top: 1px solid var(--el-border-color-darker);
}

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
  background: #625C70;
  color: #dddddd;
}
</style>