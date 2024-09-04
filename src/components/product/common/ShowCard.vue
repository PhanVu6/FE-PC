<script setup lang="ts">
import {Edit} from "@element-plus/icons-vue";
import {computed, ref} from "vue";


const checkLengthDesc = ref(true)

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

const props = withDefaults(
    defineProps<{
      result: FormInp;
      files: File[];
      imageLink: string;
      isUpdate?: boolean;
      modifiedBy?: string;
      modifiedDate?: string;
    }>(),
    {
      isUpdate: false,
      modifiedBy: '',
      modifiedDate: '',
    }
);
const imageUrls = computed(() => {
  // Tạo một mảng tạm để lưu các URL ảnh
  return props.files.map(file => URL.createObjectURL(file));
});
const changeLengthDesc = () => {
  checkLengthDesc.value = !checkLengthDesc.value
}

</script>

<template>
  <el-card style="max-width: 480px;">
    <div style="max-width: 30vw; max-height:33vh; overflow: auto">
      <!--      <el-image class="img-card" :src="imageLink"/>-->
      <el-image v-for="(url, index) in imageUrls" :key="index" :src="url"
                style="max-width: 25vw"/>
    </div>
    <span @click="$emit('visibleImage')" class="demonstration" style="margin-top: 20px">
          <el-icon>
            <Edit/>
          </el-icon>
        </span>
    <div class="information-main">
      <h1>{{ result.name }}</h1>
      <button disabled="true" style="margin-top: 10px; font-weight: 700;">
        Product Code: {{ result.product_code }}
      </button>
      <br>
      <div style="display:inline-flex; align-content: center; margin-top: 10px;">
        <h1>State:&nbsp; </h1>
        <h1 :style="result.state==='AVAILABLE'?'color:var(--el-color-success)':'color:var(--el-color-warning)'">
          {{ result.state }}
        </h1>
      </div>
    </div>

    <el-form :model="result" label-width="auto">
      <el-form-item label="Price: ">
        <el-form>{{ result.price }}</el-form>
      </el-form-item>
      <el-form-item label="Quantity: ">
        <el-form>{{ result.quantity }}</el-form>
      </el-form-item>
      <el-form-item label="Create by: ">
        <el-form>{{ result.createdBy }}</el-form>
      </el-form-item>
      <el-form-item label="Create date: ">
        <el-form>{{ result.createdDate }}</el-form>
      </el-form-item>
      <el-form-item v-if="isUpdate" label="Modified by: ">
        <el-form>{{ modifiedBy }}</el-form>
      </el-form-item>
      <el-form-item v-if="isUpdate" label="Modified date: ">
        <el-form>{{ modifiedDate }}</el-form>
      </el-form-item>
      <el-form-item label="Desc: ">
        <el-form>
          <p v-if="checkLengthDesc">
            {{ result.desc.slice(0, 100) }}
            <span
                class="more"
                v-if="result.desc.length > 100"
                @click="changeLengthDesc()"
            >
              ...See more
            </span>
          </p>
          <p v-else-if="!checkLengthDesc">
            {{ result.desc }}
            <span
                class="more"
                v-if="result.desc.length > 100"
                @click="changeLengthDesc()"
            >
              .See less
            </span>
          </p>
        </el-form>
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

.description h3 {
  background: #d6d0cf;
  height: 34px;
  align-content: center;
  padding: 5px;
}

.description p {
  margin: 14px 3px;
}

.more {
  color: rgb(50, 118, 206);
}

.more:hover {
  cursor: pointer;
  color: green;
  text-decoration: underline;
}

</style>
