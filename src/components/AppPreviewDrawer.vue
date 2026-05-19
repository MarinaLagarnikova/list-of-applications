<template>
  <BaseDrawer :open="open" @close="$emit('close')">

    <!-- Scrollable body -->
    <div class="flex-1 overflow-y-auto">

      <!-- Header -->
      <div class="px-6 pt-6 pb-0">
        <div class="flex items-center gap-2">
          <!-- Avatar -->
          <div class="size-8 rounded-full bg-[#e4e4e7] flex items-center justify-center shrink-0">
            <span class="text-sm font-medium text-[#71717a] leading-none">{{ app.initials }}</span>
          </div>
          <span class="flex-1 text-[20px] leading-[32px] font-semibold text-[#18181b]">{{ app.client }}</span>
          <button @click="$emit('close')" class="flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors">
            <XIcon :size="20" />
          </button>
        </div>

        <!-- Action buttons -->
        <div class="flex items-center gap-x-2 mt-4">
          <button class="flex flex-1 items-center justify-center gap-x-1.5 rounded-lg border border-zinc-950/10 bg-white px-3 h-9 text-[14px] leading-[20px] font-medium text-[#09090b] shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors">
            Перейти в заявку
            <ExternalLinkIcon :size="14" class="shrink-0" />
          </button>
          <button v-if="!hideIconButtons" class="flex size-9 shrink-0 items-center justify-center rounded-lg border border-zinc-950/10 bg-white shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors">
            <Link2Icon :size="16" class="text-[#18181b]" />
          </button>

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
                class="absolute right-0 bottom-full mb-1 z-50 w-48 origin-bottom-right rounded-xl p-1 bg-white/75 backdrop-blur-xl shadow-lg ring-1 ring-zinc-950/10"
              >
                <button class="flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-zinc-100 transition-colors text-left">
                  <CopyIcon :size="14" class="shrink-0 text-zinc-500" />
                  Скопировать ссылку
                </button>
                <button class="flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-zinc-100 transition-colors text-left">
                  <PrinterIcon :size="14" class="shrink-0 text-zinc-500" />
                  Распечатать
                </button>
                <div class="mx-3 my-1 h-px bg-zinc-950/5" />
                <button class="flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-red-600 hover:bg-red-50 transition-colors text-left">
                  <TrashIcon :size="14" class="shrink-0" />
                  Удалить заявку
                </button>
              </div>
            </Transition>
          </div>
        </div>
      </div>

      <!-- Detail rows -->
      <div class="px-6 pt-6">
        <template v-for="row in rows" :key="row.label">
          <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
            <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ row.label }}</span>
            <CatalystListbox
              v-if="row.options"
              :options="row.options"
              :model-value="selectValues[row.label] ?? row.value"
              @update:model-value="selectValues[row.label] = $event"
              placeholder="Выбрать"
            />
            <span v-else-if="row.date" class="flex items-center gap-x-3 text-[14px] leading-[20px] font-normal text-[#18181b]">
              <span>{{ row.value }}</span>
              <span class="text-[#71717a]">{{ row.date }}</span>
              <button v-if="row.copy" @click="navigator.clipboard.writeText(row.copy)" class="text-zinc-900 hover:text-zinc-600 transition-colors">
                <CopyIcon :size="14" />
              </button>
            </span>
            <span v-else class="flex items-center gap-x-3 text-[14px] leading-[20px] font-normal text-[#18181b]">
              <span>{{ row.value }}</span>
              <button v-if="row.copy" @click="navigator.clipboard.writeText(row.copy)" class="text-zinc-900 hover:text-zinc-600 transition-colors">
                <CopyIcon :size="14" />
              </button>
            </span>
          </div>
        </template>
      </div>

      <!-- Подписи section -->
      <template v-if="showSignatures">
        <div class="px-6 pt-[40px] pb-[40px]">
          <div class="flex items-center justify-between">
            <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Подписи</span>
            <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
          </div>
          <div class="mt-2 flex flex-col">
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">НЕ</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">Новикова Е.Д.</span>
                <div class="flex items-center gap-x-1.5">
                  <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">+7 931 208-05-14</span>
                  <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                  <span class="text-[14px] leading-[20px] font-normal text-[#16a34a]">Подпись выпущена</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>

      <!-- История section -->
      <div v-if="showHistory" class="px-6 pt-6">
        <div class="flex flex-col gap-6">
          <div v-for="(event, i) in historyEvents" :key="i" class="relative flex items-start gap-4">
            <div v-if="i < historyEvents.length - 1" class="absolute left-[9px] top-[10px] w-px bg-[#e5e7eb]" style="height: calc(100% + 24px)" />
            <span class="relative z-10 shrink-0 flex size-5 items-center justify-center rounded-full" :class="event.gray ? 'bg-zinc-300' : 'bg-[#16a34a]'">
              <CheckIcon :size="14" class="text-white" />
            </span>
            <div class="flex flex-1 items-center gap-1 min-w-0 pt-[3px]">
              <span class="text-[14px] leading-[20px] font-normal text-zinc-900" v-html="event.text" />
              <span class="text-[14px] leading-[20px] font-light text-zinc-500 shrink-0">{{ event.time }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Анкета section -->
      <div v-if="showAnketa" class="px-8 pt-[40px] pb-[40px]">
        <div class="flex items-center justify-between">
          <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Анкета</span>
          <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
        </div>
        <div class="mt-2 flex flex-col">
          <!-- Страховка: один участник с телефоном -->
          <template v-if="anketaPhoneMode">
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">{{ app.initials }}</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">{{ app.client }}</span>
                <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">{{ app.phone }}</span>
              </div>
            </div>
          </template>
          <!-- Ипотека: два участника с ролями и PDF -->
          <template v-else>
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">НЕ</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">Новикова Е. Д.</span>
                <div class="flex items-center gap-x-1.5">
                  <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">Заемщик</span>
                  <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Новикова.pdf</a>
                  <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                </div>
              </div>
            </div>
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">НА</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">Новиков А. С.</span>
                <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">Созаемщик</span>
              </div>
            </div>
          </template>
        </div>
      </div>

      <!-- Одобрение section -->
      <template v-if="showApproval">
        <div class="px-6 pt-0 pb-[40px]">
          <div class="flex items-center justify-between">
            <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Одобрение</span>
            <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
          </div>
          <div class="mt-2 flex flex-col">
            <div v-for="bank in approvalBanks" :key="bank.name" class="flex items-center py-3 border-b border-[#f4f4f5] last:border-0">
              <div class="flex flex-1 flex-col gap-y-0.5">
                <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">{{ bank.name }}</span>
                <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ bank.payment }}</span>
              </div>
              <div class="flex-1 flex justify-center">
                <span class="text-[14px] leading-[20px] font-light text-[#71717a] w-12 text-left">{{ bank.rate }}</span>
              </div>
              <span :class="[
                'inline-flex items-center rounded-md px-2 py-1 text-[12px] leading-[16px] font-medium shrink-0',
                bank.status === 'Одобрена'
                  ? 'bg-green-50 text-green-700'
                  : 'bg-violet-50 text-violet-600',
              ]">{{ bank.status }}</span>
            </div>
          </div>
        </div>
      </template>

      <!-- Документы + Подписи (регистрация) -->
      <template v-if="showDocuments">

        <!-- Документы section -->
        <div class="px-6 pt-[40px]">
          <div class="flex items-center justify-between">
            <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Документы</span>
            <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
          </div>
          <div class="mt-3 flex flex-col gap-y-2">
            <div v-for="doc in regDocuments" :key="doc.name" class="flex items-center gap-3 rounded-2xl border border-[#e4e4e7] bg-white px-4 py-2">
              <span class="shrink-0 rounded-md bg-[rgba(82,82,91,0.1)] px-1.5 py-1 text-[12px] leading-[16px] font-medium text-[#3f3f46]">{{ doc.ext }}</span>
              <div class="flex flex-1 min-w-0 flex-col">
                <span class="text-[13px] leading-[20px] font-semibold text-zinc-900 truncate">{{ doc.name }}</span>
                <span class="text-[11px] leading-[16px] font-normal text-zinc-500">{{ doc.size }}</span>
              </div>
              <button class="flex size-10 shrink-0 items-center justify-center rounded-lg hover:bg-zinc-50 transition-colors">
                <MoreHorizontalIcon :size="16" class="text-[#18181b]" />
              </button>
            </div>
          </div>
        </div>

        <!-- Подписи section -->
        <div class="px-6 pt-[40px] pb-[40px]">
          <div class="flex items-center justify-between">
            <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Подписи</span>
            <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
          </div>
          <div class="mt-2 flex flex-col">
            <div class="flex items-center py-3">
              <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">{{ app.initials }}</span>
              <div class="flex flex-col gap-y-0.5 pl-4">
                <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">{{ app.client }}</span>
                <div class="flex items-center gap-x-1.5">
                  <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">{{ app.fullPhone }}</span>
                  <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                  <span class="text-[14px] leading-[20px] font-normal text-[#16a34a]">Подпись выпущена</span>
                </div>
              </div>
            </div>
          </div>
        </div>

      </template>


    </div>
  </BaseDrawer>
</template>

<script setup>
import { ref, computed, watch, reactive } from 'vue'
import BaseDrawer from './BaseDrawer.vue'
import CatalystListbox from './CatalystListbox.vue'
import {
  X as XIcon,
  Copy as CopyIcon,
  ExternalLink as ExternalLinkIcon,
  Link2 as Link2Icon,
  Check as CheckIcon,
  BadgeCheck as BadgeCheckIcon,
  MoreHorizontal as MoreHorizontalIcon,
  Printer as PrinterIcon,
  Trash2 as TrashIcon,
  FileText as FileTextIcon,
} from 'lucide-vue-next'

const moreMenuOpen = ref(false)
const selectValues = reactive({})

const props = defineProps({
  open: Boolean,
  app: { type: Object, default: null },
  rows: { type: Array, default: () => [] },
  history: { type: Array, default: null },
  showDocuments: { type: Boolean, default: false },
  showApproval: { type: Boolean, default: false },
  showAnketa: { type: Boolean, default: true },
  anketaPhoneMode: { type: Boolean, default: false },
  hideIconButtons: { type: Boolean, default: false },
  showHistory: { type: Boolean, default: true },
  showSignatures: { type: Boolean, default: false },
})

const approvalBanks = [
  { name: 'МТС',           rate: '15%',   payment: '156 000 ₽/мес', status: 'Отправлена' },
  { name: 'Газпромбанк',   rate: '15.5%', payment: '166 678 ₽/мес', status: 'Отправлена' },
  { name: 'Промсвязьбанк', rate: '16%',   payment: '189 567 ₽/мес', status: 'Отправлена' },
]

const regDocuments = [
  { ext: 'PDF', name: 'Договор купли-продажи.pdf', size: '2.3 MB' },
  { ext: 'JPG', name: 'Паспорт — разворот.jpg',    size: '4.9 MB' },
]

defineEmits(['close'])

const appDate = computed(() => {
  if (!props.app?.createdAt) return ''
  const [day, month, year] = props.app.createdAt.split('.')
  const d = new Date(+year, +month - 1, +day)
  return d.toLocaleDateString('ru-RU', { day: 'numeric', month: 'long' }).replace(' г.', '')
})

const defaultHistory = computed(() => [
  { text: 'Заявка создана', time: '2 дня назад' },
  { text: 'Банки подобраны', time: '1 день назад' },
  { text: 'Анкета заполнена', time: '1 день назад' },
  { text: 'Клиент проверен', time: '2 часа назад' },
  { text: 'Анкета подписана', time: '', gray: true },
  { text: 'Получено одобрение', time: '', gray: true },
])

const historyEvents = computed(() => props.history ?? defaultHistory.value)
</script>
