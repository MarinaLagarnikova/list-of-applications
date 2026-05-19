<template>
  <TransitionRoot as="template" :show="open">
    <Dialog class="relative z-50" @close="$emit('close')">

      <!-- Backdrop -->
      <TransitionChild
        as="template"
        enter="ease-out duration-200" enter-from="opacity-0" enter-to="opacity-100"
        leave="ease-in duration-150" leave-from="opacity-100" leave-to="opacity-0"
      >
        <div class="fixed inset-0 bg-gray-500/50 transition-opacity" />
      </TransitionChild>

      <div class="fixed inset-0 z-10 overflow-y-auto">
        <div class="flex min-h-full items-center justify-center p-4">
          <TransitionChild
            as="template"
            enter="ease-out duration-200" enter-from="opacity-0 scale-95" enter-to="opacity-100 scale-100"
            leave="ease-in duration-150" leave-from="opacity-100 scale-100" leave-to="opacity-0 scale-95"
          >
            <DialogPanel class="relative w-full max-w-lg transform overflow-hidden rounded-2xl bg-white shadow-xl transition-all">

              <!-- Header -->
              <div class="flex items-center justify-between px-6 pt-6 pb-4 border-b border-[#f4f4f5]">
                <DialogTitle class="text-[18px] leading-[28px] font-semibold text-[#18181b]">
                  Создание заявки
                </DialogTitle>
                <button
                  @click="$emit('close')"
                  class="flex size-6 items-center justify-center text-[#71717a] hover:text-[#18181b] transition-colors"
                >
                  <XIcon :size="20" />
                </button>
              </div>

              <!-- Body -->
              <div class="px-6 py-6 flex flex-col gap-6">
                <fieldset v-if="mode === 'mortgage'">
                  <legend class="sr-only">Тип заявки</legend>
                  <div class="grid grid-cols-3 gap-3">
                    <label
                      v-for="option in applicationTypes"
                      :key="option.id"
                      class="group relative flex flex-col gap-2 rounded-lg border border-[#d1d5db] bg-white p-[17px] cursor-pointer drop-shadow-[0px_1px_1px_rgba(0,0,0,0.05)]"
                    >
                      <input type="radio" name="application-type" :value="option.id" v-model="selectedType" class="sr-only" />
                      <!-- Overlay при выборе -->
                      <div v-if="selectedType === option.id" class="pointer-events-none absolute inset-[-1px] rounded-lg border-2 border-indigo-600" />
                      <div class="flex items-center justify-between">
                        <component :is="option.icon" :size="18" :class="selectedType === option.id ? 'text-indigo-600' : 'text-zinc-400'" />
                        <span v-if="selectedType === option.id" class="flex size-[18px] shrink-0 items-center justify-center rounded-full bg-indigo-600">
                          <svg width="10" height="8" viewBox="0 0 10 8" fill="none">
                            <path d="M1 4L3.5 6.5L9 1" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                          </svg>
                        </span>
                      </div>
                      <span class="text-[13px] font-medium text-[#18181b]">{{ option.label }}</span>
                    </label>
                  </div>
                </fieldset>

                <!-- ФИО -->
                <div class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-[#18181b]">ФИО</label>
                  <input
                    v-model="fullName"
                    type="text"
                    placeholder="Иванов Иван Иванович"
                    class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-[#18181b] placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                  />
                </div>

                <!-- Телефон -->
                <div class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-[#18181b]">Телефон</label>
                  <input
                    v-model="phone"
                    type="tel"
                    placeholder="+7 (967) 890-45-34"
                    class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-[#18181b] placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                  />
                </div>

                <!-- Проект -->
                <div class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-[#18181b]">Проект</label>
                  <div class="relative">
                    <select
                      v-model="project"
                      class="w-full appearance-none rounded-lg border border-[#e4e4e7] bg-white px-3 py-2 pr-8 text-[14px] font-normal leading-[20px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition cursor-pointer"
                      :class="project ? 'text-[#18181b]' : 'text-[#a1a1aa]'"
                    >
                      <option value="" disabled>Выберите проект</option>
                      <option v-for="p in projects" :key="p" :value="p">{{ p }}</option>
                    </select>
                    <ChevronDownIcon :size="16" class="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 text-[#18181b]" />
                  </div>
                </div>
              </div>

              <!-- Footer -->
              <div class="flex items-center justify-end gap-3 px-6 py-4 border-t border-[#f4f4f5]">
                <button
                  @click="$emit('close')"
                  class="inline-flex h-9 items-center justify-center rounded-lg border border-zinc-950/10 bg-white px-4 text-[14px] font-medium text-[#09090b] shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors"
                >
                  Отмена
                </button>
                <button
                  :disabled="!phone.trim()"
                  class="inline-flex h-9 items-center justify-center rounded-lg bg-indigo-600 px-4 text-[14px] font-semibold text-white shadow-xs transition-colors disabled:opacity-40 disabled:cursor-not-allowed enabled:hover:bg-indigo-700 enabled:active:bg-indigo-800"
                >
                  Создать
                </button>
              </div>

            </DialogPanel>
          </TransitionChild>
        </div>
      </div>

    </Dialog>
  </TransitionRoot>
</template>

<script setup>
import { ref } from 'vue'
import { Dialog, DialogPanel, DialogTitle, TransitionChild, TransitionRoot } from '@headlessui/vue'
import { X as XIcon, Home as HomeIcon, ChartPie as ChartPieIcon, ShieldCheck as ShieldCheckIcon, CircleCheck as CircleCheckIcon, ChevronDown as ChevronDownIcon } from 'lucide-vue-next'

defineProps({ open: Boolean, mode: { type: String, default: 'all' } })
defineEmits(['close'])

const selectedType = ref('mortgage')
const fullName = ref('')
const phone = ref('')
const project = ref('')

const applicationTypes = [
  { id: 'mortgage', label: 'Ипотека',  icon: HomeIcon },
  { id: 'scoring',  label: 'Скоринг',  icon: ChartPieIcon },
  { id: 'insurance',label: 'Страховка', icon: ShieldCheckIcon },
]

const projects = [
  'Самолет/Новые Ватутинки',
  'А101/Прокшино',
  'MR Group/Савёловский Сити',
  'ФСК/Южная Битца',
  'ЛСР/Морская набережная',
  'Донстрой/Символ',
  'ПИК/Люберцы Парк',
  'Эталон/Галактика',
]
</script>
