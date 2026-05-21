<template>
  <div class="relative flex h-full flex-col overflow-hidden">

    <!-- Поиск + фильтры -->
    <div :class="['shrink-0 flex items-center justify-between px-8 py-2', isScrolled && !(filterTags && filterTags.length) && 'border-b border-[#f4f4f5]']">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ФИО, телефону или названию страховой" />
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

              <!-- Страховщик + Тип страховки -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
                <div class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ item.insurer }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.insuranceType }}</span>
                </div>
              </td>

              <!-- Цена + КВ -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
                <div class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ item.price }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">КВ {{ item.kv }}</span>
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
  filterTags: { type: Array, default: () => [] },
  filters: { type: Object, default: null },
})
const emit = defineEmits(['open-filter', 'open-preview', 'update:count', 'reset-filters'])


const sortOrder = ref('new')
const isScrolled = ref(false)
const searchQuery = ref('')
const checkedRows = ref(new Set())

const pluralize = (n) => {
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return `найдено ${n} заявок`
  if (mod10 === 1) return `найдена ${n} заявка`
  if (mod10 >= 2 && mod10 <= 4) return `найдено ${n} заявки`
  return `найдено ${n} заявок`
}

const toggleRow = (id) => {
  const s = new Set(checkedRows.value)
  s.has(id) ? s.delete(id) : s.add(id)
  checkedRows.value = s
}

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (status === 'Оформлено')    return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'На оформлении') return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'В работе')     return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerSelections = reactive({})

const items = [
  { id: 'СТ 1047', date: '12.05', initials: 'НЕ', client: 'Новикова Елена',  phone: '+7 916 ***-**-09', fullPhone: '+7 916 234-11-09', company: 'ООО «Ипотека»',     insurer: 'Сбербанк Страхование', insuranceType: 'Имущество + Жизнь', address: 'ул. Полярная, 14/2',         price: '14 200 ₽', kv: '1 420 ₽',  status: 'Оформлено',    manager: 'Смирнова Юлия'  },
  { id: 'СТ 1046', date: '11.05', initials: 'ОД', client: 'Орлов Дмитрий',   phone: '+7 903 ***-**-72', fullPhone: '+7 903 811-45-72', company: 'ООО «ДомФинанс»', insurer: 'ВТБ Страхование',      insuranceType: 'Имущество',         address: 'пр. Ленина, 88, кв. 34',     price: '7 500 ₽',  kv: '525 ₽',    status: 'На оформлении', manager: 'Орлов Дмитрий'  },
  { id: 'СТ 1045', date: '10.05', initials: 'СЮ', client: 'Смирнова Юлия',   phone: '+7 925 ***-**-18', fullPhone: '+7 925 007-33-18', company: 'ООО «Ипотека»',     insurer: 'Альфастрахование',     insuranceType: 'Жизнь',             address: 'ул. Берёзовая, 5, кв. 12',   price: '9 800 ₽',  kv: '980 ₽',    status: 'В работе',     manager: 'Лебедев Игорь'  },
  { id: 'СТ 1044', date: '07.05', initials: 'ЛИ', client: 'Лебедев Игорь',   phone: '+7 499 ***-**-55', fullPhone: '+7 499 123-00-55', company: 'ООО «СтройКредит»', insurer: 'Ингосстрах',           insuranceType: 'Имущество + Жизнь', address: 'ул. Советская, 3, кв. 201',  price: '18 600 ₽', kv: '2 232 ₽',  status: 'Оформлено',    manager: 'Воронова Анна'  },
  { id: 'СТ 1043', date: '05.05', initials: 'ВА', client: 'Воронова Анна',    phone: '+7 916 ***-**-77', fullPhone: '+7 916 540-29-77', company: 'ООО «Ипотека»',     insurer: 'Росгосстрах',          insuranceType: 'Имущество',         address: 'ул. Светлая, 22, кв. 7',     price: '6 300 ₽',  kv: '378 ₽',    status: 'На оформлении', manager: 'Смирнова Юлия'  },
  { id: 'СТ 1042', date: '04.05', initials: 'МС', client: 'Морозов Сергей',   phone: '+7 912 ***-**-88', fullPhone: '+7 912 300-14-88', company: 'ООО «ДомФинанс»', insurer: 'Сбербанк Страхование', insuranceType: 'Жизнь',             address: 'пр. Мира, 101, кв. 56',      price: '11 100 ₽', kv: '1 110 ₽',  status: 'Оформлено',    manager: 'Орлов Дмитрий'  },
  { id: 'СТ 1041', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария',  phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', company: 'ООО «РегЦентр»',   insurer: 'ВТБ Страхование',      insuranceType: 'Имущество + Жизнь', address: 'ул. Садовая, 9, кв. 3',      price: '15 400 ₽', kv: '1 848 ₽',  status: 'В работе',     manager: 'Морозов Сергей' },
  { id: 'СТ 1040', date: '28.04', initials: 'АП', client: 'Алексеев Пётр',    phone: '+7 915 ***-**-31', fullPhone: '+7 915 462-55-31', company: 'ООО «Ипотека»',     insurer: 'Альфастрахование',     insuranceType: 'Имущество',         address: 'ул. Новая, 15, кв. 88',      price: '8 900 ₽',  kv: '712 ₽',    status: 'Оформлено',    manager: 'Лебедев Игорь'  },
  { id: 'СТ 1039', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',   phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', company: 'ООО «СтройКредит»', insurer: 'Ингосстрах',           insuranceType: 'Жизнь',             address: 'наб. Невская, 4, кв. 19',    price: '10 200 ₽', kv: '1 275 ₽',  status: 'На оформлении', manager: 'Воронова Анна'  },
  { id: 'СТ 1038', date: '22.04', initials: 'ТВ', client: 'Тихонов Виктор',   phone: '+7 903 ***-**-45', fullPhone: '+7 903 159-60-45', company: 'ООО «РегЦентр»',   insurer: 'Росгосстрах',          insuranceType: 'Имущество + Жизнь', address: 'ул. Промышленная, 7, кв. 2', price: '13 700 ₽', kv: '1 644 ₽',  status: 'Оформлено',    manager: 'Смирнова Юлия'  },
]

items.forEach(item => { managerSelections[item.id] = item.manager })

const openApp = (item) => {
  window.open(`/app-page.html?id=${item.id}&title=${encodeURIComponent('Это страница заявки')}`, '_blank')
}

const parseItemDate = (str) => {
  if (!str) return null
  const p = str.split('.')
  if (p.length === 3) return new Date(+p[2], +p[1] - 1, +p[0])
  if (p.length === 2) return new Date(2026, +p[1] - 1, +p[0])
  return null
}
const inRange = (date, from, to) => {
  if (!date) return !from && !to
  if (from && date < from) return false
  if (to) { const end = new Date(to); end.setHours(23, 59, 59); if (date > end) return false }
  return true
}
const pd = (s) => {
  if (!s) return null
  const p = s.split('.')
  return p.length === 3 ? new Date(+p[2], +p[1] - 1, +p[0]) : null
}

const filteredItems = computed(() => {
  let result = [...items]
  if (props.activeSubItem === 'Мои заявки') result = result.filter(i => i.manager === 'Смирнова Юлия')
  const q = searchQuery.value.trim().toLowerCase()
  if (q) result = result.filter(i =>
    i.id.toLowerCase().includes(q) ||
    i.client.toLowerCase().includes(q) ||
    i.address.toLowerCase().includes(q) ||
    i.insurer.toLowerCase().includes(q) ||
    i.company.toLowerCase().includes(q)
  )
  const f = props.filters
  if (f) {
    if (f.statuses?.length)  result = result.filter(i => f.statuses.includes(i.status))
    if (f.managers?.length)  result = result.filter(i => f.managers.includes(i.manager))
    if (f.types?.length)     result = result.filter(i => f.types.includes(i.insuranceType))
    if (f.insurers?.length)  result = result.filter(i => f.insurers.includes(i.insurer))
    const df = pd(f.dateFrom), dt = pd(f.dateTo)
    if (df || dt) result = result.filter(i => inRange(parseItemDate(i.date), df, dt))
  }
  result.sort((a, b) => {
    const na = parseInt(a.id.split(' ')[1]), nb = parseInt(b.id.split(' ')[1])
    return sortOrder.value === 'old' ? na - nb : nb - na
  })
  return result
})
watch(filteredItems, v => emit('update:count', v.length), { immediate: true })
</script>
