<template>
  <Transition
    enter-active-class="transition ease-out duration-200"
    enter-from-class="opacity-0 translate-y-2"
    enter-to-class="opacity-100 translate-y-0"
    leave-active-class="transition ease-in duration-150"
    leave-from-class="opacity-100 translate-y-0"
    leave-to-class="opacity-0 translate-y-2"
  >
    <div v-if="count > 0" class="absolute bottom-6 left-1/2 -translate-x-1/2 z-50 flex items-center rounded-2xl bg-indigo-600 px-4 py-3 shadow-lg whitespace-nowrap">
      <!-- Счётчик + Сбросить -->
      <span class="text-[14px] font-normal text-white">Выбрано {{ count }}</span>
      <button
        @click="$emit('clear')"
        class="ml-2 flex items-center gap-x-1 rounded-lg px-2 py-1 text-[14px] font-normal text-white transition-colors"
      >
        <XIcon :size="14" />
        Сбросить
      </button>

      <!-- Слот для доп. кнопок -->
      <div v-if="$slots.actions" class="ml-12 flex items-center gap-x-3">
        <slot name="actions" />
      </div>

      <!-- Удалить -->
      <button
        @click="$emit('delete')"
        class="ml-3 flex items-center gap-x-1.5 rounded-xl bg-indigo-500 hover:bg-indigo-400 px-3 py-1.5 text-[14px] font-medium text-white transition-colors"
      >
        <Trash2Icon :size="14" />
        {{ deleteLabel }}
      </button>

      <!-- Закрыть без фона -->
      <button
        @click="$emit('clear')"
        class="ml-6 flex items-center justify-center rounded-lg p-1 text-white transition-colors"
      >
        <XIcon :size="16" />
      </button>
    </div>
  </Transition>
</template>

<script setup>
import { X as XIcon, Trash2 as Trash2Icon } from 'lucide-vue-next'

defineProps({ count: { type: Number, default: 0 }, deleteLabel: { type: String, default: 'Удалить' } })
defineEmits(['clear', 'delete'])
</script>
