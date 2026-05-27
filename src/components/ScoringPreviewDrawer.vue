<template>
  <BaseDrawer :open="open" @close="$emit('close')">
    <div class="flex flex-col h-full">
    <div class="flex-1 overflow-y-auto">

      <!-- Header -->
      <div class="px-6 pt-6 pb-0">

        <!-- Строка бейджей: ID + статус + закрыть -->
        <div class="flex items-center gap-2">
          <span class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">
            {{ item.id }}
          </span>
          <span v-if="item.status" :class="statusBadgeClass(item.status)">{{ item.status }}</span>
          <button @click="$emit('close')" class="ml-auto flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors">
            <XIcon :size="16" />
          </button>
        </div>

        <!-- Заголовок: ФИО -->
        <h2 class="mt-2 text-[24px] leading-[32px] font-medium text-[#18181b]">{{ item.fullName }}</h2>

        <!-- Подзаголовок: телефон -->
        <div class="group/subtitle mt-0.5 flex items-center gap-x-2">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ item.fullPhone ?? item.phone }}</span>
          <button
            @click="navigator.clipboard.writeText(item.fullPhone ?? item.phone)"
            class="invisible group-hover/subtitle:visible text-zinc-900 hover:text-zinc-600 transition-colors"
          >
            <CopyIcon :size="14" />
          </button>
        </div>

        <!-- Строка действий -->
        <div class="flex items-center gap-x-2 mt-4">
          <button class="flex flex-1 items-center justify-center gap-x-1.5 rounded-lg px-3 h-9 text-[14px] leading-[20px] font-medium text-white bg-[#5B4FCF] hover:bg-[#4e44b8] active:bg-[#443aaa] transition-colors">
            Обновить рейтинг
            <ArrowUpRightIcon :size="14" class="shrink-0" />
          </button>
        </div>
      </div>

      <!-- Детали + Проверка -->
      <div class="px-6 pt-[40px]">

        <!-- Дата создания -->
        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Дата создания</span>
          <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ formatDate(item.date) }}</span>
        </div>

        <!-- Дата проверки -->
        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Дата проверки</span>
          <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.lastCheckDate ? formatDate(item.lastCheckDate) : '—' }}</span>
        </div>

        <!-- Рейтинг -->
        <template v-if="checks[0]">
          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Рейтинг</span>
            <div class="flex items-center gap-x-2">
              <div class="relative h-2 w-[61px] rounded-full bg-[#e4e4e7] overflow-hidden shrink-0">
                <div class="absolute inset-y-0 left-0 rounded-full" :style="{ width: scoreBarWidth(checks[0].score), background: scoreDotColor(checks[0].score) }" />
              </div>
              <span class="text-[14px] leading-[20px] font-normal text-[#18181b] w-8 text-right">{{ checks[0].score }}</span>
            </div>
          </div>

          <!-- Долговая нагрузка -->
          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Долговая нагрузка</span>
            <div v-if="checks[0].debtLoad != null" class="flex items-center gap-x-2">
              <div class="relative h-2 w-[61px] rounded-full bg-[#e4e4e7] overflow-hidden shrink-0">
                <div class="absolute inset-y-0 left-0 rounded-full" :style="{ width: checks[0].debtLoad + '%', background: debtLoadColor(checks[0].debtLoad) }" />
              </div>
              <span class="text-[14px] leading-[20px] font-normal text-[#18181b] w-8 text-right">{{ checks[0].debtLoad }}%</span>
            </div>
            <span v-else class="text-[14px] leading-[20px] font-light text-[#71717a]">—</span>
          </div>

          <!-- Сумма платежей -->
          <div class="flex items-start justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Сумма всех платежей по кредитам</span>
            <span class="text-[14px] leading-[20px] font-normal text-[#18181b] shrink-0 ml-4">{{ checks[0].totalPayments ?? '—' }}</span>
          </div>
        </template>

        <!-- Менеджер -->
        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Менеджер</span>
          <CatalystListbox :options="managerOptions" v-model="managerValue" placeholder="Менеджер" />
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
      <div class="px-6 pt-[40px]">
        <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Анкета</span>
        <div class="mt-2 flex flex-col">
          <div class="flex items-center py-3">
            <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">{{ item.initials }}</span>
            <div class="flex flex-col gap-y-0.5 pl-4">
              <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item.fullName }}</span>
              <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ item.fullPhone ?? item.phone }}</span>
            </div>
            <a href="#" class="ml-auto text-[14px] leading-[20px] font-medium text-[#5B4FCF] hover:opacity-80">Открыть</a>
          </div>
        </div>
      </div>

    </div>

    <!-- Футер -->
    <div class="border-t border-[#f4f4f5] bg-[#fafafa] px-6 py-3 shrink-0">
      <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">Чат с поддержкой</span>
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
  Copy as CopyIcon,
  CircleX as CircleXIcon,
  ArrowUpRight as ArrowUpRightIcon,
} from 'lucide-vue-next'

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (status === 'Выполнена')       return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'На проверке')     return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'Ошибка проверки') return `${base} bg-red-50 text-red-700 inset-ring-red-600/10`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerValue = ref('Смирнова Юлия')

const props = defineProps({
  open: Boolean,
  item: { type: Object, default: null },
})

defineEmits(['close'])


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
