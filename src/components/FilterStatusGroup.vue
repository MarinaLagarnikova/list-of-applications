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

    <div class="flex flex-col gap-y-3">
      <!-- Все -->
      <div class="flex items-center justify-between cursor-pointer" @click="toggleAll">
        <span class="text-[14px] font-light text-zinc-900">Все</span>
        <span :class="cbClass(allSelected)">
          <svg :class="['size-3 stroke-white transition-opacity', allSelected ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
            <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
        </span>
      </div>

      <!-- Опции -->
      <div
        v-for="option in displayedOptions"
        :key="option.name"
        class="flex items-center justify-between cursor-pointer"
        @click="toggle(option.name)"
      >
        <div class="flex items-center gap-x-2.5">
          <span class="flex w-6 shrink-0 items-center justify-center">
            <span :class="['size-3 rounded-[4px] border', option.dot]" />
          </span>
          <span class="text-[14px] font-light text-zinc-900">{{ option.name }}</span>
        </div>
        <span :class="cbClass(modelValue.has(option.name))">
          <svg :class="['size-3 stroke-white transition-opacity', modelValue.has(option.name) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none">
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
import { ChevronDown as ChevronDownIcon } from 'lucide-vue-next'

const VISIBLE_COUNT = 8

const props = defineProps({
  label: { type: String, required: true },
  options: { type: Array, required: true }, // [{ name, dot }]
  modelValue: { type: Set, required: true },
})
const emit = defineEmits(['update:modelValue'])

const expanded = ref(false)

const displayedOptions = computed(() => {
  if (expanded.value || props.options.length <= VISIBLE_COUNT) return props.options
  const first = props.options.slice(0, VISIBLE_COUNT)
  const selectedBeyond = props.options.slice(VISIBLE_COUNT).filter(o => props.modelValue.has(o.name))
  return [...first, ...selectedBeyond]
})

const hasMore = computed(() => props.options.length > VISIBLE_COUNT)

const hiddenCount = computed(() => {
  const selectedBeyond = props.options.slice(VISIBLE_COUNT).filter(o => props.modelValue.has(o.name)).length
  return props.options.length - VISIBLE_COUNT - selectedBeyond
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
