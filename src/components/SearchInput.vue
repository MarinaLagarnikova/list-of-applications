<template>
  <div class="relative flex flex-1 items-center gap-x-5 mr-8">
    <Search :size="16" :stroke-width="2" style="stroke:#A1A1AA;" class="shrink-0 aspect-square" aria-hidden="true" />
    <input
      type="text"
      :placeholder="placeholder"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      @focus="onFocus"
      @blur="onBlur"
      @keydown.enter="onEnter"
      @keydown.escape="isOpen = false"
      class="bg-transparent text-[14px] font-light leading-normal text-[#18181b] placeholder:text-[#71717a] focus:placeholder:text-zinc-400 focus:outline-none w-full"
    />

    <Transition
      enter-active-class="transition ease-out duration-200"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition ease-in duration-150"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div
        v-if="isOpen && history.length > 0"
        class="absolute top-[calc(100%+14px)] left-9 z-50 w-[300px] origin-top-left rounded-xl bg-white/75 p-1 shadow-lg ring-1 ring-zinc-950/10 backdrop-blur-xl"
      >
        <div class="px-3 py-2">
          <div class="text-xs text-zinc-500">Вы искали</div>
        </div>
        <button
          v-for="item in history.slice(0, 5)"
          :key="item"
          class="group flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-indigo-600 hover:text-white transition-colors text-left"
          @mousedown.prevent="select(item)"
        >
          <Clock class="size-4 shrink-0 text-zinc-500 group-hover:text-white" />
          <span class="truncate">{{ item }}</span>
        </button>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { Search, Clock } from 'lucide-vue-next'

const props = defineProps({
  modelValue: { type: String, default: '' },
  placeholder: { type: String, default: 'Поиск' },
})
const emit = defineEmits(['update:modelValue'])

const STORAGE_KEY = 'app-search-history'
const MAX_ITEMS = 7

const isOpen = ref(false)
const isFocused = ref(false)
const history = ref(loadHistory())

function loadHistory() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
  } catch {
    return []
  }
}

function saveToHistory(query) {
  const q = query.trim()
  if (!q || q.length < 2) return
  const list = history.value.filter(h => h !== q)
  list.unshift(q)
  history.value = list.slice(0, MAX_ITEMS)
  localStorage.setItem(STORAGE_KEY, JSON.stringify(history.value))
}

function onFocus() {
  isFocused.value = true
  if (!props.modelValue && history.value.length > 0) isOpen.value = true
}

function onBlur() {
  isFocused.value = false
  setTimeout(() => { isOpen.value = false }, 150)
  if (props.modelValue.trim().length >= 2) saveToHistory(props.modelValue)
}

function onEnter() {
  saveToHistory(props.modelValue)
  isOpen.value = false
}

function select(item) {
  emit('update:modelValue', item)
  isOpen.value = false
}

watch(() => props.modelValue, (val) => {
  if (val) isOpen.value = false
  else if (isFocused.value && history.value.length > 0) isOpen.value = true
})
</script>
