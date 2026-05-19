<template>
  <div class="select-none">
    <!-- Navigation -->
    <div class="flex items-center">
      <h2 class="flex-auto text-[14px] font-medium text-gray-900">{{ monthName }} {{ currentYear }}</h2>
      <button type="button" @click="prevMonth" class="-my-1.5 flex flex-none items-center justify-center p-1.5 text-gray-400 hover:text-gray-500 transition-colors">
        <ChevronLeftIcon class="size-4" />
      </button>
      <button type="button" @click="nextMonth" class="-my-1.5 -mr-1.5 ml-1 flex flex-none items-center justify-center p-1.5 text-gray-400 hover:text-gray-500 transition-colors">
        <ChevronRightIcon class="size-4" />
      </button>
    </div>

    <!-- Week headers -->
    <div class="mt-4 grid grid-cols-7 text-center text-xs text-gray-400">
      <div v-for="d in weekDays" :key="d">{{ d }}</div>
    </div>

    <!-- Days -->
    <div class="mt-1 grid grid-cols-7 text-sm">
      <div
        v-for="(day, idx) in days"
        :key="idx"
        :class="cellClass(day)"
      >
        <button
          type="button"
          @click="selectDay(day)"
          @mouseenter="hoverDate = day.dateStr"
          @mouseleave="hoverDate = null"
          :class="btnClass(day)"
        >{{ day.day }}</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { ChevronLeftIcon, ChevronRightIcon } from '@heroicons/vue/20/solid'

const props = defineProps({
  modelValue: { type: Object, default: () => ({ start: null, end: null }) },
})
const emit = defineEmits(['update:modelValue'])

const todayReal = new Date(2026, 4, 15)
const toStr = (d) => `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`
const todayStr = toStr(todayReal)

const currentYear = ref(todayReal.getFullYear())
const currentMonth = ref(todayReal.getMonth())
const hoverDate = ref(null)
const startDate = ref(props.modelValue?.start ?? null)
const endDate = ref(props.modelValue?.end ?? null)

const weekDays = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']
const monthNames = ['Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь', 'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь']
const monthName = computed(() => monthNames[currentMonth.value])

const prevMonth = () => {
  if (currentMonth.value === 0) { currentMonth.value = 11; currentYear.value-- }
  else currentMonth.value--
}
const nextMonth = () => {
  if (currentMonth.value === 11) { currentMonth.value = 0; currentYear.value++ }
  else currentMonth.value++
}

const days = computed(() => {
  const year = currentYear.value
  const month = currentMonth.value
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)

  // Monday-based (0=Mon, 6=Sun)
  let startDow = firstDay.getDay()
  startDow = startDow === 0 ? 6 : startDow - 1

  const result = []

  for (let i = startDow; i > 0; i--)
    result.push({ date: new Date(year, month, 1 - i), isCurrentMonth: false })

  for (let d = 1; d <= lastDay.getDate(); d++)
    result.push({ date: new Date(year, month, d), isCurrentMonth: true })

  const remaining = result.length % 7
  if (remaining > 0)
    for (let i = 1; i <= 7 - remaining; i++)
      result.push({ date: new Date(year, month + 1, i), isCurrentMonth: false })

  return result.map(({ date, isCurrentMonth }) => ({
    date,
    dateStr: toStr(date),
    day: date.getDate(),
    isCurrentMonth,
    isToday: toStr(date) === todayStr,
  }))
})

const effectiveRange = computed(() => {
  if (!startDate.value) return { start: null, end: null }
  if (endDate.value) return { start: startDate.value, end: endDate.value }
  const b = hoverDate.value
  if (b) return startDate.value <= b ? { start: startDate.value, end: b } : { start: b, end: startDate.value }
  return { start: startDate.value, end: startDate.value }
})

const cellClass = (day) => {
  const { start, end } = effectiveRange.value
  if (!start || start === end) return 'py-0.5'
  if (day.dateStr === start) return 'py-0.5 bg-indigo-50 rounded-l-full'
  if (day.dateStr === end) return 'py-0.5 bg-indigo-50 rounded-r-full'
  if (day.dateStr > start && day.dateStr < end) return 'py-0.5 bg-indigo-50'
  return 'py-0.5'
}

const btnClass = (day) => {
  const base = 'mx-auto flex size-7 items-center justify-center rounded-full text-[13px] transition-colors cursor-pointer'
  const { start, end } = effectiveRange.value
  if (day.dateStr === start || day.dateStr === end)
    return `${base} bg-indigo-600 text-white font-semibold`
  if (start && end && day.dateStr > start && day.dateStr < end)
    return `${base} text-indigo-700 hover:bg-indigo-100`
  if (day.isToday)
    return `${base} font-semibold text-indigo-600 hover:bg-gray-100`
  if (!day.isCurrentMonth)
    return `${base} text-gray-300 hover:bg-gray-100`
  return `${base} text-gray-900 hover:bg-gray-100`
}

const selectDay = (day) => {
  if (!startDate.value || endDate.value) {
    startDate.value = day.dateStr
    endDate.value = null
  } else {
    if (day.dateStr < startDate.value) {
      endDate.value = startDate.value
      startDate.value = day.dateStr
    } else {
      endDate.value = day.dateStr
    }
    emit('update:modelValue', { start: startDate.value, end: endDate.value })
  }
}

watch(() => props.modelValue, (v) => {
  startDate.value = v?.start ?? null
  endDate.value = v?.end ?? null
})
</script>
