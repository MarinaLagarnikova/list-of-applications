<template>
  <div class="shrink-0 border-t border-[#f4f4f5] px-6 py-4 flex flex-col gap-y-3">

    <FilterTagsBar
      v-if="tags.length"
      :tags="tags"
      @reset="emit('reset')"
    />

    <button
      @click="!isLoading && count > 0 && emit('close')"
      :class="['flex w-full items-center justify-center rounded-lg h-9 px-4 text-[14px] font-medium text-white transition-colors',
        isLoading ? 'bg-indigo-600 cursor-default' : count > 0 ? 'bg-indigo-600 hover:bg-indigo-500' : 'bg-indigo-300 cursor-default']"
    >
      <span v-if="isLoading" class="flex items-center gap-x-1">
        <span class="size-1.5 rounded-full bg-white animate-bounce" style="animation-delay: 0ms" />
        <span class="size-1.5 rounded-full bg-white animate-bounce" style="animation-delay: 150ms" />
        <span class="size-1.5 rounded-full bg-white animate-bounce" style="animation-delay: 300ms" />
      </span>
      <span v-else>{{ buttonLabel }}</span>
    </button>

  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import FilterTagsBar from './FilterTagsBar.vue'

const props = defineProps({
  count: { type: Number, default: 0 },
  hasActiveFilters: { type: Boolean, default: false },
  filterCount: { type: Number, default: 0 },
  tags: { type: Array, default: () => [] },
})
const emit = defineEmits(['close', 'reset'])

const isLoading = ref(false)
let loadingTimer = null

watch(() => props.filterCount, () => {
  isLoading.value = true
  clearTimeout(loadingTimer)
  loadingTimer = setTimeout(() => { isLoading.value = false }, 700)
})

const buttonLabel = computed(() => {
  if (props.count === 0) return 'Заявок нет'
  const n = props.count, m10 = n % 10, m100 = n % 100
  if (m100 >= 11 && m100 <= 14) return `Показать ${n} заявок`
  if (m10 === 1) return `Показать ${n} заявку`
  if (m10 >= 2 && m10 <= 4) return `Показать ${n} заявки`
  return `Показать ${n} заявок`
})
</script>
