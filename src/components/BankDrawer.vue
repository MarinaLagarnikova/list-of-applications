<template>
  <BaseDrawer :open="open" @close="$emit('close')">

    <!-- ─── Scrollable body ──────────────────────────── -->
    <div class="flex-1 overflow-y-auto min-h-0">

      <!-- Header -->
      <div class="px-6 pt-6 pb-0">
        <div class="flex items-center gap-2">
          <!-- Bank logo badge -->
          <div
            class="size-8 rounded-full flex items-center justify-center shrink-0"
            :style="{ backgroundColor: bank.color }"
          >
            <span
              class="text-sm font-semibold leading-none"
              :class="bank.lightInitial ? 'text-zinc-900' : 'text-white'"
            >{{ bank.initial }}</span>
          </div>
          <!-- Bank name -->
          <span class="flex-1 text-xl font-semibold text-zinc-900 leading-8">{{ bank.name }}</span>
          <!-- Close -->
          <button
            @click="$emit('close')"
            class="flex size-6 items-center justify-center text-zinc-500 hover:text-zinc-900 transition-colors"
          >
            <XIcon :size="20" />
          </button>
        </div>
      </div>

      <!-- Content -->
      <div class="px-6 pt-6 pb-6">

        <!-- Program title + description -->
        <div class="flex flex-col gap-1 mb-4">
          <span class="text-sm font-semibold text-zinc-900 leading-5">{{ bank.programTitle }}</span>
          <p class="text-sm font-light text-zinc-500 leading-5">{{ bank.programDesc }}</p>
        </div>

        <!-- Action buttons -->
        <div class="flex items-center gap-4 pb-10">
          <button class="inline-flex items-center gap-1.5 h-9 px-3 rounded-lg border border-zinc-200 shadow-xs text-sm font-semibold text-zinc-900 bg-white hover:bg-zinc-50 transition-colors">
            <ChartColumnBigIcon :size="16" />
            График платежей
          </button>
          <button class="inline-flex items-center gap-1.5 h-9 px-3 rounded-lg border border-zinc-200 shadow-xs text-sm font-semibold text-zinc-900 bg-white hover:bg-zinc-50 transition-colors">
            <BookmarkIcon :size="16" />
            Сохранить
          </button>
          <button class="size-9 flex items-center justify-center rounded-lg border border-zinc-200 shadow-xs bg-white hover:bg-zinc-50 transition-colors shrink-0">
            <ExternalLinkIcon :size="16" />
          </button>
        </div>

        <!-- Details rows -->
        <div class="flex flex-col">

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Полная стоимость</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ bank.psk }}</span>
          </div>

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Ставка</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ bank.rate }}</span>
          </div>

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Первый взнос</span>
            <div class="flex items-center gap-2">
              <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ pvAmount }}</span>
              <span class="bg-[rgba(82,82,91,0.1)] text-zinc-700 text-xs font-medium px-1.5 py-1 rounded-md leading-4">{{ pvPct }}</span>
            </div>
          </div>

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Объект</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ bank.cost }}</span>
          </div>

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Кредит</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ bank.credit }}</span>
          </div>

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Переплата</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ bank.overpayment }}</span>
          </div>

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Доход</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ bank.income }}</span>
          </div>

          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Страховка</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ bank.insurance }}</span>
          </div>

        </div>

        <!-- Название программы -->
        <div class="flex flex-col gap-1 pt-6">
          <span class="text-sm font-medium text-zinc-900 leading-5">Название программы</span>
          <p class="text-sm font-light text-zinc-500 leading-5">{{ bank.programName }}</p>
        </div>

      </div>
    </div>

  </BaseDrawer>
</template>

<script setup>
import { computed } from 'vue'
import { X as XIcon, ChartColumnBig as ChartColumnBigIcon, Bookmark as BookmarkIcon, ExternalLink as ExternalLinkIcon } from 'lucide-vue-next'
import BaseDrawer from './BaseDrawer.vue'

const props = defineProps({
  open: Boolean,
  bank: Object,
})

defineEmits(['close'])

const pvAmount = computed(() => {
  if (!props.bank?.pv) return ''
  return props.bank.pv.split(' / ')[0]?.trim() ?? props.bank.pv
})
const pvPct = computed(() => {
  if (!props.bank?.pv) return ''
  return props.bank.pv.split(' / ')[1]?.trim() ?? ''
})
</script>
