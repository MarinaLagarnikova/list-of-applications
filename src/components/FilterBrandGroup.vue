<template>
  <div class="px-6 py-4">
    <div class="flex items-center justify-between mb-3">
      <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">{{ label }}</span>
      <button
        v-if="modelValue.size > 0"
        @click="emit('update:modelValue', new Set())"
        class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
      >Сбросить</button>
    </div>

    <div class="flex items-center gap-x-2 rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition mb-5">
      <SearchIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
      <input
        type="text"
        :placeholder="searchPlaceholder"
        v-model="searchQuery"
        class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-zinc-500 placeholder:font-normal focus:outline-none"
      />
      <button
        v-if="searchQuery"
        @click="searchQuery = ''"
        class="shrink-0 text-[#a1a1aa] hover:text-zinc-600 transition-colors"
      >
        <XIcon :size="16" />
      </button>
    </div>

    <!-- Все -->
    <div v-if="!searchQuery" class="flex items-center justify-between cursor-pointer" @click="toggleAll">
      <span class="text-[14px] font-light text-[#18181b]">Все</span>
      <span :class="cbClass(allSelected)">
        <svg :class="['size-3 stroke-white transition-opacity', allSelected ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
          <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
      </span>
    </div>

    <!-- Пустой поиск -->
    <div v-if="searchQuery && displayedOptions.length === 0" class="flex flex-col items-center justify-center py-6 text-center">
      <FolderSearchIcon :size="32" class="mx-auto text-zinc-400" />
      <p class="mt-2 text-[13px] font-medium text-zinc-900">{{ emptyText }}</p>
    </div>

    <!-- Список -->
    <div :class="['flex flex-col gap-y-3', searchQuery ? '' : 'mt-5']">
      <div
        v-for="item in displayedOptions"
        :key="item.name"
        class="flex items-center justify-between cursor-pointer"
        @click="toggle(item.name)"
      >
        <div class="flex items-center gap-x-2.5">
          <span
            class="inline-flex size-6 shrink-0 items-center justify-center rounded-full"
            :style="{ backgroundColor: hexToRgba(item.color, 0.15) }"
          >
            <span class="text-xs font-medium" :style="{ color: item.color }">{{ item.initial }}</span>
          </span>
          <span class="text-[14px] font-light text-[#18181b]">{{ item.name }}</span>
        </div>
        <span :class="cbClass(modelValue.has(item.name))">
          <svg :class="['size-3 stroke-white transition-opacity', modelValue.has(item.name) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
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

const hexToRgba = (hex, alpha) => {
  const r = parseInt(hex.slice(1, 3), 16)
  const g = parseInt(hex.slice(3, 5), 16)
  const b = parseInt(hex.slice(5, 7), 16)
  return `rgba(${r}, ${g}, ${b}, ${alpha})`
}

const props = defineProps({
  label: { type: String, required: true },
  options: { type: Array, required: true }, // [{ name, initial, color }]
  modelValue: { type: Set, required: true },
  searchPlaceholder: { type: String, default: 'Название' },
  emptyText: { type: String, default: 'Ничего не найдено' },
})
const emit = defineEmits(['update:modelValue'])

const searchQuery = ref('')
const expanded = ref(false)

const filteredOptions = computed(() => {
  const base = searchQuery.value.trim()
    ? props.options.filter(o => o.name.toLowerCase().includes(searchQuery.value.trim().toLowerCase()))
    : props.options
  return [...base].sort((a, b) => a.name.localeCompare(b.name, 'ru'))
})

const displayedOptions = computed(() => {
  const base = filteredOptions.value
  if (searchQuery.value || expanded.value || base.length <= VISIBLE_COUNT) return base
  const first = base.slice(0, VISIBLE_COUNT)
  const selectedBeyond = base.slice(VISIBLE_COUNT).filter(o => props.modelValue.has(o.name))
  return [...first, ...selectedBeyond]
})

const hasMore = computed(() =>
  !searchQuery.value && filteredOptions.value.length > VISIBLE_COUNT
)

const hiddenCount = computed(() => {
  const base = filteredOptions.value
  const selectedBeyond = base.slice(VISIBLE_COUNT).filter(o => props.modelValue.has(o.name)).length
  return base.length - VISIBLE_COUNT - selectedBeyond
})

const allSelected = computed(() =>
  props.options.length > 0 && props.modelValue.size === props.options.length
)

const cbClass = (checked) => [
  'relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors',
  checked ? 'bg-indigo-600 border-indigo-600' : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]',
]

const toggle = (name) => {
  const s = new Set(props.modelValue)
  s.has(name) ? s.delete(name) : s.add(name)
  emit('update:modelValue', s)
}

const toggleAll = () => {
  emit('update:modelValue', allSelected.value ? new Set() : new Set(props.options.map(o => o.name)))
}
</script>
