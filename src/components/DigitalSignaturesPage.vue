<template>
  <div class="relative flex h-full flex-col overflow-hidden">

    <!-- Поиск + фильтры -->
    <div :class="['shrink-0 flex items-center justify-between px-8 py-2', isScrolled && !(filterTags && filterTags.length) && 'border-b border-[#f4f4f5]']">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ФИО, телефону или названию центра" />
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
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top pt-[16px] pb-3 pl-8 pr-2" @click.stop="toggleRow(item.id)">
                <label class="inline-flex cursor-pointer">
                  <span :class="['relative flex size-4 items-center justify-center rounded-sm border', checkedRows.has(item.id) ? 'bg-indigo-600 border-transparent' : 'bg-white border-zinc-950/15 hover:border-zinc-950/30']">
                    <svg :class="['size-3 stroke-white transition-opacity', checkedRows.has(item.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
                  </span>
                </label>
              </td>

              <!-- ID + дата -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top pl-3 pr-[22px] py-4 whitespace-nowrap">
                <div class="flex flex-col gap-y-1">
                  <TableLink @click="$emit('open-preview', item)">{{ item.id }}</TableLink>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">от {{ item.date }}</span>
                </div>
              </td>

              <!-- Аватар + ФИО + Телефон -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                <div class="flex items-start gap-x-3">
                  <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] font-medium text-[#71717a]">{{ item.initials }}</span>
                  <div class="flex flex-col gap-y-1">
                    <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.client }}</span>
                    <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.phone }}</span>
                  </div>
                </div>
              </td>

              <!-- Статус -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                <span :class="statusBadgeClass(item.status)">{{ item.status }}</span>
              </td>

              <!-- Способ идентификации + Название центра -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                <div class="flex flex-col gap-y-1">
                  <span class="inline-flex items-center gap-x-1.5 text-[14px] leading-[20px] font-light text-zinc-900">
                    <component
                      :is="item.signMethod === 'Офис' ? OfficeIcon : item.signMethod === 'Курьер' ? CourierIcon : PassportIcon"
                      :size="16"
                      class="shrink-0 text-zinc-900"
                    />
                    {{ item.signMethod }}
                  </span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.center }}</span>
                </div>
              </td>

              <!-- Встреча с курьером -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                <div v-if="item.signMethod === 'Курьер' && item.courierMeeting" class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">Встреча</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.courierMeeting }}</span>
                </div>
              </td>

              <!-- Дата активации + Дата истечения -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                <div v-if="item.status === 'Активирована'" class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">С {{ formatFull(item.activationDate) }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-500">Действует {{ item.duration }}</span>
                </div>
              </td>

              <!-- Менеджер -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top pl-[22px] pr-8 py-4 whitespace-nowrap" @click.stop>
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

        <!-- Empty state — список пуст -->
        <div v-if="filteredItems.length === 0 && !searchQuery.trim() && !props.filterCount" class="flex flex-col items-center justify-center py-24 text-center">
          <FilePlusCornerIcon :size="48" class="mx-auto text-zinc-400" />
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Пока здесь пусто</h3>
          <p class="mt-1 text-sm font-light text-zinc-500">В этом списке пока нет заявок</p>
          <Button outline class="mt-6" @click="emit('create')">
            <svg class="size-4" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true"><path d="M10.75 4.75a.75.75 0 00-1.5 0v4.5h-4.5a.75.75 0 000 1.5h4.5v4.5a.75.75 0 001.5 0v-4.5h4.5a.75.75 0 000-1.5h-4.5v-4.5z" /></svg>
            Создать
          </Button>
        </div>

        <!-- Empty state -->
        <div v-if="filteredItems.length === 0 && (searchQuery.trim() || props.filterCount > 0)" class="flex flex-col items-center justify-center py-24 text-center">
          <FileSearchCornerIcon :size="48" class="mx-auto text-zinc-400" />
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
import { Search as SearchIcon, SlidersHorizontal as FunnelIcon, ChevronDown as ChevronDownIcon, Landmark as OfficeIcon, UserRound as CourierIcon, IdCard as PassportIcon, FilePlusCorner as FilePlusCornerIcon, FileSearchCorner as FileSearchCornerIcon } from 'lucide-vue-next'
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
  filters: { type: Object, default: null },
  filterTags: { type: Array, default: () => [] },
})
const emit = defineEmits(['open-filter', 'open-preview', 'update:count', 'reset-filters', 'create'])


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
  if (status === 'Активирована')                  return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'Процесс выпуска создан')         return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
  if (status === 'Встреча с курьером состоялась')  return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
  if (status === 'Ожидает назначение курьера')     return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'Встреча с курьером назначена')   return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'Встреча с курьером отменена')    return `${base} bg-red-50 text-red-700 inset-ring-red-600/20`
  if (status === 'Запрос на выпуск истек')         return `${base} bg-red-50 text-red-700 inset-ring-red-600/20`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const months = ['января','февраля','марта','апреля','мая','июня','июля','августа','сентября','октября','ноября','декабря']

const formatShort = (d) => {
  const [day, month] = d.split('.')
  return `${parseInt(day)} ${months[parseInt(month) - 1]}`
}

const formatFull = (d) => {
  const [day, month, year] = d.split('.')
  return `${parseInt(day)} ${months[parseInt(month) - 1]} ${year}`
}

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerSelections = reactive({})

const items = [
  { id: 'ЦП 3047', date: '12.05', initials: 'НЕ', client: 'Новикова Елена',  phone: '+7 916 ***-**-09', fullPhone: '+7 916 234-11-09', center: 'SignMe',  signMethod: 'Офис',    activationDate: '12.05.2026', expiryDate: '12.05.2027', duration: '1 год',    courierDate: null,           status: 'Активирована',                   manager: 'Смирнова Юлия'  },
  { id: 'ЦП 3046', date: '11.05', initials: 'ОД', client: 'Орлов Дмитрий',   phone: '+7 903 ***-**-72', fullPhone: '+7 903 811-45-72', center: 'Контур',  signMethod: 'Курьер',  activationDate: '11.05.2026', expiryDate: '11.11.2027', duration: '1,5 года', courierDate: '18.05.2026', courierMeeting: '18 мая с 10 до 15', status: 'Встреча с курьером назначена',   manager: 'Орлов Дмитрий'  },
  { id: 'ЦП 3045', date: '10.05', initials: 'СЮ', client: 'Смирнова Юлия',   phone: '+7 925 ***-**-18', fullPhone: '+7 925 007-33-18', center: 'SignMe',  signMethod: 'Офис',    activationDate: '10.05.2026', expiryDate: '10.05.2027', duration: '1 год',    courierDate: null,           status: 'Процесс выпуска создан',         manager: 'Лебедев Игорь'  },
  { id: 'ЦП 3044', date: '07.05', initials: 'ЛИ', client: 'Лебедев Игорь',   phone: '+7 499 ***-**-55', fullPhone: '+7 499 123-00-55', center: 'Тензор',  signMethod: 'Паспорт', activationDate: '07.05.2026', expiryDate: '07.11.2027', duration: '1,5 года', courierDate: '14 мая 2026',  courierMeeting: '14 мая с 12 до 17', status: 'Ожидает назначение курьера',     manager: 'Воронова Анна'  },
  { id: 'ЦП 3043', date: '05.05', initials: 'ВА', client: 'Воронова Анна',    phone: '+7 916 ***-**-77', fullPhone: '+7 916 540-29-77', center: 'Контур',  signMethod: 'Офис',    activationDate: '05.05.2026', expiryDate: '05.05.2027', duration: '1 год',    courierDate: '16 мая 2026',  courierMeeting: '16 мая с 9 до 14', status: 'Встреча с курьером назначена',   manager: 'Смирнова Юлия'  },
  { id: 'ЦП 3042', date: '04.05', initials: 'МС', client: 'Морозов Сергей',   phone: '+7 912 ***-**-88', fullPhone: '+7 912 300-14-88', center: 'SignMe',  signMethod: 'Курьер',  activationDate: '04.05.2026', expiryDate: '04.11.2027', duration: '1,5 года', courierDate: '20.05.2026', courierMeeting: '20 мая с 10 до 14', status: 'Встреча с курьером назначена',   manager: 'Орлов Дмитрий'  },
  { id: 'ЦП 3041', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария',  phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', center: 'Тензор',  signMethod: 'Паспорт', activationDate: '01.05.2026', expiryDate: '01.05.2027', duration: '1 год',    courierDate: '19 мая 2026',  courierMeeting: '19 мая с 11 до 16', status: 'Встреча с курьером состоялась',  manager: 'Морозов Сергей' },
  { id: 'ЦП 3040', date: '28.04', initials: 'АП', client: 'Алексеев Пётр',    phone: '+7 915 ***-**-31', fullPhone: '+7 915 462-55-31', center: 'Контур',  signMethod: 'Курьер',  activationDate: '28.04.2026', expiryDate: '28.10.2027', duration: '1,5 года', courierDate: '25.05.2026', courierMeeting: '25 мая с 11 до 15', status: 'Встреча с курьером назначена',   manager: 'Лебедев Игорь'  },
  { id: 'ЦП 3039', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',   phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', center: 'SignMe',  signMethod: 'Офис',    activationDate: '25.04.2026', expiryDate: '25.04.2027', duration: '1 год',    courierDate: null,           status: 'Активирована',                   manager: 'Воронова Анна'  },
  { id: 'ЦП 3038', date: '22.04', initials: 'ТВ', client: 'Тихонов Виктор',   phone: '+7 903 ***-**-45', fullPhone: '+7 903 159-60-45', center: 'Тензор',  signMethod: 'Паспорт', activationDate: '22.04.2026', expiryDate: '22.10.2027', duration: '1,5 года', courierDate: null,           status: 'Активирована',                   manager: 'Смирнова Юлия'  },
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

const parseCourierDate = (str) => {
  if (!str) return null
  const m = { 'января':0,'февраля':1,'марта':2,'апреля':3,'мая':4,'июня':5,'июля':6,'августа':7,'сентября':8,'октября':9,'ноября':10,'декабря':11 }
  const [day, month, year] = str.split(' ')
  return new Date(+year, m[month], +day)
}

const inRange = (date, from, to) => {
  if (!date) return !from && !to
  if (from && date < from) return false
  if (to) { const end = new Date(to); end.setHours(23, 59, 59); if (date > end) return false }
  return true
}

const pd = (s) => { // parse filter date string dd.mm.yyyy
  if (!s) return null
  const p = s.split('.')
  return p.length === 3 ? new Date(+p[2], +p[1] - 1, +p[0]) : null
}

const filteredItems = computed(() => {
  let result = [...items]

  if (props.activeSubItem === 'Мои заявки')
    result = result.filter(i => i.manager === 'Смирнова Юлия')

  const q = searchQuery.value.trim().toLowerCase()
  if (q) result = result.filter(i =>
    i.id.toLowerCase().includes(q) ||
    i.client.toLowerCase().includes(q) ||
    i.phone.replace(/\s/g, '').includes(q.replace(/\s/g, '')) ||
    i.center.toLowerCase().includes(q)
  )

  const f = props.filters
  if (f) {
    if (f.statuses?.length)  result = result.filter(i => f.statuses.includes(i.status))
    if (f.managers?.length)  result = result.filter(i => f.managers.includes(i.manager))
    if (f.methods?.length)   result = result.filter(i => f.methods.includes(i.signMethod))

    const df = pd(f.dateFrom), dt = pd(f.dateTo)
    if (df || dt) result = result.filter(i => inRange(parseItemDate(i.date), df, dt))

    const af = pd(f.activationFrom), at = pd(f.activationTo)
    if (af || at) result = result.filter(i => inRange(parseItemDate(i.activationDate), af, at))

    const ef = pd(f.expiryFrom), et = pd(f.expiryTo)
    if (ef || et) result = result.filter(i => inRange(parseItemDate(i.expiryDate), ef, et))

    const cf = pd(f.courierFrom), ct = pd(f.courierTo)
    if (cf || ct) result = result.filter(i => inRange(parseCourierDate(i.courierDate), cf, ct))

    result.sort((a, b) => {
      const na = parseInt(a.id.split(' ')[1]), nb = parseInt(b.id.split(' ')[1])
      return sortOrder.value === 'old' ? na - nb : nb - na
    })
  }

  return result
})
watch(filteredItems, v => emit('update:count', v.length), { immediate: true })
</script>
