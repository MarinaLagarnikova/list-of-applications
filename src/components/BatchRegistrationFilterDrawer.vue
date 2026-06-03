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
            @click="selectedDateBadge = null; dateCustom = ''"
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
          <button v-if="dateCustom" @click="selectedDateBadge = null; dateCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

      <!-- Статус -->
      <FilterStatusGroup
        label="Статус"
        :options="statuses"
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

      <!-- Тип заявления -->
      <FilterCheckboxGroup
        label="Тип заявления"
        :options="applicationTypes"
        :model-value="selectedApplicationTypes"
        @update:model-value="selectedApplicationTypes = $event"
      />

      <!-- Способ оплаты -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Способ оплаты</span>
          <button
            v-if="selectedPayments.size > 0"
            @click="selectedPayments = new Set()"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2">
          <button
            v-for="method in paymentMethods"
            :key="method"
            @click="togglePayment(method)"
            :class="selectedPayments.has(method)
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ method }}</button>
        </div>
      </div>

      <!-- Проект -->
      <FilterProjectGroup
        label="Проект"
        :groups="projectGroups"
        :model-value="selectedProjects"
        @update:model-value="selectedProjects = $event"
      />

      <!-- Дата регистрации -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата регистрации</span>
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
            :class="selectedRegDateBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="regDateCustom"
            @input="selectedRegDateBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="regDateCustom" @click="selectedRegDateBadge = null; regDateCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
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
import FilterProjectGroup from './FilterProjectGroup.vue'
import FilterManagerGroup from './FilterManagerGroup.vue'
import FilterStatusGroup from './FilterStatusGroup.vue'
import FilterFooter from './FilterFooter.vue'
import {
  X as XIcon,
  Calendar as CalendarIcon,
} from 'lucide-vue-next'

const props = defineProps({ open: Boolean, count: { type: Number, default: 0 } })
const emit = defineEmits(['close', 'update:filterCount', 'update:filters', 'update:filterTags'])

// Проект
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

// Менеджер
const managers = [
  { name: 'Смирнова Юлия',  initials: 'СЮ', role: 'Администратор'     },
  { name: 'Орлов Дмитрий',  initials: 'ОД', role: 'Продажи'           },
  { name: 'Лебедев Игорь',  initials: 'ЛИ', role: 'Менеджер ипотеки'  },
  { name: 'Воронова Анна',  initials: 'ВА', role: 'Менеджер ипотеки'  },
  { name: 'Морозов Сергей', initials: 'МС', role: 'Руководитель'       },
]
const selectedManagers = ref(new Set())

// Статус
const statuses = [
  { name: 'Новая заявка',                                  dot: 'border-zinc-400 bg-zinc-50'   },
  { name: 'Подготовка документов',                         dot: 'border-blue-400 bg-blue-50'   },
  { name: 'Готова к генерации',                             dot: 'border-blue-400 bg-blue-50'   },
  { name: 'Ошибки при генерации заявления',                 dot: 'border-red-400 bg-red-50'     },
  { name: 'Готова к подписанию',                           dot: 'border-blue-400 bg-blue-50'   },
  { name: 'На подписании',                                 dot: 'border-amber-400 bg-amber-50' },
  { name: 'Подписано',                                     dot: 'border-green-400 bg-green-50' },
  { name: 'Проверка не пройдена',                          dot: 'border-red-400 bg-red-50'     },
  { name: 'Ошибка отправки в Росреестр',                    dot: 'border-red-400 bg-red-50'     },
  { name: 'Отправлена в Росреестр',                        dot: 'border-amber-400 bg-amber-50' },
  { name: 'КУВД получен',                                  dot: 'border-amber-400 bg-amber-50' },
  { name: 'Назначены УИНы',                                dot: 'border-amber-400 bg-amber-50' },
  { name: 'В работе у регистратора',                       dot: 'border-amber-400 bg-amber-50' },
  { name: 'Зарегистрирована',                               dot: 'border-green-400 bg-green-50' },
  { name: 'Регистрация приостановлена',                    dot: 'border-orange-400 bg-orange-50' },
  { name: 'Отказ в регистрации',                            dot: 'border-red-400 bg-red-50'     },
  { name: 'Регистрация отменена',                           dot: 'border-red-400 bg-red-50'     },
]
const selectedStatuses = ref(new Set())

// Дата создания
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

// Дата регистрации
const selectedRegDateBadge = ref(null)
const regDateCustom = ref('')

// Способ оплаты
const paymentMethods = [
  'Самостоятельная оплата',
  'Частичная автооплата за покупателя',
  'Полная автооплата',
  'Частичная автооплата за продавца',
]
const selectedPayments = ref(new Set())
const togglePayment = (method) => {
  const s = new Set(selectedPayments.value)
  s.has(method) ? s.delete(method) : s.add(method)
  selectedPayments.value = s
}

// Тип заявления
const applicationTypes = [
  'Соглашение об изменении условий договора уступки',
  'Отправка документов к заявлению',
  'Ипотека в силу закона',
  'Договор уступки прав требования',
  'Соглашение о расторжении договора уступки',
  'Постановка на ГКУ и регистрация прав',
  'Дополнительное соглашение к ДДУ',
  'Договор ДДУ',
  'Соглашение о расторжении ДДУ',
  'Погашение записи об ипотеке',
]
const selectedApplicationTypes = ref(new Set())

const hasActiveFilters = computed(() =>
  selectedProjects.value.size > 0 ||
  selectedManagers.value.size > 0 ||
  selectedStatuses.value.size > 0 ||
  selectedDateBadge.value !== null ||
  dateCustom.value !== '' ||
  selectedRegDateBadge.value !== null ||
  regDateCustom.value !== '' ||
  selectedPayments.value.size > 0 ||
  selectedApplicationTypes.value.size > 0
)

const filterCount = computed(() =>
  selectedProjects.value.size +
  selectedManagers.value.size +
  selectedStatuses.value.size +
  (selectedDateBadge.value !== null || dateCustom.value !== '' ? 1 : 0) +
  (selectedRegDateBadge.value !== null || regDateCustom.value !== '' ? 1 : 0) +
  selectedPayments.value.size +
  selectedApplicationTypes.value.size
)

watch(filterCount, val => emit('update:filterCount', val), { immediate: true })

const parseBadgeRange = (badge, custom) => {
  const str = badge ? badgeToDate(badge) : custom
  if (!str) return { from: null, to: null }
  if (str.includes('—')) {
    const [f, t] = str.split('—').map(s => s.trim())
    return { from: f, to: t }
  }
  return { from: str, to: str }
}

const activeFilters = computed(() => {
  const dateR    = parseBadgeRange(selectedDateBadge.value, dateCustom.value)
  const regDateR = parseBadgeRange(selectedRegDateBadge.value, regDateCustom.value)
  return {
    statuses:         [...selectedStatuses.value],
    managers:         [...selectedManagers.value],
    projects:         [...selectedProjects.value],
    payments:         [...selectedPayments.value],
    applicationTypes: [...selectedApplicationTypes.value],
    dateFrom:         dateR.from,
    dateTo:           dateR.to,
    regDateFrom:      regDateR.from,
    regDateTo:        regDateR.to,
  }
})
watch(activeFilters, val => emit('update:filters', val), { immediate: true, deep: true })

const activeFilterTags = computed(() => {
  const tags = []
  if (selectedDateBadge.value || dateCustom.value)
    tags.push({ label: `Дата создания: ${selectedDateBadge.value || dateCustom.value}`, reset: () => { selectedDateBadge.value = null; dateCustom.value = '' } })
  selectedStatuses.value.forEach(s =>
    tags.push({ label: s, reset: () => { const set = new Set(selectedStatuses.value); set.delete(s); selectedStatuses.value = set } })
  )
  selectedManagers.value.forEach(m =>
    tags.push({ label: m, reset: () => { const set = new Set(selectedManagers.value); set.delete(m); selectedManagers.value = set } })
  )
  selectedProjects.value.forEach(p =>
    tags.push({ label: p, reset: () => { const set = new Set(selectedProjects.value); set.delete(p); selectedProjects.value = set } })
  )
  if (selectedRegDateBadge.value || regDateCustom.value)
    tags.push({ label: `Дата регистрации: ${selectedRegDateBadge.value || regDateCustom.value}`, reset: () => { selectedRegDateBadge.value = null; regDateCustom.value = '' } })
  selectedPayments.value.forEach(p =>
    tags.push({ label: p, reset: () => { const set = new Set(selectedPayments.value); set.delete(p); selectedPayments.value = set } })
  )
  selectedApplicationTypes.value.forEach(t =>
    tags.push({ label: t, reset: () => { const set = new Set(selectedApplicationTypes.value); set.delete(t); selectedApplicationTypes.value = set } })
  )
  return tags
})
watch(activeFilterTags, val => emit('update:filterTags', val), { immediate: true })

const resetAll = () => {
  selectedProjects.value    = new Set()
  selectedManagers.value    = new Set()
  selectedStatuses.value    = new Set()
  selectedDateBadge.value    = null
  dateCustom.value           = ''
  selectedRegDateBadge.value = null
  regDateCustom.value        = ''
  selectedPayments.value         = new Set()
  selectedApplicationTypes.value = new Set()
}

defineExpose({ resetAll })
</script>
