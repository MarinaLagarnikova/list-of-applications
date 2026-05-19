<template>
  <BaseDrawer :open="open" @close="$emit('close')">
    <div class="flex-1 overflow-y-auto">

      <!-- Header -->
      <div class="px-6 pt-6 pb-0">
        <div class="flex items-center gap-2">
          <span class="flex-1 text-[20px] leading-[32px] font-semibold text-[#18181b]">{{ item?.name }}</span>
          <button @click="$emit('close')" class="flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors">
            <XIcon :size="20" />
          </button>
        </div>

        <!-- Action buttons -->
        <div class="flex items-center gap-x-2 mt-4">
          <button class="flex flex-1 items-center justify-center gap-x-1.5 rounded-lg border border-zinc-950/10 bg-white px-3 h-9 text-[14px] leading-[20px] font-medium text-[#09090b] shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors">
            Перейти в пакет
            <ExternalLinkIcon :size="14" class="shrink-0" />
          </button>
        </div>
      </div>

      <!-- Data rows -->
      <div class="px-6 pt-6">

        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">ID</span>
          <span class="flex items-center gap-x-3 text-[14px] leading-[20px] font-normal text-[#18181b]">
            <span>{{ item?.id }}</span>
            <button @click="navigator.clipboard.writeText(item?.id)" class="text-zinc-900 hover:text-zinc-600 transition-colors">
              <CopyIcon :size="14" />
            </button>
          </span>
        </div>

        <div class="flex items-center justify-between border-b border-[#f4f4f5] py-3">
          <span class="text-[14px] leading-[20px] font-light text-[#71717a]">Статус</span>
          <span class="inline-flex items-center rounded-md bg-[rgba(82,82,91,0.1)] px-1.5 py-1 text-[12px] leading-[16px] font-medium text-[#3f3f46]">{{ item?.status }}</span>
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

      <!-- Баннер подписания -->
      <div class="px-6 pt-[40px] pb-2">
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

      <!-- Документы section -->
      <div class="px-6 pt-[40px]">
        <div class="flex items-center justify-between">
          <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Документы</span>
          <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
        </div>
        <div class="mt-3 flex flex-col gap-y-2">
          <div v-for="doc in documents" :key="doc.name" class="flex items-center gap-3 rounded-2xl border border-[#e4e4e7] bg-white px-4 py-2">
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

      <!-- Анкета section -->
      <div class="px-6 pt-[40px] pb-[40px]">
        <div class="flex items-center justify-between">
          <span class="text-[16px] leading-[24px] font-medium text-[#111827]">Подписи</span>
          <a href="#" class="text-[14px] leading-[20px] font-medium text-indigo-600 hover:text-indigo-700">Открыть</a>
        </div>
        <div class="mt-2 flex flex-col">
          <!-- Заемщик — подписана -->
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
          <!-- Созаемщик — не подписана -->
          <div class="flex items-center py-3">
            <span class="inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] leading-[20px] font-medium text-[#71717a]">НА</span>
            <div class="flex flex-col gap-y-0.5 pl-4">
              <span class="text-[14px] leading-[20px] font-medium text-[#18181b]">Новиков А. С.</span>
              <div class="flex items-center gap-x-1.5">
                <span class="text-[14px] leading-[20px] font-normal text-[#71717a]">+7 931 208-05-14</span>
                <BadgeCheckIcon :size="16" class="text-[#16a34a]" />
                <span class="text-[14px] leading-[20px] font-normal text-[#16a34a]">Подпись выпущена</span>
              </div>
            </div>
          </div>
        </div>
      </div>


    </div>
  </BaseDrawer>
</template>

<script setup>
import BaseDrawer from './BaseDrawer.vue'
import CatalystListbox from './CatalystListbox.vue'
import { ref, watch } from 'vue'
import { X as XIcon, ExternalLink as ExternalLinkIcon, Copy as CopyIcon, MoreHorizontal as MoreHorizontalIcon, BadgeCheck as BadgeCheckIcon, Check as CheckIcon, FileText as FileTextIcon } from 'lucide-vue-next'

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
