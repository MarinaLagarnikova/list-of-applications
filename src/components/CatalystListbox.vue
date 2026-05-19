<template>
  <div class="flex flex-col gap-3">

    <!-- Label -->
    <label v-if="label" class="text-sm/6 font-medium text-zinc-950 select-none dark:text-white">
      {{ label }}
    </label>

    <Listbox v-model="selected" as="div" class="relative" v-slot="{ open }">

      <!-- Trigger button — белый inset + shadow + focus ring (точно как Catalyst) -->
      <ListboxButton
        :class="[
          'group relative block w-full',
          'before:absolute before:inset-px before:rounded-[calc(var(--radius-lg)-1px)] before:bg-white before:shadow-sm',
          'dark:before:hidden',
          'focus:outline-none',
          'after:pointer-events-none after:absolute after:inset-0 after:rounded-lg after:ring-transparent after:ring-inset',
          'focus-within:after:ring-2 focus-within:after:ring-blue-500',
          'data-disabled:opacity-50 data-disabled:before:bg-zinc-950/5 data-disabled:before:shadow-none',
        ]"
      >
        <!-- Selected value display -->
        <span
          :class="[
            'relative block w-full appearance-none rounded-lg text-left',
            'py-[calc(--spacing(2.5)-1px)] sm:py-[calc(--spacing(1.5)-1px)]',
            'min-h-11 sm:min-h-9',
            'pr-[calc(--spacing(7)-1px)] pl-[calc(--spacing(3.5)-1px)] sm:pl-[calc(--spacing(3)-1px)]',
            'text-base/6 sm:text-sm/6',
            selected ? 'text-zinc-950 dark:text-white' : 'text-zinc-500',
            'border border-zinc-950/10 group-hover:border-zinc-950/20 dark:border-white/10 dark:group-hover:border-white/20',
            'bg-transparent dark:bg-white/5',
            'truncate',
          ]"
        >
          {{ selected || placeholder }}
        </span>

        <!-- Chevron icon -->
        <span class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-2">
          <ChevronDownIcon :size="16" class="text-zinc-500 dark:text-zinc-400" />
        </span>
      </ListboxButton>

      <!-- Dropdown -->
      <TransitionRoot
        :show="open"
        leave="transition ease-in duration-100"
        leave-from="opacity-100"
        leave-to="opacity-0"
      >
        <ListboxOptions
          static
          :class="[
            'absolute z-50 mt-1 w-full',
            'isolate scroll-py-1 rounded-xl p-1 select-none',
            'outline outline-transparent focus:outline-none',
            'overflow-y-auto overscroll-contain max-h-60',
            'bg-white/75 backdrop-blur-xl dark:bg-zinc-800/75',
            'shadow-lg ring-1 ring-zinc-950/10 dark:ring-white/10 dark:ring-inset',
          ]"
        >
          <ListboxOption
            v-for="option in options"
            :key="option"
            :value="option"
            v-slot="{ active, selected: isSelected }"
            as="template"
          >
            <li
              :class="[
                'grid cursor-default items-baseline gap-x-2 rounded-lg',
                'grid-cols-[theme(spacing.5)_1fr] py-2.5 pr-3.5 pl-2',
                'sm:grid-cols-[theme(spacing.4)_1fr] sm:py-1.5 sm:pr-3 sm:pl-1.5',
                'text-base/6 sm:text-sm/6 outline-none',
                active ? 'bg-indigo-600 text-white' : 'text-zinc-950 dark:text-white',
              ]"
            >
              <!-- Check mark (слева, как в Catalyst) -->
              <svg
                v-if="isSelected"
                :class="[
                  'relative size-5 self-center sm:size-4',
                  active ? 'stroke-white' : 'stroke-zinc-950 dark:stroke-white',
                ]"
                viewBox="0 0 16 16"
                fill="none"
                aria-hidden="true"
              >
                <path d="M4 8.5l3 3L12 4" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
              </svg>
              <span v-else />

              <!-- Label (col-start-2) -->
              <span class="col-start-2 flex min-w-0 items-center truncate">{{ option }}</span>
            </li>
          </ListboxOption>
        </ListboxOptions>
      </TransitionRoot>

    </Listbox>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { ChevronDown as ChevronDownIcon } from 'lucide-vue-next'
import {
  Listbox,
  ListboxButton,
  ListboxOptions,
  ListboxOption,
  TransitionRoot,
} from '@headlessui/vue'

const props = defineProps({
  label: { type: String, default: '' },
  options: { type: Array, default: () => [] },
  modelValue: { type: String, default: '' },
  placeholder: { type: String, default: 'Выберите...' },
})

const emit = defineEmits(['update:modelValue'])

const selected = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val),
})
</script>
