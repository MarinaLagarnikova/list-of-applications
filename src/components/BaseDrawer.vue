<template>
  <!-- Backdrop -->
  <div v-if="open" class="fixed inset-0 z-40 bg-zinc-500/50" @click="$emit('close')" />

  <!-- Slide panel -->
  <Transition
    enter-active-class="transform transition ease-out duration-300"
    enter-from-class="translate-x-full"
    enter-to-class="translate-x-0"
    leave-active-class="transform transition ease-out duration-300"
    leave-from-class="translate-x-0"
    leave-to-class="translate-x-full"
  >
    <div
      v-if="open"
      class="fixed top-2 bottom-2 right-2 z-50 flex flex-col bg-white shadow-xl rounded-3xl ring-1 ring-gray-200 overflow-hidden"
      :style="{ width }"
    >
      <slot />
    </div>
  </Transition>
</template>

<script setup>
import { watch, onUnmounted } from 'vue'

const props = defineProps({
  open: Boolean,
  width: { type: String, default: '448px' },
})
const emit = defineEmits(['close'])

const onKeydown = (e) => {
  if (e.key === 'Escape') emit('close')
}

watch(() => props.open, (val) => {
  if (val) window.addEventListener('keydown', onKeydown)
  else window.removeEventListener('keydown', onKeydown)
})

onUnmounted(() => window.removeEventListener('keydown', onKeydown))
</script>
