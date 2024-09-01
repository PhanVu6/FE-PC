<template>
  <div style="margin-bottom: 100px">
    <section>
      <div class="flex gap-4 mb-5 items-center">
        <el-input
            v-model="search"
            style="width: 30%; margin: 30px 0"
            placeholder="Search"
            clearable
        />
        <el-button @click="getAllProduct()">
          <el-icon>
            <Search :suffix-icon="Search"/>
          </el-icon>
        </el-button>
      </div>
      <div class="create-button">
        <el-button @click="$emit('openCreate')" type="primary" :icon="CirclePlus" size="default">
          Create Product
        </el-button>
      </div>
    </section>

    <section>
      <el-table :data="products" border>
        <el-table-column fixed type="index" label="No" width="90"/>
        <el-table-column prop="name" label="Name" width="120"/>
        <el-table-column prop="description" label="Description" width="290">
          <template #default="scope">
            <div class="description-text">
              {{ scope.row.description }}
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="status" label="Status" width="130">
          <template #default="scope">
            <span
                :style="
                scope.row.status === 'AVAILABLE'
                  ? 'color:var(--el-color-success)'
                  : 'color:var(--el-color-warning)'
              "
            >
              {{ scope.row.status }}
            </span>
          </template>
        </el-table-column>
        <el-table-column prop="price" label="Price" width="120"/>
        <el-table-column prop="product_code" label="Product Code" width="100"/>
        <el-table-column prop="quantity" label="Quantity" width="100"/>
        <el-table-column prop="createdBy" label="Saler" width="120"/>
        <el-table-column fixed="right" label="Operations" min-width="80px">
          <template #default="scope">
            <el-button
                @click="$emit('viewDetail', scope.row.name, scope.row.id)"
                type="primary"
                size="small"
                circle
            >
              <el-icon>
                <Document/>
              </el-icon>
            </el-button>
            <el-button type="warning" @click="$emit('update', scope.row.id)" size="small" circle>
              <el-icon>
                <Edit/>
              </el-icon>
            </el-button>
            <el-button type="danger" size="small" @click="confirmDelete(scope.row.id)" circle>
              <el-icon>
                <DeleteFilled/>
              </el-icon>
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-dialog v-model="visible" title="Confirm Delete" width="500" draggable>
        <span>Are you sure you want to delete this product?</span>
        <template #footer>
          <div class="dialog-footer">
            <el-button @click="visible = false"> Cancel</el-button>
            <el-button type="primary" @click="Delete"> Delete</el-button>
          </div>
        </template>
      </el-dialog>
    </section>

    <section>
      <div class="demo-pagination-block" style="margin-top: 20px">
        <div class="demonstration">Change page size</div>
        <el-pagination
            v-model:current-page="page.currentPage"
            v-model:page-size="page.pageSize"
            :page-sizes="[1, 5, 10]"
            :size="size"
            :background="true"
            layout="sizes, prev, pager, next"
            :total="page.totalElement"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
        />
      </div>
    </section>
  </div>
</template>

<script lang="ts" setup>
import axios from 'axios'
import {onMounted, reactive, ref, watch} from 'vue'
import {CirclePlus, DeleteFilled, Document, Edit, Search} from '@element-plus/icons-vue'
import type {ComponentSize} from 'element-plus'

onMounted(() => {
  getAllProduct()
})

interface EditingRows {
  [key: string]: boolean
}

const emit = defineEmits(['update'])
const props = defineProps({
  loadTable: Boolean
})

const URL_PRODUCT = 'http://localhost:8080/product'
const products = ref([])
const search = ref('')
const size = ref<ComponentSize>('default')
const visible = ref(false)
const editingRows = ref<EditingRows>({})
const deleteProductId = ref()
const page = reactive({
  currentPage: 1,
  pageSize: 10,
  totalElement: 0
})

interface FormInp {
  id: number
  name: string
  price: number | null
  product_code: string
  quantity: number | null
  imageLink: string
  status: 'UNAVAILABLE' | 'AVAILABLE'
  description: string
  createdDate: string
  createdBy: string
}

const getAllProduct = async () => {
  try {
    page.currentPage--

    const params = new URLSearchParams({
      name: search.value,
      page: String(page.currentPage),
      size: String(page.pageSize)
    })

    const {data} = await axios.get(`${URL_PRODUCT}`, {params})

    products.value = data?.result?.content ?? []
    page.totalElement = data?.result?.totalElements

    const pageable = data?.result?.pageable
    page.currentPage = pageable.pageNumber + 1
    page.pageSize = pageable.pageSize

    if (products.value.length === 0) {
      console.warn('No students found or response structure is incorrect.')
    }
  } catch (error) {
    console.error('Failed to fetch students:', error)
  }
}

const Delete = async () => {
  try {
    const {data} = await axios.delete(`${URL_PRODUCT}/${deleteProductId.value}`)

    if (data) {
      await getAllProduct()
      visible.value = false
      return data
    } else {
      console.warn('Error deleting student or response structure is incorrect.')
    }
  } catch (error) {
    console.error('Error delete student:', error.message)
  }
}

const handleSizeChange = (val: number) => {
  page.pageSize = val
  page.currentPage = 1
  getAllProduct()
  console.log(`${val} items per page`)
}
const handleCurrentChange = (val: number) => {
  page.currentPage = val
  getAllProduct()
  console.log(`current page: ${val}`)
}

const confirmDelete = (id: number) => {
  deleteProductId.value = id // Lưu ID của dòng cần xóa
  visible.value = true // Mở dialog xác nhận
}

watch(
    () => props.loadTable,
    () => {
      getAllProduct()
    }
)
</script>

<style scoped>
.create-button {
  display: flex;
  width: 100%;
  justify-content: flex-end;
  margin: 10px 0;
}

.my-autocomplete li {
  line-height: normal;
  padding: 7px;
}

.my-autocomplete li .name {
  text-overflow: ellipsis;
  overflow: hidden;
}

.my-autocomplete li .addr {
  font-size: 12px;
  color: #b4b4b4;
}

.my-autocomplete li .highlighted .addr {
  color: #ddd;
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
