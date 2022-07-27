<script setup lang="ts">
import { ref, computed } from 'vue'
import { ChevronLeftIcon, ChevronRightIcon } from '@heroicons/vue/outline'
import type { Dayjs } from 'dayjs'
import dayjs from 'dayjs'
import isBetween from 'dayjs/plugin/isBetween'
dayjs.extend(isBetween)

const props = withDefaults(
  defineProps<{
  endDate: Date
  startDate: Date
  validDates?: Date[]
  duration?: boolean
  styles?: {
    SELECTED_COLOR: string
    VALID_COLOR: string
    NOT_SELECTED_COLOR: string
    BUTTON_COLOR: string
    BACKGROUND_COLOR: string
  }
}>(), {
  styles: {
  SELECTED_COLOR: 'text-sky-600',
  VALID_COLOR: 'text-black',
  NOT_SELECTED_COLOR: 'text-gray-500',
  BUTTON_COLOR: 'bg-sky-500',
  BACKGROUND_COLOR: 'bg-sky-100'
}
}
) 

const days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']

const prependZero = (num: number, digitCount = 2) =>
  ('0'.repeat(digitCount - 1) + num).slice(digitCount * -1)

const serializeMonthDates = (date: Dayjs) => {
  const dates = []
  const maxDate = date.daysInMonth()
  const month = prependZero(date.month() + 1)
  const year = date.year()
  for (let i = 1; i < maxDate + 1; i++) {
    const formattedDate = dayjs(`${year}-${month}-${i}`).format('YYYY-MM-DD')
    dates.push(formattedDate)
  }
  return dates
}

const dates = ref(serializeMonthDates(dayjs(props.endDate)))
const currentMonth = ref(dayjs(props.endDate))
const selectedDate = ref(dayjs(props.endDate))
const startDate = ref(dayjs(props.startDate).format('YYYY-MM-DD'))

const selectDateHandler = (date: string) => {
  selectedDate.value = dayjs(date)
}

const monthIncrease = () => {
  const newDate = dayjs(currentMonth.value).add(1, 'month')
  currentMonth.value = newDate
  dates.value = serializeMonthDates(newDate)
}

const monthDecrease = () => {
  const newDate = dayjs(currentMonth.value).subtract(1, 'month')
  currentMonth.value = newDate
  dates.value = serializeMonthDates(newDate)
}

const paddedDates = computed(() => {
  const month = currentMonth.value.format('YYYY-MM')
  const firstDay = dayjs(month).day()
  for (let i = 0; i < firstDay - 1; i++) dates.value.unshift('')
  return dates.value
})

// const lockDuration = computed(() => {
//   return dayjs(endDate.value).diff(startDate.value, 'w')
// })

const selected = (date: string) => {
  console.log(date)
  return props.duration
    ? dayjs(date).isBetween(startDate.value, selectedDate.value, 'day', '[]')
    : date === selectedDate.value.format('YYYY-MM-DD')
}

const formattedValidDates = computed(() => {
  return props.validDates
    ? props.validDates!.map((validDate) => {
        return dayjs(validDate).format('YYYY-MM-DD')
      })
    : []
})

const emit = defineEmits<{
  (event: 'update:model-value', selectedDate: Date): void
}>()
</script>
<template>
  <div class="md:w-1/2 xl:w-1/3">
    <div
      class="mt-4 justify-center flex flex-col items-center rounded-xl border-2 p-6 sm:p-8"
      :class="styles.BACKGROUND_COLOR"
    >
      <div
        class="mt-4 flex justify-center items-center text-2xl font-bold text-black"
      >
        <slot name="header" />
      </div>

      <div class="mt-4">
        <div class="flex items-center justify-center text-black">
          <button class="text-black" @click="monthDecrease">
            prev
          </button>
          <div class="flex-grow" />
          <div>{{ dayjs(currentMonth).format('MMM YYYY') }}</div>
          <div class="flex-grow" />
          <button class="text-black" @click="monthIncrease">
            next
          </button>
        </div>
        <div class="mt-8 grid grid-cols-7">
          <div
            v-for="day in days"
            :key="day"
            class="flex w-10 justify-center text-sm font-semibold text-gray-400"
          >
            {{ day }}
          </div>
        </div>

        <div class="mt-4 grid grid-cols-7 justify-center gap-y-2">
          <button
            v-for="date in paddedDates"
            :key="date.toString()"
            :disabled="
              validDates
                ? date === '' || !formattedValidDates.includes(date)
                : date === ''
            "
            :class="
              selected(date)
                ? styles.SELECTED_COLOR
                : validDates
                ? formattedValidDates.includes(date)
                  ? styles.VALID_COLOR
                  : styles.NOT_SELECTED_COLOR
                : styles.VALID_COLOR
            "
            class="flex items-center justify-center"
            @click="selectDateHandler(date)"
          >
            {{ date.split('-')[2] }}
          </button>
        </div>
      </div>

      <div
        class="mt-8 flex w-full items-center justify-center p-4 text-water-200 sm:px-20"
      >
        <div class="flex items-center flex-col">
          Date Selected:
          <br />
          <span class="font-bold text-grass">
            {{ selectedDate.format('DD MMM YYYY') }}
          </span>
        </div>
        <slot name="bottom-card" />
      </div>
      <button
        class="mt-8 rounded-full mb-4 px-20 py-2 sm:mb-0 sm:w-auto"
        :class="styles.BUTTON_COLOR"
        @click="emit('update:model-value', selectedDate.toDate())"
      >
        <slot name="button-text" />
      </button>
    </div>
    <span class="text-sm text-water-200">
      <slot name="footer" />
    </span>
  </div>
</template>
