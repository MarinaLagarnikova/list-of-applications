<template>
  <button
    @click.stop="$emit('click')"
    :class="[
      'cursor-pointer text-[14px] leading-[20px] text-zinc-900 group-hover:text-indigo-600 hover:underline transition-colors',
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
