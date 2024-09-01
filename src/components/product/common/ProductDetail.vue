<script setup lang="ts">
import {onMounted, reactive, ref, watch} from 'vue'
import axios from 'axios'
import moment from 'moment/moment'

onMounted(() => {
  getDetailProduct(props.idProduct)
})

const props = defineProps({
  idProduct: {
    type: Number,
    required: true
  },
  loadTable: Boolean
})

const URL_PRODUCT = 'http://localhost:8080/product'
const checkLengthDesc = ref(true)
const infor = reactive({
  id: null,
  name: '',
  description: '',
  price: '',
  product_code: '',
  quantity: null,
  imageLink: '',
  status: '',
  createdDate: '',
  modifiedDate: '',
  createdBy: '',
  modifiedBy: '',
  categories: []
})

const getDetailProduct = async (id: number) => {
  try {
    const {data} = await axios.get(`${URL_PRODUCT}/${id}`)

    if (data?.result) {
      infor.id = data?.result?.id
      infor.name = data?.result?.name
      infor.description = data?.result?.description
      infor.price = formatPrice(data?.result?.price)
      infor.product_code = data?.result?.product_code
      infor.quantity = data?.result?.quantity
      infor.imageLink = data?.result?.imageLink
      infor.status = data?.result?.status
      infor.createdDate = formatDate(data?.result?.createdDate)
      infor.createdBy = data?.result?.createdBy
      infor.modifiedBy = data?.result?.modifiedBy
      infor.modifiedDate = formatDate(data?.result?.modifiedDate)
      infor.categories = data.result.categories
    }
  } catch (error) {
    console.error('Failed to fetch students:', error)
  }
}
const formatDate = (date: any) => {
  return moment(date).format('DD.MM.YYYY HH:mm:ss')
}

const formatPrice = (value: number) => {
  let val = (value / 1).toFixed(0).replace('.', ',')
  return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.')
}

const changeLengthDesc = () => {
  checkLengthDesc.value = !checkLengthDesc.value
}

watch(
    () => props.loadTable,
    () => {
      getDetailProduct(props.idProduct)
    }
)
</script>

<template>
  <el-row :gutter="20">
    <el-col :span="10">
      <el-image class="img-detail" :src="infor.imageLink"/>
    </el-col>
    <el-col :span="14">
      <div class="header">
        <h1>{{ infor.name }}</h1>
      </div>
      <div class="container">
        <div class="product_code-status">
          <p>Product code: {{ infor.product_code }}</p>
          <span>State: {{ infor.status }}</span>
        </div>
        <div class="price">₫{{ infor.price }}</div>

        <div class="quantity">Quantity: {{ infor.quantity }}</div>
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
  <el-table :data="infor.categories" style="width: 100%" border>
    <el-table-column prop="name" label="Name" width="180"/>
    <el-table-column prop="status" label="Status" width="180"/>
    <el-table-column prop="category_code" label="Category code" width="180"/>
    <el-table-column prop="description" label="Description">
      <template #default="scope">
        <div class="description-text">
          {{ scope.row.description }}
        </div>
      </template>
    </el-table-column>
  </el-table>
</template>

<style scoped>
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
