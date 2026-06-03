<template>
  <BaseDrawer :open="open" width="560px" @close="$emit('close')">

    <!-- Header -->
    <div class="shrink-0 px-6 pt-6 pb-4 border-b border-[#f4f4f5]">
      <div class="flex items-center justify-between">
        <span class="text-[20px] leading-[32px] font-medium text-zinc-900">Импорт</span>
        <button
          @click="$emit('close')"
          class="flex size-6 items-center justify-center text-zinc-500 hover:text-zinc-900 transition-colors"
        >
          <XIcon :size="20" />
        </button>
      </div>
    </div>

  </BaseDrawer>
</template>

<script setup>
import { X as XIcon } from 'lucide-vue-next'
import BaseDrawer from './BaseDrawer.vue'

defineProps({ open: Boolean })
defineEmits(['close'])
</script>
