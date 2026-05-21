<template>
  <div class="relative flex h-full flex-col overflow-hidden">

  <!-- Поиск + фильтры — фиксированная строка -->
  <div :class="['shrink-0 flex items-center justify-between px-8 py-2', isScrolled && !(filterTags && filterTags.length) && 'border-b border-[#f4f4f5]']">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ФИО или телефону" />
      <div class="flex items-center gap-x-3">
        <span v-if="searchQuery.trim()" class="text-[14px] font-light text-[#71717a] mr-5">
          {{ pluralize(filteredItems.length) }}
        </span>
        <Button outline @click="$emit('open-filter')">
          <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
          Фильтры
          <span v-if="props.filterCount > 0" class="inline-flex items-center justify-center size-[18px] rounded-full bg-indigo-600 text-white text-[11px] font-medium leading-none">{{ props.filterCount }}</span>
        </Button>
        <SortDropdown v-model="sortOrder" />
        <ExportPopover :count="filteredItems.length" />
      </div>
  </div>

  <!-- Applied filters strip -->
  <div v-if="filterTags && filterTags.length > 0" :class="['shrink-0 pl-[60px] pr-8 pt-2 pb-4', isScrolled && 'border-b border-[#f4f4f5]']">
    <FilterTagsBar :tags="filterTags" @reset="$emit('reset-filters')" @show-more="$emit('open-filter')" />
  </div>

  <div class="flex-1 overflow-auto" @scroll="isScrolled = $event.target.scrollTop > 0">

    <!-- Список -->
    <div class="pt-6 pb-10">
      <table class="w-full border-separate border-spacing-0">
        <tbody>
          <tr
            v-for="item in filteredItems"
            :key="item.id"
            :class="['group cursor-pointer', checkedRows.has(item.id) && 'selected']"
            @click="openApp(item)"
          >
            <!-- Checkbox -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top pt-[16px] pb-3 pl-8 pr-2" @click.stop="toggleRow(item.id)">
              <label class="inline-flex cursor-pointer">
                <span :class="['relative flex size-4 items-center justify-center rounded-sm border', checkedRows.has(item.id) ? 'bg-indigo-600 border-transparent' : 'bg-white border-zinc-950/15 hover:border-zinc-950/30']">
                  <svg :class="['size-3 stroke-white transition-opacity', checkedRows.has(item.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
                </span>
              </label>
            </td>

            <!-- ID + дата -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top pl-3 pr-[22px] py-4 whitespace-nowrap">
              <div class="flex flex-col items-start gap-y-1">
                <TableLink @click="$emit('open-preview', item)">{{ item.id }}</TableLink>
                <span class="text-[14px] leading-[20px] font-light text-zinc-900">от {{ item.date }}</span>
              </div>
            </td>

            <!-- Аватар + ФИО + Телефон -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
              <div class="flex items-start gap-x-3">
                <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] font-medium text-[#71717a]">{{ item.initials }}</span>
                <div class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.client }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.phone }}</span>
                </div>
              </div>
            </td>

            <!-- Статус -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
              <span :class="statusBadgeClass(item.status)">{{ item.status }}</span>
            </td>

            <!-- Рейтинг 1 + Дата -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <div v-if="item.lastCheck" class="flex items-center gap-x-1.5">
                  <span :style="{ display:'flex', padding:'4px', borderRadius:'9999px', background:scoreHaloBg(item.lastCheck.score), flexShrink:0 }">
                    <span class="size-2 rounded-full" :style="{ background: scoreDotColor(item.lastCheck.score) }" />
                  </span>
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ scoreLabel(item.lastCheck.score) }}</span>
                </div>
                <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ formatCheckDate(item.lastCheckDate) ?? '' }}</span>
              </div>
            </td>

            <!-- Рейтинг 2 + Дата -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <div v-if="item.prevCheck" class="flex items-center gap-x-1.5">
                  <span :style="{ display:'flex', padding:'4px', borderRadius:'9999px', background:scoreHaloBg(item.prevCheck.score), flexShrink:0 }">
                    <span class="size-2 rounded-full" :style="{ background: scoreDotColor(item.prevCheck.score) }" />
                  </span>
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ scoreLabel(item.prevCheck.score) }}</span>
                </div>
                <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ formatCheckDate(item.prevCheckDate) ?? '' }}</span>
              </div>
            </td>

            <!-- Менеджер -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top pl-[22px] pr-8 py-4 whitespace-nowrap" @click.stop>
              <CatalystListbox
                :options="managerOptions"
                :model-value="managerSelections[item.id]"
                @update:model-value="managerSelections[item.id] = $event"
                placeholder="Менеджер"
              />
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Empty state -->
      <div v-if="filteredItems.length === 0 && (searchQuery.trim() || props.filterCount > 0)" class="flex flex-col items-center justify-center py-24 text-center">
        <svg class="mx-auto size-12 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true">
          <path vector-effect="non-scaling-stroke" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 13h6m-3-3v6m-9 1V7a2 2 0 012-2h6l2 2h6a2 2 0 012 2v8a2 2 0 01-2 2H5a2 2 0 01-2-2z" />
        </svg>
        <h3 class="mt-3 text-sm font-medium text-zinc-900">Заявки не найдены</h3>
        <p class="mt-1 text-sm font-light text-zinc-500">Попробуйте изменить параметры поиска или фильтрации</p>
        <Button v-if="searchQuery.trim() && !props.filterCount" outline class="mt-6" @click="searchQuery = ''">Сбросить поиск</Button>
        <Button v-else-if="!searchQuery.trim() && props.filterCount > 0" outline class="mt-6" @click="$emit('reset-filters')">Сбросить фильтры</Button>
      </div>

    </div>

  </div><!-- /overflow-auto -->
  <SelectionBar :count="checkedRows.size" @clear="checkedRows = new Set()" @delete="checkedRows = new Set()" />
  </div><!-- /relative wrapper -->

</template>

<script setup>
import { ref, computed, watch } from 'vue'
import SearchInput from './SearchInput.vue'
import Button from './Button.vue'
import TableLink from './TableLink.vue'
import CatalystListbox from './CatalystListbox.vue'
import SelectionBar from './SelectionBar.vue'
import ExportPopover from './ExportPopover.vue'
import SortDropdown from './SortDropdown.vue'
import FilterTagsBar from './FilterTagsBar.vue'
import {
  Search as MagnifyingGlassIcon,
  SlidersHorizontal as FunnelIcon,
  ChevronDown as ChevronDownIcon,
} from 'lucide-vue-next'

const props = defineProps({
  activeSubItem: { type: String, default: 'Все заявки' },
  filterCount: { type: Number, default: 0 },
  filters: { type: Object, default: () => ({}) },
  filterTags: { type: Array, default: () => [] },
})

const emit = defineEmits(['open-filter', 'open-preview', 'update:count', 'reset-filters'])


const sortOrder = ref('new')
const isScrolled = ref(false)
const searchQuery = ref('')
const checkedRows = ref(new Set())

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (status === 'Выполнена') return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'На проверке')        return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'Ошибка проверки')    return `${base} bg-red-50 text-red-700 inset-ring-red-600/10`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const managerOptions = ['Демо ДВИЖ', 'Иванов А.В.', 'Петрова М.С.', 'Сидоров К.Н.']

const managerSelections = ref(
  Object.fromEntries(
    ['СК 1','СК 2','СК 3','СК 4','СК 5','СК 6','СК 7','СК 8'].map(id => [id, 'Демо ДВИЖ'])
  )
)

const filterPills = [
  { label: 'Статус проверки' },
  { label: 'Рейтинг' },
  { label: 'Дата создания' },
  { label: 'Дата последней проверки' },
  { label: 'Дата предпоследней проверки' },
  { label: 'Менеджер' },
]

const toggleRow = (id) => {
  const s = new Set(checkedRows.value)
  s.has(id) ? s.delete(id) : s.add(id)
  checkedRows.value = s
}

const statusDotColor = (status) => {
  if (status === 'Выполнена') return 'bg-[#16a34a]'
  if (status === 'Ошибка проверки') return 'bg-[#dc2626]'
  return 'bg-[#ca8a04]'
}

const formatCheckDate = (dateStr) => {
  if (!dateStr) return null
  const [day, month] = dateStr.split('.')
  const months = ['января','февраля','марта','апреля','мая','июня','июля','августа','сентября','октября','ноября','декабря']
  return `Проверка от ${parseInt(day)} ${months[parseInt(month) - 1]}`
}

const scoreLabel = (score) => {
  if (score >= 800) return 'Отличный'
  if (score >= 700) return 'Высокий'
  if (score >= 600) return 'Средний'
  if (score >= 500) return 'Низкий'
  return 'Низкий'
}

const scoreDotColor = (score) => {
  if (score >= 800) return '#16a34a' // green-600
  if (score >= 700) return '#65a30d' // lime-600
  if (score >= 600) return '#ca8a04' // yellow-600
  return '#e11d48'                   // rose-600
}

const debtLoadColor = (v) => {
  if (v <= 20) return '#16a34a'  // green-600  — Низкая
  if (v <= 50) return '#ca8a04'  // yellow-600 — Допустимая
  if (v <= 80) return '#b45309'  // amber-700  — Средняя
  return '#dc2626'               // red-600    — Высокая
}

const scoreHaloBg = (score) => {
  if (score >= 800) return 'rgba(22,163,74,0.10)'
  if (score >= 700) return 'rgba(101,163,13,0.10)'
  if (score >= 600) return 'rgba(202,138,4,0.10)'
  return 'rgba(225,29,72,0.10)'
}

const pluralize = (n) => {
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return `найдено ${n} заявок`
  if (mod10 === 1) return `найдена ${n} заявка`
  if (mod10 >= 2 && mod10 <= 4) return `найдено ${n} заявки`
  return `найдено ${n} заявок`
}

const items = [
  {
    id: 'СК 8', date: '29.04', mine: true,
    status: 'Выполнена',
    initials: 'MM', client: 'Могиль М.В.', phone: '+7 901 ***-**-19', fullPhone: '+7 901 234 56 19',
    fullName: 'Могиль Михаил Валерьевич',
    lastCheck:  { score: 831, debtLoad: 15, totalPayments: '57 991 ₽/мес' },
    prevCheck:  { score: 797, debtLoad: 20, totalPayments: '57 991 ₽/мес' },
    lastCheckDate: '06.05.2026',
    prevCheckDate: '29.04.2026',
    manager: 'Демо ДВИЖ',
  },
  {
    id: 'СК 7', date: '28.04', mine: false,
    status: 'Выполнена',
    initials: 'НЕ', client: 'Новикова Е.Д.', phone: '+7 925 ***-**-31', fullPhone: '+7 925 123 45 31',
    fullName: 'Новикова Екатерина Дмитриевна',
    lastCheck:  { score: 742, debtLoad: 28, totalPayments: '68 200 ₽/мес' },
    prevCheck:  { score: 718, debtLoad: 35 },
    lastCheckDate: '05.05.2026',
    prevCheckDate: '20.04',
    manager: 'Демо ДВИЖ',
  },
  {
    id: 'СК 6', date: '27.04', mine: true,
    status: 'Выполнена',
    initials: 'ОД', client: 'Орлов Д.П.', phone: '+7 912 ***-**-07', fullPhone: '+7 912 345 67 07',
    fullName: 'Орлов Дмитрий Павлович',
    lastCheck:  { score: 815, debtLoad: 18, totalPayments: '41 300 ₽/мес' },
    prevCheck:  { score: 799, debtLoad: 22 },
    lastCheckDate: '04.05.2026',
    prevCheckDate: '18.04',
    manager: 'Демо ДВИЖ',
  },
  {
    id: 'СК 5', date: '26.04', mine: false,
    status: 'На проверке',
    initials: 'ВА', client: 'Воронова А.С.', phone: '+7 926 ***-**-64', fullPhone: '+7 926 456 78 64',
    fullName: 'Воронова Анна Сергеевна',
    lastCheck:  { score: 610, debtLoad: 58, totalPayments: '112 700 ₽/мес' },
    prevCheck:  null,
    lastCheckDate: '26.04.2026',
    manager: 'Демо ДВИЖ',
  },
  {
    id: 'СК 4', date: '25.04', mine: true,
    status: 'Выполнена',
    initials: 'ЛИ', client: 'Лебедев И.Н.', phone: '+7 967 ***-**-19', fullPhone: '+7 967 567 89 19',
    fullName: 'Лебедев Игорь Николаевич',
    lastCheck:  { score: 779, debtLoad: 33, totalPayments: '87 400 ₽/мес' },
    prevCheck:  { score: 755, debtLoad: 40 },
    lastCheckDate: '03.05.2026',
    prevCheckDate: '15.04',
    manager: 'Демо ДВИЖ',
  },
  {
    id: 'СК 3', date: '24.04', mine: false,
    status: 'Ошибка проверки',
    initials: 'МС', client: 'Морозов С.В.', phone: '+7 963 ***-**-93', fullPhone: '+7 963 678 90 93',
    fullName: 'Морозов Сергей Владимирович',
    lastCheck:  null,
    prevCheck:  null,
    lastCheckDate: null,
    manager: 'Демо ДВИЖ',
  },
  {
    id: 'СК 2', date: '23.04', mine: true,
    status: 'Выполнена',
    initials: 'СЮ', client: 'Смирнова Ю.А.', phone: '+7 918 ***-**-56', fullPhone: '+7 918 789 01 56',
    fullName: 'Смирнова Юлия Андреевна',
    lastCheck:  { score: 831, debtLoad: 12, totalPayments: '31 800 ₽/мес' },
    prevCheck:  { score: 810, debtLoad: 16 },
    lastCheckDate: '02.05.2026',
    prevCheckDate: '10.04',
    manager: 'Демо ДВИЖ',
  },
  {
    id: 'СК 1', date: '22.04', mine: false,
    status: 'Выполнена',
    initials: 'АР', client: 'Андреев Р.Е.', phone: '+7 909 ***-**-74', fullPhone: '+7 909 890 12 74',
    fullName: 'Андреев Роман Евгеньевич',
    lastCheck:  { score: 708, debtLoad: 42, totalPayments: '95 100 ₽/мес' },
    prevCheck:  null,
    lastCheckDate: '01.05.2026',
    manager: 'Демо ДВИЖ',
  },
]

const openApp = (item) => { window.open(`/app-page.html?id=${item.id}&title=${encodeURIComponent('Это страница заявки')}`, '_blank') }

const filteredItems = computed(() => {
  let result = items
  if (props.activeSubItem === 'Мои заявки') result = result.filter(item => item.mine)
  if (props.filters?.statuses?.size > 0) result = result.filter(item => props.filters.statuses.has(item.status))
  if (props.filters?.ratings?.size > 0) result = result.filter(item => item.lastCheck && props.filters.ratings.has(scoreLabel(item.lastCheck.score)))
  if (props.filters?.managers?.size > 0) result = result.filter(item => props.filters.managers.has(item.manager))
  if (sortOrder.value === 'old') result = [...result].reverse()
  const q = searchQuery.value.trim().toLowerCase()
  if (q) result = result.filter(item =>
    item.id.toLowerCase().includes(q) ||
    item.fullName.toLowerCase().includes(q) ||
    item.phone.replace(/\s/g, '').includes(q.replace(/\s/g, ''))
  )
  return result
})
watch(filteredItems, v => emit('update:count', v.length), { immediate: true })
</script>
