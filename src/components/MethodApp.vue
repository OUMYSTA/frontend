<template>
  <div>
    <div class="m-8 grid gap-y-2">
      <div>
        <p class="font-bold text-xl">ສາຂາຕົ້ນທາງ</p>
        <button
          class="btn rounded-md btn-outline border-yellow-400 w-full mr-2 mt-2"
          @click="openModal('start')"
        >
          {{ startLocationText }}
        </button>
        <dialog ref="startModal" class="modal">
          <div class="bg-white w-[20rem] h-[28rem] rounded-xl">
            <div class="modal_box pt-5">
              <SelectBox
                :initialLocation="'ນະຄອນຫຼວງວຽງຈັນ'"
                @locationChange="handleStartLocation"
                @subLocationChange="handleStartSubLocation"
              />
            </div>
            <div class="modal-action flex justify-center">
              <button @click="selectLocation('start')" class="btn bg-yellow-400 rounded-md ml-2">
                ເລືອກ
              </button>
            </div>
          </div>
        </dialog>
      </div>
      <div>
        <p class="font-bold text-xl">ສາຂາປາຍທາງ</p>
        <button
          class="btn rounded-md btn-ghost mr-2 btn-outline border-yellow-400 w-full mt-2"
          @click="openModal('end')"
        >
          {{ endLocationText }}
        </button>
        <dialog ref="endModal" class="modal">
          <div class="bg-white w-[20rem] h-[28rem] rounded-xl">
            <div class="modal_box pt-5">
              <SelectBox
                :initialLocation="'ນະຄອນຫຼວງວຽງຈັນ'"
                @locationChange="handleEndLocation"
                @subLocationChange="handleEndSubLocation"
              />
            </div>
            <div class="modal-action flex justify-center">
              <button @click="selectLocation('end')" class="btn bg-yellow-400 rounded-md ml-2">
                ເລືອກ
              </button>
            </div>
          </div>
        </dialog>
      </div>
      <div>
        <p class="font-bold text-xl">ສິນຄ້າຈັດສົ່ງ</p>
        <button
          class="btn rounded-md btn-ghost mr-2 btn-outline border-yellow-400 w-full mt-2"
          @click="openModal('item')"
        >
          {{ productTypeText }}
        </button>
        <dialog ref="itemModal" class="modal">
          <div class="bg-white w-[20rem] h-[28rem] rounded-xl">
            <div class="modal_box pt-5">
              <SelectProduct
                :initialProduct="'ສັດ'"
                @productChange="handleProductType"
                @subItemChange="handleSubItemChange"
              />
            </div>
            <div class="modal-action flex justify-center">
              <button @click="selectProduct" class="btn bg-yellow-400 rounded-md ml-2">
                ເລືອກ
              </button>
            </div>
          </div>
        </dialog>
      </div>
      <div>
        <p class="font-bold text-xl">ຄິດໄລ່ຂະໜາດ</p>
        <CalSize />
      </div>
    </div>

    <div class="flex text-center mb-10 items-center justify-around">
      <button @click="submit" class="btn bg-[#FFC107] rounded-md text-xl border-0">ຄິດໄລ່</button>
      <div v-if="shippingCost !== null">ຄ່າຂົນສົ່ງ: {{ shippingCost }} ກີບ</div>
      <div v-if="errorMessage">
        {{ errorMessage }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import SelectBox from './SelectBox.vue'
import SelectProduct from './SelectProduct.vue'
import CalSize from './CalSize.vue'

const endLocation = ref<string>('ນະຄອນຫຼວງວຽງຈັນ')
const endLocationText = ref<string>('ເລືອກສາຂາປາຍທາງ')
const endSubLocation = ref<string>('')
const productType = ref<string>('ສັດ')
const productTypeText = ref<string>('ເລືອກສິນຄ້າ')
const subItem = ref<string>('')
const shippingCost = ref<number | null>(null)
const errorMessage = ref<string>('')
const startLocation = ref<string>('ນະຄອນຫຼວງວຽງຈັນ')
const startSubLocation = ref<string>('')
const startLocationText = ref<string>('ເລືອກສາຂາຕົ້ນທາງ')
const startModal = ref<HTMLDialogElement | null>(null)
const endModal = ref<HTMLDialogElement | null>(null)
const itemModal = ref<HTMLDialogElement | null>(null)

const openModal = (modalType: 'start' | 'end' | 'item') => {
  if (modalType === 'start') {
    startModal.value?.showModal()
  } else if (modalType === 'end') {
    endModal.value?.showModal()
  } else {
    itemModal.value?.showModal()
  }
}

const closeModal = () => {
  startModal.value?.close()
  endModal.value?.close()
  itemModal.value?.close()
}

const selectLocation = (modalType: 'start' | 'end') => {
  if (modalType === 'start' && startSubLocation.value) {
    startLocationText.value = startSubLocation.value
  } else if (modalType === 'end' && endSubLocation.value) {
    endLocationText.value = endSubLocation.value
  }
  closeModal()
}

const selectProduct = () => {
  productTypeText.value = subItem.value
  closeModal()
}

const handleStartLocation = (value: string) => {
  startLocation.value = value
}

const handleStartSubLocation = (value: string) => {
  startSubLocation.value = value
}

const handleEndLocation = (value: string) => {
  endLocation.value = value
}

const handleEndSubLocation = (value: string) => {
  endSubLocation.value = value
}

const handleProductType = (value: string) => {
  productType.value = value
}

const handleSubItemChange = (value: string) => {
  subItem.value = value
}


const submit = async () => {
  if (startLocation.value && endSubLocation.value && productType.value && subItem.value) {
    try {
      const response = await fetch(`https://backend-app-price.onrender.com/check-product`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          start: startLocation.value,
          endSub: endSubLocation.value,
          productType: productType.value,
          subItem: subItem.value
        })
      })

      const result = await response.json()
      if (response.ok) {
        shippingCost.value = result.price || 0
        errorMessage.value = ''
      } else {
        shippingCost.value = null
        errorMessage.value = result.error || 'An error occurred'
      }
    } catch (error) {
      errorMessage.value = 'Unable to retrieve shipping cost. Please try again.'
    }
  } else {
    errorMessage.value = 'Please fill out all fields before submitting.'
  }
}
</script>
