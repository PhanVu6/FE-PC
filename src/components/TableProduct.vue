<template>
  <section>
    <div class="flex gap-4 mb-4 items-center">
      <el-input @input="getAllProduct()" v-model="search" style="width: 30%; margin: 30px 0;" placeholder="Search"
                :suffix-icon="Search"
      />
    </div>
  </section>

  <section>
    <el-table :data="products" border style="width: 100%">
      <el-table-column fixed type="index" label="No" width="90"/>
      <el-table-column prop="name" label="Name" width="120">
        <template #default="scope">
          <el-input v-model="scope.row.name" v-if="editingRows[scope.row.id]"/>
          <span v-else>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="description" label="Description" width="290">
        <template #default="scope">
          <el-input v-model="scope.row.description" v-if="editingRows[scope.row.id]"/>
          <span class="ellipsis" v-else>{{ scope.row.description }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="status" label="Status" width="120">
        <template #default="scope">
          <el-input v-model="scope.row.status" v-if="editingRows[scope.row.id]"/>
          <span v-else>{{ scope.row.status }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="price" label="Price" width="120">
        <template #default="scope">
          <el-input v-model="scope.row.price" v-if="editingRows[scope.row.id]"/>
          <span v-else>{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="product_code" label="Product Code" width="100">
        <template #default="scope">
          <el-input v-model="scope.row.product_code" v-if="editingRows[scope.row.id]"/>
          <span v-else>{{ scope.row.product_code }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="quantity" label="Quantity" width="100">
        <template #default="scope">
          <el-input v-model="scope.row.quantity" v-if="editingRows[scope.row.id]"/>
          <span v-else>{{ scope.row.quantity }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="createdBy" label="Saler" width="120">
        <template #default="scope">
          <el-input v-model="scope.row.createdBy" v-if="editingRows[scope.row.id]"/>
          <span v-else>{{ scope.row.createdBy }}</span>
        </template>
      </el-table-column>
      <el-table-column fixed="right" label="Operations" min-width="120">
        <template #default="scope">
          <el-button link :type="editingRows[scope.row.id] ? 'warning' :'primary'" size="small"
                     @click="Update(scope.row)">
            {{ editingRows[scope.row.id] ? 'Save' : 'Edit' }}
          </el-button>
          <el-button v-if="editingRows[scope.row.id]" link type="primary" size="small"
                     @click="closeFunction(scope.row.id)">
            <el-icon>
              <CircleCloseFilled/>
            </el-icon>
          </el-button>
          <el-button link type="danger" size="small" @click="confirmDelete(scope.row.id)">
            Delete
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog v-model="visible" title="Confirm Delete" width="500" draggable>
      <span>Are you sure you want to delete this product?</span>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="visible = false">
            Cancel
          </el-button>
          <el-button type="primary" @click="Delete">
            Delete
          </el-button>
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
          :page-sizes="[1,5,10]"
          :size="size"
          :background="true"
          layout="sizes, prev, pager, next"
          :total="page.totalElement"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
      />
    </div>
  </section>
</template>

<script lang="ts" setup>
import axios from 'axios';
import {onMounted, reactive, ref} from "vue";
import {CircleCloseFilled, Search} from '@element-plus/icons-vue';
import type {ComponentSize} from "element-plus";

onMounted(() => {
  getAllProduct()
})

interface EditingRows {
  [key: string]: boolean;
}

const URL_PRODUCT = "http://localhost:8080/product";
const products = ref([]);
const search = ref('');
const size = ref<ComponentSize>('default');
const visible = ref(false)
const editingRows = ref<EditingRows>({});
const deleteProductId = ref();
const page = reactive({
  currentPage: 1,
  pageSize: 10,
  totalElement: 0,
});


const getAllProduct = async () => {
  try {
    page.currentPage--;

    const params = new URLSearchParams({
      search: search.value,
      page: page.currentPage,
      size: page.pageSize,
    });

    const {data} = await axios.get(`${URL_PRODUCT}`, {params});

    products.value = data?.result?.content ?? [];
    page.totalElement = data?.result?.totalElements;

    const pageable = data?.result?.pageable;
    page.currentPage = pageable.pageNumber + 1;
    page.pageSize = pageable.pageSize;

    if (products.value.length === 0) {
      console.warn("No students found or response structure is incorrect.");
    }

  } catch (error) {
    console.error("Failed to fetch students:", error);
  }
};

const Update = async (product) => {
  try {
    if (editingRows.value[product.id]) {
      const body = {
        id: product.id,
        name: product.name,
        description: product.description,
        price: product.price,
        product_code: product.product_code,
        quantity: product.quantity,
        imageLink: product.imageLink,
        status: product.status,
      };


      const {data} = await axios.put(`${URL_PRODUCT}`, body);

      if (data) {
        await getAllProduct();
        editingRows.value[product.id] = false;
        return data;
      } else {
        console.warn("Error updating student or response structure is incorrect.");
      }
    } else {
      editingRows.value[product.id] = true;
    }
  } catch (error) {
    console.error("Error updating student:", error.message);
  }
};

const Delete = async () => {
  try {
    const {data} = await axios.delete(`${URL_PRODUCT}/${deleteProductId.value}`)

    if (data) {
      await getAllProduct();
      visible.value = false
      return data;
    } else {
      console.warn("Error deleting student or response structure is incorrect.");
    }
  } catch (error) {
    console.error("Error delete student:", error.message);
  }
}

const handleSizeChange = (val: number) => {
  page.pageSize = val;
  page.currentPage = 1;
  getAllProduct();
  console.log(`${val} items per page`)
}
const handleCurrentChange = (val: number) => {
  page.currentPage = val;
  getAllProduct();
  console.log(`current page: ${val}`)
}

const confirmDelete = (id) => {
  deleteProductId.value = id;  // Lưu ID của dòng cần xóa
  visible.value = true;    // Mở dialog xác nhận
}

const closeFunction = (id) => {
  editingRows.value[id] = false;
  getAllProduct();
}


/**
 * Chưa fix
 * */

interface LinkItem {
  value: string
  link: string
}

const state = ref('')
const links = ref<LinkItem[]>([])

const querySearch = (queryString: string, cb) => {
  const results = queryString
      ? links.value.filter(createFilter(queryString))
      : links.value
  // call callback function to return suggestion objects
  cb(results)
}
const createFilter = (queryString) => {
  return (restaurant) => {
    return (
        restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
    )
  }
}
const loadAll = () => {
  return [
    {value: 'vue', link: 'https://github.com/vuejs/vue'},
    {value: 'element', link: 'https://github.com/ElemeFE/element'},
    {value: 'cooking', link: 'https://github.com/ElemeFE/cooking'},
    {value: 'mint-ui', link: 'https://github.com/ElemeFE/mint-ui'},
    {value: 'vuex', link: 'https://github.com/vuejs/vuex'},
    {value: 'vue-router', link: 'https://github.com/vuejs/vue-router'},
    {value: 'babel', link: 'https://github.com/babel/babel'},
  ]
}
const handleSelect = (item: LinkItem) => {
  console.log(item)
}

const handleIconClick = (ev: Event) => {
  console.log(ev)
}

onMounted(() => {
  links.value = loadAll()
})

</script>

<style scoped>
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

.ellipsis {
  display: inline-block;
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>