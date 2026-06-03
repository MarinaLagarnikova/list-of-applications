<template>
  <BaseDrawer :open="open" @close="$emit('close')">
    <!-- ─── Sticky header ───────────────────────── -->
      <div class="shrink-0 px-6 pt-6 pb-4">
        <div class="flex items-center justify-between">
          <span class="text-[20px] leading-[32px] font-medium text-zinc-900">Фильтры</span>
          <button
            @click="$emit('close')"
            class="flex size-6 items-center justify-center text-zinc-500 hover:text-zinc-900 transition-colors"
          >
            <XIcon :size="20" />
          </button>
        </div>
      </div>

      <!-- ─── Scrollable content ──────────────────── -->
      <div class="flex-1 overflow-y-auto min-h-0">

        <!-- Дата создания -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата создания</span>
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
              :class="selectedDateBadge === badge
                ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
                : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
            >{{ badge }}</button>
          </div>
          <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
            <input
              type="text"
              placeholder=""
              v-model="dateCustom"
              @input="selectedDateBadge = null"
              class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
            />
            <button v-if="dateCustom" @click="resetDate" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
              <XIcon :size="16" />
            </button>
            <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
          </div>
        </div>

        <!-- Статус -->
        <FilterStatusGroup
          label="Статус"
          :options="applicationStatuses"
          :model-value="selectedStatuses"
          @update:model-value="selectedStatuses = $event"
        />

        <!-- Менеджер -->
        <FilterManagerGroup
          label="Менеджер"
          :options="managers"
          :model-value="selectedManagers"
          @update:model-value="selectedManagers = $event"
        />

        <!-- Регион -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Регион</span>
            <button
              v-if="selectedRegions.size > 0"
              @click="selectedRegions = new Set()"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >Сбросить</button>
          </div>
          <div class="flex flex-wrap gap-2">
            <button
              v-for="region in regionOptions"
              :key="region"
              @click="toggleRegion(region)"
              :class="selectedRegions.has(region)
                ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
                : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
            >{{ region }}</button>
          </div>
        </div>

        <!-- Проект -->
        <FilterProjectGroup
          label="Проект"
          :groups="projectGroups"
          :model-value="selectedProjects"
          @update:model-value="selectedProjects = $event"
        />

        <!-- Тип ипотеки -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Тип ипотеки</span>
            <button
              v-if="selectedMortgageTypes.size > 0"
              @click="selectedMortgageTypes = new Set()"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >Сбросить</button>
          </div>
          <div class="flex flex-wrap gap-2">
            <button
              v-for="type in mortgageTypes"
              :key="type"
              @click="toggleMortgageType(type)"
              :class="selectedMortgageTypes.has(type)
                ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
                : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
            >{{ type }}</button>
          </div>
        </div>

        <!-- Банк -->
        <FilterBrandGroup
          label="Банк"
          :options="banks"
          :model-value="selectedBanks"
          search-placeholder="Название"
          empty-text="Банк не найден"
          @update:model-value="selectedBanks = $event"
        />

        <!-- Статус сделки -->
        <FilterStatusGroup
          label="Статус сделки"
          :options="dealStatuses"
          :model-value="selectedDealStatuses"
          @update:model-value="selectedDealStatuses = $event"
        />

        <!-- Стоимость объекта -->
        <FilterPriceRange
          label="Стоимость объекта"
          :model-value="priceObject"
          @update:model-value="priceObject = $event"
        />

        <!-- Стоимость кредита -->
        <FilterPriceRange
          label="Сумма кредита"
          :model-value="priceCredit"
          @update:model-value="priceCredit = $event"
        />

        <!-- Первый взнос -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Первый взнос</span>
            <button
              v-if="selectedDownPayments.size > 0"
              @click="selectedDownPayments = new Set()"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >Сбросить</button>
          </div>
          <div class="flex flex-wrap gap-2">
            <button
              v-for="pct in downPaymentOptions"
              :key="pct"
              @click="toggleDownPayment(pct)"
              :class="selectedDownPayments.has(pct)
                ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
                : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
            >{{ pct }}%</button>
          </div>
        </div>

        <!-- Дата отправки -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата отправки</span>
            <button
              v-if="selectedSendDateBadge || sendDateCustom"
              @click="selectedSendDateBadge = null; sendDateCustom = ''"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >Сбросить</button>
          </div>
          <div class="flex flex-wrap gap-2 mb-3">
            <button
              v-for="badge in dateBadges"
              :key="badge"
              @click="selectedSendDateBadge === badge ? (selectedSendDateBadge = null, sendDateCustom = '') : (selectedSendDateBadge = badge, sendDateCustom = badgeToDate(badge))"
              :class="selectedSendDateBadge === badge
                ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
                : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
            >{{ badge }}</button>
          </div>
          <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
            <input type="text" placeholder="" v-model="sendDateCustom" @input="selectedSendDateBadge = null" class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none" />
            <button v-if="sendDateCustom" @click="selectedSendDateBadge = null; sendDateCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors"><XIcon :size="16" /></button>
            <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
          </div>
        </div>

        <!-- Дата выдачи кредита -->
        <div class="px-6 py-4">
          <div class="flex items-center justify-between mb-3">
            <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата выдачи кредита</span>
            <button
              v-if="selectedIssueDateBadge || issueDateCustom"
              @click="selectedIssueDateBadge = null; issueDateCustom = ''"
              class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
            >Сбросить</button>
          </div>
          <div class="flex flex-wrap gap-2 mb-3">
            <button
              v-for="badge in dateBadges"
              :key="badge"
              @click="selectedIssueDateBadge === badge ? (selectedIssueDateBadge = null, issueDateCustom = '') : (selectedIssueDateBadge = badge, issueDateCustom = badgeToDate(badge))"
              :class="selectedIssueDateBadge === badge
                ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
                : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
            >{{ badge }}</button>
          </div>
          <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
            <input type="text" placeholder="" v-model="issueDateCustom" @input="selectedIssueDateBadge = null" class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none" />
            <button v-if="issueDateCustom" @click="selectedIssueDateBadge = null; issueDateCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors"><XIcon :size="16" /></button>
            <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
          </div>
        </div>

        <div class="h-4" />
      </div>

      <!-- ─── Sticky footer ───────────────────────── -->
      <FilterFooter
        :count="props.count"
        :has-active-filters="hasActiveFilters"
        :filter-count="filterCount"
        :tags="activeFilterTags"
        @reset="resetAll"
        @close="$emit('close')"
      />

  </BaseDrawer>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import BaseDrawer from './BaseDrawer.vue'
import FilterCheckboxGroup from './FilterCheckboxGroup.vue'
import FilterManagerGroup from './FilterManagerGroup.vue'
import FilterProjectGroup from './FilterProjectGroup.vue'
import FilterBrandGroup from './FilterBrandGroup.vue'
import FilterPriceRange from './FilterPriceRange.vue'
import FilterStatusGroup from './FilterStatusGroup.vue'
import FilterFooter from './FilterFooter.vue'
import {
  X as XIcon,
  ChevronRight as ChevronRightIcon,
  Calendar as CalendarIcon,
} from 'lucide-vue-next'

const props = defineProps({ open: Boolean, count: { type: Number, default: 0 } })
const emit = defineEmits(['close', 'update:filterCount', 'update:filters', 'update:filterTags'])

const dateBadges = ['Сегодня', 'Вчера', 'Неделя', '2 недели', 'Май', '2 квартал']
const selectedDateBadge = ref(null)
const dateCustom = ref('')

const today = new Date(2026, 4, 15)
const fmtDate = (d) => d.toLocaleDateString('ru-RU', { day: '2-digit', month: '2-digit', year: 'numeric' })
const addDays = (d, n) => { const r = new Date(d); r.setDate(r.getDate() + n); return r }
const badgeToDate = (badge) => {
  if (badge === 'Сегодня') return fmtDate(today)
  if (badge === 'Вчера') return fmtDate(addDays(today, -1))
  if (badge === 'Неделя') return `${fmtDate(addDays(today, -7))} — ${fmtDate(today)}`
  if (badge === '2 недели') return `${fmtDate(addDays(today, -14))} — ${fmtDate(today)}`
  if (badge === 'Май') return '01.05.2026 — 31.05.2026'
  if (badge === '2 квартал') return '01.04.2026 — 30.06.2026'
  return ''
}

const resetDate = () => {
  selectedDateBadge.value = null
  dateCustom.value = ''
}

const managers = [
  { name: 'Смирнова Юлия',  initials: 'СЮ', role: 'Администратор'    },
  { name: 'Орлов Дмитрий',  initials: 'ОД', role: 'Продажи'          },
  { name: 'Лебедев Игорь',  initials: 'ЛИ', role: 'Менеджер ипотеки' },
  { name: 'Воронова Анна',  initials: 'ВА', role: 'Менеджер ипотеки' },
  { name: 'Морозов Сергей', initials: 'МС', role: 'Руководитель'      },
]
const selectedManagers = ref(new Set())

const applicationStatuses = [
  { name: 'Новая заявка',         dot: 'border-gray-400 bg-gray-50'   },
  { name: 'Подготовка',           dot: 'border-gray-400 bg-gray-50'   },
  { name: 'Консультация',         dot: 'border-gray-400 bg-gray-50'   },
  { name: 'На подписании',        dot: 'border-amber-400 bg-amber-50' },
  { name: 'Банк выбран',          dot: 'border-blue-400 bg-blue-50'   },
  { name: 'Ожидает решения',      dot: 'border-amber-400 bg-amber-50' },
  { name: 'Одобрена',             dot: 'border-green-400 bg-green-50' },
  { name: 'Согласование ДДУ',     dot: 'border-amber-400 bg-amber-50' },
  { name: 'ДДУ согласован',       dot: 'border-green-400 bg-green-50' },
  { name: 'ДДУ зарегистрирован',  dot: 'border-green-400 bg-green-50' },
  { name: 'Объект на согласовании', dot: 'border-amber-400 bg-amber-50' },
  { name: 'Кредит выдан',         dot: 'border-green-400 bg-green-50' },
  { name: 'Отказано',             dot: 'border-red-400 bg-red-50'     },
]
const selectedStatuses = ref(new Set())

const projectGroups = [
  {
    city: 'Москва',
    options: [
      'Самолет/Новые Ватутинки',
      'Самолет/Пригород Лесное',
      'Самолет/Жилой район Южный',
      'Самолет/Митино О2',
      'Самолет/Путилково',
    ],
  },
  {
    city: 'Санкт-Петербург',
    options: [
      'ЛСР/Морская набережная (СПб)',
      'Группа ЛСР/Зенит (СПб)',
      'ЛСР/Цивилизация (СПб)',
      'ПИК/Кудрово (СПб)',
    ],
  },
  {
    city: 'Краснодар',
    options: [
      'DOGMA/Самолет (Краснодар)',
    ],
  },
]
const selectedProjects = ref(new Set())

const regionOptions = [
  'Город Москва',
  'Город Санкт-Петербург',
  'Красноярский край',
]
const selectedRegions = ref(new Set())

// Банк
const banks = [
  { name: 'Сбербанк',         initial: 'С', color: '#21A038' },
  { name: 'ВТБ',              initial: 'В', color: '#009FDF' },
  { name: 'Альфа-Банк',       initial: 'А', color: '#EF3124' },
  { name: 'Газпромбанк',      initial: 'Г', color: '#0066B3' },
  { name: 'Россельхозбанк',   initial: 'Р', color: '#00703C' },
  { name: 'Открытие',         initial: 'О', color: '#FF6600' },
  { name: 'Райффайзенбанк',   initial: 'Р', color: '#FFDD00' },
  { name: 'Совкомбанк',       initial: 'С', color: '#E31E24' },
  { name: 'Промсвязьбанк',    initial: 'П', color: '#FF6B00' },
  { name: 'Росбанк',          initial: 'Р', color: '#6C2D8F' },
  { name: 'Уралсиб',          initial: 'У', color: '#0070C0' },
  { name: 'МКБ',              initial: 'М', color: '#E4002B' },
]
const selectedBanks = ref(new Set())

// Статус сделки
const dealStatuses = [
  { name: 'Одобрена',               dot: 'border-green-400 bg-green-50'  },
  { name: 'Сделка выбрана',         dot: 'border-gray-400 bg-gray-50'    },
  { name: 'Отказ клиента',          dot: 'border-red-400 bg-red-50'      },
  { name: 'ДДУ согласован',         dot: 'border-green-400 bg-green-50'  },
  { name: 'ДДУ зарегистрирован',    dot: 'border-amber-400 bg-amber-50'  },
  { name: 'Объект на согласовании', dot: 'border-blue-400 bg-blue-50'    },
  { name: 'Кредит выдан',           dot: 'border-amber-400 bg-amber-50'  },
  { name: 'На подписании',          dot: 'border-amber-400 bg-amber-50'  },
  { name: 'Новая сделка',           dot: 'border-gray-400 bg-gray-50'    },
  { name: 'Готово к отправке',      dot: 'border-green-400 bg-green-50'  },
  { name: 'Отказано',               dot: 'border-red-400 bg-red-50'      },
  { name: 'На доработке',           dot: 'border-gray-400 bg-gray-50'    },
  { name: 'Отправлена',             dot: 'border-blue-400 bg-blue-50'    },
  { name: 'Ошибка',                 dot: 'border-red-400 bg-red-50'      },
]
const selectedDealStatuses = ref(new Set())

const priceObject = ref({ from: null, to: null })
const priceCredit = ref({ from: null, to: null })

const downPaymentOptions = [15, 20, 30, 40, 50, 60, 70]
const selectedDownPayments = ref(new Set())
const toggleDownPayment = (pct) => {
  const s = new Set(selectedDownPayments.value)
  s.has(pct) ? s.delete(pct) : s.add(pct)
  selectedDownPayments.value = s
}

const toggleRegion = (region) => {
  const s = new Set(selectedRegions.value)
  s.has(region) ? s.delete(region) : s.add(region)
  selectedRegions.value = s
}

const collapsibleSections = []

const mortgageTypes = ['Семейная', 'Гос.поддержка', 'Стандартная', 'Военная', 'ИТ', 'Сельская', 'Дальневосточная', 'Арктическая']
const selectedMortgageTypes = ref(new Set())
const toggleMortgageType = (type) => {
  const s = new Set(selectedMortgageTypes.value)
  s.has(type) ? s.delete(type) : s.add(type)
  selectedMortgageTypes.value = s
}

// Дата отправки
const selectedSendDateBadge = ref(null)
const sendDateCustom = ref('')

// Дата выдачи кредита
const selectedIssueDateBadge = ref(null)
const issueDateCustom = ref('')
const toggleSection = (_name) => { /* future expansion */ }

const hasActiveFilters = computed(() =>
  selectedDateBadge.value !== null ||
  dateCustom.value !== '' ||
  selectedManagers.value.size > 0 ||
  selectedStatuses.value.size > 0 ||
  selectedProjects.value.size > 0 ||
  selectedRegions.value.size > 0 ||
  selectedBanks.value.size > 0 ||
  selectedDealStatuses.value.size > 0 ||
  priceObject.value.from !== null ||
  priceObject.value.to !== null ||
  priceCredit.value.from !== null ||
  priceCredit.value.to !== null ||
  selectedDownPayments.value.size > 0 ||
  selectedSendDateBadge.value !== null || sendDateCustom.value !== '' ||
  selectedIssueDateBadge.value !== null || issueDateCustom.value !== '' ||
  selectedMortgageTypes.value.size > 0
)

const filterCount = computed(() =>
  (selectedDateBadge.value !== null || dateCustom.value !== '' ? 1 : 0) +
  selectedManagers.value.size +
  selectedStatuses.value.size +
  selectedProjects.value.size +
  selectedRegions.value.size +
  selectedBanks.value.size +
  selectedDealStatuses.value.size +
  (priceObject.value.from !== null || priceObject.value.to !== null ? 1 : 0) +
  (priceCredit.value.from !== null || priceCredit.value.to !== null ? 1 : 0) +
  selectedDownPayments.value.size +
  (selectedSendDateBadge.value !== null || sendDateCustom.value !== '' ? 1 : 0) +
  (selectedIssueDateBadge.value !== null || issueDateCustom.value !== '' ? 1 : 0) +
  selectedMortgageTypes.value.size
)

watch(filterCount, val => emit('update:filterCount', val), { immediate: true })

const activeFilterTags = computed(() => {
  const tags = []
  if (selectedDateBadge.value || dateCustom.value)
    tags.push({ label: `Дата создания: ${selectedDateBadge.value || dateCustom.value}`, reset: resetDate })
  selectedStatuses.value.forEach(s =>
    tags.push({ label: s, reset: () => { const set = new Set(selectedStatuses.value); set.delete(s); selectedStatuses.value = set } })
  )
  selectedManagers.value.forEach(m =>
    tags.push({ label: m, reset: () => { const set = new Set(selectedManagers.value); set.delete(m); selectedManagers.value = set } })
  )
  selectedProjects.value.forEach(p =>
    tags.push({ label: p, reset: () => { const set = new Set(selectedProjects.value); set.delete(p); selectedProjects.value = set } })
  )
  selectedRegions.value.forEach(r =>
    tags.push({ label: r, reset: () => { const set = new Set(selectedRegions.value); set.delete(r); selectedRegions.value = set } })
  )
  selectedBanks.value.forEach(b =>
    tags.push({ label: b, reset: () => { const set = new Set(selectedBanks.value); set.delete(b); selectedBanks.value = set } })
  )
  selectedDealStatuses.value.forEach(ds =>
    tags.push({ label: ds, reset: () => { const set = new Set(selectedDealStatuses.value); set.delete(ds); selectedDealStatuses.value = set } })
  )
  selectedMortgageTypes.value.forEach(mt =>
    tags.push({ label: mt, reset: () => { const set = new Set(selectedMortgageTypes.value); set.delete(mt); selectedMortgageTypes.value = set } })
  )
  selectedDownPayments.value.forEach(dp =>
    tags.push({ label: `ПВ ${dp}%`, reset: () => { const set = new Set(selectedDownPayments.value); set.delete(dp); selectedDownPayments.value = set } })
  )
  if (priceObject.value.from !== null || priceObject.value.to !== null)
    tags.push({ label: `Стоимость объекта: ${priceObject.value.from ?? ''}–${priceObject.value.to ?? ''}`, reset: () => { priceObject.value = { from: null, to: null } } })
  if (priceCredit.value.from !== null || priceCredit.value.to !== null)
    tags.push({ label: `Сумма кредита: ${priceCredit.value.from ?? ''}–${priceCredit.value.to ?? ''}`, reset: () => { priceCredit.value = { from: null, to: null } } })
  if (selectedSendDateBadge.value || sendDateCustom.value)
    tags.push({ label: `Дата отправки: ${selectedSendDateBadge.value || sendDateCustom.value}`, reset: () => { selectedSendDateBadge.value = null; sendDateCustom.value = '' } })
  if (selectedIssueDateBadge.value || issueDateCustom.value)
    tags.push({ label: `Дата выдачи: ${selectedIssueDateBadge.value || issueDateCustom.value}`, reset: () => { selectedIssueDateBadge.value = null; issueDateCustom.value = '' } })
  return tags
})
watch(activeFilterTags, val => emit('update:filterTags', val), { immediate: true })

const emitFilters = () => emit('update:filters', {
  statuses: selectedStatuses.value,
  managers: selectedManagers.value,
  projects: selectedProjects.value,
  regions: selectedRegions.value,
  banks: selectedBanks.value,
  dealStatuses: selectedDealStatuses.value,
  priceObject: priceObject.value,
  priceCredit: priceCredit.value,
  downPayments: selectedDownPayments.value,
  sendDate: sendDateCustom.value || null,
  issueDate: issueDateCustom.value || null,
  mortgageTypes: selectedMortgageTypes.value,
})
watch([selectedStatuses, selectedManagers, selectedProjects, selectedRegions, selectedBanks, selectedDealStatuses, priceObject, priceCredit, selectedDownPayments, sendDateCustom, issueDateCustom, selectedMortgageTypes], emitFilters, { immediate: true })

const resetAll = () => {
  resetDate()
  selectedManagers.value = new Set()
  selectedStatuses.value = new Set()
  selectedProjects.value = new Set()
  selectedRegions.value = new Set()
  selectedBanks.value = new Set()
  selectedDealStatuses.value = new Set()
  priceObject.value = { from: null, to: null }
  priceCredit.value = { from: null, to: null }
  selectedDownPayments.value = new Set()
  selectedSendDateBadge.value = null
  sendDateCustom.value = ''
  selectedIssueDateBadge.value = null
  issueDateCustom.value = ''
  selectedMortgageTypes.value = new Set()
}

defineExpose({ resetAll })
</script>
