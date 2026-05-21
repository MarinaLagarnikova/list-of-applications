<template>
  <div class="relative" ref="container">
    <button
      @click="open = !open"
      class="inline-flex h-8 items-center gap-x-1.5 rounded-lg border border-zinc-950/10 bg-white px-3 text-sm/6 font-medium text-zinc-900 hover:bg-zinc-50 transition-colors"
    >
      <ArrowDownIcon v-if="modelValue === 'new'" :size="14" class="shrink-0 text-zinc-500" />
      <ArrowUpIcon v-else :size="14" class="shrink-0 text-zinc-500" />
      {{ modelValue === 'new' ? 'Сначала новые' : 'Сначала старые' }}
      <ChevronDownIcon :size="14" :class="['shrink-0 text-zinc-500 transition-transform duration-150', open ? 'rotate-180' : '']" />
    </button>
    <div
      v-if="open"
      class="absolute right-0 z-20 mt-1 w-44 rounded-lg bg-white py-1 shadow-lg ring-1 ring-black/5"
    >
      <button
        v-for="opt in options"
        :key="opt.value"
        @click="select(opt.value)"
        class="flex w-full items-center gap-x-2 px-3 py-2 text-[14px] text-zinc-900 hover:bg-zinc-50"
        :class="modelValue === opt.value ? 'font-medium' : 'font-light'"
      >
        <CheckIcon v-if="modelValue === opt.value" :size="14" class="shrink-0 text-indigo-600" />
        <span v-else class="size-3.5 shrink-0" />
        {{ opt.label }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { ArrowDown as ArrowDownIcon, ArrowUp as ArrowUpIcon, ChevronDown as ChevronDownIcon, Check as CheckIcon } from 'lucide-vue-next'

const props = defineProps({
  modelValue: { type: String, default: 'new' },
})
const emit = defineEmits(['update:modelValue'])

const open = ref(false)
const container = ref(null)

const options = [
  { value: 'new', label: 'Сначала новые' },
  { value: 'old', label: 'Сначала старые' },
]

const select = (value) => {
  emit('update:modelValue', value)
  open.value = false
}

const handleClickOutside = (e) => {
  if (container.value && !container.value.contains(e.target)) open.value = false
}

onMounted(() => document.addEventListener('click', handleClickOutside))
onUnmounted(() => document.removeEventListener('click', handleClickOutside))
</script>
