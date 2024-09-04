<script setup lang="ts">
import {onMounted, reactive, ref, watch} from 'vue'
import axios from 'axios'
import moment from 'moment/moment'

interface ImageCategory {
  "id": number,
  "imageName": string,
  "imagePath": string
}

interface FormInp {
  id: number | null;
  name: string;
  category_code: string;
  imageLink: string;
  status: 'UNAVAILABLE' | 'AVAILABLE';
  description: string;
  createdDate: string;
  createdBy: string;
  modifiedBy: string;
  modifiedDate: string;
  imageCategories: ImageCategory[],
}

onMounted(() => {
  getDetailCategory(props.idCategory)
})

const props = defineProps({
  idCategory: {
    type: Number,
    required: true
  },
  loadTable: Boolean
})


const URL_CATEGORY = 'http://localhost:8080/category'
const URL_IMAGES = 'http://localhost:8080/image-category/api/images';
const checkLengthDesc = ref(true)
const infor = reactive<FormInp>({
  id: null,
  name: '',
  description: '',
  category_code: '',
  imageLink: '',
  status: 'UNAVAILABLE',
  createdDate: '',
  modifiedDate: '',
  createdBy: '',
  modifiedBy: '',
  imageCategories: [],
})


const getDetailCategory = async (id: number) => {
  try {
    const {data} = await axios.get(`${URL_CATEGORY}/${id}`)

    if (data?.result) {
      infor.id = data?.result?.id
      infor.name = data?.result?.name
      infor.description = data?.result?.description
      infor.category_code = data?.result?.category_code
      infor.imageLink = data?.result?.imageLink
      infor.status = data?.result?.status
      infor.createdDate = formatDate(data?.result?.createdDate)
      infor.createdBy = data?.result?.createdBy
      infor.modifiedBy = data?.result?.modifiedBy
      infor.modifiedDate = formatDate(data?.result?.modifiedDate)
      infor.imageCategories = data.result.imageCategories.map(img => ({
        ...img,
        imagePath: `${URL_IMAGES}?imageName=${encodeURIComponent(img.imageName)}` // Thay đổi đường dẫn đến URL API
      }));
    }
  } catch (error) {
    console.error('Failed to fetch students:', error)
  }
}
const formatDate = (date: any) => {
  return moment(date).format('DD.MM.YYYY HH:mm:ss')
}

const changeLengthDesc = () => {
  checkLengthDesc.value = !checkLengthDesc.value
}

watch(
    () => props.loadTable,
    () => {
      getDetailCategory(props.idCategory)
    }
)
</script>

<template>
  <el-row :gutter="20">
    <el-col :span="10">
      <div class="demo-image__lazy">
        <!--        <el-image class="img-detail" :src="infor.imageLink"/>-->
        <el-image v-for="url in infor.imageCategories" :key="url?.id" :src="url?.imagePath" lazy/>
      </div>
    </el-col>
    <el-col :span="14">
      <div class="header">
        <h1>{{ infor.name }}</h1>
      </div>
      <div class="container">
        <div class="product_code-status">
          <p>Category code: {{ infor.category_code }}</p>
          <span>State: {{ infor.status }}</span>
        </div>
        <div class="description">
          <h3>Describe</h3>
          <p v-if="checkLengthDesc">
            {{ infor.description.slice(0, 150) }}
            <span
                class="more"
                v-if="infor.description.length > 150"
                @click="changeLengthDesc()"
            >
              ...See more
            </span>
          </p>
          <p v-else-if="!checkLengthDesc">
            {{ infor.description }}
            <span
                class="more"
                v-if="infor.description.length > 150"
                @click="changeLengthDesc()"
            >
              .See less
            </span>
          </p>
        </div>
      </div>
    </el-col>
  </el-row>
</template>

<style scoped>
.demo-image__lazy {
  height: 450px;
  overflow-y: auto;
}

.demo-image__lazy .el-image {
  display: block;
  min-height: 200px;
  margin-bottom: 10px;
}

.demo-image__lazy .el-image:last-child {
  margin-bottom: 0;
}

.img-detail {
  border-radius: 30px;
}

.product_code-status {
  display: flex;
  margin-top: 10px;
}

.product_code-status p {
  border-right: 1px solid burlywood;
  margin-right: 5px;
}

.product_code-status p,
.product_code-status span {
  padding: 4px;
  margin-bottom: 3px;
}

.price {
  height: 50px;
  text-align: center;
  align-content: center;
  background: burlywood;
  color: #ee4d2d;
  font-size: 1.875rem;
  font-weight: 500;
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

.quantity {
  margin: 16px 0;
}

.el-table {
  margin-bottom: 200px;
}

.description-text {
  display: -webkit-box;
  -webkit-line-clamp: 2; /* Số dòng tối đa */
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: normal;
}
</style>
