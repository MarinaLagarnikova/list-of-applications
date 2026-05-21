<template>
  <div class="relative flex h-full flex-col overflow-hidden">


    <!-- Поиск + фильтры -->
    <div :class="['shrink-0 flex items-center justify-between px-8 py-2', isScrolled && !(filterTags && filterTags.length) && 'border-b border-[#f4f4f5]']">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID или названию" />
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
              <td class="w-px group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top pt-[16px] pb-3 pl-8 pr-2" @click.stop="toggleRow(item.id)">
                <label class="inline-flex cursor-pointer">
                  <span :class="['relative flex size-4 items-center justify-center rounded-sm border', checkedRows.has(item.id) ? 'bg-indigo-600 border-transparent' : 'bg-white border-zinc-950/15 hover:border-zinc-950/30']">
                    <svg :class="['size-3 stroke-white transition-opacity', checkedRows.has(item.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
                  </span>
                </label>
              </td>

              <!-- ID + дата -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top pl-3 pr-[22px] py-4 whitespace-nowrap">
                <div class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-normal text-indigo-600 hover:text-indigo-800 cursor-pointer transition-colors" @click.stop="$emit('open-preview', item)">{{ item.id }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">от {{ item.date }}</span>
                </div>
              </td>

              <!-- Статус -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-middle px-[22px] py-4 whitespace-nowrap">
                <span :class="statusBadgeClass(item.status)">{{ item.status }}</span>
              </td>

              <!-- Название -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-middle px-[22px] py-4 whitespace-nowrap">
                <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.name }}</span>
              </td>

              <!-- Менеджер -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-middle pl-[22px] pr-8 py-4 whitespace-nowrap" @click.stop>
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
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Пакеты не найдены</h3>
          <p class="mt-1 text-sm font-light text-zinc-500">Попробуйте изменить параметры поиска или фильтрации</p>
          <Button v-if="searchQuery.trim() && !props.filterCount" outline class="mt-6" @click="searchQuery = ''">Сбросить поиск</Button>
          <Button v-else-if="!searchQuery.trim() && props.filterCount > 0" outline class="mt-6" @click="$emit('reset-filters')">Сбросить фильтры</Button>
        </div>

      </div>
    </div>

  <SelectionBar :count="checkedRows.size" @clear="checkedRows = new Set()" @delete="checkedRows = new Set()" />
  </div>

</template>

<script setup>
import { ref, computed, reactive, watch } from 'vue'
import { Search as SearchIcon, SlidersHorizontal as FunnelIcon, ChevronDown as ChevronDownIcon } from 'lucide-vue-next'
import ExportPopover from './ExportPopover.vue'
import SearchInput from './SearchInput.vue'
import Button from './Button.vue'
import TableLink from './TableLink.vue'
import CatalystListbox from './CatalystListbox.vue'
import SelectionBar from './SelectionBar.vue'
import SortDropdown from './SortDropdown.vue'
import FilterTagsBar from './FilterTagsBar.vue'

const props = defineProps({
  activeSubItem: { type: String, default: '' },
  filterCount: { type: Number, default: 0 },
  filters: { type: Object, default: () => ({}) },
  filterTags: { type: Array, default: () => [] },
})
const emit = defineEmits(['open-filter', 'open-preview', 'update:count', 'reset-filters'])


const sortOrder = ref('new')
const isScrolled = ref(false)
const searchQuery = ref('')
const checkedRows = ref(new Set())

const toggleRow = (id) => {
  const s = new Set(checkedRows.value)
  s.has(id) ? s.delete(id) : s.add(id)
  checkedRows.value = s
}

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (status === 'Подписан')            return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'Готов к подписанию')  return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'Черновик')            return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const pluralize = (n) => {
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return `найдено ${n} пакетов`
  if (mod10 === 1) return `найден ${n} пакет`
  if (mod10 >= 2 && mod10 <= 4) return `найдено ${n} пакета`
  return `найдено ${n} пакетов`
}

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerSelections = reactive({})

const items = [
  { id: 'ПД 1150', date: '17.04', status: 'Подписан',           statusDate: '17.04', name: 'Ипотека Сбербанк — Новикова Е.Д.',      manager: 'Новиков Алексей' },
  { id: 'ПД 1149', date: '17.04', status: 'Подписан',           statusDate: '17.04', name: 'Семейная ипотека — Орлов Д.П.',          manager: 'Орлов Дмитрий'   },
  { id: 'ПД 1148', date: '16.04', status: 'Черновик',           statusDate: null,    name: 'Льготная ипотека — Смирнова Ю.А.',       manager: 'Смирнова Юлия'   },
  { id: 'ПД 1147', date: '08.04', status: 'Готов к подписанию', statusDate: null,    name: 'Рефинансирование — Лебедев И.Н.',        manager: 'Лебедев Игорь'   },
  { id: 'ПД 1146', date: '21.01', status: 'Готов к подписанию', statusDate: null,    name: 'Ипотека ВТБ — Воронова А.С.',           manager: 'Воронова Анна'   },
  { id: 'ПД 1145', date: '15.01', status: 'Подписан',           statusDate: '16.01', name: 'Договор долевого участия — Могиль М.В.', manager: 'Новиков Алексей' },
  { id: 'ПД 1144', date: '10.01', status: 'Черновик',           statusDate: null,    name: 'IT-ипотека — Андреев Р.Е.',             manager: 'Морозов Сергей'  },
  { id: 'ПД 1143', date: '05.01', status: 'Готов к подписанию', statusDate: null,    name: 'Ипотека Альфа-Банк — Морозов С.В.',     manager: 'Смирнова Юлия'  },
]

items.forEach(item => { managerSelections[item.id] = item.manager })

const openApp = (item) => { window.open(`/app-page.html?id=${item.id}&title=${encodeURIComponent('Это страница пакета')}`, '_blank') }

const filteredItems = computed(() => {
  let result = items
  if (props.activeSubItem === 'Мои') result = result.filter(item => item.manager === 'Новиков Алексей')
  if (props.filters?.statuses?.size > 0) result = result.filter(item => props.filters.statuses.has(item.status))
  if (props.filters?.managers?.size > 0) result = result.filter(item => props.filters.managers.has(item.manager))
  if (sortOrder.value === 'old') result = [...result].reverse()
  const q = searchQuery.value.trim().toLowerCase()
  if (q) result = result.filter(item => item.id.toLowerCase().includes(q) || item.name.toLowerCase().includes(q))
  return result
})
watch(filteredItems, v => emit('update:count', v.length), { immediate: true })
</script>
