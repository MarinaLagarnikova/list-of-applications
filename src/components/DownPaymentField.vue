<template>
  <div class="flex flex-col gap-3">

    <!-- Label -->
    <label class="text-sm/6 font-medium text-zinc-950 select-none dark:text-white">
      Первоначальный взнос
    </label>

    <!-- Control wrapper — такой же как Catalyst Input -->
    <span
      class="relative block w-full
        before:absolute before:inset-px before:rounded-[calc(var(--radius-lg)-1px)] before:bg-white before:shadow-sm
        dark:before:hidden
        after:pointer-events-none after:absolute after:inset-0 after:rounded-lg after:ring-transparent after:ring-inset
        sm:focus-within:after:ring-2 sm:focus-within:after:ring-blue-500"
    >
      <div
        class="relative flex w-full items-baseline rounded-lg border border-zinc-950/10 hover:border-zinc-950/20 bg-transparent
               px-[calc(--spacing(3.5)-1px)] py-[calc(--spacing(2.5)-1px)]
               sm:px-[calc(--spacing(3)-1px)] sm:py-[calc(--spacing(1.5)-1px)]
               dark:border-white/10 dark:bg-white/5"
      >
        <!-- Левая часть: редактируемый % -->
        <div class="flex items-baseline gap-1 min-w-0">
          <input
            :value="modelValue"
            @input="$emit('update:modelValue', $event.target.value)"
            :style="{ width: (modelValue?.length || 1) + 'ch' }"
            class="bg-transparent text-base/6 text-zinc-950 sm:text-sm/6 dark:text-white focus:outline-none"
          />
          <span class="text-base/6 text-zinc-950 sm:text-sm/6 dark:text-white">%</span>
        </div>

        <!-- Правая часть: сумма в рублях -->
        <span class="ml-auto text-base/6 text-zinc-500 sm:text-sm/6 dark:text-zinc-400 shrink-0">
          {{ formattedRubles }} ₽
        </span>
      </div>
    </span>

  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  modelValue:    { type: String, default: '30' },
  propertyPrice: { type: String, default: '0' },
})

defineEmits(['update:modelValue'])

const formattedRubles = computed(() => {
  const pct   = parseFloat(props.modelValue) || 0
  const price = parseFloat(props.propertyPrice.replace(/\s/g, '')) || 0
  const amount = Math.round(pct / 100 * price)
  return amount.toLocaleString('ru-RU')
})
</script>
