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

    <div v-if="searchable" class="flex items-center gap-x-2 rounded-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition mb-3">
      <SearchIcon :size="16" class="shrink-0 text-[#a1a1aa] pointer-events-none" />
      <input
        type="text"
        :placeholder="searchPlaceholder"
        v-model="searchQuery"
        class="flex-1 min-w-0 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] focus:outline-none"
      />
    </div>

    <div class="flex flex-col gap-y-3">
      <!-- Все -->
      <div class="flex items-center justify-between cursor-pointer" @click="toggleAll">
        <span class="text-[14px] font-light text-[#18181b]">Все</span>
        <span :class="cbClass(allSelected)">
          <svg :class="['size-3 stroke-white transition-opacity', allSelected ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
            <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
        </span>
      </div>
      <!-- Опции -->
      <div
        v-for="option in displayedOptions"
        :key="option"
        class="flex items-center justify-between cursor-pointer"
        @click="toggle(option)"
      >
        <span class="text-[14px] font-light text-[#18181b]">{{ option }}</span>
        <span :class="cbClass(modelValue.has(option))">
          <svg :class="['size-3 stroke-white transition-opacity', modelValue.has(option) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
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
import { Search as SearchIcon, ChevronDown as ChevronDownIcon } from 'lucide-vue-next'

const VISIBLE_COUNT = 8

const props = defineProps({
  label: { type: String, required: true },
  options: { type: Array, required: true },
  modelValue: { type: Set, required: true },
  searchable: { type: Boolean, default: false },
  searchPlaceholder: { type: String, default: 'Поиск' },
})
const emit = defineEmits(['update:modelValue'])

const searchQuery = ref('')
const expanded = ref(false)

const filteredOptions = computed(() => {
  const base = (props.searchable && searchQuery.value.trim())
    ? props.options.filter(o => o.toLowerCase().includes(searchQuery.value.trim().toLowerCase()))
    : props.options
  return [...base].sort((a, b) => a.localeCompare(b, 'ru'))
})

const displayedOptions = computed(() => {
  const base = filteredOptions.value
  if (searchQuery.value || expanded.value || base.length <= VISIBLE_COUNT) return base
  const first = base.slice(0, VISIBLE_COUNT)
  const selectedBeyond = base.slice(VISIBLE_COUNT).filter(o => props.modelValue.has(o))
  return [...first, ...selectedBeyond]
})

const hasMore = computed(() =>
  !searchQuery.value && filteredOptions.value.length > VISIBLE_COUNT
)

const hiddenCount = computed(() => {
  const base = filteredOptions.value
  const selectedBeyond = base.slice(VISIBLE_COUNT).filter(o => props.modelValue.has(o)).length
  return base.length - VISIBLE_COUNT - selectedBeyond
})

const allSelected = computed(() =>
  props.options.length > 0 && props.modelValue.size === props.options.length
)

const cbClass = (checked) => [
  'relative flex size-4 shrink-0 items-center justify-center rounded-sm border transition-colors',
  checked ? 'bg-indigo-600 border-indigo-600' : 'bg-white border-[#d4d4d8] hover:border-[#a1a1aa]',
]

const toggle = (option) => {
  const s = new Set(props.modelValue)
  s.has(option) ? s.delete(option) : s.add(option)
  emit('update:modelValue', s)
}

const toggleAll = () => {
  emit('update:modelValue', allSelected.value ? new Set() : new Set(props.options))
}
</script>
