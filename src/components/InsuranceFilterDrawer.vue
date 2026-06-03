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
            v-if="selectedDateBadge || dateCustom"
            @click="selectedDateBadge = null; dateCustom = ''"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedDateBadge === badge ? (selectedDateBadge = null, dateCustom = '') : (selectedDateBadge = badge, dateCustom = badgeToDate(badge))"
            :class="selectedDateBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
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
          <button v-if="dateCustom" @click="selectedDateBadge = null; dateCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

      <!-- Статус -->
      <FilterStatusGroup
        label="Статус"
        :options="statuses"
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

      <!-- Вид страхования -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Вид страхования</span>
          <button
            v-if="selectedTypes.size > 0"
            @click="selectedTypes = new Set()"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2">
          <button
            v-for="type in insuranceTypes"
            :key="type"
            @click="selectedTypes.has(type) ? selectedTypes.delete(type) : selectedTypes.add(type); selectedTypes = new Set(selectedTypes)"
            :class="selectedTypes.has(type)
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ type }}</button>
        </div>
      </div>

      <!-- Страховая -->
      <FilterBrandGroup
        label="Страховая"
        :options="insurers"
        :model-value="selectedInsurers"
        search-placeholder="Название"
        empty-text="Страховая не найдена"
        @update:model-value="selectedInsurers = $event"
      />

      <!-- Банк -->
      <FilterBrandGroup
        label="Банк"
        :options="banks"
        :model-value="selectedBanks"
        search-placeholder="Название"
        empty-text="Банк не найден"
        @update:model-value="selectedBanks = $event"
      />

      <!-- Дата оплаты -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата оплаты</span>
          <button
            v-if="selectedPayDateBadge || payDateCustom"
            @click="selectedPayDateBadge = null; payDateCustom = ''"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedPayDateBadge === badge ? (selectedPayDateBadge = null, payDateCustom = '') : (selectedPayDateBadge = badge, payDateCustom = badgeToDate(badge))"
            :class="selectedPayDateBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="payDateCustom"
            @input="selectedPayDateBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="payDateCustom" @click="selectedPayDateBadge = null; payDateCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

      <!-- Действует до -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Действует до</span>
          <button
            v-if="selectedExpDateBadge || expDateCustom"
            @click="selectedExpDateBadge = null; expDateCustom = ''"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedExpDateBadge === badge ? (selectedExpDateBadge = null, expDateCustom = '') : (selectedExpDateBadge = badge, expDateCustom = badgeToDate(badge))"
            :class="selectedExpDateBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="expDateCustom"
            @input="selectedExpDateBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="expDateCustom" @click="selectedExpDateBadge = null; expDateCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
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
import FilterFooter from './FilterFooter.vue'
import FilterBrandGroup from './FilterBrandGroup.vue'
import FilterManagerGroup from './FilterManagerGroup.vue'
import FilterStatusGroup from './FilterStatusGroup.vue'
import {
  X as XIcon,
  Calendar as CalendarIcon,
} from 'lucide-vue-next'

const props = defineProps({ open: Boolean, count: { type: Number, default: 0 } })
const emit = defineEmits(['close', 'update:filterCount', 'update:filters', 'update:filterTags'])

// Даты
const dateBadges = ['Сегодня', 'Вчера', 'Неделя', '2 недели', 'Май', '2 квартал']
const selectedDateBadge = ref(null)
const dateCustom = ref('')
const selectedPayDateBadge = ref(null)
const payDateCustom = ref('')
const selectedExpDateBadge = ref(null)
const expDateCustom = ref('')

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

// Менеджер
const managers = [
  { name: 'Смирнова Юлия',  initials: 'СЮ', role: 'Администратор'     },
  { name: 'Орлов Дмитрий',  initials: 'ОД', role: 'Продажи'           },
  { name: 'Лебедев Игорь',  initials: 'ЛИ', role: 'Менеджер ипотеки'  },
  { name: 'Воронова Анна',  initials: 'ВА', role: 'Менеджер ипотеки'  },
  { name: 'Морозов Сергей', initials: 'МС', role: 'Руководитель'       },
]
const selectedManagers = ref(new Set())

// Статус
const statuses = [
  { name: 'Черновик',        dot: 'border-gray-400 bg-gray-50'   },
  { name: 'Оффер выбран',    dot: 'border-blue-400 bg-blue-50'   },
  { name: 'Согласование',    dot: 'border-blue-400 bg-blue-50'   },
  { name: 'Ожидает оплаты',  dot: 'border-amber-400 bg-amber-50' },
  { name: 'Оплачен',         dot: 'border-green-400 bg-green-50' },
  { name: 'Выпущен',         dot: 'border-green-400 bg-green-50' },
  { name: 'Отменен',         dot: 'border-gray-400 bg-gray-50'   },
  { name: 'Ошибка',          dot: 'border-red-400 bg-red-50'     },
]
const selectedStatuses = ref(new Set())

// Банк
const banks = [
  { name: 'Сбербанк',         initial: 'С', color: '#21A038' },
  { name: 'ВТБ',              initial: 'В', color: '#009FDF' },
  { name: 'Альфа-Банк',       initial: 'А', color: '#EF3124' },
  { name: 'Газпромбанк',      initial: 'Г', color: '#0066B3' },
  { name: 'Россельхозбанк',   initial: 'Р', color: '#00703C' },
  { name: 'Открытие',         initial: 'О', color: '#FF6600' },
  { name: 'Райффайзенбанк',   initial: 'Р', color: '#FFDD00' },
  { name: 'Совкомбанк',       initial: 'С', color: '#E31E24' },
  { name: 'Промсвязьбанк',    initial: 'П', color: '#FF6B00' },
  { name: 'Росбанк',          initial: 'Р', color: '#6C2D8F' },
  { name: 'Уралсиб',          initial: 'У', color: '#0070C0' },
  { name: 'МКБ',              initial: 'М', color: '#E4002B' },
]
const selectedBanks = ref(new Set())

// Вид страхования
const insuranceTypes = ['Имущество', 'Жизнь', 'Имущество + Жизнь']
const selectedTypes = ref(new Set())

// Страховая
const insurers = [
  { name: 'Сбербанк Страхование', initial: 'С', color: '#21A038', logo: 'https://favicon.yandex.net/favicon/v2/sberbank-insurance.ru?size=32&stub=1' },
  { name: 'ВТБ Страхование',      initial: 'В', color: '#009FDF', logo: 'https://favicon.yandex.net/favicon/v2/vtb-ins.ru?size=32&stub=1' },
  { name: 'Альфастрахование',     initial: 'А', color: '#EF3124', logo: 'https://favicon.yandex.net/favicon/v2/alfastrah.ru?size=32&stub=1' },
  { name: 'Ингосстрах',           initial: 'И', color: '#0033A0', logo: 'https://favicon.yandex.net/favicon/v2/ingos.ru?size=32&stub=1' },
  { name: 'Росгосстрах',          initial: 'Р', color: '#CC0000', logo: 'https://favicon.yandex.net/favicon/v2/rgs.ru?size=32&stub=1' },
]
const selectedInsurers = ref(new Set())

const hasActiveFilters = computed(() =>
  selectedDateBadge.value !== null ||
  dateCustom.value !== '' ||
  selectedPayDateBadge.value !== null ||
  payDateCustom.value !== '' ||
  selectedExpDateBadge.value !== null ||
  expDateCustom.value !== '' ||
  selectedManagers.value.size > 0 ||
  selectedStatuses.value.size > 0 ||
  selectedBanks.value.size > 0 ||
  selectedTypes.value.size > 0 ||
  selectedInsurers.value.size > 0
)

const filterCount = computed(() =>
  (selectedDateBadge.value !== null || dateCustom.value !== '' ? 1 : 0) +
  (selectedPayDateBadge.value !== null || payDateCustom.value !== '' ? 1 : 0) +
  (selectedExpDateBadge.value !== null || expDateCustom.value !== '' ? 1 : 0) +
  selectedManagers.value.size +
  selectedStatuses.value.size +
  selectedBanks.value.size +
  selectedTypes.value.size +
  selectedInsurers.value.size
)

watch(filterCount, val => emit('update:filterCount', val), { immediate: true })

const parseBadgeRange = (badge, custom) => {
  const str = badge ? badgeToDate(badge) : custom
  if (!str) return { from: null, to: null }
  if (str.includes('—')) {
    const [f, t] = str.split('—').map(s => s.trim())
    return { from: f, to: t }
  }
  return { from: str, to: str }
}

const activeFilters = computed(() => {
  const dateR    = parseBadgeRange(selectedDateBadge.value, dateCustom.value)
  const payDateR = parseBadgeRange(selectedPayDateBadge.value, payDateCustom.value)
  const expDateR = parseBadgeRange(selectedExpDateBadge.value, expDateCustom.value)
  return {
    statuses:    [...selectedStatuses.value],
    managers:    [...selectedManagers.value],
    types:       [...selectedTypes.value],
    insurers:    [...selectedInsurers.value],
    banks:       [...selectedBanks.value],
    dateFrom:    dateR.from,
    dateTo:      dateR.to,
    payDateFrom: payDateR.from,
    payDateTo:   payDateR.to,
    expDateFrom: expDateR.from,
    expDateTo:   expDateR.to,
  }
})
watch(activeFilters, val => emit('update:filters', val), { immediate: true, deep: true })

const activeFilterTags = computed(() => {
  const tags = []
  if (selectedDateBadge.value || dateCustom.value)
    tags.push({ label: `Дата создания: ${selectedDateBadge.value || dateCustom.value}`, reset: () => { selectedDateBadge.value = null; dateCustom.value = '' } })
  selectedStatuses.value.forEach(s =>
    tags.push({ label: s, reset: () => { const set = new Set(selectedStatuses.value); set.delete(s); selectedStatuses.value = set } })
  )
  selectedManagers.value.forEach(m =>
    tags.push({ label: m, reset: () => { const set = new Set(selectedManagers.value); set.delete(m); selectedManagers.value = set } })
  )
  selectedTypes.value.forEach(t =>
    tags.push({ label: t, reset: () => { const set = new Set(selectedTypes.value); set.delete(t); selectedTypes.value = set } })
  )
  selectedInsurers.value.forEach(i =>
    tags.push({ label: i, reset: () => { const set = new Set(selectedInsurers.value); set.delete(i); selectedInsurers.value = set } })
  )
  selectedBanks.value.forEach(b =>
    tags.push({ label: b, reset: () => { const set = new Set(selectedBanks.value); set.delete(b); selectedBanks.value = set } })
  )
  if (selectedPayDateBadge.value || payDateCustom.value)
    tags.push({ label: `Дата оплаты: ${selectedPayDateBadge.value || payDateCustom.value}`, reset: () => { selectedPayDateBadge.value = null; payDateCustom.value = '' } })
  if (selectedExpDateBadge.value || expDateCustom.value)
    tags.push({ label: `Действует до: ${selectedExpDateBadge.value || expDateCustom.value}`, reset: () => { selectedExpDateBadge.value = null; expDateCustom.value = '' } })
  return tags
})
watch(activeFilterTags, val => emit('update:filterTags', val), { immediate: true })

const resetAll = () => {
  selectedDateBadge.value    = null
  dateCustom.value           = ''
  selectedPayDateBadge.value = null
  payDateCustom.value        = ''
  selectedExpDateBadge.value = null
  expDateCustom.value        = ''
  selectedManagers.value  = new Set()
  selectedStatuses.value  = new Set()
  selectedBanks.value     = new Set()
  selectedTypes.value     = new Set()
  selectedInsurers.value  = new Set()
}

defineExpose({ resetAll })
</script>
