<template>
  <div :class="['flex flex-col gap-3', readonly ? 'opacity-50' : '']">

    <!-- Label -->
    <label class="text-sm/6 font-medium text-zinc-950 select-none dark:text-white">
      {{ label }}
    </label>

    <!-- Control wrapper — белый inset + shadow + focus ring -->
    <span
      :class="[
        'relative block w-full',
        // before: — серый фон для readonly, белый для обычного
        'before:absolute before:inset-px before:rounded-[calc(var(--radius-lg)-1px)]',
        readonly ? 'before:bg-zinc-950/5 before:shadow-none' : 'before:bg-white before:shadow-sm',
        'dark:before:hidden',
        // focus ring
        'after:pointer-events-none after:absolute after:inset-0 after:rounded-lg after:ring-transparent after:ring-inset',
        readonly ? '' : 'sm:focus-within:after:ring-2 sm:focus-within:after:ring-blue-500',
      ]"
    >
      <input
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        :readonly="readonly"
        :placeholder="placeholder"
        :class="[
          'relative block w-full appearance-none rounded-lg',
          'px-[calc(--spacing(3.5)-1px)] py-[calc(--spacing(2.5)-1px)]',
          'sm:px-[calc(--spacing(3)-1px)] sm:py-[calc(--spacing(1.5)-1px)]',
          suffix ? 'pr-8' : '',
          'text-base/6 text-zinc-950 placeholder:text-zinc-500 sm:text-sm/6 dark:text-white',
          // border
          'border dark:border-white/10',
          readonly
            ? 'border-zinc-950/20 bg-transparent cursor-default'
            : 'border-zinc-950/10 bg-transparent hover:border-zinc-950/20 dark:bg-white/5 dark:hover:border-white/20',
          'focus:outline-none dark:scheme-dark',
        ]"
      />
      <!-- Suffix (₽, %) -->
      <span
        v-if="suffix"
        class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3 text-sm/6 text-zinc-500 dark:text-zinc-400"
      >
        {{ suffix }}
      </span>
    </span>

  </div>
</template>

<script setup>
defineProps({
  label:      { type: String,  required: true },
  modelValue: { type: String,  default: '' },
  suffix:     { type: String,  default: '' },
  readonly:   { type: Boolean, default: false },
  placeholder:{ type: String,  default: '' },
})
defineEmits(['update:modelValue'])
</script>
