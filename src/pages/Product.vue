<template>
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
      <component
          :is="item.content"
          :props="item.props"
          @open-create="openCreate"
          @view-detail="viewDetailProduct"
          @update="injectUpdate"
      />
    </el-tab-pane>
  </el-tabs>
</template>

<script setup lang="ts">
import {ref} from 'vue'
import type {TabPaneName} from 'element-plus'
import ProductDetail from "@/components/product/common/ProductDetail.vue";

const editableTabsValue = ref('table-product')
const editableTabs = ref<{ title: string, name: string, content: any, props?: Record<string, any> }[]>([])
const isCreate = ref(false)
const handleTabsAdd = (title: string, targetName: TabPaneName | undefined, content: any, props?: any) => {
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

  editableTabsValue.value = activeName
  editableTabs.value = tabs.filter((tab) => tab.name !== targetName)
}

const openCreate = () => {
  isCreate.value = true
}

const viewDetailProduct = (nameProduct: string, id: number) => {
  nameProduct = 'View Detail: ' + nameProduct
  handleTabsAdd(nameProduct, id, ProductDetail, {idProduct: id, loadTable: updateTable})
}

const injectUpdate = (id: number) => {
  // Define how to handle update
}
</script>

<style scoped>
/* Add any styles specific to the tabs component here */
</style>
