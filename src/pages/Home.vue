<script setup lang="ts">
import {Menu as IconMenu} from '@element-plus/icons-vue';
import type {TabPaneName} from 'element-plus'
import TableProduct from "@/components/TableProduct.vue";
import {onMounted, ref} from "vue";
import CreateForm from "@/components/CreateForm.vue";
import UpdateForm from "@/components/UpdateForm.vue";

onMounted(() => {
  viewProduct();
})
const idProduct = ref();
const isViewProduct = ref(true);
const isCreate = ref(false);
const isUpdate = ref(false);
const updateTable = ref(false);
const isViewProductTab = ref(false); // Thêm biến này để kiểm soát việc bật/tắt tab TableProduct
interface FormTabs {
  title: string,
  name: string,
  content: string,
}

let tabIndex = 1
const editableTabsValue = ref('table-product')
const editableTabs = ref<FormTabs[]>([])

const handleTabsEdit = (
    targetName: TabPaneName | undefined,
    action: 'remove' | 'add'
) => {
  if (action === 'add') {
    const newTabName = `${++tabIndex}`
    editableTabs.value.push({
      title: 'New Tab',
      name: newTabName,
      content: 'New Tab content',
    })
    editableTabsValue.value = newTabName
  } else if (action === 'remove') {
    const tabs = editableTabs.value
    let activeName = editableTabsValue.value
    if (activeName === targetName) {
      tabs.forEach((tab, index) => {
        if (tab.name === targetName) {
          const nextTab = tabs[index + 1] || tabs[index - 1]
          if (nextTab) {
            activeName = nextTab.name
          }
        }
      })
    }

    if (targetName === 'table-product') {
      isViewProductTab.value = false; // Cập nhật trạng thái khi tab bị xóa
    }

    editableTabsValue.value = activeName
    editableTabs.value = tabs.filter((tab) => tab.name !== targetName)
  }
}

const injectUpdate = (id: number) => {
  isUpdate.value = true;
  idProduct.value = id;
}
const viewProduct = () => {
  isViewProductTab.value = !isViewProductTab.value;

  if (isViewProductTab.value) {
    editableTabs.value.push({
      title: 'Table Product',
      name: 'table-product',
      content: '',
    });
    editableTabsValue.value = 'table-product';
  } else {
    handleTabsEdit('table-product', 'remove');
  }
}

const viewCreate = () => {
  isCreate.value = false;
}
const openCreate = () => {
  isCreate.value = true
}

const loadTable = () => {
  updateTable.value = !updateTable.value;
}

const viewUpate = () => {
  isUpdate.value = false;
}

</script>

<template>

  <el-container class="layout-container-demo" style="height: 100vh;">
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
            <el-menu-item index="1-1" @click="viewProduct"
                          :style="isViewProductTab?
                          'background: var(--el-menu-hover-bg-color); color: cornflowerblue;'
                          :''">
              Manager Product
            </el-menu-item>
          </el-sub-menu>
        </el-menu>
      </el-scrollbar>
    </el-aside>


    <el-main>
      <el-tabs
          v-model="editableTabsValue"
          type="card"
          editable
          class="demo-tabs"
          @edit="handleTabsEdit"
      >
        <el-tab-pane
            v-for="item in editableTabs"
            :key="item.name"
            :label="item.title"
            :name="item.name"
        >
          <div v-if="item.name === 'table-product'">
            <TableProduct :loadTable="updateTable" @open-create="openCreate" @update="injectUpdate"/>
          </div>
        </el-tab-pane>
      </el-tabs>

      <!-- Dialog Create -->
      <el-dialog v-model="isCreate" title="Create Product" style="width: 80vw; top:-13%;">
        <CreateForm v-if="isCreate" @close-create="viewCreate" @load-table="loadTable"/>
      </el-dialog>

      <!-- Dialog Update -->
      <el-dialog v-model="isUpdate" title="Update Product" style="width: 80vw; top:-13%;">
        <UpdateForm v-if="isUpdate" @close-update="viewUpate" :idProduct="idProduct" @load-table="loadTable"/>
      </el-dialog>
    </el-main>

  </el-container>

</template>

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
  background: #625C70;
  color: #dddddd;
}

.demo-tabs > .el-tabs__content {
  color: #6b778c;
  font-size: 32px;
  font-weight: 600;
}
</style>
