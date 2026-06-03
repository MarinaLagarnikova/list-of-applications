<template>
  <div class="px-6 py-4">
    <div class="flex items-center justify-between mb-3">
      <span class="text-[14px] leading-[24px] font-medium text-zinc-900">{{ label }}</span>
      <button
        v-if="modelValue.size > 0"
        @click="emit('update:modelValue', new Set())"
        class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
      >Сбросить</button>
    </div>

    <!-- Поиск -->
    <div v-if="showSearch" class="flex items-center gap-x-2 rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition mb-3">
      <SearchIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
      <input
        type="text"
        placeholder="Название"
        v-model="searchQuery"
        class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-zinc-500 placeholder:font-normal focus:outline-none"
      />
      <button v-if="searchQuery" @click="searchQuery = ''" class="shrink-0 text-[#a1a1aa] hover:text-zinc-600 transition-colors">
        <XIcon :size="16" />
      </button>
    </div>

    <!-- Теги городов (только если есть поиск) -->
    <div v-if="showSearch && !searchQuery" class="flex flex-wrap gap-2 mb-5">
      <button
        v-for="group in groups"
        :key="group.city"
        @click="toggleCity(group)"
        :class="citySelected(group)
          ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
          : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
      >{{ group.city }}</button>
    </div>

    <!-- Все -->
    <div v-if="showAll && !searchQuery" class="flex items-center justify-between cursor-pointer" @click="toggleAll">
      <span class="text-[14px] font-light text-zinc-900">Все</span>
      <span :class="cbClass(allSelected)">
        <svg :class="['size-3 stroke-white transition-opacity', allSelected ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
          <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
      </span>
    </div>

    <!-- Пустой стейт поиска -->
    <div v-if="searchQuery && displayedOptions.length === 0" class="flex flex-col items-center justify-center py-6 text-center">
      <FolderSearchIcon :size="32" class="mx-auto text-zinc-400" />
      <p class="mt-2 text-[13px] font-medium text-zinc-900">Ничего не найдено</p>
    </div>

    <!-- Список проектов -->
    <div :class="['flex flex-col gap-y-3', searchQuery ? '' : 'mt-5']">
      <div
        v-for="item in displayedOptions"
        :key="item.value"
        class="flex items-center justify-between cursor-pointer"
        @click="toggle(item.value)"
      >
        <div class="flex items-center gap-x-2">
          <span class="text-[14px] font-light text-zinc-900">{{ item.name }}</span>
          <span v-if="!hideGroupLabel" class="text-[14px] font-light text-zinc-500">{{ item.city }}</span>
        </div>
        <span :class="cbClass(modelValue.has(item.value))">
          <svg :class="['size-3 stroke-white transition-opacity', modelValue.has(item.value) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
            <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
        </span>
      </div>
    </div>

    <!-- Показать ещё / Свернуть -->
    <button
      v-if="hasMore"
      @click="expanded = !expanded"
      class="mt-3 flex items-center gap-x-1 text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
    >
      <span>{{ expanded ? 'Свернуть' : `Показать ещё ${hiddenCount}` }}</span>
      <ChevronDownIcon :size="16" :class="['transition-transform duration-200', expanded ? 'rotate-180' : '']" />
    </button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { Search as SearchIcon, X as XIcon, FolderSearch as FolderSearchIcon, ChevronDown as ChevronDownIcon } from 'lucide-vue-next'

const VISIBLE_COUNT = 8

const props = defineProps({
  label: { type: String, required: true },
  groups: { type: Array, required: true }, // [{ city: string, options: string[] }]
  modelValue: { type: Set, required: true },
  hideGroupLabel: { type: Boolean, default: false },
  showSearch: { type: Boolean, default: true },
  showAll: { type: Boolean, default: true },
})
const emit = defineEmits(['update:modelValue'])

const searchQuery = ref('')
const expanded = ref(false)

// Плоский список со всеми данными
const allItems = computed(() =>
  props.groups.flatMap(g =>
    g.options.map(v => ({
      value: v,
      name: stripCity(v),
      city: g.city,
    }))
  )
)

const stripCity = (value) => {
  const name = value.includes('/') ? value.split('/').slice(1).join('/').trim() : value
  return name.replace(/\s*\([^)]+\)\s*$/, '').trim()
}

const filteredItems = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  const base = q
    ? allItems.value.filter(i => i.name.toLowerCase().includes(q) || i.city.toLowerCase().includes(q))
    : allItems.value
  return [...base].sort((a, b) => a.name.localeCompare(b.name, 'ru'))
})

const displayedOptions = computed(() => {
  const base = filteredItems.value
  if (searchQuery.value || expanded.value || base.length <= VISIBLE_COUNT) return base
  const first = base.slice(0, VISIBLE_COUNT)
  const selectedBeyond = base.slice(VISIBLE_COUNT).filter(i => props.modelValue.has(i.value))
  return [...first, ...selectedBeyond]
})

const hasMore = computed(() =>
  !searchQuery.value && filteredItems.value.length > VISIBLE_COUNT
)

const hiddenCount = computed(() => {
  const base = filteredItems.value
  const selectedBeyond = base.slice(VISIBLE_COUNT).filter(i => props.modelValue.has(i.value)).length
  return base.length - VISIBLE_COUNT - selectedBeyond
})

const allSelected = computed(() =>
  allItems.value.length > 0 && allItems.value.every(i => props.modelValue.has(i.value))
)

const citySelected = (group) =>
  group.options.length > 0 && group.options.every(v => props.modelValue.has(v))

const cbClass = (checked) => [
  'relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors',
  checked ? 'bg-indigo-600 border-indigo-600' : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]',
]

const toggle = (value) => {
  const s = new Set(props.modelValue)
  s.has(value) ? s.delete(value) : s.add(value)
  emit('update:modelValue', s)
}

const toggleAll = () => {
  emit('update:modelValue', allSelected.value ? new Set() : new Set(allItems.value.map(i => i.value)))
}

const toggleCity = (group) => {
  const s = new Set(props.modelValue)
  if (citySelected(group)) {
    group.options.forEach(v => s.delete(v))
  } else {
    group.options.forEach(v => s.add(v))
  }
  emit('update:modelValue', s)
}
</script>
