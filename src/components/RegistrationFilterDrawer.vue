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
          <input id="sort-reg-new" name="sort-reg" type="radio" value="new" v-model="sortOrder" class="relative size-4 appearance-none rounded-full border border-gray-300 bg-white before:absolute before:inset-1 before:rounded-full before:bg-white not-checked:before:hidden checked:border-indigo-600 checked:bg-indigo-600 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600 forced-colors:appearance-auto forced-colors:before:hidden" />
          <label for="sort-reg-new" class="ml-3 text-[14px] font-light text-[#18181b]">Сначала новые</label>
        </div>
        <div class="flex items-center">
          <input id="sort-reg-old" name="sort-reg" type="radio" value="old" v-model="sortOrder" class="relative size-4 appearance-none rounded-full border border-gray-300 bg-white before:absolute before:inset-1 before:rounded-full before:bg-white not-checked:before:hidden checked:border-indigo-600 checked:bg-indigo-600 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600 forced-colors:appearance-auto forced-colors:before:hidden" />
          <label for="sort-reg-old" class="ml-3 text-[14px] font-light text-[#18181b]">Сначала старые</label>
        </div>
      </div>

      <!-- Дата создания -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Дата создания</span>
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
            @click="selectedStatuses.size > 0 ? resetSet(selectedStatuses) : fillSet(selectedStatuses, statuses)"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >{{ selectedStatuses.size > 0 ? 'Сбросить' : 'Выбрать все' }}</button>
        </div>
        <div class="flex flex-col gap-y-3">
          <div
            v-for="status in statuses"
            :key="status"
            class="flex items-center justify-between cursor-pointer"
            @click="toggleSet(selectedStatuses, status)"
          >
            <span class="text-[14px] font-light text-[#18181b]">{{ status }}</span>
            <span :class="checkboxClass(selectedStatuses.has(status))">
              <svg :class="['size-3 stroke-white transition-opacity', selectedStatuses.has(status) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
            </span>
          </div>
        </div>
      </div>

      <!-- Менеджер -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Менеджер</span>
          <button
            @click="selectedManagers.size > 0 ? resetSet(selectedManagers) : fillSet(selectedManagers, managers.map(m => m.id))"
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
            @click="toggleSet(selectedManagers, mgr.id)"
          >
            <span class="text-[14px] font-light text-[#18181b]">{{ mgr.name }}</span>
            <span :class="checkboxClass(selectedManagers.has(mgr.id))">
              <svg :class="['size-3 stroke-white transition-opacity', selectedManagers.has(mgr.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
            </span>
          </div>
        </div>
      </div>

      <!-- Проект -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Проект</span>
          <button
            @click="selectedProjects.size > 0 ? resetSet(selectedProjects) : fillSet(selectedProjects, projects.map(p => p.id))"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >{{ selectedProjects.size > 0 ? 'Сбросить' : 'Выбрать все' }}</button>
        </div>
        <div class="flex flex-col gap-y-3">
          <div
            v-for="project in projects"
            :key="project.id"
            class="flex items-center justify-between cursor-pointer"
            @click="toggleSet(selectedProjects, project.id)"
          >
            <span class="text-[14px] font-light text-[#18181b]">{{ project.name }}</span>
            <span :class="checkboxClass(selectedProjects.has(project.id))">
              <svg :class="['size-3 stroke-white transition-opacity', selectedProjects.has(project.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
            </span>
          </div>
        </div>
      </div>

      <!-- Дата регистрации -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Дата регистрации</span>
          <button
            v-if="selectedRegDateBadge || regDateCustom"
            @click="selectedRegDateBadge = null; regDateCustom = ''"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedRegDateBadge === badge ? (selectedRegDateBadge = null, regDateCustom = '') : (selectedRegDateBadge = badge, regDateCustom = badgeToDate(badge))"
            :class="[
              'inline-flex items-center rounded-full border px-3 h-6 text-[13px] leading-none font-medium transition-colors',
              selectedRegDateBadge === badge
                ? 'bg-indigo-50 border-indigo-200 text-indigo-700'
                : 'bg-white border-[#e4e4e7] text-[#3f3f46] hover:bg-zinc-50',
            ]"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="regDateCustom"
            @input="selectedRegDateBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-medium text-[#18181b] placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

      <!-- Способ оплаты -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">Способ оплаты</span>
          <button
            @click="selectedPayments.size > 0 ? resetSet(selectedPayments) : fillSet(selectedPayments, paymentMethods)"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >{{ selectedPayments.size > 0 ? 'Сбросить' : 'Выбрать все' }}</button>
        </div>
        <div class="flex flex-col gap-y-3">
          <div
            v-for="method in paymentMethods"
            :key="method"
            class="flex items-center justify-between cursor-pointer"
            @click="toggleSet(selectedPayments, method)"
          >
            <span class="text-[14px] font-light text-[#18181b]">{{ method }}</span>
            <span :class="checkboxClass(selectedPayments.has(method))">
              <svg :class="['size-3 stroke-white transition-opacity', selectedPayments.has(method) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
            </span>
          </div>
        </div>
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
  Calendar as CalendarIcon,
  Search as SearchIcon,
} from 'lucide-vue-next'

const props = defineProps({ open: Boolean, count: { type: Number, default: 0 } })
defineEmits(['close'])

const sortOrder = ref('new')

const checkboxClass = (checked) => [
  'relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors',
  checked ? 'bg-indigo-600 border-indigo-600' : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]',
]

const toggleSet = (setRef, key) => {
  const s = new Set(setRef.value)
  s.has(key) ? s.delete(key) : s.add(key)
  setRef.value = s
}
const resetSet = (setRef) => { setRef.value = new Set() }
const fillSet  = (setRef, items) => { setRef.value = new Set(items) }

// Проект
const projects = [
  { id: 1,  name: 'ЖК «Северный»'    },
  { id: 2,  name: 'ЖК «Парковый»'    },
  { id: 3,  name: 'ЖК «Лесной»'      },
  { id: 4,  name: 'ЖК «Центральный»' },
  { id: 5,  name: 'ЖК «Солнечный»'   },
  { id: 6,  name: 'ЖК «Западный»'    },
  { id: 7,  name: 'ЖК «Восточный»'   },
  { id: 8,  name: 'ЖК «Уютный»'      },
  { id: 9,  name: 'ЖК «Речной»'      },
  { id: 10, name: 'ЖК «Новый город»' },
]
const selectedProjects = ref(new Set())

// Менеджер
const managers = [
  { id: 1, name: 'Смирнова Юлия'  },
  { id: 2, name: 'Орлов Дмитрий'  },
  { id: 3, name: 'Лебедев Игорь'  },
  { id: 4, name: 'Воронова Анна'  },
  { id: 5, name: 'Морозов Сергей' },
]
const managerSearch = ref('')
const selectedManagers = ref(new Set())
const filteredManagers = computed(() =>
  managerSearch.value
    ? managers.filter(m => m.name.toLowerCase().includes(managerSearch.value.toLowerCase()))
    : managers
)

// Статус
const statuses = ['В работе', 'На регистрации', 'Зарегистрировано']
const selectedStatuses = ref(new Set())

// Дата создания
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

// Дата регистрации
const selectedRegDateBadge = ref(null)
const regDateCustom = ref('')

// Способ оплаты
const paymentMethods = ['Ипотека', 'Наличные', 'Рассрочка', 'Материнский капитал', 'Субсидия']
const selectedPayments = ref(new Set())

const hasActiveFilters = computed(() =>
  selectedProjects.value.size > 0 ||
  selectedManagers.value.size > 0 ||
  selectedStatuses.value.size > 0 ||
  selectedDateBadge.value !== null ||
  dateCustom.value !== '' ||
  selectedRegDateBadge.value !== null ||
  regDateCustom.value !== '' ||
  selectedPayments.value.size > 0
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
  resetSet(selectedProjects)
  resetSet(selectedManagers)
  resetSet(selectedStatuses)
  selectedDateBadge.value    = null
  dateCustom.value           = ''
  selectedRegDateBadge.value = null
  regDateCustom.value        = ''
  resetSet(selectedPayments)
}
</script>
