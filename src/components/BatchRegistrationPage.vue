<template>
  <div class="relative flex h-full flex-col overflow-hidden">

    <!-- Поиск + фильтры -->
    <div :class="['shrink-0 flex items-center justify-between px-8 py-2', isScrolled && !(filterTags && filterTags.length) && 'border-b border-[#f4f4f5]']">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ИНН или названию пакета" />
      <div class="flex items-center gap-x-3">
        <span v-if="searchQuery.trim()" class="text-[14px] font-light text-[#71717a] mr-5">
          {{ pluralize(totalCount) }}
        </span>
        <Button outline @click="$emit('open-filter')">
          <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
          Фильтры
          <span v-if="props.filterCount > 0" class="inline-flex items-center justify-center size-[18px] rounded-full bg-indigo-600 text-white text-[11px] font-medium leading-none">{{ props.filterCount }}</span>
        </Button>
        <SortDropdown v-model="sortOrder" />
        <ExportPopover :count="totalCount" />
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
            <template v-for="(group, gi) in groupedFilteredItems" :key="group.packageName">

              <!-- Отступ между группами -->
              <tr v-if="gi > 0" aria-hidden="true">
                <td colspan="9" class="pt-4" />
              </tr>

              <!-- Заголовок группы -->
              <tr>
                <td
                  colspan="8"
                  class="sticky left-0 z-10 bg-white pb-1 pl-8"
                >
                  <div class="flex items-center gap-x-[18px] cursor-pointer" @click="toggleGroup(group.packageName)">
                    <ChevronUpIcon
                      :size="18"
                      :class="['shrink-0 text-zinc-400 transition-transform duration-200', collapsedGroups.has(group.packageName) ? 'rotate-90' : '']"
                    />
                    <div class="flex items-center gap-x-2">
                      <span class="text-[14px] font-medium text-zinc-900">{{ group.packageName }}</span>
                      <span class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">{{ pluralizeApps(group.items.length) }}</span>
                    </div>
                  </div>
                </td>
                <td class="sticky right-0 z-10 bg-white pb-1 pr-8 text-right whitespace-nowrap">
                  <Button plain color="zinc" @click.stop="deletePackage(group.packageName)">
                    <Trash2Icon :size="14" class="shrink-0" />
                    Удалить
                  </Button>
                </td>
              </tr>

              <!-- Строки заявок -->
              <template v-if="!collapsedGroups.has(group.packageName)">
                <tr
                  v-for="item in group.items"
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

                  <!-- Аватар + ФИО + Телефон / Юрлицо -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <div class="flex items-start gap-x-3">
                      <template v-if="item.isCompany">
                        <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[#71717a]">
                          <LandmarkIcon :size="16" />
                        </span>
                        <div class="flex flex-col gap-y-1">
                          <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.companyName }}</span>
                          <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.inn }}</span>
                        </div>
                      </template>
                      <template v-else>
                        <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] font-medium text-[#71717a]">{{ item.initials }}</span>
                        <div class="flex flex-col gap-y-1">
                          <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.client }}</span>
                          <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.phone }}</span>
                        </div>
                      </template>
                    </div>
                  </td>

                  <!-- Статус -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <span :class="statusBadgeClass(item.status)">{{ item.status }}</span>
                  </td>

                  <!-- Компания + Проект -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <div class="flex flex-col gap-y-1">
                      <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ item.project }}</span>
                      <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.company }}</span>
                    </div>
                  </td>

                  <!-- Тип недвижимости + Адрес -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4">
                    <div class="flex flex-col gap-y-1" style="width: 400px">
                      <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ item.propertyType }}</span>
                      <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.address }}</span>
                    </div>
                  </td>

                  <!-- РР + КУВД -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <div v-if="item.status === 'Зарегистрировано'" class="flex flex-col gap-y-1">
                      <span class="text-[14px] leading-[20px] font-light text-[#18181b]">РР {{ item.rr }}</span>
                      <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.kuvd }}</span>
                    </div>
                  </td>

                  <!-- Дата отправки + Регистрация -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <div v-if="item.status === 'Зарегистрировано'" class="flex flex-col gap-y-1">
                      <span class="text-[14px] leading-[20px] font-light text-[#18181b]">Отправка {{ item.sendDate }}</span>
                      <span class="text-[14px] leading-[20px] font-light text-zinc-900">Регистрация {{ item.regDate }}</span>
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
              </template>

            </template>
          </tbody>
        </table>

        <!-- Empty state -->
        <div v-if="totalCount === 0 && (searchQuery.trim() || props.filterCount > 0)" class="flex flex-col items-center justify-center py-24 text-center">
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
import { SlidersHorizontal as FunnelIcon, ChevronUp as ChevronUpIcon, Landmark as LandmarkIcon, Trash2 as Trash2Icon } from 'lucide-vue-next'
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
const deletedPackages = ref(new Set())
const searchQuery = ref('')
const checkedRows = ref(new Set())
const collapsedGroups = ref(new Set())

const toggleGroup = (name) => {
  const s = new Set(collapsedGroups.value)
  s.has(name) ? s.delete(name) : s.add(name)
  collapsedGroups.value = s
}

const pluralize = (n) => {
  const m10 = n % 10, m100 = n % 100
  if (m100 >= 11 && m100 <= 14) return `найдено ${n} заявок`
  if (m10 === 1) return `найдена ${n} заявка`
  if (m10 >= 2 && m10 <= 4) return `найдено ${n} заявки`
  return `найдено ${n} заявок`
}

const pluralizeApps = (n) => {
  const m10 = n % 10, m100 = n % 100
  if (m100 >= 11 && m100 <= 14) return `${n} заявок`
  if (m10 === 1) return `${n} заявка`
  if (m10 >= 2 && m10 <= 4) return `${n} заявки`
  return `${n} заявок`
}

const deletePackage = (packageName) => {
  deletedPackages.value = new Set([...deletedPackages.value, packageName])
  const s = new Set(checkedRows.value)
  items.filter(i => i.packageName === packageName).forEach(i => s.delete(i.id))
  checkedRows.value = s
}

const toggleRow = (id) => {
  const s = new Set(checkedRows.value)
  s.has(id) ? s.delete(id) : s.add(id)
  checkedRows.value = s
}

const isGroupChecked = (packageName) => {
  const group = groupedFilteredItems.value.find(g => g.packageName === packageName)
  return group && group.items.length > 0 && group.items.every(item => checkedRows.value.has(item.id))
}

const isGroupIndeterminate = (packageName) => {
  const group = groupedFilteredItems.value.find(g => g.packageName === packageName)
  if (!group) return false
  const checked = group.items.filter(item => checkedRows.value.has(item.id)).length
  return checked > 0 && checked < group.items.length
}

const toggleGroupRows = (packageName) => {
  const group = groupedFilteredItems.value.find(g => g.packageName === packageName)
  if (!group) return
  const s = new Set(checkedRows.value)
  if (isGroupChecked(packageName)) {
    group.items.forEach(item => s.delete(item.id))
  } else {
    group.items.forEach(item => s.add(item.id))
  }
  checkedRows.value = s
}

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (status === 'Зарегистрировано') return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'На регистрации')   return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'В работе')         return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerSelections = reactive({})

const items = [
  // Пакет ЖК «Северный» — покупатель ООО «Старт», продавец ООО «СеверСтрой»
  { id: 'РГ 2047', date: '12.05', isCompany: true, companyName: 'ООО «Старт»', inn: '7701234567', company: 'ООО «СеверСтрой»', project: 'ЖК «Северный»', propertyType: 'Квартира',          address: 'Москва, Северный АО, ул. Полярная, д. 14, корп. 2, кв. 1',                          status: 'Зарегистрировано', regDate: '14 мая',    rr: '77:09:0002001:1234', kuvd: 'КУВД-001/2026-47', sendDate: '13 мая',    manager: 'Смирнова Юлия',  packageName: 'Пакет ЖК «Северный»' },
  { id: 'РГ 2046', date: '12.05', isCompany: true, companyName: 'ООО «Старт»', inn: '7701234567', company: 'ООО «СеверСтрой»', project: 'ЖК «Северный»', propertyType: 'Апартаменты',       address: 'Москва, Северный АО, ул. Полярная, д. 14, корп. 2, апарт. 34',                      status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Орлов Дмитрий',  packageName: 'Пакет ЖК «Северный»' },
  { id: 'РГ 2045', date: '12.05', isCompany: true, companyName: 'ООО «Старт»', inn: '7701234567', company: 'ООО «СеверСтрой»', project: 'ЖК «Северный»', propertyType: 'Квартира',          address: 'Москва, Северный АО, ул. Полярная, д. 14, корп. 2, кв. 12',                         status: 'В работе',         regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Лебедев Игорь',  packageName: 'Пакет ЖК «Северный»' },
  // Пакет ЖК «Западный» — покупатель АО «РегионСтрой», продавец АО «ЗападДевелопмент»
  { id: 'РГ 2044', date: '07.05', isCompany: true, companyName: 'АО «РегионСтрой»', inn: '5034891203', company: 'АО «ЗападДевелопмент»', project: 'ЖК «Западный»', propertyType: 'Таунхаус',  address: 'Московская область, г. Красногорск, Западный мкр., пр. Западный, д. 3, кв. 201',    status: 'Зарегистрировано', regDate: '9 мая',     rr: '77:01:0003002:5678', kuvd: 'КУВД-001/2026-44', sendDate: '8 мая',     manager: 'Воронова Анна',  packageName: 'Пакет ЖК «Западный»' },
  { id: 'РГ 2043', date: '07.05', isCompany: true, companyName: 'АО «РегионСтрой»', inn: '5034891203', company: 'АО «ЗападДевелопмент»', project: 'ЖК «Западный»', propertyType: 'Квартира',   address: 'Московская область, г. Красногорск, Западный мкр., пр. Западный, д. 3, кв. 7',      status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Смирнова Юлия',  packageName: 'Пакет ЖК «Западный»' },
  { id: 'РГ 2042', date: '07.05', isCompany: true, companyName: 'АО «РегионСтрой»', inn: '5034891203', company: 'АО «ЗападДевелопмент»', project: 'ЖК «Западный»', propertyType: 'Квартира',   address: 'Московская область, г. Красногорск, Западный мкр., пр. Западный, д. 3, кв. 56',     status: 'Зарегистрировано', regDate: '6 мая',     rr: '77:06:0001003:9012', kuvd: 'КУВД-001/2026-42', sendDate: '5 мая',     manager: 'Орлов Дмитрий',  packageName: 'Пакет ЖК «Западный»' },
  // Пакет ЖК «Уютный» — покупатель Кузнецова Мария, продавец ООО «УютСтрой»
  { id: 'РГ 2041', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария', phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', company: 'ООО «УютСтрой»', project: 'ЖК «Уютный»', propertyType: 'Машино-место', address: 'Москва, ЗАО, Очаково-Матвеевское, ул. Уютная, д. 9, м/м 3',                        status: 'В работе',         regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Морозов Сергей', packageName: 'Пакет ЖК «Уютный»' },
  { id: 'РГ 2040', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария', phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', company: 'ООО «УютСтрой»', project: 'ЖК «Уютный»', propertyType: 'Квартира',     address: 'Москва, ЗАО, Очаково-Матвеевское, ул. Уютная, д. 15, кв. 88',                       status: 'Зарегистрировано', regDate: '30 апреля', rr: '77:03:0004004:3456', kuvd: 'КУВД-001/2026-40', sendDate: '29 апреля', manager: 'Лебедев Игорь',  packageName: 'Пакет ЖК «Уютный»' },
  // Пакет ЖК «Речной» — покупатель Захарова Ольга, продавец ООО «РечСтрой»
  { id: 'РГ 2039', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',   phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', company: 'ООО «РечСтрой»',  project: 'ЖК «Речной»', propertyType: 'Апартаменты',       address: 'Санкт-Петербург, Невский р-н, наб. Невская, д. 4, апарт. 19',                       status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Воронова Анна',  packageName: 'Пакет ЖК «Речной»' },
  { id: 'РГ 2038', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',   phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', company: 'ООО «РечСтрой»',  project: 'ЖК «Речной»', propertyType: 'Нежилое помещение', address: 'Санкт-Петербург, Невский р-н, наб. Невская, д. 4, пом. 2',                          status: 'Зарегистрировано', regDate: '24 апреля', rr: '77:22:0005005:7890', kuvd: 'КУВД-001/2026-38', sendDate: '23 апреля', manager: 'Смирнова Юлия',  packageName: 'Пакет ЖК «Речной»' },
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
  let result = [...items].filter(i => !deletedPackages.value.has(i.packageName))
  if (props.activeSubItem === 'Мои заявки') result = result.filter(i => i.manager === 'Смирнова Юлия')
  const q = searchQuery.value.trim().toLowerCase()
  if (q) result = result.filter(i =>
    i.id.toLowerCase().includes(q) ||
    (i.client && i.client.toLowerCase().includes(q)) ||
    (i.phone && i.phone.replace(/\s/g, '').includes(q.replace(/\s/g, ''))) ||
    (i.companyName && i.companyName.toLowerCase().includes(q)) ||
    (i.inn && i.inn.includes(q)) ||
    i.address.toLowerCase().includes(q) ||
    i.project.toLowerCase().includes(q) ||
    i.company.toLowerCase().includes(q) ||
    i.packageName.toLowerCase().includes(q) ||
    (i.rr && i.rr.toLowerCase().includes(q)) ||
    (i.kuvd && i.kuvd.toLowerCase().includes(q))
  )
  const f = props.filters
  if (f) {
    if (f.statuses?.length) result = result.filter(i => f.statuses.includes(i.status))
    if (f.managers?.length) result = result.filter(i => f.managers.includes(i.manager))
    const df = pd(f.dateFrom), dt = pd(f.dateTo)
    if (df || dt) result = result.filter(i => inRange(parseItemDate(i.date), df, dt))
  }
  result.sort((a, b) => {
    const na = parseInt(a.id.split(' ')[1]), nb = parseInt(b.id.split(' ')[1])
    return sortOrder.value === 'old' ? na - nb : nb - na
  })
  return result
})

// Группировка по packageName с сохранением порядка групп
const groupedFilteredItems = computed(() => {
  const map = new Map()
  for (const item of filteredItems.value) {
    if (!map.has(item.packageName)) map.set(item.packageName, [])
    map.get(item.packageName).push(item)
  }
  return [...map.entries()].map(([packageName, items]) => ({ packageName, items }))
})

const totalCount = computed(() => filteredItems.value.length)
const packageCount = computed(() => groupedFilteredItems.value.length)

watch(packageCount, v => emit('update:count', v), { immediate: true })
</script>
