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

      <!-- Способ идентификации -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Способ идентификации</span>
          <button
            v-if="selectedMethods.size > 0"
            @click="selectedMethods = new Set()"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2">
          <button
            v-for="method in signMethods"
            :key="method.name"
            @click="toggleMethod(method.name)"
            :class="selectedMethods.has(method.name)
              ? 'flex items-center gap-1.5 rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'flex items-center gap-1.5 rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >
            <component :is="method.icon" :size="12" />
            {{ method.name }}
          </button>
        </div>
      </div>

      <!-- Дата активации -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата активации</span>
          <button
            v-if="selectedActivationBadge || activationCustom"
            @click="selectedActivationBadge = null; activationCustom = ''"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedActivationBadge === badge ? (selectedActivationBadge = null, activationCustom = '') : (selectedActivationBadge = badge, activationCustom = badgeToDate(badge))"
            :class="selectedActivationBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="activationCustom"
            @input="selectedActivationBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="activationCustom" @click="selectedActivationBadge = null; activationCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

      <!-- Действует до -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Действует до</span>
          <button
            v-if="selectedExpiryBadge || expiryCustom"
            @click="selectedExpiryBadge = null; expiryCustom = ''"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedExpiryBadge === badge ? (selectedExpiryBadge = null, expiryCustom = '') : (selectedExpiryBadge = badge, expiryCustom = badgeToDate(badge))"
            :class="selectedExpiryBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="expiryCustom"
            @input="selectedExpiryBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="expiryCustom" @click="selectedExpiryBadge = null; expiryCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
            <XIcon :size="16" />
          </button>
          <CalendarIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
        </div>
      </div>

      <!-- Дата встречи с курьером -->
      <div class="px-6 py-4">
        <div class="flex items-center justify-between mb-3">
          <span class="text-[14px] leading-[24px] font-medium text-zinc-900">Дата встречи с курьером</span>
          <button
            v-if="selectedCourierBadge || courierCustom"
            @click="selectedCourierBadge = null; courierCustom = ''"
            class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
          >Сбросить</button>
        </div>
        <div class="flex flex-wrap gap-2 mb-3">
          <button
            v-for="badge in dateBadges"
            :key="badge"
            @click="selectedCourierBadge === badge ? (selectedCourierBadge = null, courierCustom = '') : (selectedCourierBadge = badge, courierCustom = badgeToDate(badge))"
            :class="selectedCourierBadge === badge
              ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
              : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
          >{{ badge }}</button>
        </div>
        <div class="flex items-center rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition">
          <input
            type="text"
            placeholder=""
            v-model="courierCustom"
            @input="selectedCourierBadge = null"
            class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
          />
          <button v-if="courierCustom" @click="selectedCourierBadge = null; courierCustom = ''" class="shrink-0 mr-4 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
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
import FilterManagerGroup from './FilterManagerGroup.vue'
import FilterStatusGroup from './FilterStatusGroup.vue'
import FilterFooter from './FilterFooter.vue'
import {
  X as XIcon,
  Calendar as CalendarIcon,
  UserRound as UserRoundIcon,
  Landmark as LandmarkIcon,
  IdCard as IdCardIcon,
} from 'lucide-vue-next'

const props = defineProps({ open: Boolean, count: { type: Number, default: 0 } })
const emit = defineEmits(['close', 'update:filterCount', 'update:filters', 'update:filterTags'])

// Даты
const dateBadges = ['Сегодня', 'Вчера', 'Неделя', '2 недели', 'Май', '2 квартал']
const selectedActivationBadge = ref(null)
const activationCustom = ref('')
const selectedCourierBadge = ref(null)
const courierCustom = ref('')
const selectedDateBadge = ref(null)
const dateCustom = ref('')
const selectedExpiryBadge = ref(null)
const expiryCustom = ref('')

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

// Способ идентификации
const signMethods = [
  { name: 'Курьер',                 icon: UserRoundIcon },
  { name: 'Офис',                   icon: LandmarkIcon  },
  { name: 'Паспорт с биометрией',   icon: IdCardIcon    },
]
const selectedMethods = ref(new Set())
const toggleMethod = (name) => {
  const s = new Set(selectedMethods.value)
  s.has(name) ? s.delete(name) : s.add(name)
  selectedMethods.value = s
}

// Статус
const statuses = [
  { name: 'Активирована',                  dot: 'border-green-400 bg-green-50'  },
  { name: 'Заявление создано в УЦ',        dot: 'border-zinc-400 bg-zinc-50'    },
  { name: 'Процесс выпуска создан',        dot: 'border-blue-400 bg-blue-50'    },
  { name: 'Ожидает назначение курьера',    dot: 'border-amber-400 bg-amber-50'  },
  { name: 'Встреча с курьером назначена',  dot: 'border-amber-400 bg-amber-50'  },
  { name: 'Встреча с курьером отменена',   dot: 'border-red-400 bg-red-50'      },
  { name: 'Встреча с курьером состоялась', dot: 'border-blue-400 bg-blue-50'    },
  { name: 'Запрос на выпуск истек',        dot: 'border-red-400 bg-red-50'      },
]
const selectedStatuses = ref(new Set())

// Менеджер
const managers = [
  { name: 'Смирнова Юлия',  initials: 'СЮ', role: 'Администратор'     },
  { name: 'Орлов Дмитрий',  initials: 'ОД', role: 'Продажи'           },
  { name: 'Лебедев Игорь',  initials: 'ЛИ', role: 'Менеджер ипотеки'  },
  { name: 'Воронова Анна',  initials: 'ВА', role: 'Менеджер ипотеки'  },
  { name: 'Морозов Сергей', initials: 'МС', role: 'Руководитель'       },
]
const selectedManagers = ref(new Set())

const hasActiveFilters = computed(() =>
  selectedActivationBadge.value !== null ||
  activationCustom.value !== '' ||
  selectedCourierBadge.value !== null ||
  courierCustom.value !== '' ||
  selectedDateBadge.value !== null ||
  dateCustom.value !== '' ||
  selectedExpiryBadge.value !== null ||
  expiryCustom.value !== '' ||
  selectedMethods.value.size > 0 ||
  selectedStatuses.value.size > 0 ||
  selectedManagers.value.size > 0
)

const filterCount = computed(() =>
  (selectedActivationBadge.value !== null || activationCustom.value !== '' ? 1 : 0) +
  (selectedCourierBadge.value !== null || courierCustom.value !== '' ? 1 : 0) +
  (selectedDateBadge.value !== null || dateCustom.value !== '' ? 1 : 0) +
  (selectedExpiryBadge.value !== null || expiryCustom.value !== '' ? 1 : 0) +
  selectedMethods.value.size +
  selectedStatuses.value.size +
  selectedManagers.value.size
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
  const dateR       = parseBadgeRange(selectedDateBadge.value,       dateCustom.value)
  const activationR = parseBadgeRange(selectedActivationBadge.value, activationCustom.value)
  const expiryR     = parseBadgeRange(selectedExpiryBadge.value,     expiryCustom.value)
  const courierR    = parseBadgeRange(selectedCourierBadge.value,    courierCustom.value)
  return {
    statuses:       [...selectedStatuses.value],
    managers:       [...selectedManagers.value],
    methods:        [...selectedMethods.value],
    dateFrom:       dateR.from,
    dateTo:         dateR.to,
    activationFrom: activationR.from,
    activationTo:   activationR.to,
    expiryFrom:     expiryR.from,
    expiryTo:       expiryR.to,
    courierFrom:    courierR.from,
    courierTo:      courierR.to,
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
  selectedMethods.value.forEach(method =>
    tags.push({ label: method, reset: () => { const set = new Set(selectedMethods.value); set.delete(method); selectedMethods.value = set } })
  )
  if (selectedActivationBadge.value || activationCustom.value)
    tags.push({ label: `Дата активации: ${selectedActivationBadge.value || activationCustom.value}`, reset: () => { selectedActivationBadge.value = null; activationCustom.value = '' } })
  if (selectedExpiryBadge.value || expiryCustom.value)
    tags.push({ label: `Действует до: ${selectedExpiryBadge.value || expiryCustom.value}`, reset: () => { selectedExpiryBadge.value = null; expiryCustom.value = '' } })
  if (selectedCourierBadge.value || courierCustom.value)
    tags.push({ label: `Дата встречи с курьером: ${selectedCourierBadge.value || courierCustom.value}`, reset: () => { selectedCourierBadge.value = null; courierCustom.value = '' } })
  return tags
})
watch(activeFilterTags, val => emit('update:filterTags', val), { immediate: true })

const resetAll = () => {
  selectedActivationBadge.value = null
  activationCustom.value        = ''
  selectedCourierBadge.value    = null
  courierCustom.value           = ''
  selectedDateBadge.value       = null
  dateCustom.value              = ''
  selectedExpiryBadge.value     = null
  expiryCustom.value            = ''
  selectedMethods.value  = new Set()
  selectedStatuses.value = new Set()
  selectedManagers.value = new Set()
}

defineExpose({ resetAll })
</script>
