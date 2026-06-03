<template>
  <div class="min-h-screen bg-[#fafafa]">
    <div class="px-8 py-6">
      <button
        @click="goBack"
        class="inline-flex items-center gap-x-1.5 text-[14px] font-medium text-zinc-500 hover:text-zinc-900 transition-colors mb-6"
      >
        <ArrowLeftIcon :size="16" class="shrink-0" />
        Назад
      </button>
      <h1 class="text-[24px] leading-[32px] font-semibold text-zinc-900">{{ pageTitle }}</h1>
    </div>
  </div>
</template>

<script setup>
import { ArrowLeft as ArrowLeftIcon } from 'lucide-vue-next'

const params = new URLSearchParams(window.location.search)
const pageTitle = params.get('title') ?? 'Это страница заявки'

const goBack = () => window.close()
</script>
