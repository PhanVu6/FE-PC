<script setup lang="ts">
import {Menu as IconMenu} from '@element-plus/icons-vue'
import type {TabPaneName} from 'element-plus'
import TableProduct from '@/components/product/TableProduct.vue'
import {onMounted, ref} from 'vue'
import InputFormProduct from '@/components/product/InputFormProduct.vue'
import ProductDetail from '@/components/product/common/ProductDetail.vue'

onMounted(() => {
  viewProduct()
})
const idProduct = ref()
const isCreate = ref(false)
const isUpdate = ref(false)
const updateTable = ref(false)
const isViewProductTab = ref(false)

interface FormTabs {
  title: string
  name: string
  content: string
  props?: Record<string, any>
}

const editableTabsValue = ref('table-product')
const editableTabs = ref<FormTabs[]>([])

const handleTabsAdd = (
    title: string,
    targetName: TabPaneName | undefined,
    content: any,
    props?: any
) => {
  const existTabs = editableTabs.value.find((tab) => tab.name === String(targetName))
  if (!existTabs) {
    editableTabs.value.push({
      title: title,
      name: String(targetName),
      content: content,
      props: props
    })
  }
  editableTabsValue.value = String(targetName)
}

const handleCloseTabs = (targetName: TabPaneName | undefined) => {
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
    isViewProductTab.value = false // Cập nhật trạng thái khi tab bị xóa
  }

  editableTabsValue.value = activeName
  editableTabs.value = tabs.filter((tab) => tab.name !== targetName)
}

const injectUpdate = (id: number) => {
  isUpdate.value = true
  idProduct.value = id
}
const viewProduct = () => {
  isViewProductTab.value = !isViewProductTab.value

  if (isViewProductTab.value) {
    handleTabsAdd('Table Product', 'table-product', TableProduct, {loadTable: updateTable})
  } else {
    handleCloseTabs('table-product')
  }
}

const viewDetailProduct = (nameProduct: string, id: number) => {
  nameProduct = 'View Detail: ' + nameProduct
  handleTabsAdd(nameProduct, id, ProductDetail, {idProduct: id, loadTable: updateTable})
}

const viewCreate = () => {
  isCreate.value = false
}
const openCreate = () => {
  isCreate.value = true
}

const loadTable = () => {
  updateTable.value = !updateTable.value
}

const viewUpate = () => {
  isUpdate.value = false
}
</script>

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
            <el-menu-item
                index="1-1"
                @click="viewProduct"
                :style="
                isViewProductTab
                  ? 'background: var(--el-menu-hover-bg-color); color: cornflowerblue;'
                  : ''
              "
            >
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
          closable
          class="demo-tabs"
          @tab-remove="handleCloseTabs"
      >
        <el-tab-pane
            v-for="item in editableTabs"
            :key="item.name"
            :label="item.title"
            :name="item.name"
        >
          <!--  open-create , view-detail , update: là của product   -->
          <component
              :is="item.content"
              :="item.props"
              @open-create="openCreate"
              @view-detail="viewDetailProduct"
              @update="injectUpdate"
          />
        </el-tab-pane>
      </el-tabs>

      <!-- Dialog Create -->
      <!--      <el-dialog v-model="isCreate" title="Create Product" style="width: 80vw; top: -13%">-->
      <!--        <CreateForm v-if="isCreate" @close-create="viewCreate" @load-table="loadTable" />-->
      <!--      </el-dialog>-->
      <el-dialog v-model="isCreate" title="Create Product" style="width: 80vw; top: -13%">
        <InputFormProduct v-if="isCreate" :isCreate="isCreate" @close-create="viewCreate"
                          :idProduct="idProduct"
                          @load-table="loadTable"/>
      </el-dialog>

      <!-- Dialog Update -->
      <el-dialog v-model="isUpdate" title="Update Product" style="width: 80vw; top: -13%">
        <InputFormProduct
            :isCreate="isCreate"
            v-if="isUpdate"
            @close-update="viewUpate"
            :idProduct="idProduct"
            @load-table="loadTable"
        />
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
  background: #625c70;
  color: #dddddd;
}

.demo-tabs > .el-tabs__content {
  color: #6b778c;
  font-size: 32px;
  font-weight: 600;
}
</style>
