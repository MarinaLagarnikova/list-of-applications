<template>
  <component
    :is="href ? 'a' : 'button'"
    :href="href"
    :type="href ? undefined : (type || 'button')"
    :disabled="disabled || undefined"
    :class="classes"
    v-bind="$attrs"
  >
    <span
      class="absolute top-1/2 left-1/2 size-[max(100%,2.75rem)] -translate-x-1/2 -translate-y-1/2 [@media(pointer:fine)]:hidden"
      aria-hidden="true"
    />
    <slot />
  </component>
</template>

<script>
export default { inheritAttrs: false }
</script>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  href:     { type: String,  default: null },
  type:     { type: String,  default: 'button' },
  color:    { type: String,  default: null },
  outline:  { type: Boolean, default: false },
  plain:    { type: Boolean, default: false },
  soft:     { type: Boolean, default: false },
  disabled: { type: Boolean, default: false },
})

const base = [
  'relative isolate inline-flex items-center justify-center gap-x-2 rounded-lg border text-sm/6 font-medium',
  'h-8 px-3',
  'focus:outline-none focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-500',
  'disabled:opacity-50 cursor-pointer transition-colors',
]

const variants = {
  outline: 'border-zinc-950/10 bg-white text-zinc-950 shadow-xs hover:bg-zinc-50 active:bg-zinc-100',
  plain:   'border-transparent bg-transparent text-zinc-500 hover:bg-zinc-100 active:bg-zinc-200',
}

const softColors = {
  indigo: 'border-transparent bg-indigo-50 text-indigo-600 shadow-xs hover:bg-indigo-100 dark:bg-indigo-500/20 dark:text-indigo-400 dark:hover:bg-indigo-500/30',
  zinc:   'border-transparent bg-zinc-100 text-zinc-600 shadow-xs hover:bg-zinc-200 dark:bg-zinc-500/20 dark:text-zinc-400 dark:hover:bg-zinc-500/30',
  red:    'border-transparent bg-red-50 text-red-600 shadow-xs hover:bg-red-100 dark:bg-red-500/20 dark:text-red-400 dark:hover:bg-red-500/30',
  green:  'border-transparent bg-green-50 text-green-600 shadow-xs hover:bg-green-100 dark:bg-green-500/20 dark:text-green-400 dark:hover:bg-green-500/30',
}

const colors = {
  'dark/zinc': 'border-transparent bg-zinc-900 text-white shadow-xs hover:bg-zinc-800 active:bg-zinc-700',
  light:       'border-zinc-950/10 bg-white text-zinc-950 shadow-xs hover:bg-zinc-50',
  dark:        'border-transparent bg-zinc-900 text-white shadow-xs hover:bg-zinc-800',
  white:       'border-zinc-950/10 bg-white text-zinc-950 shadow-xs hover:bg-zinc-50',
  zinc:        'border-transparent bg-zinc-600 text-white shadow-xs hover:bg-zinc-500',
  indigo:      'border-transparent bg-indigo-600 text-white shadow-xs hover:bg-indigo-700 active:bg-indigo-800',
  cyan:        'border-transparent bg-cyan-300 text-cyan-950 shadow-xs hover:bg-cyan-400',
  red:         'border-transparent bg-red-600 text-white shadow-xs hover:bg-red-500',
  orange:      'border-transparent bg-orange-500 text-white shadow-xs hover:bg-orange-600',
  amber:       'border-transparent bg-amber-400 text-amber-950 shadow-xs hover:bg-amber-500',
  yellow:      'border-transparent bg-yellow-300 text-yellow-950 shadow-xs hover:bg-yellow-400',
  lime:        'border-transparent bg-lime-300 text-lime-950 shadow-xs hover:bg-lime-400',
  green:       'border-transparent bg-green-600 text-white shadow-xs hover:bg-green-500',
  emerald:     'border-transparent bg-emerald-600 text-white shadow-xs hover:bg-emerald-500',
  teal:        'border-transparent bg-teal-600 text-white shadow-xs hover:bg-teal-500',
  sky:         'border-transparent bg-sky-500 text-white shadow-xs hover:bg-sky-600',
  blue:        'border-transparent bg-blue-600 text-white shadow-xs hover:bg-blue-500',
  violet:      'border-transparent bg-violet-500 text-white shadow-xs hover:bg-violet-600',
  purple:      'border-transparent bg-purple-500 text-white shadow-xs hover:bg-purple-600',
  fuchsia:     'border-transparent bg-fuchsia-500 text-white shadow-xs hover:bg-fuchsia-600',
  pink:        'border-transparent bg-pink-500 text-white shadow-xs hover:bg-pink-600',
  rose:        'border-transparent bg-rose-500 text-white shadow-xs hover:bg-rose-600',
}

const classes = computed(() => [
  ...base,
  props.outline
    ? variants.outline
    : props.plain
      ? variants.plain
      : props.soft
        ? (softColors[props.color ?? 'zinc'])
        : (colors[props.color ?? 'dark/zinc']),
])
</script>
