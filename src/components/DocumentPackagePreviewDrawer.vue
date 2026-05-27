<template>
  <BaseDrawer :open="open" @close="$emit('close')">
    <div class="flex-1 overflow-y-auto">

      <!-- Header -->
      <div class="px-6 pt-6 pb-0">

        <!-- Строка бейджей: ID + статус + закрыть -->
        <div class="flex items-center gap-2">
          <span class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10">
            {{ item?.id }}
          </span>
          <span v-if="item?.status" :class="statusBadgeClass(item.status)">{{ item.status }}</span>
          <button @click="$emit('close')" class="ml-auto flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors">
            <XIcon :size="16" />
          </button>
        </div>

        <!-- Заголовок: название пакета -->
        <h2 class="mt-2 text-[24px] leading-[32px] font-medium text-[#18181b]">{{ item?.name }}</h2>

        <!-- Подзаголовок: количество подписантов -->
        <p class="mt-0.5 text-[14px] leading-[20px] font-light text-[#71717a]">{{ pluralizeSigners(item?.signers) }}</p>

        <!-- Строка действий: кнопка на всю ширину -->
        <div class="mt-4">
          <button class="flex w-full items-center justify-center gap-x-1.5 rounded-lg px-3 h-9 text-[14px] leading-[20px] font-medium text-white bg-[#5B4FCF] hover:bg-[#4e44b8] active:bg-[#443aaa] transition-colors">
            Перейти в пакет
            <ArrowUpRightIcon :size="14" class="shrink-0" />
          </button>
        </div>
      </div>

      <!-- Баннер подписания -->
      <div v-if="item?.status === 'Подписан'" class="px-6 pt-6 pb-2">
        <div class="rounded-2xl bg-green-50 px-4 py-4">
          <div>
            <div>
              <p class="text-[14px] leading-[20px] font-semibold text-[#18181b]">Документы подписаны!</p>
              <p class="mt-0.5 text-[14px] leading-[20px] font-normal text-[#71717a]">Скачайте подписанный пакет</p>
              <div class="mt-3 flex items-center gap-x-3">
                <div class="flex size-10 shrink-0 items-center justify-center rounded-xl bg-white shadow-sm">
                  <FileTextIcon :size="20" class="text-zinc-300" />
                </div>
                <div class="flex flex-col">
                  <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">Ипотека Сбербанк — Новикова Е.Д.</span>
                  <a href="#" class="text-[14px] leading-[20px] font-normal text-indigo-600 hover:text-indigo-700">Скачать</a>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Data rows -->
      <div class="px-6 pt-6">

        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Дата создания</span>
          <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item?.date ? formatDate(item.date) : '—' }}</span>
        </div>

        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Дата подписания</span>
          <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ item?.statusDate ? formatDate(item.statusDate) : '—' }}</span>
        </div>

        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Менеджер</span>
          <CatalystListbox
            :options="managerOptions"
            :model-value="managerValue"
            @update:model-value="managerValue = $event"
            placeholder="Менеджер"
          />
        </div>

      </div>

      <!-- Подписи section -->
      <div class="px-6 pt-[40px] pb-[60px]">
        <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Подписи</span>
        <div class="mt-2 flex flex-col">
          <div v-for="signer in item?.signersList" :key="signer.name" class="flex items-center py-3">
            <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">{{ signer.initials }}</span>
            <div class="flex flex-col gap-y-0.5 pl-4">
              <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ signer.name }}</span>
              <div class="flex items-center gap-x-1.5">
                <span class="text-[14px] leading-[20px] font-light text-[#71717a]">{{ signer.phone }}</span>
                <template v-if="signer.signed">
                  <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                  <span class="text-[14px] leading-[20px] font-normal text-[#16a34a]">Выпущена</span>
                </template>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Футер -->
      <div class="mt-[80px] border-t border-[#f4f4f5] bg-[#fafafa] px-6 py-3">
        <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">Чат с поддержкой</span>
      </div>

    </div>
  </BaseDrawer>
</template>

<script setup>
import BaseDrawer from './BaseDrawer.vue'
import CatalystListbox from './CatalystListbox.vue'
import { ref, watch } from 'vue'
import { X as XIcon, Copy as CopyIcon, MoreHorizontal as MoreHorizontalIcon, BadgeCheck as BadgeCheckIcon, FileText as FileTextIcon, ArrowUpRight as ArrowUpRightIcon } from 'lucide-vue-next'

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (status === 'Подписан')           return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'Готов к подписанию') return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const pluralizeSigners = (n) => {
  if (!n) return ''
  const m10 = n % 10, m100 = n % 100
  if (m100 >= 11 && m100 <= 14) return `${n} подписантов`
  if (m10 === 1) return `${n} подписант`
  if (m10 >= 2 && m10 <= 4) return `${n} подписанта`
  return `${n} подписантов`
}

const documents = [
  { ext: 'PDF', name: 'Договор купли-продажи.pdf', size: '2.3 MB' },
  { ext: 'JPG', name: 'Паспорт — разворот.jpg', size: '4.9 MB' },
]

const props = defineProps({ open: Boolean, item: { type: Object, default: null } })
defineEmits(['close'])

const managerOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const managerValue = ref(null)
watch(() => props.item, (item) => { managerValue.value = item?.manager ?? null }, { immediate: true })

const months = ['января','февраля','марта','апреля','мая','июня','июля','августа','сентября','октября','ноября','декабря']
const formatDate = (d) => {
  const [day, month] = d.split('.')
  return `${parseInt(day)} ${months[parseInt(month) - 1]}`
}
</script>
