<template>
  <div class="relative flex h-full flex-col overflow-hidden">

    <!-- Поиск + фильтры -->
    <div :class="['shrink-0 flex items-center justify-between px-8 py-2', isScrolled && !(filterTags && filterTags.length) && 'border-b border-[#f4f4f5]']">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ФИО, телефону, ИНН или номерам регистрации" />
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
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top pt-[16px] pb-3 pl-8 pr-2" @click.stop="toggleRow(item.id)">
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

              <!-- Компания + Проект -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4">
                <div class="flex flex-col gap-y-1" style="width: 400px">
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ item.project }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.company }}</span>
                </div>
              </td>

              <!-- Тип недвижимости + Адрес -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4">
                <div class="flex flex-col gap-y-1" style="width: 400px">
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ item.propertyType }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.address }}</span>
                </div>
              </td>

              <!-- РР + КУВД -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
                <div v-if="item.status === 'Зарегистрировано'" class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">РР {{ item.rr }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.kuvd }}</span>
                </div>
              </td>

              <!-- Дата отправки в РР + Регистрация -->
              <td class="group-hover:bg-zinc-50 [.selected_&]:bg-indigo-50 align-top px-[22px] py-4 whitespace-nowrap">
                <div v-if="item.status === 'Зарегистрировано'" class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-[#18181b]">Отправка {{ item.sendDate }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">Регистрация {{ item.regDate }}</span>
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
  if (status === 'Зарегистрировано') return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'На регистрации')   return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'В работе')         return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerSelections = reactive({})

const items = [
  { id: 'РГ 2047', date: '12.05', initials: 'НЕ', client: 'Новикова Елена',   phone: '+7 916 ***-**-09', fullPhone: '+7 916 234-11-09', company: 'ООО «Ипотека»',       project: 'ЖК «Северный»',    propertyType: 'Квартира',          address: 'Москва, ул. Полярная, д. 14, корп. 2, кв. 78',                                    status: 'Зарегистрировано', regDate: '14 мая',    rr: '77:09:0002001:1234', kuvd: 'КУВД-001/2026-47', sendDate: '13 мая',    manager: 'Смирнова Юлия'  },
  { id: 'РГ 2046', date: '11.05', initials: 'ОД', client: 'Орлов Дмитрий',    phone: '+7 903 ***-**-72', fullPhone: '+7 903 811-45-72', company: 'ООО «ДомФинанс»',     project: 'ЖК «Парковый»',    propertyType: 'Апартаменты',       address: 'Санкт-Петербург, Петроградский р-н, пр. Ленина, д. 88, кв. 34',                   status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Орлов Дмитрий'  },
  { id: 'РГ 2045', date: '10.05', initials: 'СЮ', client: 'Смирнова Юлия',    phone: '+7 925 ***-**-18', fullPhone: '+7 925 007-33-18', company: 'ООО «Ипотека»',       project: 'ЖК «Лесной»',      propertyType: 'Квартира',          address: 'Московская область, г. Одинцово, ул. Берёзовая, д. 5, кв. 12',                    status: 'В работе',         regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Лебедев Игорь'  },
  { id: 'РГ 2044', date: '07.05', initials: 'ЛИ', client: 'Лебедев Игорь',    phone: '+7 499 ***-**-55', fullPhone: '+7 499 123-00-55', company: 'ООО «СтройКредит»',   project: 'ЖК «Центральный»', propertyType: 'Таунхаус',          address: 'Москва, Новомосковский АО, пос. Московский, ул. Советская, д. 3, кв. 201',        status: 'Зарегистрировано', regDate: '9 мая',     rr: '77:01:0003002:5678', kuvd: 'КУВД-001/2026-44', sendDate: '8 мая',     manager: 'Воронова Анна'  },
  { id: 'РГ 2043', date: '05.05', initials: 'ВА', client: 'Воронова Анна',     phone: '+7 916 ***-**-77', fullPhone: '+7 916 540-29-77', company: 'ООО «Ипотека»',       project: 'ЖК «Солнечный»',   propertyType: 'Квартира',          address: 'Ленинградская область, г. Всеволожск, ул. Светлая, д. 22, кв. 7',                 status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Смирнова Юлия'  },
  { id: 'РГ 2042', date: '04.05', initials: 'МС', client: 'Морозов Сергей',    phone: '+7 912 ***-**-88', fullPhone: '+7 912 300-14-88', company: 'ООО «ДомФинанс»',     project: 'ЖК «Западный»',    propertyType: 'Квартира',          address: 'Москва, Западный АО, пр. Мира, д. 101, кв. 56',                                   status: 'Зарегистрировано', regDate: '6 мая',     rr: '77:06:0001003:9012', kuvd: 'КУВД-001/2026-42', sendDate: '5 мая',     manager: 'Орлов Дмитрий'  },
  { id: 'РГ 2041', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария',   phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', company: 'ООО «РегЦентр»',      project: 'ЖК «Восточный»',   propertyType: 'Машино-место',      address: 'Московская область, г. Балашиха, мкр. Железнодорожный, ул. Садовая, д. 9, м/м 3', status: 'В работе',         regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Морозов Сергей' },
  { id: 'РГ 2040', date: '28.04', initials: 'АП', client: 'Алексеев Пётр',     phone: '+7 915 ***-**-31', fullPhone: '+7 915 462-55-31', company: 'ООО «Ипотека»',       project: 'ЖК «Уютный»',      propertyType: 'Квартира',          address: 'Москва, ул. Новая, д. 15, кв. 88',                                                status: 'Зарегистрировано', regDate: '30 апреля', rr: '77:03:0004004:3456', kuvd: 'КУВД-001/2026-40', sendDate: '29 апреля', manager: 'Лебедев Игорь'  },
  { id: 'РГ 2039', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',    phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', company: 'ООО «СтройКредит»',   project: 'ЖК «Речной»',      propertyType: 'Апартаменты',       address: 'Санкт-Петербург, Невский р-н, наб. Невская, д. 4, кв. 19',                        status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Воронова Анна'  },
  { id: 'РГ 2038', date: '22.04', initials: 'ТВ', client: 'Тихонов Виктор',    phone: '+7 903 ***-**-45', fullPhone: '+7 903 159-60-45', company: 'ООО «РегЦентр»',      project: 'ЖК «Новый город»', propertyType: 'Нежилое помещение', address: 'Москва, ТиНАО, г. Троицк, ул. Промышленная, д. 7, пом. 2',                        status: 'Зарегистрировано', regDate: '24 апреля', rr: '77:22:0005005:7890', kuvd: 'КУВД-001/2026-38', sendDate: '23 апреля', manager: 'Смирнова Юлия'  },
  { id: 'РГ 2037', date: '20.04', initials: 'ПА', client: 'Петрова Анастасия', phone: '+7 926 ***-**-63', fullPhone: '+7 926 381-74-63', company: 'ООО «СтройКредит»',   project: 'ЖК «Северный»',    propertyType: 'Квартира',          address: 'Московская область, г. Красногорск, ул. Первомайская, д. 14, корп. 3, кв. 256',   status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Лебедев Игорь'  },
  { id: 'РГ 2036', date: '18.04', initials: 'БД', client: 'Борисов Денис',     phone: '+7 915 ***-**-82', fullPhone: '+7 915 044-22-82', company: 'Акционерное общество «Специализированный застройщик Северо-Западная строительная корпорация»',                        project: 'ЖК «Речной»',      propertyType: 'Апартаменты',       address: 'Санкт-Петербург, Центральный р-н, наб. Обводного канала, д. 118, апарт. 45',      status: 'В работе',         regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Воронова Анна'  },
  { id: 'РГ 2035', date: '15.04', initials: 'ЕС', client: 'Егоров Станислав',  phone: '+7 916 ***-**-37', fullPhone: '+7 916 772-09-37', company: 'Общество с ограниченной ответственностью «Инвестиционно-строительная компания Перспектива Девелопмент»', project: 'ЖК «Уютный»',      propertyType: 'Квартира',          address: 'Московская область, г. Люберцы, ул. Октябрьская, д. 3, кв. 112',                  status: 'Зарегистрировано', regDate: '17 апреля', rr: '77:08:0006001:2341', kuvd: 'КУВД-001/2026-35', sendDate: '16 апреля', manager: 'Орлов Дмитрий'  },
  { id: 'РГ 2034', date: '12.04', initials: 'НВ', client: 'Никитина Вера',     phone: '+7 903 ***-**-91', fullPhone: '+7 903 514-88-91', company: 'Публичное акционерное общество «Группа компаний ПИК — Специализированный застройщик»',                         project: 'ЖК «Солнечный»',   propertyType: 'Квартира',          address: 'Москва, Северо-Восточный АО, ул. Ярославская, д. 9, корп. 1, кв. 344',           status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Смирнова Юлия'  },
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
    i.phone.replace(/\s/g, '').includes(q.replace(/\s/g, '')) ||
    i.address.toLowerCase().includes(q) ||
    i.project.toLowerCase().includes(q) ||
    i.company.toLowerCase().includes(q) ||
    (i.rr && i.rr.toLowerCase().includes(q)) ||
    (i.kuvd && i.kuvd.toLowerCase().includes(q))
  )
  const f = props.filters
  if (f) {
    if (f.statuses?.length)  result = result.filter(i => f.statuses.includes(i.status))
    if (f.managers?.length)  result = result.filter(i => f.managers.includes(i.manager))
    if (f.projects?.length)  result = result.filter(i => f.projects.includes(i.project))
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
