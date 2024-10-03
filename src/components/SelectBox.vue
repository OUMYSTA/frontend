<template>
  <div class="w-full carousel carousel-vertical h-[20rem] overflow-y-auto">
    <div class="p-10 carousel-item flex-col">
      <!-- Location Dropdown -->
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

      <!-- Sub-Data List -->
      <div v-if="subData.length > 0" class="mt-4">
        <ul class="menu bg-base-200 w-56 rounded-box">
          <li v-for="(item, idx) in subData" :key="idx">
            <a @click="selectSubItem(item)" :class="{ active: item === selectedSubItem }">{{ item }}</a>
          </li>
        </ul>
      </div>

      <!-- No Sub-Data -->
      <div v-if="subData.length === 0 && !errorMessage && selectedLocation" class="mt-2">
        No sub-data available for the selected location.
      </div>

      <!-- Error Message -->
      <div v-if="errorMessage" class="text-red-500 mt-2">
        {{ errorMessage }}
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from 'vue'

// Define Props
const props = defineProps<{
  initialLocation?: string
}>()

// Define Emitted Events
const emit = defineEmits(['locationChange', 'subLocationChange'])

// Reactive Data
const selectedLocation = ref<string>(props.initialLocation || '')
const selectedSubItem = ref<string>('') 
const subData = ref<string[]>([])
const locationsList = ref<string[]>([])
const errorMessage = ref<string | null>(null)

// Fetch Location Names
const fetchLocations = async () => {
  try {
    const response = await fetch(`https://backend-app-price.onrender.com/location_names`)
    if (!response.ok) throw new Error('Failed to fetch locations')
    locationsList.value = await response.json()
  } catch (error) {
    errorMessage.value = 'Could not load locations. Please try again.'
  }
}

// Fetch Sub-Data Based on Location
const fetchSubData = async () => {
  subData.value = []
  selectedSubItem.value = ''
  errorMessage.value = null

  if (selectedLocation.value) {
    try {
      const response = await fetch(`https://backend-app-price.onrender.com/location_data/${selectedLocation.value}`)
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

// Handle Location Change
const onLocationChange = async () => {
  emit('locationChange', selectedLocation.value)
  await fetchSubData()
}

// Handle Sub-Item Selection
const selectSubItem = (item: string) => {
  selectedSubItem.value = item
  emit('subLocationChange', item)
}

// Fetch Locations on Mounted
onMounted(() => {
  fetchLocations()
  if (props.initialLocation) {
    fetchSubData()
  }
})

// Watch Prop Change for Initial Location
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
