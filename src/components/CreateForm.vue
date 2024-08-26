<template>
  <div class="demo-image__placeholder">
    <section class="block">
      <el-alert v-if="isCreated" title="Successful" type="success" show-icon style="max-width: 480px;"/>
      <GetCard :create-completed="createCompleted" :imageLink="form.imageLink" @visible-image="visibleImage"/>
    </section>

    <section class="block">
      <el-form ref="formRef" :model="form" :rules="rules" label-width="auto" style="max-width: 600px; margin-top: 30px">
        <el-form-item prop="name" label="Product name">
          <el-input v-model="form.name" clearable/>
        </el-form-item>
        <el-form-item prop="price" label="Price">
          <el-input v-model="form.price"/>
        </el-form-item>
        <el-form-item prop="product_code" label="Product Code">
          <el-input v-model="form.product_code"/>
        </el-form-item>
        <el-form-item prop="quantity" label="Quantity">
          <el-input v-model="form.quantity"/>
        </el-form-item>
        <el-form-item label="Product state">
          <el-select v-model="form.state" :disabled="!(form.quantity > 0)">
            <el-option label="AVAILABLE" value="AVAILABLE"/>
            <el-option label="UNAVAILABLE" value="UNAVAILABLE"/>
          </el-select>
        </el-form-item>
        <el-form-item label="Product description">
          <el-input v-model="form.desc" type="textarea"/>
        </el-form-item>
        <el-form-item>
          <el-button v-if="!isCreated" @click="validateForm" type="primary">Create</el-button>
          <el-button @click="$emit('closeCreate')">Cancel</el-button>
        </el-form-item>
      </el-form>
    </section>

    <section>
      <el-dialog v-model="isVisibleImage" title="Inject" width="500" center>
        <span>
          Input your link at here!
        </span>
        <el-form-item label="Create Image">
          <el-input v-model="inputLinkImage"/>
        </el-form-item>
        <template #footer>
          <div class="dialog-footer">
            <el-button @click="isVisibleImage=false">Cancel</el-button>
            <el-button type="primary" @click="inputImage(inputLinkImage)">
              Confirm
            </el-button>
          </div>
        </template>
      </el-dialog>
    </section>
  </div>
</template>

<script lang="ts" setup>
import {onMounted, reactive, ref, watch} from 'vue'
import axios from "axios";
import moment from 'moment';
import GetCard from "@/components/common/ShowCard.vue";
import type {FormInstance} from "element-plus";


onMounted(() => {
  isCreated.value = false;
})
const emits = defineEmits(['closeCreate'])

const URL_PRODUCT = "http://localhost:8080/product";
const isVisibleImage = ref(false);
const inputLinkImage = ref('');
const isCreated = ref(false);
const formRef = ref<FormInstance>();
const form = reactive({
  name: '',
  price: null,
  product_code: '',
  quantity: null,
  imageLink: 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg',
  state: "UNAVAILABLE",
  desc: '',
})

const rules = {
  name: [
    {required: true, message: 'Please input product name', trigger: 'blur'}
  ],
  price: [
    {required: true, message: 'Please input price', trigger: 'blur'}
  ],
  product_code: [
    {required: true, message: 'Please input product code', trigger: 'blur'}
  ],
  quantity: [
    {required: true, message: 'Please input quantity', trigger: 'blur'}
  ]
}

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

const createCompleted = reactive<FormInp>({
  id: null,
  name: '',
  price: null,
  product_code: '',
  quantity: null,
  imageLink: '',
  state: 'UNAVAILABLE',
  desc: '',
  createdDate: '',
  createdBy: '',
})

const validateForm = () => {
  formRef.value?.validate((valid) => {
    if (valid) {
      create();
    } else {
      console.warn('Validation failed!');
    }
  });
};

const create = async () => {
  isCreated.value = false
  const body = {
    name: form.name,
    price: form.price,
    product_code: form.product_code,
    quantity: form.quantity,
    imageLink: form.imageLink,
    status: form.state,
    description: form.desc,
  }

  const {data} = await axios.post(URL_PRODUCT, body)

  if (data) {
    Object.assign(createCompleted, {
      name: form.name,
      price: form.price,
      product_code: form.product_code,
      quantity: form.quantity,
      imageLink: form.imageLink,
      state: form.state,
      desc: form.desc,
      id: data?.result?.id,
      createdDate: formatDate(data?.result?.createdDate),
      createdBy: data?.result?.createdBy,
    })

    isCreated.value = true
  } else {
    console.warn('Error creating product or response structure is incorrect.')
  }
}


const visibleImage = () => {
  isVisibleImage.value = true;
}


const inputImage = (image: string | null) => {
  if (image !== null && image !== '') {
    form.imageLink = image;
  }
  isVisibleImage.value = false;
}

const formatDate = (date: any) => {
  return moment(date).format('DD.MM.YYYY HH:mm:ss');
}


watch(() => isCreated.value == false, () => {
  if (!isCreated.value) {
    form.name = '';
    form.price = null;
    form.product_code = '';
    form.quantity = null;
    form.imageLink = 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg';
    form.state = 'UNAVAILABLE';
    form.desc = '';

  }
})

</script>


<style scoped>
.demo-image__placeholder .block {
  position: relative;
  padding: 30px 0;
  text-align: center;
  display: inline-block;
  width: 49%;
  box-sizing: border-box;
  vertical-align: top;
}

.demo-image__placeholder .el-image {
  padding: 0 5px;
  max-width: 300px;
  max-height: 50%;
}

.demo-image__placeholder .dot {
  animation: dot 2s infinite steps(3, start);
  overflow: hidden;
}

</style>