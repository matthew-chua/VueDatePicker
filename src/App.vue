<script setup lang="ts">
import { ref } from "vue";
import DatePicker from "./DatePicker.vue";

const validEndDates = [
  new Date('2022-07-27'),
  new Date('2022-07-28'),
  new Date('2022-07-29'),
  new Date('2022-07-30'),
  new Date('2022-07-31'),
  new Date('2022-08-01'),
  new Date('2022-08-02'),
]

const today = ref(new Date())
const test = (res: Date) => {
  console.log(res)
}

const styles = {
  SELECTED_COLOR: 'text-red-600',
  VALID_COLOR: 'text-black',
  NOT_SELECTED_COLOR: 'text-gray-500',
  BUTTON_COLOR: 'bg-sky-900',
  BACKGROUND_COLOR: 'bg-white'
}

</script>

<template>
<div class="flex justify-center">
  <DatePicker :start-date="today" :end-date="today" @update:model-value="test" :styles="styles">
  <template #header>
    Select Lockup Date
  </template>
  <template #button-text>
    Confirm Date
  </template>
  
  </DatePicker>
  </div>
</template>



