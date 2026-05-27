<template>
  <BaseDrawer :open="open" @close="$emit('close')">

    <!-- Scrollable body -->
    <div class="flex-1 overflow-y-auto">
      <div class="flex flex-col min-h-full">

      <!-- 1. Шапка -->
      <div class="px-6 pt-6 pb-0">

        <!-- Строка бейджей: ID + статус + закрыть -->
        <div class="flex items-center gap-2">
          <span class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">
            {{ app.id }}
          </span>
          <span v-if="app.status" :class="statusBadgeClass(app.status)">{{ app.status }}</span>
          <button @click="$emit('close')" class="ml-auto flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors">
            <XIcon :size="16" />
          </button>
        </div>

        <!-- Заголовок: ФИО / компания / документ -->
        <h2 class="mt-2 text-[24px] leading-[32px] font-medium text-[#18181b]">{{ displayName }}</h2>

        <!-- Подзаголовок: телефон / ИНН / кол-во подписантов -->
        <div class="group/subtitle mt-0.5 flex items-center gap-x-2">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ subtitle ?? (app.inn ? 'ИНН ' + app.inn : null) ?? app.fullPhone ?? app.phone }}</span>
          <button
            @click="navigator.clipboard.writeText(subtitle ?? (app.inn ? 'ИНН ' + app.inn : null) ?? app.fullPhone ?? app.phone)"
            class="invisible group-hover/subtitle:visible text-zinc-900 hover:text-zinc-600 transition-colors"
          >
            <CopyIcon :size="14" />
          </button>
        </div>

        <!-- Строка действий -->
        <div class="flex items-center gap-x-2 mt-4">
          <button :class="['flex items-center justify-center gap-x-1.5 rounded-lg px-3 h-9 text-[14px] leading-[20px] font-medium text-white bg-[#5B4FCF] hover:bg-[#4e44b8] active:bg-[#443aaa] transition-colors', hideIconButtons ? 'w-full' : 'flex-1']">
            {{ primaryActionLabel }}
            <ArrowUpRightIcon :size="14" class="shrink-0" />
          </button>
          <div v-if="!hideIconButtons" class="relative group/link">
            <button
              @click="copyLink"
              class="flex size-9 shrink-0 items-center justify-center rounded-lg border border-zinc-950/10 bg-white shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors"
            >
              <Link2Icon :size="16" class="text-[#18181b]" />
            </button>
            <div class="pointer-events-none absolute top-full left-1/2 -translate-x-1/2 mt-2 hidden group-hover/link:flex flex-col items-center z-50">
              <div class="size-0 border-x-4 border-x-transparent border-b-4 border-b-zinc-900" />
              <div class="rounded-lg bg-zinc-900 px-2.5 py-1.5 text-[12px] leading-[16px] font-medium text-white whitespace-nowrap">
                {{ linkCopied ? 'Скопирована' : 'Скопировать ссылку' }}
              </div>
            </div>
          </div>

          <!-- Три точки -->
          <div v-if="!hideIconButtons" class="relative">
            <div v-if="moreMenuOpen" class="fixed inset-0 z-40" @click="moreMenuOpen = false" />
            <button
              @click="moreMenuOpen = !moreMenuOpen"
              class="flex size-9 shrink-0 items-center justify-center rounded-lg border border-zinc-950/10 bg-white shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors"
            >
              <MoreHorizontalIcon :size="16" class="text-[#18181b]" />
            </button>
            <Transition
              enter-active-class="transition ease-out duration-100"
              enter-from-class="opacity-0 scale-95"
              enter-to-class="opacity-100 scale-100"
              leave-active-class="transition ease-in duration-75"
              leave-from-class="opacity-100 scale-100"
              leave-to-class="opacity-0 scale-95"
            >
              <div
                v-if="moreMenuOpen"
                class="absolute right-0 top-full mt-1 z-50 w-56 origin-top-right rounded-xl p-1 bg-white/75 backdrop-blur-xl shadow-lg ring-1 ring-zinc-950/10"
              >
                <button @click="copyLink" class="group flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-indigo-600 hover:text-white transition-colors text-left">
                  <CopyIcon :size="16" class="shrink-0 text-zinc-500 group-hover:text-white" />
                  Создать копию
                </button>
                <div class="mx-3 my-1 h-px bg-zinc-950/5" />
                <button class="group flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-red-600 hover:bg-red-50 transition-colors text-left">
                  <TrashIcon :size="16" class="shrink-0" />
                  Удалить заявку
                </button>
              </div>
            </Transition>
          </div>
        </div>
      </div>

      <!-- 2. Контекстный блок — Встреча с курьером (цифровые подписи) -->
      <template v-if="app?.status === 'Встреча с курьером назначена' && app?.courierMeeting">
        <div class="px-6 pt-6">
          <div class="flex items-center gap-x-3 rounded-xl bg-[#fffbeb] px-4 py-4">
            <CalendarIcon :size="16" class="shrink-0 text-[#d97706]" />
            <span class="text-[14px] leading-[20px] text-[#92400e]"><span class="font-light">Встреча с курьером </span><span class="font-medium">{{ app.courierMeeting }}</span></span>
          </div>
        </div>
      </template>

      <!-- 2. Контекстный блок — Документы подписаны (цифровые подписи) -->
      <template v-if="showSignedBanner">
        <div class="px-6 pt-6">
          <div class="flex items-center gap-x-3 rounded-xl bg-[#f0fdf4] px-4 py-4">
            <BadgeCheckIcon :size="16" class="shrink-0 text-[#16a34a]" />
            <span class="text-[14px] leading-[20px] font-medium text-[#15803d]">Документы подписаны</span>
          </div>
        </div>
      </template>

      <!-- 2. Контекстный блок — Одобрение (ипотека × статус с банками) -->
      <template v-if="showApproval && ['Ожидает решения', 'Одобрена', 'Банк выбран', 'Отказано'].includes(app.status)">
        <div class="px-6 pt-6">
          <div class="flex flex-col">
            <div v-for="bank in approvalBanks" :key="bank.name" class="flex items-center py-3 border-b border-[#f4f4f5] last:border-0">
              <div class="flex flex-1 flex-col gap-y-0.5">
                <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">{{ bank.name }}</span>
                <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ bank.payment }}</span>
              </div>
              <div class="flex-1 flex justify-center">
                <span class="text-[14px] leading-[20px] font-light text-[#71717a] w-12 text-left">{{ bank.rate }}</span>
              </div>
              <span :class="[
                'inline-flex items-center rounded-md px-2 py-0.5 text-[12px] leading-[16px] font-medium shrink-0',
                bank.status === 'Одобрена'
                  ? 'bg-[#EAF3DE] text-[#3B6D11]'
                  : 'bg-[#EEEDFE] text-[#5B4FCF]',
              ]">{{ bank.status }}</span>
            </div>
          </div>
          <div class="mt-3 flex items-center gap-x-2">
            <ZapIcon :size="16" class="shrink-0 text-zinc-500" />
            <span class="text-[12px] leading-[16px] font-light text-zinc-500">Хорошая скорость — клиента одобрили спустя 1 день после отправки заявки</span>
          </div>
        </div>
      </template>

      <!-- 3. Детали -->
      <div class="px-6 pt-[40px]">
        <template v-for="row in rows" :key="row.label">
          <div class="flex items-start justify-between border-b border-[#f4f4f5] py-3">
            <span class="shrink-0 whitespace-nowrap text-[14px] leading-[20px] font-light text-[#71717a]">{{ row.label }}</span>
            <CatalystListbox
              v-if="row.options"
              :options="row.options"
              :model-value="selectValues[row.label] ?? row.value"
              @update:model-value="selectValues[row.label] = $event"
              placeholder="Выбрать"
            />
            <span v-else-if="row.date" class="ml-4 flex items-center gap-x-3 text-right text-[14px] leading-[20px] font-normal text-[#18181b]">
              <span>{{ row.value }}</span>
              <span class="text-[#71717a]">{{ row.date }}</span>
              <button v-if="row.copy" @click="navigator.clipboard.writeText(row.copy)" class="text-zinc-900 hover:text-zinc-600 transition-colors">
                <CopyIcon :size="14" />
              </button>
            </span>
            <span v-else class="ml-4 flex items-center gap-x-1.5 text-right text-[14px] leading-[20px] font-normal text-[#18181b]">
              <HouseIcon     v-if="row.icon === 'house'"   :size="16" class="shrink-0 text-zinc-900" />
              <LandmarkIcon  v-if="row.icon === 'office'"  :size="16" class="shrink-0 text-zinc-500" />
              <UserRoundIcon v-if="row.icon === 'courier'" :size="16" class="shrink-0 text-zinc-500" />
              <IdCardIcon         v-if="row.icon === 'passport'" :size="16" class="shrink-0 text-zinc-500" />
              <CirclePercentIcon  v-if="row.icon === 'percent'"  :size="16" class="shrink-0 text-zinc-500" />
              <span>{{ row.value }}</span>
              <button v-if="row.copy" @click="navigator.clipboard.writeText(row.copy)" class="ml-1.5 text-zinc-900 hover:text-zinc-600 transition-colors">
                <CopyIcon :size="14" />
              </button>
            </span>
          </div>
        </template>
      </div>

      <!-- 4. Сущности -->

      <!-- Подписи (цифровые подписи) -->
      <template v-if="showSignatures">
        <div class="px-6 pt-[40px] pb-[40px]">
          <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Подписи</span>
          <div class="mt-2 flex flex-col">
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">НЕ</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">Новикова Е.Д.</span>
                <div class="flex items-center gap-x-1.5">
                  <span class="text-[14px] leading-[20px] font-light text-[#71717a]">+7 931 208-05-14</span>
                  <template v-if="app?.status === 'Активирована'">
                    <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                    <span class="text-[14px] leading-[20px] font-light text-[#16a34a]">Выпущена</span>
                  </template>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>

      <!-- Анкета (ипотека, страховка) -->
      <div v-if="showAnketa" class="px-6 pt-[40px] pb-[40px]">
        <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Анкета</span>
        <div class="mt-2 flex flex-col">
          <!-- Страховка: один участник с телефоном -->
          <template v-if="anketaPhoneMode">
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">{{ app.initials }}</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ app.client }}</span>
                <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ app.phone }}</span>
              </div>
              <a href="#" class="ml-auto text-[14px] leading-[20px] font-medium text-[#5B4FCF] hover:opacity-80">Открыть</a>
            </div>
          </template>
          <!-- Ипотека: два участника с ролями -->
          <template v-else>
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">НЕ</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">Новикова Е. Д.</span>
                <div class="flex items-center gap-x-1.5">
                  <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Заемщик</span>
                  <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                </div>
              </div>
              <a href="#" class="ml-auto text-[14px] leading-[20px] font-medium text-[#5B4FCF] hover:opacity-80">Открыть</a>
            </div>
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">НА</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">Новиков А. С.</span>
                <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Созаемщик</span>
              </div>
              <a href="#" class="ml-auto text-[14px] leading-[20px] font-medium text-[#5B4FCF] hover:opacity-80">Открыть</a>
            </div>
          </template>
        </div>
      </div>

      <!-- Документы + Подписи (регистрация) -->
      <template v-if="showDocuments">

        <!-- Подписи -->
        <div class="px-6 pt-[40px] pb-[40px]">
          <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Подписи</span>
          <div class="mt-2 flex flex-col">
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">{{ app.initials }}</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ app.client }}</span>
                <div class="flex items-center gap-x-1.5">
                  <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ app.fullPhone }}</span>
                  <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                  <span class="text-[14px] leading-[20px] font-light text-[#16a34a]">Выпущена</span>
                </div>
              </div>
            </div>
          </div>
        </div>

      </template>

      <!-- 5. Футер -->
      <div class="mt-auto border-t border-[#f4f4f5] bg-[#fafafa] px-6 py-3">
        <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">Чат с поддержкой</span>
      </div>

      </div>
    </div>
  </BaseDrawer>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import BaseDrawer from './BaseDrawer.vue'
import CatalystListbox from './CatalystListbox.vue'
import {
  X as XIcon,
  Copy as CopyIcon,
  Link2 as Link2Icon,
  BadgeCheck as BadgeCheckIcon,
  MoreHorizontal as MoreHorizontalIcon,
  Trash2 as TrashIcon,
  MessageCircle as MessageCircleIcon,
  ArrowUpRight as ArrowUpRightIcon,
  Zap as ZapIcon,
  House as HouseIcon,
  Calendar as CalendarIcon,
  CirclePercent as CirclePercentIcon,
  Landmark as LandmarkIcon,
  UserRound as UserRoundIcon,
  IdCard as IdCardIcon,
} from 'lucide-vue-next'

const moreMenuOpen = ref(false)
const linkCopied = ref(false)


const displayName = computed(() => {
  if (!props.app) return ''
  if (props.app.companyName) return props.app.companyName
  return props.app.fullName ?? props.app.client ?? ''
})
const selectValues = reactive({})

const copyLink = () => {
  navigator.clipboard.writeText(window.location.href)
  linkCopied.value = true
  setTimeout(() => { linkCopied.value = false }, 2000)
}

const props = defineProps({
  open: Boolean,
  app: { type: Object, default: null },
  rows: { type: Array, default: () => [] },
  subtitle: { type: String, default: null },
  primaryActionLabel: { type: String, default: 'Перейти в заявку' },
  showDocuments: { type: Boolean, default: false },
  showApproval: { type: Boolean, default: false },
  showSignedBanner: { type: Boolean, default: false },
  showAnketa: { type: Boolean, default: true },
  anketaPhoneMode: { type: Boolean, default: false },
  hideIconButtons: { type: Boolean, default: false },
  showSignatures: { type: Boolean, default: false },
})

defineEmits(['close'])

const pluralizeBanks = (n) => {
  const m10 = n % 10, m100 = n % 100
  if (m100 >= 11 && m100 <= 14) return `${n} банков`
  if (m10 === 1) return `${n} банк`
  if (m10 >= 2 && m10 <= 4) return `${n} банка`
  return `${n} банков`
}

const approvalBanks = [
  { name: 'МТС',           rate: '15%',   payment: '156 000 ₽/мес', status: 'Отправлена' },
  { name: 'Газпромбанк',   rate: '15.5%', payment: '166 678 ₽/мес', status: 'Отправлена' },
  { name: 'Промсвязьбанк', rate: '16%',   payment: '189 567 ₽/мес', status: 'Одобрена' },
]

const regDocuments = [
  { ext: 'PDF', name: 'Договор купли-продажи.pdf', size: '2.3 MB' },
  { ext: 'JPG', name: 'Паспорт — разворот.jpg',    size: '4.9 MB' },
]

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  // Зелёный — ипотека, регистрация, страховка
  if (['Одобрена', 'Кредит выдан', 'ДДУ согласован', 'ДДУ зарегистрирован', 'Зарегистрировано', 'Оформлено', 'Активирована', 'Подписан'].includes(status))
    return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  // Жёлтый — ипотека, регистрация, страховка, цифровые подписи
  if (['Ожидает решения', 'На подписании', 'Согласование ДДУ', 'Объект на согласовании', 'На регистрации', 'На оформлении', 'Ожидает назначение курьера', 'Встреча с курьером назначена'].includes(status))
    return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  // Синий — регистрация, страховка, цифровые подписи
  if (['Банк выбран', 'В работе', 'Процесс выпуска создан', 'Встреча с курьером состоялась'].includes(status))
    return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
  // Красный
  if (['Отказано', 'Встреча с курьером отменена', 'Запрос на выпуск истек'].includes(status))
    return `${base} bg-red-50 text-red-700 inset-ring-red-600/10`
  // Серый — по умолчанию (Новая заявка, Консультация, Подготовка и др.)
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}
</script>
