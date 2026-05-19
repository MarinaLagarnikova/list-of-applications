<template>
  <div class="relative flex h-full flex-col overflow-hidden">

    <!-- Поиск + фильтры -->
    <div class="shrink-0 flex items-center justify-between border-b border-[#f4f4f5] px-8 py-2">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ФИО, телефону или названию центра" />
      <div class="flex items-center gap-x-3">
        <span v-if="searchQuery.trim()" class="text-[14px] font-light text-[#71717a] mr-5">
          {{ pluralize(filteredItems.length) }}
        </span>
        <Button outline @click="$emit('open-filter')">
          <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
          Фильтры и сортировка
        </Button>
        <ExportPopover :count="filteredItems.length" />
      </div>
    </div>

    <div class="flex-1 overflow-auto">
      <div class="px-6 pt-6 pr-8 pb-10">
        <table class="border-separate border-spacing-0">
          <tbody>
            <tr
              v-for="item in filteredItems"
              :key="item.id"
              class="group cursor-pointer"
              @click="openApp(item)"
            >
              <!-- Checkbox -->
              <td class="rounded-l-2xl group-hover:bg-zinc-50 align-top py-[20.5px] pl-2 pr-2" @click.stop="toggleRow(item.id)">
                <label class="inline-flex cursor-pointer">
                  <span :class="['relative flex size-4 items-center justify-center rounded-sm border', checkedRows.has(item.id) ? 'bg-indigo-600 border-transparent' : 'bg-white border-zinc-950/15 hover:border-zinc-950/30']">
                    <svg :class="['size-3 stroke-white transition-opacity', checkedRows.has(item.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
                  </span>
                </label>
              </td>

              <!-- ID + дата -->
              <td class="group-hover:bg-zinc-50 align-top pl-3 pr-[22px] py-[20.5px] whitespace-nowrap">
                <div class="flex flex-col gap-y-1">
                  <TableLink @click="$emit('open-preview', item)">{{ item.id }}</TableLink>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">от {{ item.date }}</span>
                </div>
              </td>

              <!-- Аватар + ФИО + Телефон -->
              <td class="group-hover:bg-zinc-50 align-top px-[22px] py-[20.5px] whitespace-nowrap">
                <div class="flex items-start gap-x-3">
                  <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] font-medium text-[#71717a]">{{ item.initials }}</span>
                  <div class="flex flex-col gap-y-1">
                    <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.client }}</span>
                    <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.phone }}</span>
                  </div>
                </div>
              </td>

              <!-- Статус -->
              <td class="group-hover:bg-zinc-50 align-top px-[22px] py-[20.5px] whitespace-nowrap">
                <span :class="statusBadgeClass(item.status)">{{ item.status }}</span>
              </td>

              <!-- Название центра + Способ подписи -->
              <td class="group-hover:bg-zinc-50 align-top px-[22px] py-[20.5px] whitespace-nowrap">
                <div class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.center }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.signMethod }}</span>
                </div>
              </td>

              <!-- Встреча с курьером -->
              <td class="group-hover:bg-zinc-50 align-top px-[22px] py-[20.5px] whitespace-nowrap">
                <div v-if="item.status === 'Активна'" class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">Встреча с курьером</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.courierDate }}</span>
                </div>
              </td>

              <!-- Дата активации + Дата истечения -->
              <td class="group-hover:bg-zinc-50 align-top px-[22px] py-[20.5px] whitespace-nowrap">
                <div v-if="item.status === 'Активна'" class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">С {{ formatFull(item.activationDate) }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">Действует {{ item.duration }}</span>
                </div>
              </td>

              <!-- Менеджер -->
              <td class="rounded-r-2xl group-hover:bg-zinc-50 align-top px-[22px] py-[20.5px] whitespace-nowrap" @click.stop>
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
        <div v-if="filteredItems.length === 0" class="flex flex-col items-center justify-center py-24 text-center">
          <svg class="mx-auto size-12 text-zinc-900" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true">
            <path vector-effect="non-scaling-stroke" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 13h6m-3-3v6m-9 1V7a2 2 0 012-2h6l2 2h6a2 2 0 012 2v8a2 2 0 01-2 2H5a2 2 0 01-2-2z" />
          </svg>
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Заявки не найдены</h3>
          <p class="mt-1 text-sm font-light text-zinc-900">Попробуйте изменить параметры поиска или фильтрации</p>
          <div v-if="searchQuery.trim()" class="mt-6">
            <Button outline @click="searchQuery = ''">Сбросить поиск</Button>
          </div>
        </div>

      </div>
    </div>

  <SelectionBar :count="checkedRows.size" @clear="checkedRows = new Set()" @delete="checkedRows = new Set()" />
  </div>

</template>

<script setup>
import { ref, computed, reactive, watch } from 'vue'
import { Search as SearchIcon, SlidersHorizontal as FunnelIcon } from 'lucide-vue-next'
import ExportPopover from './ExportPopover.vue'
import SearchInput from './SearchInput.vue'
import Button from './Button.vue'
import TableLink from './TableLink.vue'
import CatalystListbox from './CatalystListbox.vue'
import SelectionBar from './SelectionBar.vue'

const props = defineProps({ activeSubItem: { type: String, default: '' } })
const emit = defineEmits(['open-filter', 'open-preview', 'update:count'])

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
  if (status === 'Активна')   return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'На выпуске') return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'В работе')  return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
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
  { id: 'ЦП 3047', date: '12.05', initials: 'НЕ', client: 'Новикова Елена',  phone: '+7 916 ***-**-09', fullPhone: '+7 916 234-11-09', center: 'SignMe',  signMethod: 'Офис',   activationDate: '12.05.2026', expiryDate: '12.05.2027', duration: '1 год',    courierDate: '12 мая 2026',  status: 'Активна',     manager: 'Смирнова Юлия'  },
  { id: 'ЦП 3046', date: '11.05', initials: 'ОД', client: 'Орлов Дмитрий',   phone: '+7 903 ***-**-72', fullPhone: '+7 903 811-45-72', center: 'Контур',  signMethod: 'Онлайн', activationDate: '11.05.2026', expiryDate: '11.11.2027', duration: '1,5 года', courierDate: null,           status: 'На выпуске',  manager: 'Орлов Дмитрий'  },
  { id: 'ЦП 3045', date: '10.05', initials: 'СЮ', client: 'Смирнова Юлия',   phone: '+7 925 ***-**-18', fullPhone: '+7 925 007-33-18', center: 'SignMe',  signMethod: 'Офис',   activationDate: '10.05.2026', expiryDate: '10.05.2027', duration: '1 год',    courierDate: null,           status: 'В работе',    manager: 'Лебедев Игорь'  },
  { id: 'ЦП 3044', date: '07.05', initials: 'ЛИ', client: 'Лебедев Игорь',   phone: '+7 499 ***-**-55', fullPhone: '+7 499 123-00-55', center: 'Тензор',  signMethod: 'Онлайн', activationDate: '07.05.2026', expiryDate: '07.11.2027', duration: '1,5 года', courierDate: '14 мая 2026',  status: 'Активна',     manager: 'Воронова Анна'  },
  { id: 'ЦП 3043', date: '05.05', initials: 'ВА', client: 'Воронова Анна',    phone: '+7 916 ***-**-77', fullPhone: '+7 916 540-29-77', center: 'Контур',  signMethod: 'Офис',   activationDate: '05.05.2026', expiryDate: '05.05.2027', duration: '1 год',    courierDate: null,           status: 'На выпуске',  manager: 'Смирнова Юлия'  },
  { id: 'ЦП 3042', date: '04.05', initials: 'МС', client: 'Морозов Сергей',   phone: '+7 912 ***-**-88', fullPhone: '+7 912 300-14-88', center: 'SignMe',  signMethod: 'Онлайн', activationDate: '04.05.2026', expiryDate: '04.11.2027', duration: '1,5 года', courierDate: '16 мая 2026',  status: 'Активна',     manager: 'Орлов Дмитрий'  },
  { id: 'ЦП 3041', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария',  phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', center: 'Тензор',  signMethod: 'Офис',   activationDate: '01.05.2026', expiryDate: '01.05.2027', duration: '1 год',    courierDate: null,           status: 'В работе',    manager: 'Морозов Сергей' },
  { id: 'ЦП 3040', date: '28.04', initials: 'АП', client: 'Алексеев Пётр',    phone: '+7 915 ***-**-31', fullPhone: '+7 915 462-55-31', center: 'Контур',  signMethod: 'Онлайн', activationDate: '28.04.2026', expiryDate: '28.10.2027', duration: '1,5 года', courierDate: '19 мая 2026',  status: 'Активна',     manager: 'Лебедев Игорь'  },
  { id: 'ЦП 3039', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',   phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', center: 'SignMe',  signMethod: 'Офис',   activationDate: '25.04.2026', expiryDate: '25.04.2027', duration: '1 год',    courierDate: null,           status: 'На выпуске',  manager: 'Воронова Анна'  },
  { id: 'ЦП 3038', date: '22.04', initials: 'ТВ', client: 'Тихонов Виктор',   phone: '+7 903 ***-**-45', fullPhone: '+7 903 159-60-45', center: 'Тензор',  signMethod: 'Онлайн', activationDate: '22.04.2026', expiryDate: '22.10.2027', duration: '1,5 года', courierDate: '21 мая 2026',  status: 'Активна',     manager: 'Смирнова Юлия'  },
]

items.forEach(item => { managerSelections[item.id] = item.manager })

const openApp = (item) => {
  window.open(`/app-page.html?id=${item.id}&title=${encodeURIComponent('Это страница заявки')}`, '_blank')
}

const filteredItems = computed(() => {
  let result = items
  if (props.activeSubItem === 'Мои заявки') result = result.filter(i => i.manager === 'Смирнова Юлия')
  const q = searchQuery.value.trim().toLowerCase()
  if (!q) return result
  return result.filter(i =>
    i.id.toLowerCase().includes(q) ||
    i.client.toLowerCase().includes(q) ||
    i.phone.replace(/\s/g, '').includes(q.replace(/\s/g, '')) ||
    i.center.toLowerCase().includes(q)
  )
})
watch(filteredItems, v => emit('update:count', v.length), { immediate: true })
</script>
