<template>
  <BaseDrawer :open="open" @close="$emit('close')">

      <!-- Sticky header -->
      <div class="shrink-0 px-6 pt-6 pb-4">
        <div class="flex items-center justify-between">
          <span class="text-[20px] leading-[32px] font-medium text-zinc-900">Фильтры</span>
          <button
            @click="$emit('close')"
            class="flex size-6 items-center justify-center text-zinc-500 hover:text-zinc-900 transition-colors"
          >
            <XIcon :size="20" />
          </button>
        </div>
      </div>

      <!-- Scrollable content -->
      <div class="flex-1 overflow-y-auto min-h-0">

        <!-- Дата создания -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата создания</span>
            <button v-if="selectedDateBadge || dateCustom" @click="resetDate" class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700">Сбросить</button>
          </div>
          <div class="flex flex-wrap gap-2 mb-3">
            <button
              v-for="badge in dateBadges"
              :key="badge"
              @click="selectedDateBadge === badge ? (selectedDateBadge = null, dateCustom = '') : (selectedDateBadge = badge, dateCustom = badgeToDate(badge))"
              :class="selectedDateBadge === badge ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100' : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
            >{{ badge }}</button>
          </div>
          <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
            <input
              type="text"
              placeholder=""
              v-model="dateCustom"
              @input="selectedDateBadge = null"
              class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
            />
            <button v-if="dateCustom" @click="resetDate" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
              <XIcon :size="16" />
            </button>
            <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
          </div>
        </div>

        <!-- Статус проверки -->
        <FilterStatusGroup
          label="Статус проверки"
          :options="scoringStatuses"
          :model-value="selectedStatuses"
          @update:model-value="selectedStatuses = $event"
        />

        <!-- Рейтинг -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Рейтинг</span>
            <button
              v-if="selectedRatings.size > 0"
              @click="selectedRatings = new Set()"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >Сбросить</button>
          </div>
          <div class="flex flex-col gap-y-3">
            <div
              v-for="rating in scoringRatings"
              :key="rating.label"
              class="flex items-center justify-between cursor-pointer"
              @click="toggleRating(rating.label)"
            >
              <div class="flex items-center gap-x-2.5">
                <span class="flex w-6 shrink-0 items-center justify-center">
                  <span :style="{ display:'flex', padding:'4px', borderRadius:'9999px', background: rating.halo }">
                    <span class="size-2 rounded-full" :style="{ background: rating.dot }" />
                  </span>
                </span>
                <span class="text-[14px] font-light text-zinc-900">{{ rating.label }}</span>
              </div>
              <span :class="['relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors', selectedRatings.has(rating.label) ? 'bg-indigo-600 border-indigo-600' : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]']">
                <svg :class="['size-3 stroke-white transition-opacity', selectedRatings.has(rating.label) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
                  <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
              </span>
            </div>
          </div>
        </div>

        <!-- Менеджер -->
        <FilterManagerGroup
          label="Менеджер"
          :options="managers"
          :model-value="selectedManagers"
          @update:model-value="selectedManagers = $event"
        />

        <div class="h-4" />
      </div>

      <!-- ─── Sticky footer ───────────────────────── -->
      <FilterFooter
        :count="props.count"
        :has-active-filters="hasActiveFilters"
        :filter-count="filterCount"
        :tags="activeFilterTags"
        @reset="resetAll"
        @close="$emit('close')"
      />

  </BaseDrawer>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import BaseDrawer from './BaseDrawer.vue'
import FilterCheckboxGroup from './FilterCheckboxGroup.vue'
import FilterManagerGroup from './FilterManagerGroup.vue'
import FilterStatusGroup from './FilterStatusGroup.vue'
import FilterFooter from './FilterFooter.vue'
import {
  X as XIcon,
  Calendar as CalendarIcon,
} from 'lucide-vue-next'

const props = defineProps({ open: Boolean, count: { type: Number, default: 0 } })
const emit = defineEmits(['close', 'update:filterCount', 'update:filters', 'update:filterTags'])

const scoringStatuses = [
  { name: 'Новая заявка',  dot: 'border-gray-400 bg-gray-50'   },
  { name: 'На проверке',   dot: 'border-amber-400 bg-amber-50' },
  { name: 'Выполнена',     dot: 'border-green-400 bg-green-50' },
  { name: 'Остановлена',   dot: 'border-red-400 bg-red-50'     },
]
const selectedStatuses = ref(new Set())

const scoringRatings = [
  { label: 'Отличный', dot: '#16a34a', halo: 'rgba(22,163,74,0.10)'  },
  { label: 'Высокий',  dot: '#65a30d', halo: 'rgba(101,163,13,0.10)' },
  { label: 'Средний',  dot: '#ca8a04', halo: 'rgba(202,138,4,0.10)'  },
  { label: 'Низкий',   dot: '#e11d48', halo: 'rgba(225,29,72,0.10)'  },
]
const selectedRatings = ref(new Set())
const toggleRating = (r) => {
  const set = new Set(selectedRatings.value)
  set.has(r) ? set.delete(r) : set.add(r)
  selectedRatings.value = set
}

const dateBadges = ['Сегодня', 'Вчера', 'Неделя', '2 недели', 'Май', '2 квартал']
const selectedDateBadge = ref(null)
const dateCustom = ref('')

const today = new Date(2026, 4, 15)
const fmtDate = (d) => d.toLocaleDateString('ru-RU', { day: '2-digit', month: '2-digit', year: 'numeric' })
const addDays = (d, n) => { const r = new Date(d); r.setDate(r.getDate() + n); return r }
const badgeToDate = (badge) => {
  if (badge === 'Сегодня') return fmtDate(today)
  if (badge === 'Вчера') return fmtDate(addDays(today, -1))
  if (badge === 'Неделя') return `${fmtDate(addDays(today, -7))} — ${fmtDate(today)}`
  if (badge === '2 недели') return `${fmtDate(addDays(today, -14))} — ${fmtDate(today)}`
  if (badge === 'Май') return '01.05.2026 — 31.05.2026'
  if (badge === '2 квартал') return '01.04.2026 — 30.06.2026'
  return ''
}

const resetDate = () => { selectedDateBadge.value = null; dateCustom.value = '' }

const managers = [
  { name: 'Иванов Иван',     initials: 'ИИ', role: 'Менеджер ипотеки' },
  { name: 'Сергеев Сергей',  initials: 'СС', role: 'Менеджер ипотеки' },
  { name: 'Петров Пётр',     initials: 'ПП', role: 'Продажи'           },
  { name: 'Сидоров Игорь',   initials: 'СИ', role: 'Руководитель'      },
  { name: 'Новиков Алексей', initials: 'НА', role: 'Администратор'     },
]
const selectedManagers = ref(new Set())

const hasActiveFilters = computed(() =>
  selectedStatuses.value.size > 0 ||
  selectedRatings.value.size > 0 ||
  selectedDateBadge.value !== null ||
  dateCustom.value !== '' ||
  selectedManagers.value.size > 0
)

const filterCount = computed(() =>
  selectedStatuses.value.size +
  selectedRatings.value.size +
  (selectedDateBadge.value !== null || dateCustom.value !== '' ? 1 : 0) +
  selectedManagers.value.size
)

watch(filterCount, val => emit('update:filterCount', val), { immediate: true })

const emitFilters = () => emit('update:filters', {
  statuses: selectedStatuses.value,
  ratings: selectedRatings.value,
  managers: selectedManagers.value,
})
watch([selectedStatuses, selectedRatings, selectedManagers], emitFilters, { immediate: true })

const activeFilterTags = computed(() => {
  const tags = []
  if (selectedDateBadge.value || dateCustom.value)
    tags.push({ label: `Дата создания: ${selectedDateBadge.value || dateCustom.value}`, reset: resetDate })
  selectedStatuses.value.forEach(s =>
    tags.push({ label: s, reset: () => { const set = new Set(selectedStatuses.value); set.delete(s); selectedStatuses.value = set } })
  )
  selectedRatings.value.forEach(r =>
    tags.push({ label: r, reset: () => { const set = new Set(selectedRatings.value); set.delete(r); selectedRatings.value = set } })
  )
  selectedManagers.value.forEach(m =>
    tags.push({ label: m, reset: () => { const set = new Set(selectedManagers.value); set.delete(m); selectedManagers.value = set } })
  )
  return tags
})
watch(activeFilterTags, val => emit('update:filterTags', val), { immediate: true })

const resetAll = () => {
  selectedStatuses.value = new Set()
  selectedRatings.value = new Set()
  resetDate()
  selectedManagers.value = new Set()
}

defineExpose({ resetAll })
</script>
