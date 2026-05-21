<template>
  <div class="px-6 py-4">
    <div class="flex items-center justify-between mb-3">
      <span class="text-[14px] leading-[24px] font-medium text-[#18181b]">{{ label }}</span>
      <button
        v-if="selectedTag || priceFrom || priceTo"
        @click="reset"
        class="text-[14px] font-medium text-indigo-600 hover:text-indigo-700"
      >Сбросить</button>
    </div>

    <!-- Теги диапазонов -->
    <div class="flex flex-wrap gap-2 mb-3">
      <button
        v-for="tag in priceTags"
        :key="tag.label"
        @click="applyTag(tag)"
        :class="selectedTag === tag.label
          ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-100'
          : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
      >{{ tag.label }}</button>
    </div>

    <!-- Сдвоенный инпут -->
    <div class="grid grid-cols-2">
      <div
        class="flex items-center rounded-l-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:relative focus-within:z-10 focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition"
      >
        <span class="shrink-0 text-[14px] text-[#a1a1aa] mr-1.5">от</span>
        <input
          type="text"
          v-model="priceFrom"
          @input="selectedTag = null; emitValue()"
          placeholder=""
          class="min-w-0 flex-1 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] placeholder:font-light focus:outline-none"
        />
        <span class="shrink-0 text-[14px] text-[#a1a1aa] ml-1">млн ₽</span>
      </div>
      <div
        class="-ml-px flex items-center rounded-r-lg border border-[#e4e4e7] bg-white px-3 py-[8px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus-within:relative focus-within:z-10 focus-within:ring-1 focus-within:ring-indigo-500 focus-within:border-indigo-500 transition"
      >
        <span class="shrink-0 text-[14px] text-[#a1a1aa] mr-1.5">до</span>
        <input
          type="text"
          v-model="priceTo"
          @input="selectedTag = null; emitValue()"
          placeholder=""
          class="min-w-0 flex-1 bg-transparent text-[14px] font-normal text-zinc-900 placeholder:text-[#a1a1aa] placeholder:font-light focus:outline-none"
        />
        <span class="shrink-0 text-[14px] text-[#a1a1aa] ml-1">млн ₽</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  label: { type: String, required: true },
  modelValue: { type: Object, default: () => ({ from: null, to: null }) },
})
const emit = defineEmits(['update:modelValue'])

const priceTags = [
  { label: 'До 5 млн ₽',     from: null, to: '5'  },
  { label: '5—10 млн ₽',     from: '5',  to: '10' },
  { label: '10—15 млн ₽',    from: '10', to: '15' },
  { label: '15—20 млн ₽',    from: '15', to: '20' },
  { label: 'От 20 млн ₽',    from: '20', to: null  },
]

const selectedTag = ref(null)
const priceFrom = ref('')
const priceTo = ref('')

const emitValue = () => emit('update:modelValue', {
  from: priceFrom.value || null,
  to: priceTo.value || null,
})

const applyTag = (tag) => {
  if (selectedTag.value === tag.label) {
    reset()
    return
  }
  selectedTag.value = tag.label
  priceFrom.value = tag.from ?? ''
  priceTo.value = tag.to ?? ''
  emitValue()
}

const reset = () => {
  selectedTag.value = null
  priceFrom.value = ''
  priceTo.value = ''
  emitValue()
}
</script>
