<template>
  <div class="flex-1 overflow-auto px-4 md:px-6 py-4 md:py-6 flex flex-col gap-5 md:gap-8">

    <!-- ── Верхние карточки ─────────────────────────────── -->
    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 md:gap-5">
      <div v-for="stat in stats" :key="stat.label" class="flex flex-col gap-1 rounded-2xl border border-zinc-100 bg-white px-6 py-5 shadow-xs">
        <span class="text-[13px] font-light text-zinc-900">{{ stat.label }}</span>
        <div class="flex items-end justify-between gap-2 mt-1">
          <span class="text-[28px] leading-[36px] font-semibold text-zinc-900">{{ stat.value }}</span>
          <span :class="['inline-flex items-center gap-1 mb-1 text-[13px] font-medium', stat.up ? 'text-[#16a34a]' : 'text-[#dc2626]']">
            <TrendingUpIcon v-if="stat.up" :size="14" />
            <TrendingDownIcon v-else :size="14" />
            {{ stat.change }}
          </span>
        </div>
        <span class="text-[12px] font-light text-zinc-900">{{ stat.sub }}</span>
      </div>
    </div>

    <!-- ── Средняя строка ───────────────────────────────── -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 md:gap-5">

      <!-- Заявки по статусам -->
      <div class="col-span-1 md:col-span-2 flex flex-col gap-5 rounded-2xl border border-zinc-100 bg-white px-6 py-5 shadow-xs">
        <div class="flex items-center justify-between">
          <span class="text-[15px] font-medium text-zinc-900">Заявки по статусам</span>
          <div class="flex items-center gap-1 rounded-lg border border-zinc-100 p-1">
            <button v-for="p in periods" :key="p" @click="activePeriod = p"
              :class="['px-3 py-1 text-[13px] font-medium rounded-md transition-colors', activePeriod === p ? 'bg-zinc-100 text-zinc-900' : 'text-zinc-500 hover:text-zinc-700']">
              {{ p }}
            </button>
          </div>
        </div>
        <!-- Бар-чарт -->
        <div class="flex items-end gap-3 h-[140px]">
          <div v-for="bar in chartBars" :key="bar.month" class="flex flex-1 flex-col items-center gap-2">
            <div class="flex w-full flex-col items-center gap-0.5 justify-end" style="height:112px">
              <div class="w-full rounded-t-md bg-indigo-500" :style="{ height: bar.approved + '%' }" />
              <div class="w-full bg-zinc-200" :style="{ height: bar.pending + '%' }" />
              <div class="w-full rounded-b-md bg-zinc-100" :style="{ height: bar.rejected + '%' }" />
            </div>
            <span class="text-[11px] font-light text-[#a1a1aa]">{{ bar.month }}</span>
          </div>
        </div>
        <!-- Легенда -->
        <div class="flex items-center gap-5">
          <div class="flex items-center gap-1.5"><span class="size-2.5 rounded-sm bg-indigo-500" /><span class="text-[12px] font-light text-zinc-500">Одобрено</span></div>
          <div class="flex items-center gap-1.5"><span class="size-2.5 rounded-sm bg-zinc-200" /><span class="text-[12px] font-light text-zinc-500">На рассмотрении</span></div>
          <div class="flex items-center gap-1.5"><span class="size-2.5 rounded-sm bg-zinc-100" /><span class="text-[12px] font-light text-zinc-500">Отказано</span></div>
        </div>
      </div>

      <!-- Топ менеджеры -->
      <div class="flex flex-col gap-4 rounded-2xl border border-zinc-100 bg-white px-6 py-5 shadow-xs">
        <span class="text-[15px] font-medium text-zinc-900">Топ менеджеры</span>
        <div class="flex flex-col gap-3">
          <div v-for="(mgr, i) in managers" :key="mgr.name" class="flex items-center gap-3">
            <span class="text-[12px] font-light text-[#a1a1aa] w-3">{{ i + 1 }}</span>
            <span class="inline-flex size-7 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[12px] font-medium text-zinc-500">{{ mgr.initials }}</span>
            <div class="flex flex-1 flex-col gap-0.5 min-w-0">
              <span class="text-[13px] font-normal text-zinc-900 truncate">{{ mgr.name }}</span>
              <div class="relative h-1.5 w-full rounded-full bg-zinc-100 overflow-hidden">
                <div class="absolute inset-y-0 left-0 rounded-full bg-indigo-500" :style="{ width: mgr.pct + '%' }" />
              </div>
            </div>
            <span class="text-[13px] font-medium text-zinc-900 shrink-0">{{ mgr.count }}</span>
          </div>
        </div>
      </div>

    </div>

    <!-- ── Нижняя строка ────────────────────────────────── -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 md:gap-5">

      <!-- Воронка -->
      <div class="flex flex-col gap-4 rounded-2xl border border-zinc-100 bg-white px-6 py-5 shadow-xs">
        <span class="text-[15px] font-medium text-zinc-900">Воронка заявок</span>
        <div class="flex flex-col gap-2">
          <div v-for="stage in funnel" :key="stage.label" class="flex flex-col gap-1">
            <div class="flex items-center justify-between">
              <span class="text-[13px] font-light text-zinc-500">{{ stage.label }}</span>
              <span class="text-[13px] font-medium text-zinc-900">{{ stage.count }}</span>
            </div>
            <div class="relative h-2 w-full rounded-full bg-zinc-100 overflow-hidden">
              <div class="absolute inset-y-0 left-0 rounded-full" :style="{ width: stage.pct + '%', background: stage.color }" />
            </div>
          </div>
        </div>
      </div>

      <!-- Типы ипотеки -->
      <div class="flex flex-col gap-4 rounded-2xl border border-zinc-100 bg-white px-6 py-5 shadow-xs">
        <span class="text-[15px] font-medium text-zinc-900">Типы ипотеки</span>
        <div class="flex flex-col gap-3">
          <div v-for="type in mortgageTypes" :key="type.label" class="flex items-center justify-between">
            <div class="flex items-center gap-2">
              <span class="size-2 rounded-full shrink-0" :style="{ background: type.color }" />
              <span class="text-[13px] font-light text-zinc-500">{{ type.label }}</span>
            </div>
            <div class="flex items-center gap-3">
              <div class="relative h-1.5 w-[80px] rounded-full bg-zinc-100 overflow-hidden">
                <div class="absolute inset-y-0 left-0 rounded-full" :style="{ width: type.pct + '%', background: type.color }" />
              </div>
              <span class="text-[13px] font-medium text-zinc-900 w-8 text-right">{{ type.pct }}%</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Средний чек -->
      <div class="flex flex-col gap-4 rounded-2xl border border-zinc-100 bg-white px-6 py-5 shadow-xs">
        <span class="text-[15px] font-medium text-zinc-900">Средние показатели</span>
        <div class="flex flex-col">
          <div v-for="avg in averages" :key="avg.label" class="flex items-center justify-between border-b border-[#f4f4f5] py-3 last:border-0">
            <span class="text-[13px] font-light text-zinc-500">{{ avg.label }}</span>
            <span class="text-[13px] font-medium text-zinc-900">{{ avg.value }}</span>
          </div>
        </div>
      </div>

    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue'
import { TrendingUp as TrendingUpIcon, TrendingDown as TrendingDownIcon } from 'lucide-vue-next'

const activePeriod = ref('Месяц')
const periods = ['Неделя', 'Месяц', 'Квартал', 'Год']

const stats = [
  { label: 'Всего заявок',       value: '12 456', change: '+8.2%',  up: true,  sub: 'За последние 30 дней' },
  { label: 'Одобрено',           value: '3 891',  change: '+12.4%', up: true,  sub: 'Конверсия 31.2%' },
  { label: 'На рассмотрении',    value: '1 247',  change: '−3.1%',  up: false, sub: 'Средний срок 4.2 дня' },
  { label: 'Сумма кредитов',     value: '₽ 4.7B', change: '+15.8%', up: true,  sub: 'Средний чек 12.4M ₽' },
]

const chartBars = [
  { month: 'Дек', approved: 40, pending: 30, rejected: 20 },
  { month: 'Янв', approved: 50, pending: 25, rejected: 15 },
  { month: 'Фев', approved: 35, pending: 35, rejected: 25 },
  { month: 'Мар', approved: 60, pending: 20, rejected: 15 },
  { month: 'Апр', approved: 55, pending: 28, rejected: 12 },
  { month: 'Май', approved: 65, pending: 22, rejected: 10 },
]

const managers = [
  { name: 'Демо ДВИЖ',    initials: 'ДМ', count: 234, pct: 100 },
  { name: 'Иванов А.В.',  initials: 'ИА', count: 187, pct: 80  },
  { name: 'Петрова М.С.', initials: 'ПМ', count: 156, pct: 67  },
  { name: 'Сидоров К.Н.', initials: 'СК', count: 98,  pct: 42  },
  { name: 'Козлова Е.П.', initials: 'КЕ', count: 74,  pct: 32  },
]

const funnel = [
  { label: 'Новые заявки',      count: '4 120', pct: 100, color: '#6366f1' },
  { label: 'Анкета заполнена',  count: '3 408', pct: 83,  color: '#818cf8' },
  { label: 'На рассмотрении',   count: '2 190', pct: 53,  color: '#a5b4fc' },
  { label: 'Одобрено',          count: '1 285', pct: 31,  color: '#c7d2fe' },
  { label: 'Кредит выдан',      count: '891',   pct: 22,  color: '#e0e7ff' },
]

const mortgageTypes = [
  { label: 'Семейная ипотека',   pct: 38, color: '#6366f1' },
  { label: 'Льготная ипотека',   pct: 27, color: '#f59e0b' },
  { label: 'Стандартная',        pct: 19, color: '#10b981' },
  { label: 'IT-ипотека',         pct: 11, color: '#3b82f6' },
  { label: 'Сельская ипотека',   pct: 5,  color: '#e11d48' },
]

const averages = [
  { label: 'Сумма кредита',    value: '12 400 000 ₽' },
  { label: 'Первый взнос',     value: '28%' },
  { label: 'Срок кредита',     value: '22 года' },
  { label: 'Ставка',           value: '14.3%' },
  { label: 'Срок одобрения',   value: '4.2 дня' },
]
</script>
