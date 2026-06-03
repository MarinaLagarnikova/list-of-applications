<template>
  <BaseDrawer :open="open" @close="handleClose()">

    <!-- ─── Sticky header ───────────────────────── -->
    <div class="shrink-0 px-6 pt-6 pb-4">
      <div class="flex items-center justify-between">
        <span class="text-[20px] leading-[32px] font-medium text-zinc-900">Выгрузка заявок</span>
        <button
          @click="handleClose()"
          class="flex size-6 items-center justify-center text-zinc-500 hover:text-zinc-900 transition-colors"
        >
          <XIcon :size="20" />
        </button>
      </div>
    </div>

    <!-- ─── Scrollable content ──────────────────── -->
    <div class="flex-1 overflow-y-auto min-h-0">
      <div class="px-6 py-4">
        <div class="mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Укажите период</span>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge.label"
            @click="selectBadge(badge)"
            :class="[
              'inline-flex items-center rounded-full border px-3 h-6 text-[13px] leading-none font-medium transition-colors',
              selectedBadge === badge.label
                ? 'bg-indigo-50 border-indigo-200 text-indigo-700'
                : 'bg-white border-[#e4e4e7] text-[#3f3f46] hover:bg-zinc-50',
            ]"
          >{{ badge.label }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="dateCustom"
            @input="selectedBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-medium text-zinc-900 placeholder:text-[#a1a1aa] placeholder:font-medium focus:outline-none"
          />
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>
      <div class="h-4" />
    </div>

    <!-- ─── Sticky footer ───────────────────────── -->
    <div class="shrink-0 border-t border-[#f4f4f5] px-6 py-4 flex items-center gap-x-3">
      <button
        @click="handleClose()"
        class="flex items-center justify-center rounded-lg border border-[#e4e4e7] h-9 px-4 text-[14px] font-medium text-zinc-900 hover:bg-zinc-50 transition-colors"
      >Отменить</button>
      <button
        :disabled="!hasDate"
        class="flex flex-1 items-center justify-center gap-x-2 rounded-lg bg-indigo-600 h-9 px-4 text-[14px] font-medium text-white transition-colors disabled:opacity-40 disabled:cursor-not-allowed hover:bg-indigo-700 active:bg-indigo-800"
      >
        <ArrowDownToLineIcon :size="15" class="shrink-0" />
        {{ exportLabel }}
      </button>
    </div>

  </BaseDrawer>
</template>

<script setup>
import { ref, computed } from 'vue'
import BaseDrawer from './BaseDrawer.vue'
import { X as XIcon, Calendar as CalendarIcon, ArrowDownToLine as ArrowDownToLineIcon } from 'lucide-vue-next'

const props = defineProps({
  open: Boolean,
  count: { type: Number, default: 0 },
})
const emit = defineEmits(['close'])

const handleClose = () => {
  selectedBadge.value = null
  dateCustom.value = ''
  emit('close')
}

const today = new Date(2026, 4, 15) // 15.05.2026

const fmt = (d) => d.toLocaleDateString('ru-RU', { day: '2-digit', month: '2-digit', year: 'numeric' }).replace(/\//g, '.')

const addDays = (d, n) => { const r = new Date(d); r.setDate(r.getDate() + n); return r }

const dateBadges = [
  { label: 'Сегодня',   date: fmt(today) },
  { label: 'Вчера',     date: fmt(addDays(today, -1)) },
  { label: 'Неделя',    date: `${fmt(addDays(today, -7))} – ${fmt(today)}` },
  { label: '2 недели',  date: `${fmt(addDays(today, -14))} – ${fmt(today)}` },
  { label: 'Май',       date: '01.05.2026 – 31.05.2026' },
]

const selectedBadge = ref(null)
const dateCustom = ref('')

const selectBadge = (badge) => {
  if (selectedBadge.value === badge.label) {
    selectedBadge.value = null
    dateCustom.value = ''
  } else {
    selectedBadge.value = badge.label
    dateCustom.value = badge.date
  }
}

const hasDate = computed(() => dateCustom.value.trim() !== '')

const plural = (n) => {
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return 'заявок'
  if (mod10 === 1) return 'заявка'
  if (mod10 >= 2 && mod10 <= 4) return 'заявки'
  return 'заявок'
}

const exportLabel = computed(() =>
  hasDate.value
    ? `Выгрузить ${props.count} ${plural(props.count)}`
    : 'Выгрузить'
)
</script>
