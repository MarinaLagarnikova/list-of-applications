<template>
  <div class="flex min-h-screen items-center justify-center bg-gray-50">
    <div class="flex flex-col items-center gap-12">

      <!-- Logo -->
      <svg width="140" height="45" viewBox="0 0 93 30" fill="none" xmlns="http://www.w3.org/2000/svg">
        <g clip-path="url(#clip0)">
          <path d="M84.3656 23.0559H89.3382L79.832 14.4849L79.6442 15.5563L92.0123 6.79249H87.2072L77.9096 15.7063L84.3656 23.0559ZM64.1832 23.0559H68.8005L79.5604 12.7493L73.7104 6.81392H68.9049L77.6171 13.8421V12.835L64.1832 23.0559ZM58.0198 23.4202C60.5687 23.4202 63.0131 21.706 63.5982 18.5776H63.6608V23.0559H66.1471V11.228H63.6608V15.5135H63.5982C63.0341 12.3422 60.5896 10.8637 58.0824 10.8637C54.8231 10.8637 52.2533 13.3707 52.2533 17.0777C52.2533 20.7632 54.8231 23.4202 58.0198 23.4202ZM58.751 20.6775C56.2021 20.6775 54.6351 19.0918 54.6351 17.0777C54.6351 15.0635 56.2021 13.6493 58.751 13.6493C61.5715 13.6493 63.5773 15.342 63.5773 17.0348C63.5773 18.749 61.5925 20.6775 58.751 20.6775ZM42.5589 23.0559H45.0452V17.3348C45.0452 15.2992 47.824 13.0493 51.4594 13.885V10.8637C47.6151 10.628 45.7556 13.3921 45.1288 15.942H45.0661V11.228H42.5799V23.0559H42.5589ZM33.1989 20.6346C30.3784 20.6346 28.8114 19.0704 28.8114 17.0562C28.8114 15.0421 30.3784 13.6493 33.1989 13.6493C35.9986 13.6493 37.6282 15.0421 37.6282 17.0562C37.6282 19.0918 35.9986 20.6346 33.1989 20.6346ZM33.1989 23.4202C37.1686 23.4202 40.01 20.7203 40.01 17.0562C40.01 13.3493 37.1477 10.8637 33.1989 10.8637C29.2292 10.8637 26.4296 13.3493 26.4296 17.0562C26.4296 20.7417 29.2292 23.4202 33.1989 23.4202ZM21.3944 23.0773H23.8806V6.79249H21.3944V23.0773ZM9.7988 23.4202C13.6222 23.4202 16.3592 21.9631 18.0098 20.7417V14.3992H9.75699V17.4205H17.759V17.6133C16.9233 18.6847 14.2699 20.3132 10.1749 20.3132C6.01717 20.2918 2.71609 18.6204 2.71609 15.0206C2.71609 11.3137 6.247 9.55663 9.7988 9.55663C13.3506 9.55663 15.1683 11.2922 15.7951 13.285L18.4694 12.385C17.4038 8.39955 13.8312 6.42822 9.75699 6.42822C5.11877 6.42822 0 9.04237 0 14.9992C0 20.4203 4.34574 23.4202 9.7988 23.4202Z" fill="#2D03E6"/>
        </g>
        <defs>
          <clipPath id="clip0">
            <rect width="92.138" height="17.1419" fill="white" transform="translate(0 6.42822)"/>
          </clipPath>
        </defs>
      </svg>

      <!-- Buttons -->
      <div class="grid grid-cols-2 gap-4">
        <button
          @click="$emit('select', 'all')"
          class="flex flex-col items-start gap-3 rounded-2xl border border-zinc-200 bg-white px-8 py-6 text-left shadow-sm transition hover:border-indigo-300 hover:shadow-md w-64 cursor-pointer"
        >
          <span class="flex size-10 items-center justify-center rounded-lg bg-indigo-50 text-indigo-600">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <rect width="7" height="7" x="3" y="3" rx="1"/><rect width="7" height="7" x="14" y="3" rx="1"/><rect width="7" height="7" x="14" y="14" rx="1"/><rect width="7" height="7" x="3" y="14" rx="1"/>
            </svg>
          </span>
          <div>
            <div class="text-sm font-semibold text-zinc-900">Все модули</div>
            <div class="mt-1 text-xs text-zinc-500">Полная версия платформы для всех продуктов</div>
          </div>
        </button>

        <button
          @click="$emit('select', 'mortgage')"
          class="flex flex-col items-start gap-3 rounded-2xl border border-zinc-200 bg-white px-8 py-6 text-left shadow-sm transition hover:border-indigo-300 hover:shadow-md w-64 cursor-pointer"
        >
          <span class="flex size-10 items-center justify-center rounded-lg bg-indigo-50 text-indigo-600">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/>
            </svg>
          </span>
          <div>
            <div class="text-sm font-semibold text-zinc-900">Модули ипотеки</div>
            <div class="mt-1 text-xs text-zinc-500">Ипотека, Скоринг, Страховка, Аналитика, Калькулятор</div>
          </div>
        </button>

        <button
          @click="$emit('select', 'mortgage-only')"
          class="flex flex-col items-start gap-3 rounded-2xl border border-zinc-200 bg-white px-8 py-6 text-left shadow-sm transition hover:border-indigo-300 hover:shadow-md w-64 cursor-pointer"
        >
          <span class="flex size-10 items-center justify-center rounded-lg bg-indigo-50 text-indigo-600">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/>
            </svg>
          </span>
          <div>
            <div class="text-sm font-semibold text-zinc-900">Только ипотека</div>
            <div class="mt-1 text-xs text-zinc-500">Один модуль ипотеки</div>
          </div>
        </button>

        <button
          @click="$emit('select', 'registration')"
          class="flex flex-col items-start gap-3 rounded-2xl border border-zinc-200 bg-white px-8 py-6 text-left shadow-sm transition hover:border-indigo-300 hover:shadow-md w-64 cursor-pointer"
        >
          <span class="flex size-10 items-center justify-center rounded-lg bg-indigo-50 text-indigo-600">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="M20 7H4a2 2 0 0 0-2 2v10a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2Z"/><path d="M16 21V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v16"/>
            </svg>
          </span>
          <div>
            <div class="text-sm font-semibold text-zinc-900">Регистрация</div>
            <div class="mt-1 text-xs text-zinc-500">Пакеты документов, Регистрация, Пакетная регистрация, Цифровые подписи</div>
          </div>
        </button>
      </div>

    </div>
  </div>
</template>

<script setup>
defineEmits(['select'])
</script>
