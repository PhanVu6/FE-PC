<template>
  <div class="demo-image__placeholder">
    <el-container>
      <el-aside class="block" width="480">
        <el-alert v-if="continueUpdate" title="Successful" type="success" show-icon style="max-width: 480px;"/>
        <el-alert v-if="isCreated" title="Successful" type="success" show-icon style="max-width: 480px"/>
        <GetCard :is-update="toComponentShowCard"
                 :modified-by="form.modifiedBy"
                 :modified-date="form.modifiedDate"
                 :result="form" :imageLink="form.imageLink"
                 @visible-image="visibleImage"/>
      </el-aside>

      <el-main>
        <el-tabs v-model="activeName" class="demo-tabs" @tab-click="handleClick">
          <el-tab-pane label="Product" name="first">
            <!--Product-->
            <el-form ref="formRef" :model="form" :rules="rules" label-width="auto"
                     style="max-width: 600px; margin-top: 30px">
              <el-form-item prop="name" label="Product name">
                <el-input v-model="form.name" clearable/>
              </el-form-item>
              <el-form-item prop="price" label="Price">
                <el-input v-model.number="form.price" clearable/>
              </el-form-item>
              <el-form-item v-if="isCreate" prop="product_code" label="Product Code">
                <el-input v-model="form.product_code"/>
              </el-form-item>
              <el-form-item prop="quantity" label="Quantity">
                <el-input v-model.number="form.quantity" clearable/>
              </el-form-item>
              <el-form-item label="Categories">
                <el-select
                    v-model="selectedCategoryIds"
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
                      :key="c.category_code"
                      :label="c.name"
                      :value="c.id"
                  />
                </el-select>
              </el-form-item>
              <el-form-item label="Product state">
                <el-switch v-model="checkpointProduct" :disabled="!(form.quantity > 0)"/>
              </el-form-item>
              <el-form-item label="Product description">
                <el-input v-model="form.desc" type="textarea" clearable/>
              </el-form-item>
              <el-form-item v-if="!isCreate">
                <el-button :disabled="continueUpdate" @click="validateFormUpdate()" type="primary">Update
                </el-button>
                <el-button @click="$emit('closeUpdate')">Cancel</el-button>
              </el-form-item>
              <el-form-item v-if="isCreate">
                <el-button v-if="!isCreated" @click="validateFormCreate" type="primary">Create</el-button>
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
                <el-switch v-model="checkpointCategory"/>
              </el-form-item>
              <el-form-item label="Category description">
                <el-input v-model="category.description" type="textarea"/>
              </el-form-item>
              <el-form-item v-if="!isCreate">
                <el-button :disabled="continueUpdate" @click="validateFormUpdate" type="primary">Update</el-button>
                <el-button :disabled="continueUpdate" @click="validateCategory" type="primary">Add Category</el-button>
                <el-button @click="$emit('closeUpdate')">Cancel</el-button>
              </el-form-item>
              <el-form-item v-if="isCreate">
                <el-button v-if="!isCreated" @click="validateFormCreate" type="primary">Create</el-button>
                <el-button v-if="!isCreated" @click="validateCategory" type="primary"
                >Add Category
                </el-button
                >
                <el-button @click="$emit('closeCreate')">Cancel</el-button>
              </el-form-item>
            </el-form>
          </el-tab-pane>
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
import GetCard from "@/components/product/common/ShowCard.vue";
import type {FormInstance, FormRules, TabsPaneContext} from "element-plus";
import {ElNotification} from "element-plus";

const props = defineProps({
  idProduct: {
    type: Number,
  },
  isCreate: {
    type: Boolean,
    default: true,
  },
});

onMounted(async () => {
  isCreated.value = false
  isFillIn.value = false;
  if (!props.isCreate) {
    await getDetailProduct(props.idProduct);
    form.state === 'UNAVAILABLE' ? checkpointProduct.value = false : checkpointProduct.value = true;
  }
  await getAllCategory();
})

const activeName = ref('first');
const emits = defineEmits(['closeCreate', 'loadTable']);

const URL_PRODUCT = "http://localhost:8080/product";
const URL_CATEGORY = "http://localhost:8080/category";

const isVisibleImage = ref(false);
const inputLinkImage = ref('');
const inputLinkImageCategory = ref('');
const isFillIn = ref(false);
const toComponentShowCard = ref(true);
const formRef = ref<FormInstance>();
const categoryRef = ref<FormInstance>();
const continueUpdate = ref(false);
const isCreated = ref(false)
const selectedCategoryIds = ref<number[]>([]);
const checkpointProduct = ref(false);
const checkpointCategory = ref(false);
const currentCategories = ref<Category[]>([]);


interface FormInp {
  id: number | null;
  name: string;
  price: number;
  product_code: string;
  quantity: number;
  imageLink: string;
  state: 'UNAVAILABLE' | 'AVAILABLE';
  desc: string;
  createdDate: string;
  createdBy: string;
  modifiedBy: string;
  modifiedDate: string;
  categoryIds: number[],
  categories: Category[],
  imageProducts: ImageProduct[],
}

interface ImageProduct {
  "id": number,
  "imageName": string,
  "imagePath": string
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
  price: 0,
  product_code: '',
  quantity: 0,
  imageLink: 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg',
  state: "UNAVAILABLE",
  desc: '',
  createdDate: '',
  createdBy: '',
  modifiedBy: '',
  modifiedDate: '',
  categoryIds: selectedCategoryIds.value,
  categories: [],
  imageProducts: [],
})
const category = reactive<Category>({
  id: null,
  name: '',
  category_code: '',
  imageLink: inputLinkImageCategory.value ? inputLinkImage.value : 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg',
  status: 'UNAVAILABLE',
  description: '',
})
const rules = reactive<FormRules<FormInp>>({
  name: [
    {required: true, message: 'Please input product name', trigger: 'blur'}
  ],
  price: [
    {required: true, message: 'Please input price', trigger: 'blur'},
    {type: "number", message: 'Price must be a number', trigger: ['blur', 'change']}
  ],
  product_code: [
    {required: true, message: 'Please input product code', trigger: 'blur'}
  ],
  quantity: [
    {required: true, message: 'Please input quantity', trigger: 'blur'},
    {type: "number", message: 'Price must be a number', trigger: ['blur', 'change']}
  ]
});
const rulesCategory = reactive<FormRules<Category>>({
  name: [
    {required: true, message: 'Please input category name', trigger: 'blur'}
  ],
  category_code: [
    {required: true, message: 'Please input category code', trigger: 'blur'}
  ],
});

// Update
const validateFormUpdate = () => {
  formRef.value?.validate(async (valid) => {
    if (valid) {
      if (selectedCategoryIds.value.length > 0 || form.categories.length > 0) {
        await update();
        emits('loadTable')
      } else {
        ElNotification({
          title: 'Error',
          message: 'You must link or create a Category.',
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
const getDetailProduct = async (id: number | any) => {
  try {
    const {data} = await axios.get(`${URL_PRODUCT}/${id}`);

    if (data?.result) {
      //Truyền vào card
      form.id = data?.result?.id;
      form.name = data?.result?.name;
      form.desc = data?.result?.description;
      form.price = data?.result?.price;
      form.product_code = data?.result?.product_code;
      form.quantity = data?.result?.quantity;
      form.imageLink = data?.result?.imageLink;
      form.state = data?.result?.status;
      form.createdDate = formatDate(data?.result?.createdDate);
      form.createdBy = data?.result?.createdBy;
      form.modifiedBy = data?.result?.modifiedBy;
      form.modifiedDate = formatDate(data?.result?.modifiedDate);
      form.categoryIds = data.result.categories?.map((c: any) => c?.id) ?? [];
      // Thêm id sẽ tự động thay đổi tới form (nhập vào)
      selectedCategoryIds.value = form.categoryIds;
    }
  } catch (error) {
    console.error("Failed to fetch students:", error);
  }

}

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

const update = async () => {
  try {
    continueUpdate.value = false
    const body = {
      id: form.id,
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

    const {data} = await axios.put(URL_PRODUCT, body)

    if (data) {
      Object.assign(form, {
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
        modifiedBy: data?.result?.modifiedBy,
        modifiedDate: formatDate(data?.result?.modifiedDate),
        categories: data?.result.categories,
      })

      // Kiểm tra nếu được tạo mới thì sẽ push và trong list currentCategories, để gọi lên các category được liên kết
      form.categories.map(c => {
        if (!currentCategories.value.find(element => element.id === c.id)) {
          currentCategories.value.push(c)
        }
      })

      // Thêm id sẽ tự động thay đổi tới form (nhập vào)
      selectedCategoryIds.value = form.categories?.map((c: any) => c?.id) ?? [];

      continueUpdate.value = true;
      setTimeout(() => {
        continueUpdate.value = false;
      }, 2000)
    } else {
      console.warn('Error creating product or response structure is incorrect.')
    }
  } catch (error) {
    console.log(error.message);
  }
}

// Create
const validateFormCreate = () => {
  formRef.value?.validate(async (valid) => {
    if (valid) {
      if (selectedCategoryIds.value.length > 0 || form.categories.length > 0) {
        await create()
        emits('loadTable')
      } else {
        ElNotification({
          title: 'Error',
          message: 'You must link or create a Category',
          type: 'error'
        })
      }
    } else {
      console.warn('Validation failed!')
    }
  })
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
    categories: form.categories
  }

  const {data} = await axios.post(URL_PRODUCT, body)

  if (data) {
    Object.assign(form, {
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
      categories: data?.result.categories
    })
    // Kiểm tra nếu được tạo mới thì sẽ push và trong list currentCategories, để gọi lên các category được liên kết
    form.categories.map((c) => {
      if (!currentCategories.value.find((element) => element.id === c.id)) {
        currentCategories.value.push(c)
      }
    })

    // Thêm id sẽ tự động thay đổi tới form (nhập vào)
    selectedCategoryIds.value = form.categories?.map((c: any) => c?.id) ?? []

    isCreated.value = true
  } else {
    console.warn('Error creating product or response structure is incorrect.')
  }
}


const handleClick = (tab: TabsPaneContext, event: Event) => {
  console.log(tab, event)
}

const addCategory = () => {
  // Giả sử `id` được tạo mới cho danh mục
  const newCategory = {
    ...category,
    id: generateNewId(), // Hàm này sẽ tạo `id` mới
  };

  form.categories.push(newCategory);
  currentCategories.value.push(newCategory);

  // Cập nhật selectedCategoryIds
  selectedCategoryIds.value.push(newCategory.id);

  // Xóa dữ liệu form nhập danh mục sau khi thêm vào
  category.name = '';
  category.category_code = '';
  inputLinkImageCategory.value = '';
  category.status = 'UNAVAILABLE';
  category.description = '';
};

// Hàm giả lập tạo ID mới
const generateNewId = () => {
  return Date.now(); // Ví dụ tạo ID bằng thời gian hiện tại
};
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

watch(isFillIn, () => {
  if (!isFillIn.value) {
    form.name = '';
    form.price = 0;
    form.product_code = '';
    form.quantity = 0;
    form.imageLink = 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg';
    form.state = 'UNAVAILABLE';
    form.desc = '';
  }
})
watch(checkpointProduct, (newValue) => {
  form.state = !newValue ? 'UNAVAILABLE' : 'AVAILABLE'
})
watch(checkpointCategory, (newValue) => {
  category.status = !newValue ? 'UNAVAILABLE' : 'AVAILABLE'
})
watch(selectedCategoryIds, (newValue) => {
  form.categoryIds = newValue;
});
</script>


<style scoped>
.demo-image__placeholder .block {
  position: relative;
  padding: 20px 0;
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