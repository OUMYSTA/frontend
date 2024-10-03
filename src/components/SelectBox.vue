<template>
  <div class="w-full carousel carousel-vertical h-[20rem] overflow-y-auto">
    <div class="p-10 carousel-item flex-col">
      <select
        id="locationSelect"
        class="select select-bordered"
        v-model="selectedLocation"
        @change="onLocationChange"
      >
        <option disabled value="">ເລືອກແຂວງ</option>
        <option v-for="location in locationsList" :key="location" :value="location">
          {{ location }}
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

      <div v-if="errorMessage" class="text-red-500 mt-2">
        {{ errorMessage }}
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps<{
  initialLocation?: string
}>()

const emit = defineEmits(['locationChange', 'subLocationChange'])

const selectedLocation = ref<string>(props.initialLocation || '')
const selectedSubItem = ref<string>('')
const subData = ref<string[]>([])
const locationsList = ref<string[]>([])
const errorMessage = ref<string | null>(null)

const onLocationChange = async () => {
  emit('locationChange', selectedLocation.value)
  await fetchSubData()
}

const selectSubItem = (item: string) => {
  selectedSubItem.value = item
  emit('subLocationChange', item)
}

const fetchLocations = async () => {
  try {
    const response = await fetch('api/location_names')
    if (!response.ok) throw new Error('Failed to fetch locations')
    locationsList.value = await response.json()
  } catch (error) {
    errorMessage.value = 'Could not load locations. Please try again.'
  }
}

const fetchSubData = async () => {
  subData.value = []
  selectedSubItem.value = ''
  errorMessage.value = null

  if (selectedLocation.value) {
    try {
      const response = await fetch(`api/location_data/${selectedLocation.value}`)
      if (!response.ok) throw new Error('Failed to fetch sub-data')
      const data = await response.json()
      if (Array.isArray(data)) {
        subData.value = data
      } else {
        throw new Error('Unexpected data format')
      }
    } catch (error) {
      errorMessage.value = 'Could not load sub-data. Please try again.'
    }
  }
}

onMounted(() => {
  fetchLocations()
  if (props.initialLocation) {
    fetchSubData()
  }
})

watch(
  () => props.initialLocation,
  (newValue) => {
    if (newValue) {
      selectedLocation.value = newValue
      fetchSubData()
    }
  }
)
</script>
