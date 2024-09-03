<script setup lang="ts">
import {Edit} from "@element-plus/icons-vue";
import {ref} from "vue";

const checkLengthDesc = ref(true)

interface FormInp {
  id: number | null;
  name: string;
  category_code: string;
  imageLink: string;
  status: 'UNAVAILABLE' | 'AVAILABLE';
  description: string;
  createdDate: string;
  createdBy: string;
}

const props = withDefaults(
    defineProps<{
      result: FormInp;
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

const changeLengthDesc = () => {
  checkLengthDesc.value = !checkLengthDesc.value
}
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
      <h1>{{ result.name }}</h1>
      <button disabled="true" style="margin-top: 10px; font-weight: 700;">
        Category Code: {{ result.category_code }}
      </button>
      <br>
      <div style="display:inline-flex; align-content: center; margin-top: 10px;">
        <h1>State:&nbsp; </h1>
        <h1 :style="result.status==='AVAILABLE'?'color:var(--el-color-success)':'color:var(--el-color-warning)'">
          {{ result.status }}
        </h1>
      </div>
    </div>

    <el-form :model="result" label-width="auto">
      <el-form-item label="Category Code: ">
        <el-form>{{ result.category_code }}</el-form>
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
            {{ result.description.slice(0, 100) }}
            <span
                class="more"
                v-if="result.description.length > 100"
                @click="changeLengthDesc()"
            >
              ...See more
            </span>
          </p>
          <p v-else-if="!checkLengthDesc">
            {{ result.description }}
            <span
                class="more"
                v-if="result.description.length > 100"
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
