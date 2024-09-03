<template>
  <div style="margin-bottom: 100px">
    <section>
      <div class="flex-container">
        <el-select
            v-model="selectedOptions"
            multiple
            collapse-tags
            collapse-tags-tooltip
            placeholder="Filter"
            style="width: 160px"
            @change="handleSelectChange"
            clearable
        >
          <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
          />
        </el-select>
        <el-input
            v-model="searchName"
            style="width: 30%"
            placeholder="Search"
            clearable
        />
        <el-input
            v-if="viewFilter.categoryCode"
            v-model="categoryCode"
            style="width: 30%;"
            placeholder="Search Category Code"
            clearable
        />
        <el-date-picker
            v-if="viewFilter.startDate"
            v-model="startDate"
            type="date"
            placeholder="Pick a date"
            value-format="YYYY-MM-DD"
            style="width: 30%"
        />
        <el-date-picker
            v-if="viewFilter.endDate"
            v-model="endDate"
            type="date"
            placeholder="Pick a date"
            value-format="YYYY-MM-DD"
            style="width: 30%"
        />
        <el-button @click="getAllCategory()">
          <el-icon>
            <Search :suffix-icon="Search"/>
          </el-icon>
        </el-button>

      </div>
      <div class="create-button">
        <el-button @click="downloadExcel" type="primary" :icon="Download" size="default">
          Download Excel
        </el-button>
        <el-button @click="$emit('openCreateCategory')" type="primary" :icon="Upload" size="default">
          Create Category
        </el-button>
      </div>
    </section>


    <section>
      <el-table :data="category" border>
        <el-table-column fixed type="index" label="No" width="90"/>
        <el-table-column prop="name" label="Name" width="120"/>
        <el-table-column prop="description" label="Description" width="390">
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
        <el-table-column prop="category_code" label="Category Code" width="100"/>
        <el-table-column prop="createdBy" label="Saler" width="120"/>
        <el-table-column fixed="right" label="Operations" min-width="80px">
          <template #default="scope">
            <el-button
                @click="$emit('viewDetailCategory', scope.row.name, scope.row.id)"
                type="primary"
                size="small"
                circle
            >
              <el-icon>
                <Document/>
              </el-icon>
            </el-button>
            <el-button type="warning" @click="$emit('updateCategory', scope.row.id)" size="small" circle>
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
        <span>Are you sure you want to delete this category?</span>
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
import {DeleteFilled, Document, Download, Edit, Search, Upload} from '@element-plus/icons-vue'
import type {ComponentSize} from 'element-plus'

onMounted(() => {
  getAllCategory()
})

const emit = defineEmits(['updateCategory', 'openCreateCategory', 'viewDetailCategory'])
const props = defineProps({
  loadTable: Boolean
})

const URL_PRODUCT = 'http://localhost:8080/category'
const URL_EXCEL = 'http://localhost:8080/api/excel/categories/download'

const category = ref([])
const searchName = ref('')
const size = ref<ComponentSize>('default')
const visible = ref(false)
const deleteCategoryId = ref()
const selectedOptions = ref([]);

const categoryCode = ref('');
const startDate = ref('')
const endDate = ref('')


const viewFilter = reactive({
  categoryCode: false,
  startDate: false,
  endDate: false,
});

const options = [
  {value: 'categoryCode', label: 'Category Code'},
  {value: 'startDate', label: 'Start Date'},
  {value: 'endDate', label: 'End Date'},
];

const handleSelectChange = (selected: any) => {
  viewFilter.categoryCode = selected.includes('categoryCode');
  viewFilter.startDate = selected.includes('startDate');
  viewFilter.endDate = selected.includes('endDate');

  if (viewFilter.categoryCode === false) {
    categoryCode.value = '';
  }
  if (viewFilter.startDate === false) {
    startDate.value = '';
  }
  if (viewFilter.endDate === false) {
    endDate.value = '';
  }

};

const page = reactive({
  currentPage: 1,
  pageSize: 10,
  totalElement: 0
})
const getAllCategory = async () => {
  try {
    // Giảm currentPage trước khi lấy dữ liệu
    page.currentPage = Math.max(page.currentPage, 1); // Đảm bảo currentPage không dưới 1

    const params = new URLSearchParams({
      name: searchName.value,
      categoryCode: categoryCode.value,
      startDate: startDate.value ? startDate.value : '',
      endDate: endDate.value ? endDate.value : '',
      page: String(page.currentPage - 1),
      size: String(page.pageSize)
    })

    const {data} = await axios.get(`${URL_PRODUCT}`, {params})

    category.value = data?.result?.content ?? []
    page.totalElement = data?.result?.totalElements

    const pageable = data?.result?.pageable
    page.currentPage = pageable.pageNumber + 1
    page.pageSize = pageable.pageSize

    if (category.value.length === 0) {
      console.warn('No students found or response structure is incorrect.')
    }
  } catch (error) {
    console.error('Failed to fetch students:', error)
  }
}

const Delete = async () => {
  try {
    const {data} = await axios.delete(`${URL_PRODUCT}/${deleteCategoryId.value}`)

    if (data) {
      await getAllCategory()
      visible.value = false
      return data
    } else {
      console.warn('Error deleting student or response structure is incorrect.')
    }
  } catch (error) {
    console.error('Error delete student:', error.message)
  }
}


const downloadExcel = async () => {
  try {
    const response = await axios.get(URL_EXCEL, {
      responseType: 'blob', // Đảm bảo rằng phản hồi được xử lý như một blob
    });

    // Tạo URL từ blob và tạo một link để tải file
    const url = window.URL.createObjectURL(new Blob([response.data]));
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', 'category.xlsx'); // Tên file bạn muốn tải về
    document.body.appendChild(link);
    link.click();

    // Sau khi tải xong thì hủy bỏ link và giải phóng bộ nhớ
    link.parentNode.removeChild(link);
    window.URL.revokeObjectURL(url);
  } catch (error) {
    console.error('Failed to download Excel file:', error);
  }
};

const handleSizeChange = (val: number) => {
  page.pageSize = val
  page.currentPage = 1
  getAllCategory()
  console.log(`${val} items per page`)
}
const handleCurrentChange = (val: number) => {
  page.currentPage = val
  getAllCategory()
  console.log(`current page: ${val}`)
}

const confirmDelete = (id: number) => {
  deleteCategoryId.value = id // Lưu ID của dòng cần xóa
  visible.value = true // Mở dialog xác nhận
}

watch(
    () => props.loadTable,
    () => {
      getAllCategory()
    }
)
</script>

<style scoped>
.flex-container {
  margin-top: 30px;
  margin-bottom: 40px;

}

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
