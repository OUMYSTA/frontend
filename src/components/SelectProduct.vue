<template>
  <div class="w-full carousel carousel-vertical h-[20rem] overflow-y-auto">
    <div class="p-10 carousel-item flex-col">
      <select
        id="productSelect"
        class="select select-bordered"
        v-model="selectedProduct"
        @change="onProductChange"
      >
        <option disabled value="">ເລືອກປະເພດ</option>
        <option v-for="(product, idx) in productsList" :key="idx" :value="product">
          {{ product }}
        </option>
      </select>
      <div v-if="subData.length > 0" class="mt-4">
        <ul class="menu bg-base-200 w-56 rounded-box">
          <li v-for="(item, idx) in subData" :key="idx">
            <a @click="selectSubItem(item)" :class="{ active: item === selectedSubItem }">{{
              item
            }}</a>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from 'vue'
import { defineEmits, defineProps } from 'vue'

const props = defineProps<{
  initialProduct?: string
}>()

const emit = defineEmits(['productChange', 'subItemChange'])

const selectedProduct = ref<string>(props.initialProduct || '')
const selectedSubItem = ref<string>('')
const subData = ref<string[]>([])
const productsList = ref<string[]>([])

const loadingSubData = ref<boolean>(false)
const errorFetchingSubData = ref<string | null>(null)

const selectSubItem = (item: string) => {
  selectedSubItem.value = item
  emit('subItemChange', item)
}

const fetchProducts = async () => {
  try {
    const response = await fetch(`https://backend-app-price.onrender.com/product_names`)
    if (!response.ok) {
      throw new Error('Failed to fetch Products')
    }
    productsList.value = await response.json()
  } catch (error) {
    errorFetchingSubData.value = 'Could not load products. Please try again.'
  }
}

const onProductChange = async () => {
  emit('productChange', selectedProduct.value)
  await fetchSubData()
}

const fetchSubData = async () => {
  subData.value = []
  selectedSubItem.value = ''
  loadingSubData.value = true
  errorFetchingSubData.value = null

  if (selectedProduct.value) {
    try {
      const response = await fetch(`https://backend-app-price.onrender.com/product_data/${selectedProduct.value}`)
      if (!response.ok) {
        throw new Error('Failed to fetch sub-data')
      }
      const data = await response.json()

      if (Array.isArray(data)) {
        subData.value = data
      } else {
        errorFetchingSubData.value = 'Unexpected data format received.'
      }
    } catch (error) {
      errorFetchingSubData.value = 'Could not load sub-items. Please try again.'
    } finally {
      loadingSubData.value = false
    }
  } else {
    loadingSubData.value = false
  }
}

onMounted(() => {
  fetchProducts()
  if (props.initialProduct) {
    fetchSubData()
  }
})

watch(
  () => props.initialProduct,
  (newValue) => {
    if (newValue) {
      selectedProduct.value = newValue
      fetchSubData()
    }
  }
)
</script>
