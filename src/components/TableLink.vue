<template>
  <button
    @click.stop="$emit('click')"
    :class="[
      'cursor-pointer text-[14px] leading-[20px] text-indigo-600 hover:text-indigo-800 transition-colors',
      weight === 'medium' ? 'font-medium' : 'font-normal',
    ]"
  >
    <slot />
  </button>
</template>

<script setup>
defineProps({
  weight: { type: String, default: 'normal' },
})
defineEmits(['click'])
</script>
