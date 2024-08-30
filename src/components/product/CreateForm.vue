<template>
  <div class="demo-image__placeholder">
    <el-container>
      <el-aside class="block" width="480">
        <el-alert v-if="isCreated" title="Successful" type="success" show-icon style="max-width: 480px;"/>
        <GetCard :create-completed="createCompleted" :imageLink="form.imageLink" @visible-image="visibleImage"/>
      </el-aside>

      <el-main>
        <el-tabs v-model="activeName" class="demo-tabs" @tab-click="handleClick">
          <!--Product-->
          <el-tab-pane label="Product" name="first">
            <el-form ref="formRef" :model="form" :rules="rules" label-width="auto"
                     style="max-width: 600px; margin-top: 30px">
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
              <el-form-item label="Categories">
                <el-select
                    v-model="idsCategory"
                    multiple
                    collapse-tags
                    collapse-tags-tooltip
                    :max-collapse-tags="3"
                    placeholder="Select"
                    style="max-width: 600px"
                    clearable
                >
                  <el-option
                      v-for="c in currentCategories"
                      :key="c.id"
                      :label="c.name"
                      :value="c.id"
                  />
                </el-select>
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
          </el-tab-pane>

          <el-tab-pane label="Create Category" name="second">
            <!--Category-->
            <el-form ref="categoryRef" :model="category" :rules="rulesCategory"
                     label-width="auto"
                     style="max-width: 600px; margin-top: 30px">
              <el-form-item prop="name" label="Category name">
                <el-input v-model="category.name" clearable/>
              </el-form-item>
              <el-form-item prop="category_code" label="Category Code">
                <el-input v-model="category.category_code"/>
              </el-form-item>
              <el-form-item prop="imageLink" label="Create Image">
                <el-input v-model="inputLinkImageCategory"/>
              </el-form-item>
              <el-form-item label="Category state">
                <el-select v-model="category.status">
                  <el-option label="AVAILABLE" value="AVAILABLE"/>
                  <el-option label="UNAVAILABLE" value="UNAVAILABLE"/>
                </el-select>
              </el-form-item>
              <el-form-item label="Category description">
                <el-input v-model="category.description" type="textarea"/>
              </el-form-item>
              <el-form-item>
                <el-button v-if="!isCreated" @click="validateForm" type="primary">Create</el-button>
                <el-button v-if="!isCreated" @click="validateCategory" type="primary">Add Category</el-button>
                <el-button @click="$emit('closeCreate')">Cancel</el-button>
              </el-form-item>
            </el-form>
          </el-tab-pane>

          <div class="box-create-category">
            <div class="category-signup-box">
              <h1>New Categories:</h1>
              <p v-for="c in form.categories" :key="c.category_code">
                <span> Name Category: {{ c.name }}</span>
                <br> Image: {{ c.imageLink }}
                <br> Category Code: {{ c.category_code }}
                <br> -----------------------------------------
              </p>
            </div>
          </div>
        </el-tabs>
      </el-main>

    </el-container>

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
import GetCard from "@/components/product/ShowCard.vue";
import type {FormInstance, TabsPaneContext} from "element-plus";
import {ElNotification} from "element-plus";

onMounted(() => {
  isCreated.value = false;
  getAllCategory();
})

const activeName = ref('first');
const emits = defineEmits(['closeCreate', 'loadTable'])

const URL_PRODUCT = "http://localhost:8080/product";
const URL_CATEGORY = "http://localhost:8080/category";

const isVisibleImage = ref(false);
const inputLinkImage = ref('');
const inputLinkImageCategory = ref('');
const isCreated = ref(false);
const isShowProduct = ref(true);
const isShowCategory = ref(false);
const formRef = ref<FormInstance>();
const categoryRef = ref<FormInstance>();

const idsCategory = ref<number[]>([]);
const currentCategories = ref<Category[]>([]);

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
  categoryIds: number[],
  categories: Category[],
}

interface Category {
  id: number | null,
  name: string,
  category_code: string,
  imageLink: string,
  status: 'UNAVAILABLE' | 'AVAILABLE',
  description: string,
}

const form = reactive<FormInp>({
  id: null,
  name: '',
  price: null,
  product_code: '',
  quantity: null,
  imageLink: 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg',
  state: "UNAVAILABLE",
  desc: '',
  createdDate: '',
  createdBy: '',
  categoryIds: idsCategory.value,
  categories: [],
})

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
  categoryIds: [],
  categories: [],
})

const category = reactive<Category>({
  id: null,
  name: '',
  category_code: '',
  imageLink: inputLinkImageCategory.value ? inputLinkImage.value : 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg',
  status: 'UNAVAILABLE',
  description: '',
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
  ],
  nameCategory: [
    {required: true, message: 'Please input product name', trigger: 'blur'}
  ],
  category_code: [
    {required: true, message: 'Please input category code', trigger: 'blur'}
  ],
}

const rulesCategory = {
  name: [
    {required: true, message: 'Please input category name', trigger: 'blur'}
  ],
  category_code: [
    {required: true, message: 'Please input category code', trigger: 'blur'}
  ],
}

const validateForm = () => {
  formRef.value?.validate(async (valid) => {
    if (valid) {
      if (idsCategory.value.length > 0 || form.categories.length > 0) {
        await create();
        emits('loadTable')
      } else {
        ElNotification({
          title: 'Error',
          message: 'You must link or create a Category',
          type: 'error',
        })
      }

    } else {
      console.warn('Validation failed!');
    }
  });
};

const validateCategory = () => {
  categoryRef.value?.validate((valid) => {
    if (valid) {
      addCategory();
    } else {
      console.warn('Validation failed!');
    }
  });
};

const getAllCategory = async () => {
  try {
    const {data} = await axios.get(`${URL_CATEGORY}/open`)

    if (data) {
      currentCategories.value = data?.result;
    } else {
      console.warn('Error getting product or response structure is incorrect.')
    }
  } catch (error) {
    console.error("Failed to fetch students:", error);
  }
}

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
    categoryIds: form.categoryIds,
    categories: form.categories,
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
      categories: data?.result.categories,
    })
    // Kiểm tra nếu được tạo mới thì sẽ push và trong list currentCategories, để gọi lên các category được liên kết
    createCompleted.categories.map(c => {
      if (!currentCategories.value.find(element => element.id === c.id)) {
        currentCategories.value.push(c)
      }
    })

    // Thêm id sẽ tự động thay đổi tới form (nhập vào)
    idsCategory.value = createCompleted.categories?.map((c: any) => c?.id) ?? [];

    isCreated.value = true
  } else {
    console.warn('Error creating product or response structure is incorrect.')
  }
}

const handleClick = (tab: TabsPaneContext, event: Event) => {
  console.log(tab, event)
}

const addCategory = () => {
  form.categories.push({...category});
  category.name = '';
  category.category_code = '';
  inputLinkImageCategory.value = '';
  category.status = 'UNAVAILABLE';
  category.description = '';
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

const showFormProduct = () => {
  isShowProduct.value = true;
  isShowCategory.value = false;
}

const showFormCategory = () => {
  isShowProduct.value = false;
  isShowCategory.value = true;
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
    form.categoryIds = [];
    form.categories = [];
  }
})
watch(idsCategory, (newValue) => {
  form.categoryIds = newValue;
});
</script>


<style scoped>
.demo-image__placeholder .block {
  position: relative;
  padding: 30px 0;
  text-align: center;
  display: inline-block;
  width: 44%;
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

.navbar {
  --el-header-padding: 0 3px;
  --el-header-height: auto;
  background-color: #3295cc;
  border-bottom: 1px solid #938d8d;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-sizing: border-box;
  flex-shrink: 0;
  height: var(--el-header-height);
  padding: var(--el-header-padding);
  padding-left: 0;
}

.navbar-menu {
  list-style: none;
  display: flex;
  gap: 20px;
}

.navbar-menu li {
  color: #fff;
  padding: 0px 10px;
  text-decoration: none;
  font-size: 1.2em;
  transition: color 0.3s;
}

.navbar-menu li:hover {
  color: #f0a500;
  cursor: pointer;
}

.box-create-category {
  display: flex;
  justify-content: flex-end;
}

.category-signup-box {
  width: 50%;
  height: 78px;
  overflow: auto;
  border: 3px solid antiquewhite;
  padding: 4px;
  background: white;
}
</style>