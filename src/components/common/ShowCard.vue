<script setup lang="ts">
import {Edit} from "@element-plus/icons-vue";


interface FormInp {
  id: number | null;
  name: string;
  price: number | null;
  product_code: string;
  quantity: number | null;
  imageLink: string;
  state: 'UNAVAILABLE' | 'AVAILABLE';
  desc: string;
  createdDate: string;
  createdBy: string;
}

const props = defineProps<{
  createCompleted: FormInp;
  imageLink: string;
}>();
</script>

<template>
  <el-card style="max-width: 480px;">
    <el-image class="img-card" :src="imageLink"/>
    <span @click="$emit('visibleImage')" class="demonstration">
          <el-icon>
            <Edit/>
          </el-icon>
        </span>
    <div class="information-main">
      <h1>{{ createCompleted.name }}</h1>
      <button disabled="true" style=" font-weight: 700;">
        Product Code: {{ createCompleted.product_code }}
      </button>
      <br>
      <div style="display:inline-flex; align-content: center; color: goldenrod;">
        <h1>State:&nbsp; </h1>
        <h5 :style="createCompleted.state==='AVAILABLE'?'color:goldenrod':'color:#A0A8AB'">
          {{ createCompleted.state }}
        </h5>
      </div>
    </div>

    <el-form :model="createCompleted" label-width="auto">
      <el-form-item label="Price: ">
        <el-form>{{ createCompleted.price }}</el-form>
      </el-form-item>
      <el-form-item label="Quantity: ">
        <el-form>{{ createCompleted.quantity }}</el-form>
      </el-form-item>
      <el-form-item label="Create by: ">
        <el-form>{{ createCompleted.createdBy }}</el-form>
      </el-form-item>
      <el-form-item label="Create date: ">
        <el-form>{{ createCompleted.createdDate }}</el-form>
      </el-form-item>
      <el-form-item label="Desc: ">
        <el-form>{{ createCompleted.desc }}</el-form>
      </el-form-item>
    </el-form>
  </el-card>
</template>

<style scoped>
.demo-image__placeholder .demonstration {
  display: block;
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 20px;
}

.img-card {
  max-width: 280px;
}
</style>
