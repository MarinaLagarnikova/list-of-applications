<template>
  <div aria-live="assertive" class="pointer-events-none fixed inset-0 z-[100] flex items-start justify-end px-4 py-6 sm:p-6">
    <div class="flex w-full flex-col items-end space-y-4">
      <Transition
        enter-active-class="transform ease-out duration-300 transition"
        enter-from-class="translate-y-2 opacity-0 sm:translate-y-0 sm:translate-x-2"
        enter-to-class="translate-y-0 sm:translate-x-0 opacity-100"
        leave-active-class="transition ease-in duration-100"
        leave-from-class="opacity-100"
        leave-to-class="opacity-0"
      >
        <div v-if="show" class="pointer-events-auto w-full max-w-sm rounded-xl bg-white shadow-lg ring-1 ring-black/5">
          <div class="p-4">
            <div class="flex items-start">
              <div class="shrink-0">
                <CheckCircleIcon class="size-6 text-green-400" aria-hidden="true" />
              </div>
              <div class="ml-3 w-0 flex-1 pt-0.5">
                <p class="text-[14px] font-medium text-gray-900">Файл готов</p>
                <p class="mt-1 text-[13px] font-light text-gray-500">{{ count }} {{ pluralLoaded(count) }} выгружены</p>
              </div>
              <div class="ml-4 flex shrink-0">
                <button
                  type="button"
                  @click="show = false"
                  class="inline-flex rounded-md text-gray-400 hover:text-gray-500"
                >
                  <XMarkIcon class="size-5" aria-hidden="true" />
                </button>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { CheckCircleIcon } from '@heroicons/vue/24/outline'
import { XMarkIcon } from '@heroicons/vue/20/solid'

const show = ref(false)
const count = ref(0)
let timer = null

const pluralLoaded = (n) => {
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return 'заявок'
  if (mod10 === 1) return 'заявка'
  if (mod10 >= 2 && mod10 <= 4) return 'заявки'
  return 'заявок'
}

const handleExport = (e) => {
  count.value = e.detail?.count ?? 0
  show.value = true
  clearTimeout(timer)
  timer = setTimeout(() => { show.value = false }, 4000)
}

onMounted(() => window.addEventListener('export-success', handleExport))
onUnmounted(() => { window.removeEventListener('export-success', handleExport); clearTimeout(timer) })
</script>
