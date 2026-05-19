<template>
  <BaseDrawer :open="open" @close="$emit('close')">
    <!-- ─── Sticky header ───────────────────────── -->
      <div class="shrink-0 px-6 pt-6 pb-4">
        <div class="flex items-center justify-between">
          <span class="text-[20px] leading-[32px] font-medium text-[#18181b]">Фильтры и сортировка</span>
          <button
            @click="$emit('close')"
            class="flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors"
          >
            <XIcon :size="20" />
          </button>
        </div>
      </div>

      <!-- ─── Scrollable content ──────────────────── -->
      <div class="flex-1 overflow-y-auto min-h-0">

        <!-- Сортировка -->
        <div class="px-6 py-4 flex flex-col gap-y-2">
          <div class="flex items-center">
            <input id="sort-filter-new" name="sort-filter" type="radio" value="new" v-model="sortOrder" class="relative size-4 appearance-none rounded-full border border-gray-300 bg-white before:absolute before:inset-1 before:rounded-full before:bg-white not-checked:before:hidden checked:border-indigo-600 checked:bg-indigo-600 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600 forced-colors:appearance-auto forced-colors:before:hidden" />
            <label for="sort-filter-new" class="ml-3 text-[14px] font-light text-[#18181b]">Сначала новые</label>
          </div>
          <div class="flex items-center">
            <input id="sort-filter-old" name="sort-filter" type="radio" value="old" v-model="sortOrder" class="relative size-4 appearance-none rounded-full border border-gray-300 bg-white before:absolute before:inset-1 before:rounded-full before:bg-white not-checked:before:hidden checked:border-indigo-600 checked:bg-indigo-600 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600 forced-colors:appearance-auto forced-colors:before:hidden" />
            <label for="sort-filter-old" class="ml-3 text-[14px] font-light text-[#18181b]">Сначала старые</label>
          </div>
        </div>

        <!-- Дата создания -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Дата создания</span>
            <button
              v-if="selectedDateBadge || dateCustom"
              @click="resetDate"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >Сбросить</button>
          </div>
          <div class="flex flex-wrap gap-2 mb-3">
            <button
              v-for="badge in dateBadges"
              :key="badge"
              @click="selectedDateBadge === badge ? (selectedDateBadge = null, dateCustom = '') : (selectedDateBadge = badge, dateCustom = badgeToDate(badge))"
              :class="[
                'inline-flex items-center rounded-full border px-3 h-6 text-[13px] leading-none font-medium transition-colors',
                selectedDateBadge === badge
                  ? 'bg-indigo-50 border-indigo-200 text-indigo-700'
                  : 'bg-white border-[#e4e4e7] text-[#3f3f46] hover:bg-zinc-50',
              ]"
            >{{ badge }}</button>
          </div>
          <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
            <input
              type="text"
              placeholder=""
              v-model="dateCustom"
              @input="selectedDateBadge = null"
              class="flex-1 min-w-0 bg-transparent text-[14px] font-medium text-[#18181b] placeholder:text-[#a1a1aa] focus:outline-none"
            />
            <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
          </div>
        </div>

        <!-- Статус -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Статус</span>
            <button
              @click="selectedStatuses.size > 0 ? resetStatuses() : selectAllStatuses()"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >{{ selectedStatuses.size > 0 ? 'Сбросить' : 'Выбрать все' }}</button>
          </div>
          <div class="flex flex-col gap-y-3">
            <div
              v-for="status in applicationStatuses"
              :key="status"
              class="flex items-center justify-between cursor-pointer"
              @click="toggleStatus(status)"
            >
              <span class="text-[14px] font-light text-[#18181b]">{{ status }}</span>
              <span
                :class="[
                  'relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors',
                  selectedStatuses.has(status)
                    ? 'bg-indigo-600 border-indigo-600'
                    : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]',
                ]"
              >
                <svg
                  :class="['size-3 stroke-white transition-opacity', selectedStatuses.has(status) ? 'opacity-100' : 'opacity-0']"
                  viewBox="0 0 14 14" fill="none"
                >
                  <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
              </span>
            </div>
          </div>
        </div>

        <!-- Менеджер -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Менеджер</span>
            <button
              @click="selectedManagers.size > 0 ? resetManagers() : selectAllManagers()"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >{{ selectedManagers.size > 0 ? 'Сбросить' : 'Выбрать все' }}</button>
          </div>
          <div class="flex items-center gap-x-2 rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition mb-3">
            <SearchIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
            <input
              type="text"
              placeholder="Фамилия или имя"
              v-model="managerSearch"
              class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-[#18181b] placeholder:text-[#a1a1aa] focus:outline-none"
            />
          </div>
          <div class="flex flex-col gap-y-3">
            <div
              v-for="mgr in filteredManagers"
              :key="mgr.id"
              class="flex items-center justify-between cursor-pointer"
              @click="toggleManager(mgr.id)"
            >
              <span class="text-[14px] font-light text-[#18181b]">{{ mgr.name }}</span>
              <span
                :class="[
                  'relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors',
                  selectedManagers.has(mgr.id)
                    ? 'bg-indigo-600 border-indigo-600'
                    : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]',
                ]"
              >
                <svg
                  :class="['size-3 stroke-white transition-opacity', selectedManagers.has(mgr.id) ? 'opacity-100' : 'opacity-0']"
                  viewBox="0 0 14 14" fill="none"
                >
                  <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
              </span>
            </div>
          </div>
        </div>

        <!-- Проект -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Проект</span>
            <button
              @click="selectedProjects.size > 0 ? resetProjects() : selectAllProjects()"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >{{ selectedProjects.size > 0 ? 'Сбросить' : 'Выбрать все' }}</button>
          </div>
          <div class="flex flex-col gap-y-3">
            <div
              v-for="project in projects"
              :key="project"
              class="flex items-center justify-between cursor-pointer"
              @click="toggleProject(project)"
            >
              <span class="text-[14px] font-light text-[#18181b]">{{ project }}</span>
              <span
                :class="[
                  'relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors',
                  selectedProjects.has(project)
                    ? 'bg-indigo-600 border-indigo-600'
                    : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]',
                ]"
              >
                <svg
                  :class="['size-3 stroke-white transition-opacity', selectedProjects.has(project) ? 'opacity-100' : 'opacity-0']"
                  viewBox="0 0 14 14" fill="none"
                >
                  <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
              </span>
            </div>
          </div>
        </div>

        <!-- Collapsible sections -->
        <div
          v-for="section in collapsibleSections"
          :key="section"
          class=""
        >
          <button
            class="flex w-full items-center justify-between px-6 py-4 hover:bg-zinc-50 transition-colors"
            @click="toggleSection(section)"
          >
            <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">{{ section }}</span>
            <ChevronRightIcon :size="16" class="text-[#a1a1aa]" />
          </button>
        </div>

        <div class="h-4" />
      </div>

      <!-- ─── Sticky footer ───────────────────────── -->
      <div class="shrink-0 border-t border-[#f4f4f5] px-6 py-4 flex items-center gap-x-3">
        <button
          @click="resetAll"
          :class="[
            'flex items-center justify-center rounded-lg border h-9 px-4 text-[14px] font-medium transition-colors',
            hasActiveFilters
              ? 'border-[#e4e4e7] text-[#18181b] hover:bg-zinc-50'
              : 'border-[#e4e4e7] text-[#a1a1aa] cursor-default',
          ]"
        >Сбросить все</button>

        <button
          @click="props.count > 0 && $emit('close')"
          :class="['flex flex-1 items-center justify-center rounded-lg h-9 px-4 text-[14px] font-medium text-white transition-colors',
            props.count > 0 ? 'bg-indigo-600 hover:bg-indigo-500' : 'bg-indigo-300 cursor-default']"
        >{{ buttonLabel }}</button>
      </div>

  </BaseDrawer>
</template>

<script setup>
import { ref, computed } from 'vue'
import BaseDrawer from './BaseDrawer.vue'
import {
  X as XIcon,
  ChevronRight as ChevronRightIcon,
  Calendar as CalendarIcon,
  Search as SearchIcon,
} from 'lucide-vue-next'

const props = defineProps({ open: Boolean, count: { type: Number, default: 0 } })
defineEmits(['close'])

const sortOrder = ref('new')

const dateBadges = ['Сегодня', 'Вчера', 'Неделя', '2 недели']
const selectedDateBadge = ref(null)
const dateCustom = ref('')

const today = new Date(2026, 4, 15)
const fmtDate = (d) => d.toLocaleDateString('ru-RU', { day: '2-digit', month: '2-digit', year: 'numeric' })
const addDays = (d, n) => { const r = new Date(d); r.setDate(r.getDate() + n); return r }
const badgeToDate = (badge) => {
  if (badge === 'Сегодня') return fmtDate(today)
  if (badge === 'Вчера') return fmtDate(addDays(today, -1))
  if (badge === 'Неделя') return `${fmtDate(addDays(today, -7))} – ${fmtDate(today)}`
  if (badge === '2 недели') return `${fmtDate(addDays(today, -14))} – ${fmtDate(today)}`
  return ''
}

const toggleDateBadge = (badge) => {
  selectedDateBadge.value = selectedDateBadge.value === badge ? null : badge
}
const resetDate = () => {
  selectedDateBadge.value = null
  dateCustom.value = ''
}

const managers = [
  { id: 1, name: 'Иванов Иван' },
  { id: 2, name: 'Сергеев Сергей' },
  { id: 3, name: 'Петров Пётр' },
  { id: 4, name: 'Сидоров Игорь' },
  { id: 5, name: 'Новиков Алексей' },
]
const managerSearch = ref('')
const selectedManagers = ref(new Set())

const filteredManagers = computed(() =>
  managerSearch.value
    ? managers.filter(m => m.name.toLowerCase().includes(managerSearch.value.toLowerCase()))
    : managers
)

const toggleManager = (id) => {
  const s = new Set(selectedManagers.value)
  s.has(id) ? s.delete(id) : s.add(id)
  selectedManagers.value = s
}
const selectAllManagers = () => { selectedManagers.value = new Set(managers.map(m => m.id)) }
const resetManagers = () => { selectedManagers.value = new Set() }

const applicationStatuses = [
  'Одобрена',
  'Банк выбран',
  'ДДУ согласован',
  'Кредит одобрен',
  'Договор подписан',
  'Средства переведены',
  'Сделка закрыта',
]
const selectedStatuses = ref(new Set())

const toggleStatus = (s) => {
  const set = new Set(selectedStatuses.value)
  set.has(s) ? set.delete(s) : set.add(s)
  selectedStatuses.value = set
}
const selectAllStatuses = () => { selectedStatuses.value = new Set(applicationStatuses) }
const resetStatuses = () => { selectedStatuses.value = new Set() }

const projects = [
  'Самолет/Новые Ватутинки',
  'А101/Прокшино',
  'MR Group/Савёловский Сити',
  'ФСК/Южная Битца',
  'ЛСР/Морская набережная',
  'Донстрой/Символ',
  'ПИК/Люберцы Парк',
  'Эталон/Галактика',
]
const selectedProjects = ref(new Set())
const toggleProject = (p) => {
  const s = new Set(selectedProjects.value)
  s.has(p) ? s.delete(p) : s.add(p)
  selectedProjects.value = s
}
const selectAllProjects = () => { selectedProjects.value = new Set(projects) }
const resetProjects = () => { selectedProjects.value = new Set() }

const collapsibleSections = [
  'Регион',
  'Банк',
  'Статус сделки',
  'Стоимость недвижимости',
  'Стоимость кредита',
  'Первый взнос',
  'Дата отправки заявки',
  'Дата выдачи кредита',
  'Тип ипотеки',
]
const toggleSection = (_name) => { /* future expansion */ }

const hasActiveFilters = computed(() =>
  selectedDateBadge.value !== null ||
  dateCustom.value !== '' ||
  selectedManagers.value.size > 0 ||
  selectedStatuses.value.size > 0 ||
  selectedProjects.value.size > 0
)

const buttonLabel = computed(() => {
  if (props.count === 0) return 'Заявок нет'
  const n = props.count, m10 = n % 10, m100 = n % 100
  if (m100 >= 11 && m100 <= 14) return `${n} заявок`
  if (m10 === 1) return `${n} заявка`
  if (m10 >= 2 && m10 <= 4) return `${n} заявки`
  return `${n} заявок`
})

const resetAll = () => {
  resetDate()
  resetManagers()
  resetStatuses()
  resetProjects()
}
</script>
