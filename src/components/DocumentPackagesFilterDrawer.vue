<template>
  <BaseDrawer :open="open" @close="$emit('close')">
    <!-- ─── Sticky header ───────────────────────── -->
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

    <!-- ─── Scrollable content ──────────────────── -->
    <div class="flex-1 overflow-y-auto min-h-0">

      <!-- Дата создания -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата создания</span>
          <button
            v-if="selectedCreatedBadge || createdCustom"
            @click="resetCreated"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedCreatedBadge === badge ? (selectedCreatedBadge = null, createdCustom = '') : (selectedCreatedBadge = badge, createdCustom = badgeToDate(badge))"
            :class="selectedCreatedBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="createdCustom"
            @input="selectedCreatedBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="createdCustom" @click="resetCreated" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

      <!-- Статус пакета -->
      <FilterStatusGroup
        label="Статус пакета"
        :options="packageStatuses"
        :model-value="selectedStatuses"
        @update:model-value="selectedStatuses = $event"
      />

      <!-- Менеджер -->
      <FilterManagerGroup
        label="Менеджер"
        :options="managers"
        :model-value="selectedManagers"
        @update:model-value="selectedManagers = $event"
      />

      <!-- Дата подписания -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата подписания</span>
          <button
            v-if="selectedSignedBadge || signedCustom"
            @click="resetSigned"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedSignedBadge === badge ? (selectedSignedBadge = null, signedCustom = '') : (selectedSignedBadge = badge, signedCustom = badgeToDate(badge))"
            :class="selectedSignedBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="signedCustom"
            @input="selectedSignedBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="signedCustom" @click="resetSigned" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

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

const dateBadges = ['Сегодня', 'Вчера', 'Неделя', '2 недели', 'Май', '2 квартал']

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

const selectedCreatedBadge = ref(null)
const createdCustom = ref('')
const resetCreated = () => { selectedCreatedBadge.value = null; createdCustom.value = '' }

const selectedSignedBadge = ref(null)
const signedCustom = ref('')
const resetSigned = () => { selectedSignedBadge.value = null; signedCustom.value = '' }

const packageStatuses = [
  { name: 'Черновик',             dot: 'border-gray-400 bg-gray-50'   },
  { name: 'В процессе обработки', dot: 'border-blue-400 bg-blue-50'   },
  { name: 'Готов к подписанию',   dot: 'border-amber-400 bg-amber-50' },
  { name: 'На подписании',        dot: 'border-amber-400 bg-amber-50' },
  { name: 'Подписан',             dot: 'border-green-400 bg-green-50' },
  { name: 'Ошибка документов',    dot: 'border-red-400 bg-red-50'     },
]
const selectedStatuses = ref(new Set())

const managers = [
  { name: 'Иванов Иван',     initials: 'ИИ', role: 'Менеджер ипотеки' },
  { name: 'Сергеев Сергей',  initials: 'СС', role: 'Менеджер ипотеки' },
  { name: 'Петров Пётр',     initials: 'ПП', role: 'Продажи'           },
  { name: 'Сидоров Игорь',   initials: 'СИ', role: 'Руководитель'      },
  { name: 'Новиков Алексей', initials: 'НА', role: 'Администратор'     },
]
const selectedManagers = ref(new Set())

const showAllTags = ref(false)

const activeFilterTags = computed(() => {
  const tags = []
  if (selectedCreatedBadge.value || createdCustom.value)
    tags.push({ label: `Дата создания: ${selectedCreatedBadge.value || createdCustom.value}`, reset: resetCreated })
  selectedStatuses.value.forEach(s =>
    tags.push({ label: s, reset: () => { const set = new Set(selectedStatuses.value); set.delete(s); selectedStatuses.value = set } })
  )
  selectedManagers.value.forEach(m =>
    tags.push({ label: m, reset: () => { const set = new Set(selectedManagers.value); set.delete(m); selectedManagers.value = set } })
  )
  if (selectedSignedBadge.value || signedCustom.value)
    tags.push({ label: `Дата подписания: ${selectedSignedBadge.value || signedCustom.value}`, reset: resetSigned })
  return tags
})

const displayedTags = computed(() =>
  showAllTags.value || activeFilterTags.value.length <= 5
    ? activeFilterTags.value
    : activeFilterTags.value.slice(0, 5)
)

const hasActiveFilters = computed(() =>
  selectedStatuses.value.size > 0 ||
  selectedCreatedBadge.value !== null ||
  createdCustom.value !== '' ||
  selectedSignedBadge.value !== null ||
  signedCustom.value !== '' ||
  selectedManagers.value.size > 0
)

const filterCount = computed(() =>
  selectedStatuses.value.size +
  (selectedCreatedBadge.value !== null || createdCustom.value !== '' ? 1 : 0) +
  (selectedSignedBadge.value !== null || signedCustom.value !== '' ? 1 : 0) +
  selectedManagers.value.size
)

watch(filterCount, val => emit('update:filterCount', val), { immediate: true })
watch(activeFilterTags, val => emit('update:filterTags', val), { immediate: true })

const emitFilters = () => emit('update:filters', {
  statuses: selectedStatuses.value,
  managers: selectedManagers.value,
})
watch([selectedStatuses, selectedManagers], emitFilters, { immediate: true })

const resetAll = () => {
  selectedStatuses.value = new Set()
  resetCreated()
  resetSigned()
  selectedManagers.value = new Set()
  showAllTags.value = false
}

defineExpose({ resetAll })
</script>
