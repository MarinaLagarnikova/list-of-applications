<template>
  <Combobox as="div" :model-value="modelValue" @update:model-value="$emit('update:modelValue', $event); query = ''" :disabled="disabled">
    <ComboboxLabel v-if="label" class="block text-sm/6 font-medium text-gray-900">{{ label }}</ComboboxLabel>
    <div :class="['relative', label ? 'mt-2' : '']">
      <ComboboxInput
        class="block w-full rounded-md bg-white py-1.5 pr-12 pl-3 text-base text-gray-900 outline-1 -outline-offset-1 outline-gray-300 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 focus:outline-indigo-600 sm:text-sm/6 disabled:cursor-not-allowed disabled:bg-gray-50 disabled:text-gray-500"
        :placeholder="placeholder"
        :display-value="(val) => val ?? ''"
        @change="query = $event.target.value"
        @blur="query = ''"
      />
      <ComboboxButton class="absolute inset-y-0 right-0 flex items-center rounded-r-md px-2 focus:outline-hidden">
        <ChevronDownIcon class="size-5 text-gray-400" aria-hidden="true" />
      </ComboboxButton>

      <transition leave-active-class="transition ease-in duration-100" leave-from-class="" leave-to-class="opacity-0">
        <ComboboxOptions
          v-if="filteredOptions.length > 0"
          class="absolute z-50 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-lg outline outline-black/5 sm:text-sm"
        >
          <ComboboxOption
            v-for="option in filteredOptions"
            :key="option"
            :value="option"
            as="template"
            v-slot="{ active }"
          >
            <li :class="['relative cursor-default select-none px-3 py-2', active ? 'bg-indigo-600 text-white' : 'text-gray-900']">
              <span class="block truncate">{{ option }}</span>
            </li>
          </ComboboxOption>
        </ComboboxOptions>
      </transition>
    </div>
  </Combobox>
</template>

<script setup>
import { ref, computed } from 'vue'
import { Combobox, ComboboxButton, ComboboxInput, ComboboxLabel, ComboboxOption, ComboboxOptions } from '@headlessui/vue'
import { ChevronDown as ChevronDownIcon } from 'lucide-vue-next'

const props = defineProps({
  options: { type: Array, default: () => [] },
  modelValue: { type: String, default: '' },
  placeholder: { type: String, default: '' },
  label: { type: String, default: '' },
  disabled: { type: Boolean, default: false },
})
defineEmits(['update:modelValue'])

const query = ref('')

const filteredOptions = computed(() =>
  query.value === ''
    ? props.options
    : props.options.filter(o => o.toLowerCase().includes(query.value.toLowerCase()))
)
</script>
