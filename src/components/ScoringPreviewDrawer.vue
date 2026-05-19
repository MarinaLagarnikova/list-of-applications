<template>
  <BaseDrawer :open="open" @close="$emit('close')">
    <div class="flex-1 overflow-y-auto">

      <!-- Header -->
      <div class="px-6 pt-6 pb-0">
        <div class="flex items-center gap-2">
          <div class="size-8 rounded-full bg-[#e4e4e7] flex items-center justify-center shrink-0">
            <span class="text-sm font-medium text-[#71717a] leading-none">{{ item.initials }}</span>
          </div>
          <span class="flex-1 text-[20px] leading-[32px] font-semibold text-[#18181b]">{{ clientShort }}</span>
          <button @click="$emit('close')" class="flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors">
            <XIcon :size="20" />
          </button>
        </div>

        <!-- Action buttons -->
        <div class="flex items-center gap-x-2 mt-4">
          <button class="flex flex-1 items-center justify-center gap-x-2 rounded-lg border border-zinc-950/10 bg-white px-3 h-9 text-[14px] leading-[20px] font-medium text-[#09090b] shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors">
            <ExternalLinkIcon :size="14" class="shrink-0" />
            Перейти в заявку
          </button>
          <button class="flex size-9 items-center justify-center rounded-lg border border-zinc-950/10 bg-white text-[#09090b] shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors shrink-0">
            <RefreshCwIcon :size="14" class="shrink-0" />
          </button>
        </div>
        <p class="mt-2 text-center text-[13px] leading-[20px] font-light text-[#71717a]">Обновить можно 8 мая с 12:00</p>
      </div>

      <!-- ID + Телефон -->
      <div class="px-6 pt-6">

        <!-- ID -->
        <div class="flex items-start justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">ID</span>
          <span class="flex items-center gap-x-3 text-[14px] leading-[20px] font-normal text-[#18181b]">
            <span>{{ item.id }}</span>
            <button @click="navigator.clipboard.writeText(item.id)" class="text-zinc-900 hover:text-zinc-600 transition-colors">
              <CopyIcon :size="14" />
            </button>
          </span>
        </div>

        <!-- Телефон -->
        <div class="flex items-start justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Телефон</span>
          <span class="flex items-center gap-x-3 text-[14px] leading-[20px] font-normal text-[#18181b]">
            <span>{{ item.fullPhone ?? item.phone }}</span>
            <button @click="navigator.clipboard.writeText(item.fullPhone ?? item.phone)" class="text-zinc-900 hover:text-zinc-600 transition-colors">
              <CopyIcon :size="14" />
            </button>
          </span>
        </div>

        <!-- Менеджер -->
        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Менеджер</span>
          <CatalystListbox :options="managerOptions" v-model="managerValue" placeholder="Менеджер" />
        </div>

      </div>

      <!-- Блоки проверок -->
      <div v-for="(check, i) in checks" :key="i" class="px-6 pt-[40px]">
        <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Рейтинг от {{ check.dateLabel }}</span>

        <!-- Рейтинг -->
        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Рейтинг</span>
          <div class="flex items-center gap-x-2">
            <div class="relative h-2 w-[61px] rounded-full bg-[#e4e4e7] overflow-hidden shrink-0">
              <div class="absolute inset-y-0 left-0 rounded-full" :style="{ width: scoreBarWidth(check.score), background: scoreDotColor(check.score) }" />
            </div>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b] w-8 text-right">{{ check.score }}</span>
          </div>
        </div>

        <!-- Долговая нагрузка -->
        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Долговая нагрузка</span>
          <div v-if="check.debtLoad != null" class="flex items-center gap-x-2">
            <div class="relative h-2 w-[61px] rounded-full bg-[#e4e4e7] overflow-hidden shrink-0">
              <div class="absolute inset-y-0 left-0 rounded-full" :style="{ width: check.debtLoad + '%', background: debtLoadColor(check.debtLoad) }" />
            </div>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b] w-8 text-right">{{ check.debtLoad }}%</span>
          </div>
          <span v-else class="text-[14px] leading-[20px] font-light text-[#71717a]">—</span>
        </div>

        <!-- Сумма платежей -->
        <div class="flex items-start justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Сумма всех платежей по кредитам</span>
          <span class="text-[14px] leading-[20px] font-normal text-[#18181b] shrink-0 ml-4">{{ check.totalPayments ?? '—' }}</span>
        </div>

      </div>

      <!-- Ошибка проверки -->
      <div v-if="item.status === 'Ошибка проверки'" class="px-6 py-6">
        <div class="flex items-start gap-x-3 rounded-xl bg-[#fef2f2] px-4 py-4">
          <CircleXIcon :size="16" class="shrink-0 mt-0.5 text-[#dc2626]" />
          <div class="flex flex-col gap-y-0.5">
            <span class="text-[14px] leading-[20px] font-medium text-[#991b1b]">Ошибка проверки</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#b91c1c]">Не удалось выполнить проверку</span>
          </div>
        </div>
      </div>

      <!-- Анкета section -->
      <div class="px-8 pt-6 pb-[40px]">
        <div class="flex items-center justify-between">
          <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Анкета</span>
          <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
        </div>
        <div class="mt-2 flex flex-col">
          <div class="flex items-center py-3">
            <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">
              {{ item.initials }}
            </span>
            <div class="flex flex-col gap-y-0.5 pl-4">
              <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">{{ item.fullName }}</span>
              <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">{{ item.phone }}</span>
            </div>
          </div>
        </div>
      </div>

    </div>
  </BaseDrawer>
</template>

<script setup>
import { computed, ref } from 'vue'
import BaseDrawer from './BaseDrawer.vue'
import CatalystListbox from './CatalystListbox.vue'
import {
  X as XIcon,
  ExternalLink as ExternalLinkIcon,
  RefreshCw as RefreshCwIcon,
  Copy as CopyIcon,
  CircleX as CircleXIcon,
} from 'lucide-vue-next'

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerValue = ref('Смирнова Юлия')

const props = defineProps({
  open: Boolean,
  item: { type: Object, default: null },
})

defineEmits(['close'])

const clientShort = computed(() => {
  if (!props.item?.fullName) return ''
  const parts = props.item.fullName.trim().split(' ')
  if (parts.length < 2) return parts[0]
  const last = parts[0]
  const first = parts[1] ? parts[1][0] + '.' : ''
  const mid = parts[2] ? parts[2][0] + '.' : ''
  return `${last} ${first} ${mid}`.trim()
})

const months = ['января','февраля','марта','апреля','мая','июня','июля','августа','сентября','октября','ноября','декабря']
const formatDate = (d) => {
  if (!d) return ''
  const parts = d.split('.')
  return `${parseInt(parts[0])} ${months[parseInt(parts[1]) - 1]}`
}

const checks = computed(() => {
  if (!props.item) return []
  const result = []
  if (props.item.lastCheck && props.item.lastCheckDate) {
    result.push({ ...props.item.lastCheck, dateLabel: formatDate(props.item.lastCheckDate) })
  }
  if (props.item.prevCheck && props.item.prevCheckDate) {
    result.push({ ...props.item.prevCheck, dateLabel: formatDate(props.item.prevCheckDate) })
  }
  return result
})

const scoreDotColor = (score) => {
  if (score >= 800) return '#16a34a'
  if (score >= 700) return '#65a30d'
  if (score >= 600) return '#ca8a04'
  return '#e11d48'
}

const scoreBarWidth = (score) => {
  const pct = Math.min(Math.round((score / 850) * 100), 100)
  return pct + '%'
}

const debtLoadColor = (v) => {
  if (v <= 20) return '#16a34a'
  if (v <= 50) return '#ca8a04'
  if (v <= 80) return '#b45309'
  return '#dc2626'
}
</script>
