<template>
  <div class="flex flex-col gap-3">

    <!-- Label — точно как Catalyst Field > Label -->
    <label class="text-sm/6 font-medium text-zinc-950 select-none dark:text-white">
      {{ label }}
    </label>

  <Combobox v-model="selected" as="div" class="relative" v-slot="{ open }">

    <!-- Control wrapper — белый inset + shadow + focus ring (точно как Catalyst) -->
    <span
      :class="[
        'relative block w-full min-w-0',
        // Белый фон + тень через pseudo-before
        'before:absolute before:inset-px before:rounded-[calc(var(--radius-lg)-1px)] before:bg-white before:shadow-sm',
        'dark:before:hidden',
        // Focus ring
        'after:pointer-events-none after:absolute after:inset-0 after:rounded-lg after:ring-transparent after:ring-inset',
        'sm:focus-within:after:ring-2 sm:focus-within:after:ring-blue-500',
      ]"
    >
      <ComboboxInput
        :class="[
          'relative block w-full appearance-none rounded-lg',
          'py-[calc(--spacing(2.5)-1px)] sm:py-[calc(--spacing(1.5)-1px)]',
          'pr-[calc(--spacing(10)-1px)] pl-[calc(--spacing(3.5)-1px)]',
          'sm:pr-[calc(--spacing(9)-1px)] sm:pl-[calc(--spacing(3)-1px)]',
          'text-base/6 text-zinc-950 placeholder:text-zinc-500 sm:text-sm/6 dark:text-white',
          'border border-zinc-950/10 hover:border-zinc-950/20 dark:border-white/10 dark:hover:border-white/20',
          'bg-transparent dark:bg-white/5',
          'focus:outline-none',
          'dark:scheme-dark',
        ]"
        :display-value="(opt) => opt ?? ''"
        @change="query = $event.target.value"
        @focus="query = ''"
        :placeholder="placeholder"
      />
      <ComboboxButton class="group absolute inset-y-0 right-0 flex items-center px-2">
        <ChevronDownIcon :size="16" class="text-zinc-500 group-hover:text-zinc-700 dark:text-zinc-400 dark:group-hover:text-zinc-300" />
      </ComboboxButton>
    </span>

    <!-- Dropdown options -->
    <TransitionRoot
      :show="open"
      leave="transition ease-in duration-100"
      leave-from="opacity-100"
      leave-to="opacity-0"
      @after-leave="query = ''"
    >
      <ComboboxOptions
        static
        :class="[
          'absolute z-50 mt-2 w-full',
          // Anchor gap / padding
          'scroll-py-1 rounded-xl p-1 select-none',
          'outline outline-transparent focus:outline-none',
          'overflow-y-auto overscroll-contain max-h-60',
          // Frosted glass background
          'bg-white/75 backdrop-blur-xl dark:bg-zinc-800/75',
          // Border + shadow
          'shadow-lg ring-1 ring-zinc-950/10 dark:ring-white/10',
          // Transition
          'transition-opacity duration-100 ease-in',
          // Empty
          'empty:invisible',
        ]"
      >
        <div
          v-if="filteredOptions.length === 0 && query !== ''"
          class="py-2.5 pr-2 pl-3.5 text-sm/6 text-zinc-500 select-none"
        >
          Ничего не найдено
        </div>

        <ComboboxOption
          v-for="option in filteredOptions"
          :key="option"
          :value="option"
          v-slot="{ active, selected: isSelected }"
          as="template"
        >
          <li
            :class="[
              'group/option grid w-full cursor-default items-baseline gap-x-2 rounded-lg',
              'grid-cols-[1fr_theme(spacing.5)] py-2.5 pr-2 pl-3.5',
              'sm:grid-cols-[1fr_theme(spacing.4)] sm:py-1.5 sm:pr-2 sm:pl-3',
              'text-base/6 sm:text-sm/6 outline-none',
              active ? 'bg-indigo-600 text-white' : 'text-zinc-950 dark:text-white',
              'data-disabled:opacity-50',
            ]"
          >
            <!-- Label -->
            <span class="flex min-w-0 items-center truncate">{{ option }}</span>
            <!-- Check mark -->
            <svg
              v-if="isSelected"
              :class="[
                'relative col-start-2 size-5 self-center sm:size-4',
                active ? 'stroke-white' : 'stroke-zinc-950 dark:stroke-white',
              ]"
              viewBox="0 0 16 16"
              fill="none"
              aria-hidden="true"
            >
              <path d="M4 8.5l3 3L12 4" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
          </li>
        </ComboboxOption>
      </ComboboxOptions>
    </TransitionRoot>

  </Combobox>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { ChevronDown as ChevronDownIcon } from 'lucide-vue-next'
import {
  Combobox,
  ComboboxInput,
  ComboboxButton,
  ComboboxOptions,
  ComboboxOption,
  TransitionRoot,
} from '@headlessui/vue'

const props = defineProps({
  options: {
    type: Array,
    default: () => [],
  },
  modelValue: {
    type: String,
    default: '',
  },
  placeholder: {
    type: String,
    default: 'Поиск...',
  },
  label: {
    type: String,
    default: 'Жилой комплекс',
  },
})

const emit = defineEmits(['update:modelValue'])

const query = ref('')

const selected = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val),
})

const filteredOptions = computed(() =>
  query.value === ''
    ? props.options
    : props.options.filter((opt) =>
        opt.toLowerCase().includes(query.value.toLowerCase())
      )
)
</script>
