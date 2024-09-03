<template>
  <div class="demo-image__placeholder">
    <el-container>
      <el-aside class="block" width="480">
        <el-alert v-if="continueUpdate" title="Successful" type="success" show-icon style="max-width: 480px;"/>
        <el-alert v-if="isCreated" title="Successful" type="success" show-icon style="max-width: 480px"/>
        <GetCard :is-update="toComponentShowCard"
                 :modified-by="category.modifiedBy"
                 :modified-date="category.modifiedDate"
                 :result="category" :imageLink="category.imageLink"
                 @visible-image="visibleImage"/>
      </el-aside>

      <el-main>

        <el-form ref="categoryRef" :model="category" :rules="rulesCategory"
                 label-width="auto"
                 style="max-width: 600px; margin-top: 30px">
          <el-form-item prop="name" label="Category name">
            <el-input v-model="category.name" clearable/>
          </el-form-item>
          <el-form-item v-if="isCreate" prop="category_code" label="Category Code">
            <el-input v-model="category.category_code"/>
          </el-form-item>
          <el-form-item label="Category state">
            <el-switch v-model="checkpointCategory"/>
          </el-form-item>
          <el-form-item label="Category description">
            <el-input v-model="category.description" type="textarea"/>
          </el-form-item>
          <el-form-item v-if="!isCreate">
            <el-button :disabled="continueUpdate" @click="validateFormUpdate" type="primary">Update</el-button>
            <el-button @click="$emit('closeUpdate')">Cancel</el-button>
          </el-form-item>
          <el-form-item v-if="isCreate">
            <el-button v-if="!isCreated" @click="validateFormCreate" type="primary">Create</el-button>

            <el-button @click="$emit('closeCreate')">Cancel</el-button>
          </el-form-item>
        </el-form>

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
import GetCard from "@/components/category/common/ShowCard.vue";
import type {FormInstance} from "element-plus";

const props = defineProps({
  idCategory: {
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
    await getDetailProduct(props.idCategory);
    category.status === 'UNAVAILABLE' ? checkpointCategory.value = false : checkpointCategory.value = true;
  }
})

const emits = defineEmits(['closeCreate', 'loadTable']);

const URL_CATEGORY = "http://localhost:8080/category";

const isVisibleImage = ref(false);
const inputLinkImage = ref('');
const isFillIn = ref(false);
const toComponentShowCard = ref(true);
const categoryRef = ref<FormInstance>();
const continueUpdate = ref(false);
const isCreated = ref(false)
const selectedCategoryIds = ref<number[]>([]);
const checkpointCategory = ref(false);

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
  modifiedBy: string;
  modifiedDate: string;
  categoryIds: number[],
  categories: Category[],
}

interface Category {
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
  modifiedBy: '',
  modifiedDate: '',
  categoryIds: selectedCategoryIds.value,
  categories: [],
})
const category = reactive<Category>({
  id: null,
  name: '',
  category_code: '',
  imageLink: inputLinkImage.value ? inputLinkImage.value : 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg',
  status: 'UNAVAILABLE',
  description: '',
  createdDate: '',
  createdBy: '',
  modifiedBy: '',
  modifiedDate: '',
})

const rulesCategory = {
  name: [
    {required: true, message: 'Please input category name', trigger: 'blur'}
  ],
  category_code: [
    {required: true, message: 'Please input category code', trigger: 'blur'}
  ],
}

// Update
const validateFormUpdate = () => {
  categoryRef.value?.validate(async (valid) => {
    if (valid) {
      await update();
      emits('loadTable')
    } else {
      console.warn('Validation failed!');
    }
  });
};

const getDetailProduct = async (id: number | any) => {
  try {
    const {data} = await axios.get(`${URL_CATEGORY}/${id}`);

    if (data?.result) {
      //Truyền vào card
      category.id = data?.result?.id;
      category.name = data?.result?.name;
      category.description = data?.result?.description;
      category.category_code = data?.result?.category_code;
      category.imageLink = data?.result?.imageLink;
      category.status = data?.result?.status;
      category.createdDate = formatDate(data?.result?.createdDate);
      category.createdBy = data?.result?.createdBy;
      category.modifiedBy = data?.result?.modifiedBy;
      category.modifiedDate = formatDate(data?.result?.modifiedDate);
    }
  } catch (error) {
    console.error("Failed to fetch students:", error);
  }

}

const update = async () => {
  try {
    continueUpdate.value = false
    const body = {
      id: category.id,
      name: category.name,
      category_code: category.category_code,
      imageLink: category.imageLink,
      status: category.status,
      description: category.description,
    }

    const {data} = await axios.put(URL_CATEGORY, body)

    if (data) {
      Object.assign(category, {
        name: category.name,
        imageLink: category.imageLink,
        state: category.status,
        desc: category.description,
        id: data?.result?.id,
        createdDate: formatDate(data?.result?.createdDate),
        createdBy: data?.result?.createdBy,
        modifiedBy: data?.result?.modifiedBy,
        modifiedDate: formatDate(data?.result?.modifiedDate),
      })

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
  categoryRef.value?.validate(async (valid) => {
    if (valid) {
      await create()
      emits('loadTable')
    } else {
      console.warn('Validation failed!')
    }
  })
}

const create = async () => {
  isCreated.value = false
  const body = {
    name: category.name,
    category_code: category.category_code,
    imageLink: category.imageLink,
    status: category.status,
    description: category.description,
  }

  const {data} = await axios.post(URL_CATEGORY, body)

  if (data) {
    Object.assign(category, {
      name: category.name,
      imageLink: category.imageLink,
      state: category.status,
      desc: category.description,
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
    category.imageLink = image;
  }
  isVisibleImage.value = false;
}

const formatDate = (date: any) => {
  return moment(date).format('DD.MM.YYYY HH:mm:ss');
}

watch(isFillIn, () => {
  if (!isFillIn.value) {
    category.name = '';
    category.category_code = '';
    category.imageLink = 'https://cdn.tgdd.vn/Files/2015/09/09/698241/zalo20.jpg';
    category.status = 'UNAVAILABLE';
    category.description = '';
  }
})
watch(checkpointCategory, (newValue) => {
  category.status = !newValue ? 'UNAVAILABLE' : 'AVAILABLE'
})
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