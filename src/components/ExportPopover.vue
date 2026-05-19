<template>
  <Popover class="relative">
    <PopoverButton as="template">
      <Button outline class="size-8 px-0">
        <ArrowDownToLineIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
      </Button>
    </PopoverButton>

    <Transition
      enter-active-class="transition ease-out duration-150"
      enter-from-class="opacity-0 translate-y-1"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition ease-in duration-100"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 translate-y-1"
    >
      <PopoverPanel v-slot="{ close }" class="absolute right-0 top-full mt-2 z-50 w-[300px] rounded-2xl bg-white shadow-lg ring-1 ring-gray-200 overflow-hidden">
        <div>

          <!-- Header -->
          <div class="px-5 pt-5 pb-0">
            <span class="text-[16px] leading-[24px] font-medium text-[#18181b]">Выгрузка заявок</span>
          </div>

          <!-- Subtitle -->
          <div class="px-5 pt-1 pb-0">
            <span class="text-[14px] leading-[20px] font-light text-zinc-500">Укажите период создания заявки</span>
          </div>

          <!-- Badges -->
          <div class="px-5 pt-4 pb-0">
            <div class="flex flex-wrap gap-1.5">
              <button
                v-for="badge in dateBadges"
                :key="badge.label"
                @click="selectBadge(badge)"
                :class="[
                  'inline-flex items-center rounded-full border px-3 h-6 text-[12px] leading-none font-medium transition-colors',
                  selectedBadge === badge.label
                    ? 'bg-indigo-50 border-indigo-200 text-indigo-700'
                    : 'bg-white border-[#e4e4e7] text-[#3f3f46] hover:bg-zinc-50',
                ]"
              >{{ badge.label }}</button>
            </div>
          </div>

          <!-- Calendar -->
          <div class="px-5 pt-4 pb-4">
            <CalendarRangePicker v-model="dateRange" />
          </div>

          <!-- Footer -->
          <div class="border-t border-[#f4f4f5] px-5 py-3 flex items-center gap-x-2">
            <button
              @click="handleCancel(close)"
              class="flex items-center justify-center rounded-lg border border-[#e4e4e7] h-8 px-3 text-[13px] font-medium text-[#18181b] hover:bg-zinc-50 transition-colors"
            >Отменить</button>
            <button
              :disabled="!hasDate"
              @click="handleExport(close)"
              class="flex flex-1 items-center justify-center gap-x-1.5 rounded-lg bg-indigo-600 h-8 px-3 text-[13px] font-medium text-white transition-colors disabled:opacity-40 disabled:cursor-not-allowed hover:bg-indigo-700 active:bg-indigo-800"
            >
              <ArrowDownToLineIcon :size="14" class="shrink-0" />
              {{ exportLabel }}
            </button>
          </div>

        </div>
      </PopoverPanel>
    </Transition>
  </Popover>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { Popover, PopoverButton, PopoverPanel } from '@headlessui/vue'
import { ArrowDownToLine as ArrowDownToLineIcon } from 'lucide-vue-next'
import Button from './Button.vue'
import CalendarRangePicker from './CalendarRangePicker.vue'

const props = defineProps({
  count: { type: Number, default: 0 },
})

const toStr = (d) => `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`
const addDays = (d, n) => { const r = new Date(d); r.setDate(r.getDate() + n); return r }
const today = new Date(2026, 4, 15)

const dateBadges = [
  { label: 'Сегодня',   start: toStr(today),              end: toStr(today) },
  { label: 'Вчера',     start: toStr(addDays(today, -1)),  end: toStr(addDays(today, -1)) },
  { label: 'Неделя',    start: toStr(addDays(today, -6)),  end: toStr(today) },
  { label: '2 недели',  start: toStr(addDays(today, -13)), end: toStr(today) },
  { label: 'Май',       start: '2026-05-01',               end: '2026-05-31' },
  { label: '2 квартал', start: '2026-04-01',               end: '2026-06-30' },
]

const selectedBadge = ref(null)
const dateRange = ref({ start: null, end: null })

const reset = () => {
  dateRange.value = { start: null, end: null }
  selectedBadge.value = null
}

const selectBadge = (badge) => {
  if (selectedBadge.value === badge.label) {
    reset()
  } else {
    selectedBadge.value = badge.label
    dateRange.value = { start: badge.start, end: badge.end }
  }
}

watch(dateRange, (v) => {
  const matched = dateBadges.find(b => b.start === v.start && b.end === v.end)
  selectedBadge.value = matched ? matched.label : null
})

const hasDate = computed(() => !!(dateRange.value.start && dateRange.value.end))

const plural = (n) => {
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return 'заявок'
  if (mod10 === 1) return 'заявка'
  if (mod10 >= 2 && mod10 <= 4) return 'заявки'
  return 'заявок'
}

const exportLabel = computed(() =>
  hasDate.value ? `Выгрузить ${props.count}` : 'Выгрузить'
)

const handleCancel = (close) => {
  reset()
  close()
}

const handleExport = (close) => {
  window.dispatchEvent(new CustomEvent('export-success', {
    detail: { count: props.count },
  }))
  reset()
  close()
}
</script>
