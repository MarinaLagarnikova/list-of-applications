<template>
  <div class="relative flex h-full flex-col overflow-hidden">

    <!-- Поиск + фильтры -->
    <div :class="['shrink-0 flex items-center justify-between px-4 md:px-6 py-2', isScrolled && !(filterTags && filterTags.length) && 'sm:border-b sm:border-[#f4f4f5]']">
      <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ИНН или названию пакета" />
      <div class="flex items-center gap-x-3">
        <span v-if="searchQuery.trim()" class="text-[14px] font-light text-zinc-500 mr-5">
          {{ pluralize(totalCount) }}
        </span>
        <div class="hidden sm:block">
          <Button outline @click="$emit('open-filter')">
            <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
            Фильтры
            <span v-if="props.filterCount > 0" class="inline-flex items-center justify-center size-[18px] rounded-full bg-indigo-600 text-white text-[11px] font-medium leading-none">{{ props.filterCount }}</span>
          </Button>
        </div>
        <div class="hidden md:flex items-center gap-x-3">
          <SortDropdown v-model="sortOrder" />
          <ExportPopover :count="totalCount" />
        </div>
      </div>
    </div>


    <!-- xs-only кнопка фильтров -->
    <div :class="['sm:hidden shrink-0 px-4 py-2', isScrolled && !(filterTags && filterTags.length) && 'border-b border-[#f4f4f5]']">
      <Button outline class="w-full justify-center" @click="$emit('open-filter')">
        <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
        Фильтры
        <span v-if="props.filterCount > 0" class="inline-flex items-center justify-center size-[18px] rounded-full bg-indigo-600 text-white text-[11px] font-medium leading-none">{{ props.filterCount }}</span>
      </Button>
    </div>

    <!-- Applied filters strip -->
    <div v-if="filterTags && filterTags.length > 0" :class="['shrink-0 pl-[60px] pr-8 pt-2 pb-4', isScrolled && 'border-b border-[#f4f4f5]']">
      <FilterTagsBar :tags="filterTags" @reset="$emit('reset-filters')" @show-more="$emit('open-filter')" />
    </div>

    <div class="flex-1 overflow-auto" @scroll="isScrolled = $event.target.scrollTop > 0">
      <div class="pt-2 pb-10">
        <table class="w-full border-separate border-spacing-0">
          <tbody>
            <template v-for="(group, gi) in groupedFilteredItems" :key="group.packageName">

              <!-- Отступ между группами -->
              <tr v-if="gi > 0" aria-hidden="true">
                <td colspan="10" class="pt-4" />
              </tr>

              <!-- Заголовок группы -->
              <tr>
                <td
                  colspan="9"
                  class="sticky left-0 z-10 bg-white pb-1 pl-4 md:pl-6"
                >
                  <div class="flex items-center gap-x-5">
                    <!-- Чекбокс группы -->
                    <label class="inline-flex cursor-pointer" @click.stop="toggleGroupRows(group.packageName)">
                      <span :class="['relative flex size-4 items-center justify-center rounded-sm border', isGroupChecked(group.packageName) ? 'bg-indigo-600 border-transparent' : isGroupIndeterminate(group.packageName) ? 'bg-indigo-600 border-transparent' : 'bg-white border-zinc-950/15 hover:border-zinc-950/30']">
                        <svg v-if="isGroupChecked(group.packageName)" viewBox="0 0 14 14" fill="none" class="size-3.5 stroke-white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 7l3 3 5-5"/></svg>
                        <svg v-else-if="isGroupIndeterminate(group.packageName)" viewBox="0 0 14 14" fill="none" class="size-3.5 stroke-white" stroke-width="2" stroke-linecap="round"><path d="M3 7h8"/></svg>
                      </span>
                    </label>
                    <!-- Название + бейдж + стрелка -->
                    <div class="flex items-center gap-x-2 cursor-pointer" @click="toggleGroup(group.packageName)">
                      <span class="text-[14px] font-medium text-zinc-900">{{ group.packageName }}</span>
                      <span class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">{{ group.items.length }}</span>
                      <ChevronUpIcon
                        :size="18"
                        :class="['shrink-0 text-zinc-900 transition-transform duration-200', collapsedGroups.has(group.packageName) ? 'rotate-180' : '']"
                      />
                    </div>
                  </div>
                </td>
                <td class="sticky right-0 z-10 bg-white pb-1 pr-8"></td>
              </tr>

              <!-- Строки заявок -->
              <template v-if="!collapsedGroups.has(group.packageName)">
                <!-- Пустой пакет -->
                <tr v-if="group.items.length === 0">
                  <td colspan="10" class="pl-[70px] py-3">
                    <span class="text-[14px] font-light text-zinc-500">Пока нет ни одной заявки</span>
                    <button
                      class="ml-2 text-[14px] font-normal text-indigo-600 hover:text-indigo-700 hover:underline transition-colors"
                      @click="openPackagePage(group.packageName)"
                    >Добавить</button>
                  </td>
                </tr>

                <tr
                  v-for="item in group.items"
                  :key="item.id"
                  :class="['group cursor-pointer', checkedRows.has(item.id) && 'selected']"
                  @click="openApp(item)"
                >
                  <!-- Checkbox -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top pt-[16px] pb-3 pl-4 md:pl-6 pr-2" @click.stop="toggleRow(item.id)">
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
                        <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-zinc-500">
                          <LandmarkIcon :size="16" />
                        </span>
                        <div class="flex flex-col gap-y-1">
                          <span class="text-[14px] leading-[20px] font-normal text-zinc-900">{{ item.companyName }}</span>
                          <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.inn }}</span>
                        </div>
                      </template>
                      <template v-else>
                        <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] font-medium text-zinc-500">{{ item.initials }}</span>
                        <div class="flex flex-col gap-y-1">
                          <span class="text-[14px] leading-[20px] font-normal text-zinc-900">{{ item.client }}</span>
                          <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ item.phone }}</span>
                        </div>
                      </template>
                    </div>
                  </td>

                  <!-- Статус -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <span :class="statusBadgeClass(item.status)">{{ item.status }}</span>
                  </td>

                  <!-- Тип заявления + Компания -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4">
                    <div class="flex flex-col gap-y-1" style="width: 280px">
                      <span class="truncate text-[14px] leading-[20px] font-light text-zinc-900">{{ item.applicationType }}</span>
                      <div class="relative group/tip" style="width: 280px">
                        <span v-if="item.company" :ref="el => checkOverflow(`company_${item.id}`, el)" class="truncate block text-[14px] leading-[20px] font-light text-zinc-500">{{ item.company }}</span>
                        <span v-else class="text-[14px] leading-[20px] font-light text-zinc-500">—</span>
                        <div v-if="overflowMap[`company_${item.id}`]" class="pointer-events-none absolute top-full left-0 mt-1.5 z-50 hidden group-hover/tip:block">
                          <div class="relative rounded-md bg-zinc-900 px-2.5 py-1.5 text-[12px] leading-[18px] text-white shadow-lg" style="width: 250px; white-space: normal;">
                            <div class="absolute -top-[5px] left-1/2 -translate-x-1/2 w-0 h-0 border-l-[5px] border-l-transparent border-r-[5px] border-r-transparent border-b-[5px] border-b-zinc-900"></div>
                            {{ item.company }}
                          </div>
                        </div>
                      </div>
                    </div>
                  </td>

                  <!-- Бейджи + Адрес -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4">
                    <div class="flex flex-col gap-y-1" style="width: 460px">
                      <div class="flex flex-wrap gap-[6px]">
                        <span v-if="item.project" class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">{{ item.project }}</span>
                        <span v-if="item.propertyType" class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">{{ item.propertyType }}</span>
                        <span v-if="item.unitNumber" class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">{{ item.unitNumber }}</span>
                        <span v-if="item.cadastral" class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">{{ item.cadastral }}</span>
                      </div>
                      <div class="relative group/tip" style="width: 460px">
                        <span :ref="el => checkOverflow(`address_${item.id}`, el)" class="block overflow-hidden whitespace-nowrap text-ellipsis text-[14px] leading-[20px] font-light text-zinc-500" style="direction: rtl; text-align: left;">{{ item.address }}</span>
                        <div v-if="overflowMap[`address_${item.id}`]" class="pointer-events-none absolute top-full left-0 mt-1.5 z-50 hidden group-hover/tip:block">
                          <div class="relative rounded-md bg-zinc-900 px-2.5 py-1.5 text-[12px] leading-[18px] text-white shadow-lg" style="width: 250px; white-space: normal;">
                            <div class="absolute -top-[5px] left-1/2 -translate-x-1/2 w-0 h-0 border-l-[5px] border-l-transparent border-r-[5px] border-r-transparent border-b-[5px] border-b-zinc-900"></div>
                            {{ item.address }}
                          </div>
                        </div>
                      </div>
                    </div>
                  </td>

                  <!-- Комментарий -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4">
                    <div v-if="item.comment" class="overflow-hidden" style="width: 280px">
                      <span class="flex items-start gap-x-1.5 text-[14px] leading-[20px] font-light text-zinc-900">
                        <PinIcon :size="16" class="shrink-0 mt-0.5 text-zinc-900" />
                        <span class="line-clamp-2">{{ item.comment }}</span>
                      </span>
                    </div>
                  </td>

                  <!-- РР + КУВД -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <div v-if="item.status === 'Зарегистрировано'" class="flex flex-col gap-y-1">
                      <span class="text-[14px] leading-[20px] font-light text-zinc-900">РР {{ item.rr }}</span>
                      <span class="text-[14px] leading-[20px] font-light text-zinc-500">{{ item.kuvd }}</span>
                    </div>
                  </td>

                  <!-- Регистрация + Отправка -->
                  <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-[22px] py-4 whitespace-nowrap">
                    <div v-if="item.status === 'Зарегистрировано'" class="flex flex-col gap-y-1">
                      <span class="text-[14px] leading-[20px] font-light text-zinc-900">Регистрация {{ item.regDate }}</span>
                      <span class="text-[14px] leading-[20px] font-light text-zinc-500">Отправка {{ item.sendDate }}</span>
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

        <!-- Empty state — список пуст -->
        <div v-if="totalCount === 0 && !searchQuery.trim() && !props.filterCount" class="flex flex-col items-center justify-center py-24 text-center">
          <FilePlusCornerIcon :size="48" class="mx-auto text-zinc-400" />
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Пока здесь пусто</h3>
          <p class="mt-1 text-sm font-light text-zinc-500">В этом списке пока нет заявок</p>
          <Button outline class="mt-6" @click="emit('create')">
            <svg class="size-4" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true"><path d="M10.75 4.75a.75.75 0 00-1.5 0v4.5h-4.5a.75.75 0 000 1.5h4.5v4.5a.75.75 0 001.5 0v-4.5h4.5a.75.75 0 000-1.5h-4.5v-4.5z" /></svg>
            Создать
          </Button>
        </div>

        <!-- Empty state -->
        <div v-if="totalCount === 0 && (searchQuery.trim() || props.filterCount > 0)" class="flex flex-col items-center justify-center py-24 text-center">
          <FileSearchCornerIcon :size="48" class="mx-auto text-zinc-400" />
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Заявки не найдены</h3>
          <p class="mt-1 text-sm font-light text-zinc-500">Попробуйте изменить параметры поиска или фильтрации</p>
          <Button v-if="searchQuery.trim() && !props.filterCount" outline class="mt-6" @click="searchQuery = ''">Сбросить поиск</Button>
          <Button v-else-if="!searchQuery.trim() && props.filterCount > 0" outline class="mt-6" @click="$emit('reset-filters')">Сбросить фильтры</Button>
        </div>

      </div>
    </div>

    <SelectionBar :count="checkedRows.size + checkedEmptyPackages.size" :deleteLabel="deleteLabel" @clear="checkedRows = new Set(); checkedEmptyPackages = new Set()" @delete="handleDelete">
      <template #actions>
        <button v-if="checkedRows.size > 0" class="flex items-center gap-x-1.5 rounded-xl bg-indigo-500 hover:bg-indigo-400 px-3 py-1.5 text-[14px] font-medium text-white transition-colors">
          <PenLineIcon :size="14" />
          Подписать
        </button>
      </template>
    </SelectionBar>
  </div>
</template>

<script setup>
import { ref, computed, reactive, watch } from 'vue'
import { SlidersHorizontal as FunnelIcon, ChevronUp as ChevronUpIcon, Landmark as LandmarkIcon, Trash2 as Trash2Icon, Pin as PinIcon, PenLine as PenLineIcon, FilePlusCorner as FilePlusCornerIcon, FileSearchCorner as FileSearchCornerIcon } from 'lucide-vue-next'
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
  extraPackages: { type: Array, default: () => [] },
})
const emit = defineEmits(['open-filter', 'open-preview', 'update:count', 'reset-filters', 'create', 'delete-extra-package'])

const sortOrder = ref('new')
const isScrolled = ref(false)
const deletedPackages = ref(new Set())

const overflowMap = reactive({})
const checkOverflow = (key, el) => {
  if (el) overflowMap[key] = el.scrollWidth > el.offsetWidth
}
const searchQuery = ref('')
const checkedRows = ref(new Set())
const checkedEmptyPackages = ref(new Set())
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

const deletePackage = (packageName) => {
  if (props.extraPackages.includes(packageName)) {
    emit('delete-extra-package', packageName)
  } else {
    deletedPackages.value = new Set([...deletedPackages.value, packageName])
  }
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
  if (!group) return false
  if (group.items.length === 0) return checkedEmptyPackages.value.has(packageName)
  return group.items.every(item => checkedRows.value.has(item.id))
}

const isGroupIndeterminate = (packageName) => {
  const group = groupedFilteredItems.value.find(g => g.packageName === packageName)
  if (!group || group.items.length === 0) return false
  const checked = group.items.filter(item => checkedRows.value.has(item.id)).length
  return checked > 0 && checked < group.items.length
}

const toggleGroupRows = (packageName) => {
  const group = groupedFilteredItems.value.find(g => g.packageName === packageName)
  if (!group) return
  if (group.items.length === 0) {
    const s = new Set(checkedEmptyPackages.value)
    s.has(packageName) ? s.delete(packageName) : s.add(packageName)
    checkedEmptyPackages.value = s
    return
  }
  const s = new Set(checkedRows.value)
  if (isGroupChecked(packageName)) {
    group.items.forEach(item => s.delete(item.id))
  } else {
    group.items.forEach(item => s.add(item.id))
  }
  checkedRows.value = s
}

// Вычисляем: выбран ли ровно один пакет целиком (с заявками или пустой)
const selectedPackageForDelete = computed(() => {
  // Пустой пакет выбран
  if (checkedEmptyPackages.value.size === 1 && checkedRows.value.size === 0) {
    return [...checkedEmptyPackages.value][0]
  }
  if (checkedRows.value.size === 0) return null
  const selectedIds = checkedRows.value
  for (const group of groupedFilteredItems.value) {
    if (group.items.length === 0) continue
    const groupIds = new Set(group.items.map(i => i.id))
    const allGroupSelected = group.items.every(i => selectedIds.has(i.id))
    const noExtraSelected = [...selectedIds].every(id => groupIds.has(id))
    if (allGroupSelected && noExtraSelected && checkedEmptyPackages.value.size === 0) return group.packageName
  }
  return null
})

const deleteLabel = computed(() => selectedPackageForDelete.value ? 'Удалить пакет' : 'Удалить')

const handleDelete = () => {
  if (selectedPackageForDelete.value) {
    deletePackage(selectedPackageForDelete.value)
    checkedEmptyPackages.value = new Set()
  } else {
    checkedRows.value = new Set()
    checkedEmptyPackages.value = new Set()
  }
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
  { id: 'РГ 2047', date: '12.05', isCompany: true, companyName: 'ООО «Старт»', inn: '7701234567', company: null, project: 'ЖК «Северный»', propertyType: 'Квартира',          unitNumber: '№ 1',   cadastral: '77:09:0002001:1001', address: 'Москва, Северный АО, ул. Полярная, д. 14, корп. 2, кв. 1',                       applicationType: 'Право собственности',        status: 'Зарегистрировано', regDate: '14 мая',    rr: '77:09:0002001:1234', kuvd: 'КУВД-001/2026-47', sendDate: '13 мая',    comment: 'Приоритетная сделка, клиент ждёт выписку', manager: 'Смирнова Юлия',  packageName: 'Пакет ЖК «Северный»' },
  { id: 'РГ 2046', date: '12.05', isCompany: true, companyName: 'ООО «Старт»', inn: '7701234567', company: null, project: 'ЖК «Северный»', propertyType: 'Апартаменты',       unitNumber: '№ 34',  cadastral: '77:09:0002001:3402', address: 'Москва, Северный АО, ул. Полярная, д. 14, корп. 2, апарт. 34',                    applicationType: 'Право собственности',        status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Орлов Дмитрий',  packageName: 'Пакет ЖК «Северный»' },
  { id: 'РГ 2045', date: '12.05', isCompany: true, companyName: 'ООО «Старт»', inn: '7701234567', company: null, project: 'ЖК «Северный»', propertyType: 'Квартира',          unitNumber: '№ 12',  cadastral: '77:09:0002001:1203', address: 'Москва, Северный АО, ул. Полярная, д. 14, корп. 2, кв. 12',                      applicationType: 'Право собственности',        status: 'В работе',         regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Лебедев Игорь',  packageName: 'Пакет ЖК «Северный»' },
  // Пакет ЖК «Западный» — покупатель АО «РегионСтрой», продавец АО «ЗападДевелопмент»
  { id: 'РГ 2044', date: '07.05', isCompany: true, companyName: 'АО «РегионСтрой»', inn: '5034891203', company: 'АО «ЗападДевелопмент»', project: 'ЖК «Западный»', propertyType: 'Таунхаус',  unitNumber: '№ 201', cadastral: '50:11:0030405:2014', address: 'Московская область, г. Красногорск, Западный мкр., пр. Западный, д. 3, кв. 201', applicationType: 'Право собственности', status: 'Зарегистрировано', regDate: '9 мая',     rr: '77:01:0003002:5678', kuvd: 'КУВД-001/2026-44', sendDate: '8 мая',     manager: 'Воронова Анна',  packageName: 'Пакет ЖК «Западный»' },
  { id: 'РГ 2043', date: '07.05', isCompany: true, companyName: 'АО «РегионСтрой»', inn: '5034891203', company: 'АО «ЗападДевелопмент»', project: 'ЖК «Западный»', propertyType: 'Квартира',   unitNumber: '№ 7',   cadastral: '50:11:0030405:0703', address: 'Московская область, г. Красногорск, Западный мкр., пр. Западный, д. 3, кв. 7',   applicationType: 'Право собственности',        status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Смирнова Юлия',  packageName: 'Пакет ЖК «Западный»' },
  { id: 'РГ 2042', date: '07.05', isCompany: true, companyName: 'АО «РегионСтрой»', inn: '5034891203', company: 'АО «ЗападДевелопмент»', project: 'ЖК «Западный»', propertyType: 'Квартира',   unitNumber: '№ 56',  cadastral: '50:11:0030405:5601', address: 'Московская область, г. Красногорск, Западный мкр., пр. Западный, д. 3, кв. 56',  applicationType: 'Право собственности',        status: 'Зарегистрировано', regDate: '6 мая',     rr: '77:06:0001003:9012', kuvd: 'КУВД-001/2026-42', sendDate: '5 мая',     comment: 'Повторная проверка пакета завершена', manager: 'Орлов Дмитрий',  packageName: 'Пакет ЖК «Западный»' },
  // Пакет ЖК «Уютный» — покупатель Кузнецова Мария, продавец ООО «УютСтрой»
  { id: 'РГ 2041', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария', phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', company: 'ООО «УютСтрой»', project: 'ЖК «Уютный»', propertyType: 'Машино-место', unitNumber: '№ 3',  cadastral: '77:07:0050607:0301', address: 'Москва, ЗАО, Очаково-Матвеевское, ул. Уютная, д. 9, м/м 3',     applicationType: 'Право собственности', status: 'В работе',         regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Морозов Сергей', packageName: 'Пакет ЖК «Уютный»' },
  { id: 'РГ 2040', date: '01.05', initials: 'КМ', client: 'Кузнецова Мария', phone: '+7 926 ***-**-04', fullPhone: '+7 926 711-88-04', company: 'ООО «УютСтрой»', project: 'ЖК «Уютный»', propertyType: 'Квартира',     unitNumber: '№ 88', cadastral: '77:07:0050607:8802', address: 'Москва, ЗАО, Очаково-Матвеевское, ул. Уютная, д. 15, кв. 88',    applicationType: 'Право собственности',                   status: 'Зарегистрировано', regDate: '30 апреля', rr: '77:03:0004004:3456', kuvd: 'КУВД-001/2026-40', sendDate: '29 апреля', manager: 'Лебедев Игорь',  packageName: 'Пакет ЖК «Уютный»' },
  // Пакет ЖК «Речной» — покупатель Захарова Ольга, продавец ООО «РечСтрой»
  { id: 'РГ 2039', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',   phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', company: 'ООО «РечСтрой»',  project: 'ЖК «Речной»', propertyType: 'Апартаменты',       unitNumber: '№ 19', cadastral: '78:12:0005006:1902', address: 'Санкт-Петербург, Невский р-н, наб. Невская, д. 4, апарт. 19', applicationType: 'Право собственности',                   status: 'На регистрации',   regDate: null,        rr: null,                  kuvd: null,                sendDate: null,        manager: 'Воронова Анна',  packageName: 'Пакет ЖК «Речной»' },
  { id: 'РГ 2038', date: '25.04', initials: 'ЗО', client: 'Захарова Ольга',   phone: '+7 968 ***-**-12', fullPhone: '+7 968 224-77-12', company: 'ООО «РечСтрой»',  project: 'ЖК «Речной»', propertyType: 'Нежилое помещение', unitNumber: '№ 2',  cadastral: '78:12:0005006:0200', address: 'Санкт-Петербург, Невский р-н, наб. Невская, д. 4, пом. 2',    applicationType: 'Право собственности',                   status: 'Зарегистрировано', regDate: '24 апреля', rr: '77:22:0005005:7890', kuvd: 'КУВД-001/2026-38', sendDate: '23 апреля', manager: 'Смирнова Юлия',  packageName: 'Пакет ЖК «Речной»' },
]

items.forEach(item => { managerSelections[item.id] = item.manager })

const openApp = (item) => {
  window.open(`/app-page.html?id=${item.id}&title=${encodeURIComponent('Это страница заявки')}`, '_blank')
}

const openPackagePage = (packageName) => {
  window.open(`/app-page.html?id=${encodeURIComponent(packageName)}&title=${encodeURIComponent('Страница пакета')}`, '_blank')
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

// Группировка по packageName с сортировкой групп по дате создания пакета
const groupedFilteredItems = computed(() => {
  const map = new Map()
  for (const item of filteredItems.value) {
    if (!map.has(item.packageName)) map.set(item.packageName, [])
    map.get(item.packageName).push(item)
  }
  // Пустые пакеты (только что созданные, без заявок)
  for (const name of props.extraPackages) {
    if (!deletedPackages.value.has(name) && !map.has(name)) {
      map.set(name, [])
    }
  }
  const groups = [...map.entries()].map(([packageName, items]) => ({ packageName, items }))
  // Сортируем группы по дате пакета; пустые пакеты — самые новые, порядок между ними по индексу в extraPackages
  groups.sort((a, b) => {
    const getDate = (group) => {
      if (group.items.length > 0) return parseItemDate(group.items[0].date)
      const idx = props.extraPackages.indexOf(group.packageName)
      return new Date(9999, 0, 1, 0, 0, 0, idx)
    }
    const aDate = getDate(a)
    const bDate = getDate(b)
    return sortOrder.value === 'old' ? aDate - bDate : bDate - aDate
  })
  return groups
})

const totalCount = computed(() => filteredItems.value.length)
const packageCount = computed(() => groupedFilteredItems.value.length)

watch(packageCount, v => emit('update:count', v), { immediate: true })
</script>
