<template>
  <div ref="containerRef" class="relative">

    <!-- Невидимая строка для замера ширин всех чипов (всегда в DOM) -->
    <div
      class="absolute top-0 left-0 flex items-center gap-x-2 opacity-0 pointer-events-none"
      aria-hidden="true"
    >
      <span
        v-for="(tag, i) in tags"
        :key="'m' + i"
        ref="measureRefs"
        class="inline-flex shrink-0 items-center gap-x-0.5 rounded-md bg-zinc-50 px-2.5 py-1 text-xs font-medium text-zinc-700 inset-ring inset-ring-zinc-700/10"
      >{{ tag.label }}<span class="inline-block size-3.5 -mr-1 shrink-0" /></span>
    </div>

    <!-- Строка 1: первые видимые чипы + кнопка Ещё (свёрнуто) + Сбросить (только если не раскрыто) -->
    <div class="flex items-center overflow-hidden">
      <div class="flex items-center gap-x-2">
        <span
          v-for="(tag, i) in tags"
          v-show="i < visibleCount"
          :key="tag.label"
          class="inline-flex shrink-0 items-center gap-x-0.5 rounded-md bg-zinc-50 px-2.5 py-1 text-xs font-medium text-zinc-700 inset-ring inset-ring-zinc-700/10"
        >
          {{ tag.label }}
          <button
            type="button"
            @click="tag.reset()"
            class="group relative -mr-1 size-3.5 rounded-sm hover:bg-zinc-600/20"
          >
            <span class="sr-only">Удалить</span>
            <svg viewBox="0 0 14 14" class="size-3.5 stroke-zinc-400 group-hover:stroke-zinc-600">
              <path d="M4 4l6 6m0-6l-6 6" fill="none" stroke-width="1.5" stroke-linecap="round" />
            </svg>
            <span class="absolute -inset-1" />
          </button>
        </span>
      </div>

      <!-- Ещё N (только когда свёрнуто) -->
      <Button
        v-if="hiddenCount > 0 && !expanded"
        outline xs
        class="shrink-0 ml-2 whitespace-nowrap"
        @click="expanded = true"
      >Ещё {{ hiddenCount }} <ChevronDownIcon :size="12" /></Button>

      <!-- Сбросить все — только на первой строке когда НЕ раскрыто -->
      <Button
        v-if="!expanded"
        ref="resetBtnRef"
        plain color="indigo" xs
        class="shrink-0 ml-6 whitespace-nowrap"
        @click="$emit('reset')"
      >Сбросить · {{ tags.length }}</Button>
    </div>

    <!-- Строка 2: оставшиеся чипы + Свернуть + Сбросить (при expanded) -->
    <div v-if="expanded && hiddenCount > 0" class="flex flex-wrap items-center gap-2 mt-2">
      <span
        v-for="(tag, i) in tags"
        v-show="i >= visibleCount"
        :key="'exp-' + tag.label"
        class="inline-flex shrink-0 items-center gap-x-0.5 rounded-md bg-zinc-50 px-2.5 py-1 text-xs font-medium text-zinc-700 inset-ring inset-ring-zinc-700/10"
      >
        {{ tag.label }}
        <button
          type="button"
          @click="tag.reset()"
          class="group relative -mr-1 size-3.5 rounded-sm hover:bg-zinc-600/20"
        >
          <span class="sr-only">Удалить</span>
          <svg viewBox="0 0 14 14" class="size-3.5 stroke-zinc-400 group-hover:stroke-zinc-600">
            <path d="M4 4l6 6m0-6l-6 6" fill="none" stroke-width="1.5" stroke-linecap="round" />
          </svg>
          <span class="absolute -inset-1" />
        </button>
      </span>
      <Button
        outline xs
        class="shrink-0 whitespace-nowrap"
        @click="expanded = false"
      >Свернуть <ChevronUpIcon :size="12" /></Button>
      <Button
        plain color="indigo" xs
        class="shrink-0 whitespace-nowrap"
        @click="$emit('reset')"
      >Сбросить · {{ tags.length }}</Button>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'
import Button from './Button.vue'
import { ChevronDown as ChevronDownIcon, ChevronUp as ChevronUpIcon } from 'lucide-vue-next'

const props = defineProps({
  tags: { type: Array, default: () => [] },
})
const emit = defineEmits(['reset'])

const containerRef = ref(null)
const measureRefs  = ref([])
const resetBtnRef  = ref(null)
const visibleCount = ref(Infinity)
const expanded     = ref(false)

const hiddenCount = computed(() => Math.max(0, props.tags.length - visibleCount.value))

// константы отступов (в пикселях)
const GAP      = 8   // gap-x-2 между чипами
const ML_MORE  = 8   // ml-2 перед кнопкой «Ещё»
const ML_RESET = 24  // ml-6 перед «Сбросить все»
const MORE_W   = 72  // оценка ширины кнопки «Ещё N ↓»

const recalculate = async () => {
  await nextTick()
  if (!containerRef.value) return

  const containerW = containerRef.value.clientWidth
  const resetEl    = resetBtnRef.value?.$el
  const resetW     = resetEl ? resetEl.offsetWidth : 90

  const els    = (measureRefs.value || []).filter(Boolean)
  const widths = els.map(el => el.offsetWidth)

  if (!widths.length) {
    visibleCount.value = props.tags.length
    return
  }

  // Считаем, влезут ли все чипы без кнопки «Ещё»
  const totalAll    = widths.reduce((s, w, i) => s + w + (i > 0 ? GAP : 0), 0)
  const availNoMore = containerW - ML_RESET - resetW

  if (totalAll <= availNoMore) {
    visibleCount.value = props.tags.length
    return
  }

  // Не влезают — ищем, сколько войдёт с кнопкой «Ещё»
  const availWithMore = containerW - ML_MORE - MORE_W - ML_RESET - resetW
  let cum = 0
  for (let i = 0; i < widths.length; i++) {
    cum += widths[i] + (i > 0 ? GAP : 0)
    if (cum > availWithMore) {
      visibleCount.value = Math.max(0, i)
      return
    }
  }
  visibleCount.value = props.tags.length
}

// Сворачиваем при изменении тэгов
watch(() => props.tags, () => { expanded.value = false; recalculate() }, { deep: true })

let ro = null
onMounted(() => {
  recalculate()
  ro = new ResizeObserver(recalculate)
  if (containerRef.value) ro.observe(containerRef.value)
})
onUnmounted(() => ro?.disconnect())
</script>
