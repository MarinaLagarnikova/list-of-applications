<template>
  <div :class="['flex h-screen overflow-hidden bg-gray-50 md:p-2 transition-all duration-300', !sidebarCollapsed && 'lg:pl-4 lg:gap-x-4']">

    <!-- ─── Backdrop (мобильный оверлей) ─────────── -->
    <Transition enter-active-class="transition-opacity duration-200" enter-from-class="opacity-0" enter-to-class="opacity-100" leave-active-class="transition-opacity duration-200" leave-from-class="opacity-100" leave-to-class="opacity-0">
      <div v-if="!isLg && mobileSidebarOpen" class="fixed inset-0 z-40 bg-black/30" @click="mobileSidebarOpen = false">
        <button
          class="absolute top-4 left-[290px] flex size-11 items-center justify-center text-white transition-colors"
          @click.stop="mobileSidebarOpen = false"
        >
          <XMarkIcon :size="24" />
        </button>
      </div>
    </Transition>

    <!-- ─── Sidebar ───────────────────────────────── -->
    <div :class="[
      'flex flex-col gap-y-5 overflow-x-hidden',
      isLg
        ? ['shrink-0 transition-[width] duration-300 ease-in-out', sidebarCollapsed ? 'w-0' : 'w-[250px]']
        : ['fixed inset-y-0 left-0 z-50 w-[282px] bg-gray-50 px-4 transition-transform duration-200 ease-in-out', mobileSidebarOpen ? 'translate-x-0' : '-translate-x-full']
    ]">

      <!-- Logo -->
      <div class="flex h-16 shrink-0 items-center pl-2">
        <svg width="93" height="30" viewBox="0 0 93 30" fill="none" xmlns="http://www.w3.org/2000/svg">
          <g clip-path="url(#clip0_470_11339)">
            <path d="M84.3656 23.0559H89.3382L79.832 14.4849L79.6442 15.5563L92.0123 6.79249H87.2072L77.9096 15.7063L84.3656 23.0559ZM64.1832 23.0559H68.8005L79.5604 12.7493L73.7104 6.81392H68.9049L77.6171 13.8421V12.835L64.1832 23.0559ZM58.0198 23.4202C60.5687 23.4202 63.0131 21.706 63.5982 18.5776H63.6608V23.0559H66.1471V11.228H63.6608V15.5135H63.5982C63.0341 12.3422 60.5896 10.8637 58.0824 10.8637C54.8231 10.8637 52.2533 13.3707 52.2533 17.0777C52.2533 20.7632 54.8231 23.4202 58.0198 23.4202ZM58.751 20.6775C56.2021 20.6775 54.6351 19.0918 54.6351 17.0777C54.6351 15.0635 56.2021 13.6493 58.751 13.6493C61.5715 13.6493 63.5773 15.342 63.5773 17.0348C63.5773 18.749 61.5925 20.6775 58.751 20.6775ZM42.5589 23.0559H45.0452V17.3348C45.0452 15.2992 47.824 13.0493 51.4594 13.885V10.8637C47.6151 10.628 45.7556 13.3921 45.1288 15.942H45.0661V11.228H42.5799V23.0559H42.5589ZM33.1989 20.6346C30.3784 20.6346 28.8114 19.0704 28.8114 17.0562C28.8114 15.0421 30.3784 13.6493 33.1989 13.6493C35.9986 13.6493 37.6282 15.0421 37.6282 17.0562C37.6282 19.0918 35.9986 20.6346 33.1989 20.6346ZM33.1989 23.4202C37.1686 23.4202 40.01 20.7203 40.01 17.0562C40.01 13.3493 37.1477 10.8637 33.1989 10.8637C29.2292 10.8637 26.4296 13.3493 26.4296 17.0562C26.4296 20.7417 29.2292 23.4202 33.1989 23.4202ZM21.3944 23.0773H23.8806V6.79249H21.3944V23.0773ZM9.7988 23.4202C13.6222 23.4202 16.3592 21.9631 18.0098 20.7417V14.3992H9.75699V17.4205H17.759V17.6133C16.9233 18.6847 14.2699 20.3132 10.1749 20.3132C6.01717 20.2918 2.71609 18.6204 2.71609 15.0206C2.71609 11.3137 6.247 9.55663 9.7988 9.55663C13.3506 9.55663 15.1683 11.2922 15.7951 13.285L18.4694 12.385C17.4038 8.39955 13.8312 6.42822 9.75699 6.42822C5.11877 6.42822 0 9.04237 0 14.9992C0 20.4203 4.34574 23.4202 9.7988 23.4202Z" fill="#2D03E6"/>
          </g>
          <defs>
            <clipPath id="clip0_470_11339">
              <rect width="92.138" height="17.1419" fill="white" transform="translate(0 6.42822)"/>
            </clipPath>
          </defs>
        </svg>
      </div>


      <!-- Sidebar skeleton on initial load -->
      <template v-if="showSidebarSkeleton">
        <div class="flex flex-col gap-1 p-1 animate-pulse">
          <!-- Create button skeleton -->
          <div class="mb-2 h-9 w-full rounded-md bg-zinc-100" />
          <!-- Nav item skeletons -->
          <div v-for="i in 7" :key="`snav-${i}`" class="flex items-center gap-3 px-2 py-2">
            <div class="size-5 shrink-0 rounded bg-zinc-100" />
            <div class="h-2.5 rounded bg-zinc-100" :style="`width: ${55 + (i % 3) * 15}%`" />
          </div>
          <!-- Divider + tools -->
          <div class="my-3 h-px bg-gray-100" />
          <div v-for="i in 2" :key="`stool-${i}`" class="flex items-center gap-3 px-2 py-2">
            <div class="size-5 shrink-0 rounded bg-zinc-100" />
            <div class="h-2.5 w-3/5 rounded bg-zinc-100" />
          </div>
        </div>
      </template>

      <template v-else-if="!appIsInitialLoading">
      <!-- Sidebar Create button (все модули кроме Калькулятора и Аналитики, когда сайдбар раскрыт) -->
      <div v-if="!sidebarCollapsed" class="shrink-0">
        <!-- mortgage-only: простая кнопка без поповера -->
        <Button v-if="mode === 'mortgage-only'" color="indigo" class="w-full justify-center" @click="createModalTitle = 'Создание заявки на ипотеку'; createModalModule = 'Ипотека'; createModalOpen = true">
          <PlusIcon :size="16" class="shrink-0" />
          Создать
        </Button>
        <!-- все остальные режимы: кнопка с шевроном и поповером -->
        <div v-else class="relative w-full">
          <div v-if="createPopoverOpen" class="fixed inset-0 z-40" @click="createPopoverOpen = false" />
          <Button color="indigo" class="w-full justify-between" @click="createPopoverOpen = !createPopoverOpen">
            <span class="flex items-center gap-2">
              <PlusIcon :size="16" class="shrink-0" />
              Создать
            </span>
            <ChevronDownIcon :size="14" class="shrink-0 transition-transform duration-150" :class="createPopoverOpen ? 'rotate-180' : ''" />
          </Button>
          <Transition
            enter-active-class="transition ease-out duration-100"
            enter-from-class="opacity-0 scale-95"
            enter-to-class="opacity-100 scale-100"
            leave-active-class="transition ease-in duration-75"
            leave-from-class="opacity-100 scale-100"
            leave-to-class="opacity-0 scale-95"
          >
            <div
              v-if="createPopoverOpen"
              class="absolute left-0 top-full mt-1 z-50 w-full origin-top rounded-xl p-1 bg-white/75 backdrop-blur-xl shadow-lg ring-1 ring-zinc-950/10"
            >
              <template v-for="item in createModuleItems" :key="item.name">
                <button
                  @click="openCreateModal(item, () => { createPopoverOpen = false })"
                  class="group flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-indigo-600 hover:text-white transition-colors text-left"
                >
                  <component :is="item.icon" :size="16" class="size-4 shrink-0 text-zinc-500 group-hover:text-white" stroke-width="1.5" />
                  {{ item.label ?? item.name }}
                </button>
              </template>
            </div>
          </Transition>
        </div>
      </div>

      <!-- Nav -->
      <nav class="flex-1 min-h-0 overflow-y-auto">
        <ul role="list" class="flex flex-col gap-y-7 min-h-full">
          <li>
            <ul role="list" class="space-y-1">
              <li v-for="item in filteredNavigation" :key="item.name">

                <!-- Simple link -->
                <a
                  v-if="!item.children"
                  :href="item.href"
                  :class="[
                    activeModule === item.name && !activeSubItem
                      ? 'bg-gray-100 text-indigo-600'
                      : 'text-zinc-700 hover:bg-gray-100 hover:text-indigo-600',
                    'group flex items-center gap-x-3 rounded-md p-2 text-sm/6 font-medium',
                  ]"
                  @click.prevent="activeModule = item.name; activeSubItem = ''"
                >
                  <component
                    :is="item.icon"
                    :class="[
                      activeModule === item.name && !activeSubItem ? 'text-indigo-600' : 'text-gray-400 group-hover:text-indigo-600',
                      item.iconClass ?? 'size-6',
                      'shrink-0',
                    ]"
                    aria-hidden="true"
                  />
                  {{ item.name }}
                </a>

                <!-- Expandable -->
                <Disclosure v-else as="div" v-slot="{ open }" :defaultOpen="(item.name === 'Ипотека' && mode !== 'registration') || (item.name === 'Регистрация' && mode === 'registration')">
                  <DisclosureButton
                    :class="[
                      activeModule === item.name
                        ? 'bg-gray-100 text-indigo-600'
                        : 'text-zinc-700 hover:bg-gray-100 hover:text-indigo-600',
                      'group flex w-full items-center gap-x-3 rounded-md p-2 text-left text-sm/6 font-medium',
                    ]"
                    @click="activeModule = item.name; activeSubItem = item.children?.some(c => c.name === activeSubItem) ? activeSubItem : item.children?.[0]?.name"
                  >
                    <component
                      :is="item.icon"
                      :class="[
                        activeModule === item.name ? 'text-indigo-600' : 'text-gray-400 group-hover:text-indigo-600',
                        item.iconClass ?? 'size-6',
                        'shrink-0',
                      ]"
                      aria-hidden="true"
                    />
                    {{ item.name }}
                    <ChevronRightIcon
                      :class="[open ? 'rotate-90 text-gray-500' : 'text-gray-400', 'ml-auto size-4 shrink-0 transition-transform']"
                      aria-hidden="true"
                    />
                  </DisclosureButton>
                  <DisclosurePanel as="ul" class="mt-1 px-2">
                    <li v-for="sub in item.children" :key="sub.name">
                      <!-- Action item (e.g. Импортировать) -->
                      <a
                        v-if="sub.action"
                        :href="sub.href"
                        class="group flex items-center justify-between rounded-md py-2 pr-0 pl-9 text-sm/6 text-zinc-700 hover:bg-gray-50 hover:text-indigo-600"
                        @click.prevent="handleSubAction(sub.action)"
                      >
                        <span>{{ sub.name }}</span>
                        <ArrowUpFromLineIcon :size="16" class="shrink-0 text-zinc-400 group-hover:text-indigo-600" />
                      </a>
                      <!-- Regular nav item -->
                      <a
                        v-else
                        :href="sub.href"
                        :class="[
                          sub.name === activeSubItem && activeModule === item.name ? 'bg-gray-50 text-indigo-600' : 'text-zinc-700 hover:bg-gray-50 hover:text-indigo-600',
                          'flex items-center justify-between rounded-md py-2 pr-2 pl-9 text-sm/6',
                        ]"
                        @click.prevent="activeSubItem = sub.name; activeModule = item.name"
                      >
                        <span>{{ sub.name }}</span>
                      </a>
                    </li>
                  </DisclosurePanel>
                </Disclosure>

              </li>
            </ul>
          </li>

          <!-- Secondary nav group -->
          <li v-if="mode !== 'mortgage-only' && mode !== 'registration'">
            <div class="pl-2 text-xs/6 font-medium text-zinc-500">Инструменты</div>
            <ul role="list" class="mt-2 space-y-1">
              <li v-for="item in secondaryNavigation" :key="item.name">
                <a
                  :href="item.href"
                  :class="[
                    activeModule === item.name
                      ? 'bg-gray-100 text-indigo-600'
                      : 'text-gray-700 hover:bg-gray-100 hover:text-indigo-600',
                    'group flex gap-x-3 rounded-md p-2 text-sm/6 font-medium',
                  ]"
                  @click.prevent="activeModule = item.name; activeSubItem = ''"
                >
                  <span :class="[
                    activeModule === item.name
                      ? 'border-indigo-600 text-indigo-600'
                      : 'border-gray-200 text-zinc-500 group-hover:border-indigo-600 group-hover:text-indigo-600',
                    'flex size-6 shrink-0 items-center justify-center rounded-lg border bg-white text-[0.625rem] font-medium',
                  ]">{{ item.initial }}</span>
                  <span class="truncate">{{ item.name }}</span>
                </a>
              </li>
            </ul>
          </li>

          <!-- Support group -->
          <li class="mt-auto">
            <ul role="list" class="space-y-1">

              <!-- Написать в поддержку -->
              <li>
                <a
                  href="#"
                  :class="[
                    activeModule === 'Написать в поддержку' && !activeSubItem
                      ? 'bg-gray-100 text-indigo-600'
                      : 'text-zinc-700 hover:bg-gray-100 hover:text-indigo-600',
                    'group flex items-center gap-x-3 rounded-md p-2 text-sm/6 font-medium',
                  ]"
                  @click.prevent="activeModule = 'Написать в поддержку'; activeSubItem = ''"
                >
                  <MessageCircleQuestionIcon
                    :class="[
                      activeModule === 'Написать в поддержку' && !activeSubItem ? 'text-indigo-600' : 'text-gray-400 group-hover:text-indigo-600',
                      'size-6 shrink-0',
                    ]"
                  />
                  Написать в поддержку
                </a>
              </li>

              <!-- Справочный центр -->
              <li>
                <a
                  href="#"
                  class="text-zinc-700 hover:bg-gray-100 hover:text-indigo-600 group flex items-center gap-x-3 rounded-md p-2 text-sm/6 font-medium"
                  @click.prevent
                >
                  <LibraryIcon class="size-6 shrink-0 text-gray-400 group-hover:text-indigo-600" />
                  Справочный центр
                </a>
              </li>

              <!-- Профиль -->
              <li>
            <div class="relative">
              <div v-if="accountMenuOpen" class="fixed inset-0 z-40" @click="accountMenuOpen = false" />
              <button
                @click="accountMenuOpen = !accountMenuOpen"
                class="group flex w-full items-center gap-x-3 rounded-md p-2 text-sm/6 font-medium text-zinc-700 hover:bg-gray-100 hover:text-indigo-600 transition-colors"
              >
                <CircleUserIcon class="size-6 shrink-0 text-gray-400 group-hover:text-indigo-600" />
                <span class="flex-1 truncate text-left" aria-hidden="true">Администратор</span>
                <EllipsisIcon class="size-4 shrink-0 text-gray-400 group-hover:text-indigo-600" />
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
                  v-if="accountMenuOpen"
                  class="absolute left-1/2 -translate-x-1/2 bottom-full mb-1 z-50 w-56 origin-bottom rounded-xl p-1 bg-white/75 backdrop-blur-xl shadow-lg ring-1 ring-zinc-950/10"
                >
                  <!-- User info block -->
                  <div class="px-3 py-2">
                    <div class="text-xs text-zinc-500">Администратор</div>
                    <div class="text-sm font-medium text-zinc-900">Смирнова Юлия</div>
                  </div>
                  <div class="mx-3 my-1 h-px bg-zinc-950/5" />
                  <button class="group flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-indigo-600 hover:text-white transition-colors text-left">
                    <ArrowLeftRightIcon class="size-4 shrink-0 text-zinc-500 group-hover:text-white" />
                    Переключить учётку
                  </button>
                  <button class="group flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-indigo-600 hover:text-white transition-colors text-left">
                    <BellIcon class="size-4 shrink-0 text-zinc-500 group-hover:text-white" />
                    Уведомления
                  </button>
                  <div class="mx-3 my-1 h-px bg-zinc-950/5" />
                  <button class="group flex w-full items-center gap-x-2.5 rounded-lg px-3 py-1.5 text-sm/6 text-zinc-950 hover:bg-indigo-600 hover:text-white transition-colors text-left">
                    <LogOutIcon class="size-4 shrink-0 text-zinc-500 group-hover:text-white" />
                    Выход
                  </button>
                </div>
              </Transition>
            </div>
              </li>
            </ul>
          </li>
        </ul>
      </nav>
      </template><!-- /sidebar skeleton v-else -->
    </div>

    <!-- ─── Main content ──────────────────────────── -->
    <div class="relative flex flex-1 flex-col overflow-hidden rounded-none md:rounded-2xl bg-white shadow-none md:shadow-sm ring-0 md:ring-1 md:ring-gray-200">

      <!-- ─── Filter Drawer ────────────────────────── -->
      <FilterDrawer ref="mortgageFilterDrawerRef" :open="filterDrawerOpen" :count="activeCount ?? 0" @close="filterDrawerOpen = false" @update:filterCount="mortgageFilterCount = $event" @update:filters="mortgageFilters = $event" @update:filterTags="mortgageFilterTags = $event" />

      <!-- ─── Scoring Filter Drawer ───────────────── -->
      <ScoringImportDrawer :open="scoringImportOpen" @close="scoringImportOpen = false" />

      <ScoringFilterDrawer ref="scoringFilterDrawerRef" :open="scoringFilterDrawerOpen" :count="activeCount ?? 0" @close="scoringFilterDrawerOpen = false" @update:filterCount="scoringFilterCount = $event" @update:filters="scoringFilters = $event" @update:filterTags="scoringFilterTags = $event" />

      <!-- ─── Document Packages Filter Drawer ─────── -->
      <DocumentPackagesFilterDrawer ref="docPackagesFilterDrawerRef" :open="docPackagesFilterOpen" :count="activeCount ?? 0" @close="docPackagesFilterOpen = false" @update:filterCount="docPackagesFilterCount = $event" @update:filters="docPackagesFilters = $event" @update:filterTags="docPackagesFilterTags = $event" />

      <!-- ─── Document Package Preview Drawer ──────── -->
      <DocumentPackagePreviewDrawer :open="docPackagePreviewOpen" :item="selectedDocPackage" @close="docPackagePreviewOpen = false" />

      <!-- ─── Bank Program Drawer ──────────────────── -->
      <BankDrawer :open="bankDrawerOpen" :bank="selectedBank" @close="bankDrawerOpen = false" />

      <!-- ─── Create Application Modal ─────────────── -->
      <CreateApplicationModal :open="createModalOpen" :mode="mode" :title="createModalTitle" :module="createModalModule" @close="createModalOpen = false" @created="handleModalCreated" />
      <ToastNotification />

      <!-- ─── App Preview Drawer (Ипотека) ───────────── -->
      <AppPreviewDrawer
        :open="drawerOpen"
        :app="selectedApp"
        :rows="selectedApp ? drawerRows(selectedApp) : []"
        :show-approval="true"
        @close="drawerOpen = false"
      />

      <!-- ─── Registration Filter Drawer ─────────────────── -->
      <RegistrationFilterDrawer ref="registrationFilterDrawerRef" :open="registrationFilterOpen" :count="activeCount ?? 0" @close="registrationFilterOpen = false" @update:filterCount="registrationFilterCount = $event" @update:filters="registrationFilters = $event" @update:filterTags="registrationFilterTags = $event" />

      <!-- ─── Batch Registration Filter Drawer ──────────────── -->
      <BatchRegistrationFilterDrawer ref="batchRegistrationFilterDrawerRef" :open="batchRegistrationFilterOpen" :count="activeCount ?? 0" @close="batchRegistrationFilterOpen = false" @update:filterCount="batchRegistrationFilterCount = $event" @update:filters="batchRegistrationFilters = $event" @update:filterTags="batchRegistrationFilterTags = $event" />

      <!-- ─── App Preview Drawer (Регистрация) ──────────── -->
      <AppPreviewDrawer
        :open="registrationDrawerOpen"
        :app="selectedRegistration ?? {}"
        :rows="selectedRegistration ? registrationDrawerRows(selectedRegistration) : []"
        :show-documents="true"
        :show-anketa="false"
        @close="registrationDrawerOpen = false"
      />

      <!-- ─── Digital Signatures Filter Drawer ───────── -->
      <DigitalSignaturesFilterDrawer ref="digitalSignaturesFilterDrawerRef" :open="digitalSignaturesFilterOpen" :count="activeCount ?? 0" @close="digitalSignaturesFilterOpen = false" @update:filterCount="digitalSignaturesFilterCount = $event" @update:filters="digitalSignaturesFilters = $event" @update:filterTags="digitalSignaturesFilterTags = $event" />

      <!-- ─── Digital Signatures Preview Drawer ───────── -->
      <AppPreviewDrawer
        :open="digitalSignaturesDrawerOpen"
        :app="selectedDigitalSignature ?? {}"
        :rows="selectedDigitalSignature ? digitalSignaturesDrawerRows(selectedDigitalSignature) : []"
        :show-anketa="false"
        :hide-icon-buttons="true"
        :show-signatures="true"
        :show-signed-banner="selectedDigitalSignature?.status === 'Подписан'"
        @close="digitalSignaturesDrawerOpen = false"
      />

      <!-- ─── Insurance Filter Drawer ──────────────────── -->
      <InsuranceFilterDrawer ref="insuranceFilterDrawerRef" :open="insuranceFilterOpen" :count="activeCount ?? 0" @close="insuranceFilterOpen = false" @update:filterCount="insuranceFilterCount = $event" @update:filters="insuranceFilters = $event" @update:filterTags="insuranceFilterTags = $event" />

      <!-- ─── App Preview Drawer (Страховка) ───────────── -->
      <AppPreviewDrawer
        :open="insuranceDrawerOpen"
        :app="selectedInsurance ?? {}"
        :rows="selectedInsurance ? insuranceDrawerRows(selectedInsurance) : []"
        :anketa-phone-mode="true"
        @close="insuranceDrawerOpen = false"
      />

      <!-- ─── App Preview Drawer (Скоринг) ────────────── -->
      <ScoringPreviewDrawer
        :open="scoringDrawerOpen"
        :item="selectedScoringApp"
        @close="scoringDrawerOpen = false"
      />

      <!-- Шапка: строка 1 — toggle + заголовок + кнопки -->
      <div class="shrink-0 flex items-center gap-x-3 pl-2 pr-4 md:pl-4 md:pr-6 pt-4 pb-2">
        <Button plain class="size-8 px-0 text-gray-400 hover:text-gray-600" @click="isLg ? (sidebarCollapsed = !sidebarCollapsed) : (mobileSidebarOpen = !mobileSidebarOpen)">
          <PanelLeftIcon :size="16" :stroke-width="1.5" class="shrink-0 aspect-square" style="width:16px;height:16px;stroke:#18181B;" aria-hidden="true" />
        </Button>
        <div class="flex items-center gap-x-4">
          <div class="flex items-center gap-x-2">
            <!-- Title + counter skeleton on initial load -->
            <template v-if="showSidebarSkeleton">
              <div class="animate-pulse flex items-center gap-x-2">
                <div class="h-6 w-32 rounded bg-zinc-100" />
                <div class="h-5 w-7 rounded bg-zinc-100" />
              </div>
            </template>
            <template v-else-if="!appIsInitialLoading">
              <span class="text-[20px] leading-[32px] font-medium text-[#18181b]">{{ activeModule }}</span>
              <span
                v-if="activeCount !== null && activeModule !== 'Калькулятор ипотеки' && activeModule !== 'Аналитика'"
                class="inline-flex items-center rounded-md bg-zinc-50 px-1.5 py-0.5 text-xs font-medium text-zinc-600 inset-ring inset-ring-zinc-500/10"
              >{{ activeCount.toLocaleString('ru-RU') }}</span>
            </template>
          </div>

          <!-- xs/sm: всегда показываем Создать в шапке -->
          <div v-if="activeModule !== 'Калькулятор ипотеки' && activeModule !== 'Аналитика'" class="md:hidden">
            <Button color="indigo" @click="openCreateModalForModule(activeModule)">
              <PlusIcon :size="16" class="shrink-0" />
              Создать
            </Button>
          </div>
          <!-- md+: только когда сайдбар скрыт -->
          <div v-if="sidebarCollapsed && activeModule !== 'Калькулятор ипотеки' && activeModule !== 'Аналитика'" class="hidden md:block">
            <Button color="indigo" @click="openCreateModalForModule(activeModule)">
              <PlusIcon :size="16" class="shrink-0" />
              Создать
            </Button>
          </div>
        </div>
      </div>

      <!-- Шапка: строка 2 — поиск + фильтры (только Ипотека) -->
      <div v-if="activeModule === 'Ипотека'" :class="['shrink-0 flex items-center justify-between px-4 md:px-6 py-2', isScrolled && !mortgageFilterTags.length && 'sm:border-b sm:border-[#f4f4f5]']">
        <!-- Filter bar skeleton on initial load -->
        <template v-if="showSidebarSkeleton">
          <div class="flex items-center gap-x-3 animate-pulse">
            <div class="h-8 w-72 rounded-md bg-zinc-100" />
            <div class="h-8 w-24 rounded-md bg-zinc-100" />
            <div class="h-8 w-32 rounded-md bg-zinc-100" />
            <div class="ml-auto h-8 w-8 rounded-md bg-zinc-100" />
          </div>
        </template>
        <template v-else-if="!appIsInitialLoading">
          <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ФИО или телефону" />
          <div class="flex items-center gap-x-3">
            <span v-if="searchQuery.trim()" class="text-[14px] font-light text-[#71717a] mr-5">
              {{ searchResultText }}
            </span>
            <!-- Фильтры: скрыт на xs, виден на sm+ -->
            <div class="hidden sm:block">
              <Button outline @click="filterDrawerOpen = true">
                <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
                Фильтры
                <span v-if="mortgageFilterCount > 0" class="inline-flex items-center justify-center size-[18px] rounded-full bg-indigo-600 text-white text-[11px] font-medium leading-none">{{ mortgageFilterCount }}</span>
              </Button>
            </div>
            <!-- Сортировка и экспорт: только md+ -->
            <div class="hidden md:flex items-center gap-x-3">
              <SortDropdown v-model="mortgageSortOrder" />
              <ExportPopover :count="filteredApplications.length" />
            </div>
          </div>
        </template>
      </div>

      <!-- Шапка: строка 3 — xs-only кнопка фильтров (полная ширина) -->
      <div v-if="activeModule === 'Ипотека' && !showSidebarSkeleton && !appIsInitialLoading" :class="['sm:hidden shrink-0 px-4 py-2', isScrolled && !mortgageFilterTags.length && 'border-b border-[#f4f4f5]']">
        <Button outline class="w-full justify-center" @click="filterDrawerOpen = true">
          <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
          Фильтры
          <span v-if="mortgageFilterCount > 0" class="inline-flex items-center justify-center size-[18px] rounded-full bg-indigo-600 text-white text-[11px] font-medium leading-none">{{ mortgageFilterCount }}</span>
        </Button>
      </div>

      <!-- Applied filters strip (Ипотека) -->
      <div v-if="activeModule === 'Ипотека' && mortgageFilterTags && mortgageFilterTags.length > 0" :class="['shrink-0 pl-[60px] pr-8 pt-2 pb-4', isScrolled && 'border-b border-[#f4f4f5]']">
        <FilterTagsBar :tags="mortgageFilterTags" @reset="mortgageFilterDrawerRef?.resetAll()" @show-more="filterDrawerOpen = true" />
      </div>

      <!-- Scrollable content -->
      <div class="flex-1 overflow-auto" @scroll="isScrolled = $event.target.scrollTop > 0">

        <!-- Ипотека -->
        <div v-if="activeModule === 'Ипотека'" class="pt-2 pb-10 relative">
        <!-- Reload: opacity-50 wrapper when reloading but skeleton not yet shown -->
        <div :class="(mortgageIsReloading && !showReloadSkeleton) ? 'opacity-50 pointer-events-none transition-opacity duration-150' : ''">
        <table class="w-full border-separate border-spacing-0">
        <tbody>

          <!-- Скелетон — первая загрузка или смена фильтра (после 500 мс) -->
          <tr v-if="mortgageIsInitialLoading || showReloadSkeleton" v-for="i in SKELETON_ROWS" :key="`sk-${i}`" class="motion-safe:animate-pulse">
            <td class="pt-[16px] pb-3 pl-4 md:pl-6 pr-2"><div class="size-4 rounded-sm bg-zinc-100" /></td>
            <td class="px-3 py-4">
              <div class="h-3 w-20 rounded bg-zinc-100" />
              <div class="mt-1.5 h-2.5 w-12 rounded bg-zinc-100" />
            </td>
            <td class="px-3 py-4">
              <div class="flex items-start gap-x-3">
                <div class="size-8 shrink-0 rounded-full bg-zinc-100" />
                <div>
                  <div class="h-3 w-28 rounded bg-zinc-100" />
                  <div class="mt-1.5 h-2.5 w-24 rounded bg-zinc-100" />
                </div>
              </div>
            </td>
            <td class="px-3 py-4"><div class="h-5 w-24 rounded-md bg-zinc-100" /></td>
            <td class="px-3 py-4">
              <div class="h-3 w-40 rounded bg-zinc-100" />
              <div class="mt-1.5 h-2.5 w-20 rounded bg-zinc-100" />
            </td>
            <td class="px-3 py-4">
              <div class="h-3 w-28 rounded bg-zinc-100" />
              <div class="mt-1.5 h-2.5 w-16 rounded bg-zinc-100" />
            </td>
            <td class="px-3 py-4">
              <div class="h-3 w-24 rounded bg-zinc-100" />
              <div class="mt-1.5 h-2.5 w-16 rounded bg-zinc-100" />
            </td>
            <td class="pl-3 pr-8 py-4"><div class="h-7 w-28 rounded bg-zinc-100" /></td>
          </tr>

          <!-- Реальные строки -->
          <tr v-if="!mortgageIsInitialLoading && !showReloadSkeleton" v-for="(app, idx) in visibleApplications" :key="app.id" :class="['group cursor-pointer', checkedRows.has(app.id) && 'selected']" @click="openApp(app)">
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top pt-[16px] pb-3 pl-4 md:pl-6 pr-2" @click.stop="toggleRow(app.id)">
              <label class="inline-flex cursor-pointer">
                <span :class="['relative flex size-4 items-center justify-center rounded-sm border', checkedRows.has(app.id) ? 'bg-indigo-600 border-transparent' : 'bg-white border-zinc-950/15 hover:border-zinc-950/30']">
                  <svg :class="['size-3 stroke-white transition-opacity', checkedRows.has(app.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
                </span>
              </label>
            </td>
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-3 py-4 whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <TableLink @click="openDrawer(app)">{{ app.id }}</TableLink>
                <span class="text-[14px] leading-[20px] font-light text-zinc-900">от {{ app.date }}</span>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-3 py-4 whitespace-nowrap">
              <div class="flex items-start gap-x-3">
                <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] font-medium text-[#71717a]">{{ app.initials }}</span>
                <div class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ app.client }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ app.phone }}</span>
                </div>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-3 py-4 whitespace-nowrap">
              <span v-if="app.status" :class="statusBadgeClass(app.status)">{{ app.status }}</span>
            </td>
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-3 py-4 whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ app.complex }}</span>
                <span class="flex items-center gap-x-1.5 text-[14px] leading-[20px] font-light text-zinc-900">
                  <HouseIcon :size="16" class="shrink-0 text-zinc-900" />
                  {{ app.complexPrice }}
                </span>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-3 py-4 whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ app.amount }}<span v-if="app.pv" class="text-zinc-400"> ПВ {{ app.pv }}</span></span>
                <span v-if="app.term" class="text-[14px] leading-[20px] font-light text-zinc-900">{{ app.term }}</span>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top px-3 py-4 whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <span v-if="app.houseType" class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ app.houseType }}</span>
                <span v-if="app.mortgageType" class="text-[14px] leading-[20px] font-light text-zinc-900">{{ app.mortgageType }}</span>
              </div>
            </td>
            <!-- Менеджер -->
            <td class="group-hover:bg-zinc-50 [.selected_&]:bg-zinc-50 [.selected_&]:group-hover:bg-zinc-100 align-top pl-3 pr-8 py-4 whitespace-nowrap" @click.stop>
              <CatalystListbox
                :options="managerSelectOptions"
                :model-value="mortgageManagerSelections[app.id]"
                @update:model-value="mortgageManagerSelections[app.id] = $event"
                placeholder="Менеджер"
              />
            </td>
          </tr>

          <!-- Sentinel — триггер подгрузки -->
          <tr v-if="!mortgageIsInitialLoading && !showReloadSkeleton && mortgageHasMore" ref="mortgageSentinelRef" aria-hidden="true">
            <td colspan="8" class="p-0 h-0" />
          </tr>

          <!-- Спиннер подгрузки -->
          <tr v-if="mortgageIsLoadingMore">
            <td colspan="8">
              <div class="flex items-center gap-x-2 py-4 pl-8 text-zinc-500">
                <svg class="size-4 animate-spin" viewBox="0 0 24 24" fill="none" aria-hidden="true">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
                </svg>
                <span class="text-sm font-normal">Загружаем заявки...</span>
              </div>
            </td>
          </tr>

          <!-- Конец списка — только если данных было больше одной страницы -->
          <tr v-if="!mortgageIsInitialLoading && !showReloadSkeleton && !mortgageHasMore && filteredApplications.length > INITIAL_PAGE_SIZE">
            <td colspan="8">
              <div class="py-4 pl-8 text-sm text-zinc-500">
                Загрузили все заявки ({{ filteredApplications.length }})
              </div>
            </td>
          </tr>

        </tbody>
        </table>
        </div><!-- /reload opacity wrapper -->

        <!-- Градиент fade-out снизу -->
        <div
          v-if="!mortgageIsInitialLoading && !showReloadSkeleton && visibleApplications.length > 0"
          class="pointer-events-none sticky bottom-0 left-0 right-0 h-20"
          style="background: linear-gradient(to bottom, transparent 0%, white 85%)"
          aria-hidden="true"
        />

        <!-- Empty state — список пуст -->
        <div v-if="filteredApplications.length === 0 && !mortgageIsInitialLoading && !mortgageIsReloading && !searchQuery.trim() && !mortgageFilterCount" class="flex flex-col items-center justify-center py-24 text-center">
          <FilePlusCornerIcon :size="48" class="mx-auto text-zinc-400" />
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Пока здесь пусто</h3>
          <p class="mt-1 text-sm font-light text-zinc-500">В этом списке пока нет заявок</p>
          <Button outline class="mt-6" @click="openCreateModalForModule(activeModule)">
            <svg class="size-4" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true"><path d="M10.75 4.75a.75.75 0 00-1.5 0v4.5h-4.5a.75.75 0 000 1.5h4.5v4.5a.75.75 0 001.5 0v-4.5h4.5a.75.75 0 000-1.5h-4.5v-4.5z" /></svg>
            Создать
          </Button>
        </div>

        <!-- Empty state при поиске или фильтрации -->
        <div v-if="filteredApplications.length === 0 && !mortgageIsInitialLoading && !mortgageIsReloading && (searchQuery.trim() || mortgageFilterCount > 0)" class="flex flex-col items-center justify-center py-24 text-center">
          <FileSearchCornerIcon :size="48" class="mx-auto text-zinc-400" />
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Заявки не найдены</h3>
          <p class="mt-1 text-sm font-light text-zinc-500">Попробуйте изменить параметры поиска или фильтрации</p>
          <Button v-if="searchQuery.trim() && !mortgageFilterCount" outline class="mt-6" @click="searchQuery = ''">Сбросить поиск</Button>
          <Button v-else-if="!searchQuery.trim() && mortgageFilterCount > 0" outline class="mt-6" @click="mortgageFilterDrawerRef?.resetAll()">Сбросить фильтры</Button>
        </div>

        </div>

        <!-- Калькулятор ипотеки -->
        <CalculatorPage v-else-if="activeModule === 'Калькулятор ипотеки'" @open-bank="openBankDrawer" />

        <!-- Скоринг -->
        <ScoringPage v-else-if="activeModule === 'Скоринг'" :active-sub-item="activeSubItem" :filter-count="scoringFilterCount" :filters="scoringFilters" :filter-tags="scoringFilterTags" @open-filter="scoringFilterDrawerOpen = true" @open-preview="openScoringDrawer" @update:count="activeCount = $event" @reset-filters="scoringFilterDrawerRef?.resetAll()" @create="openCreateModalForModule(activeModule)" />

        <!-- Пакеты документов -->
        <DocumentPackagesPage v-else-if="activeModule === 'Пакеты документов'" :active-sub-item="activeSubItem" :filter-count="docPackagesFilterCount" :filters="docPackagesFilters" :filter-tags="docPackagesFilterTags" :extra-packages="docExtraPackages" @open-filter="docPackagesFilterOpen = true" @open-preview="item => { selectedDocPackage = item; docPackagePreviewOpen = true }" @update:count="activeCount = $event" @reset-filters="docPackagesFilterDrawerRef?.resetAll()" @create="openCreateModalForModule(activeModule)" />

        <!-- Регистрация -->
        <RegistrationPage v-else-if="activeModule === 'Регистрация'" :active-sub-item="activeSubItem" :filter-count="registrationFilterCount" :filters="registrationFilters" :filter-tags="registrationFilterTags" @open-filter="registrationFilterOpen = true" @open-preview="openRegistrationPreview" @update:count="activeCount = $event" @reset-filters="registrationFilterDrawerRef?.resetAll()" @create="openCreateModalForModule(activeModule)" />

        <!-- Пакетная регистрация -->
        <BatchRegistrationPage v-else-if="activeModule === 'Пакетная регистрация'" :active-sub-item="activeSubItem" :filter-count="batchRegistrationFilterCount" :filters="batchRegistrationFilters" :filter-tags="batchRegistrationFilterTags" :extra-packages="batchExtraPackages" @open-filter="batchRegistrationFilterOpen = true" @open-preview="openRegistrationPreview" @update:count="activeCount = $event" @reset-filters="batchRegistrationFilterDrawerRef?.resetAll()" @create="openCreateModalForModule(activeModule)" @delete-extra-package="batchExtraPackages = batchExtraPackages.filter(n => n !== $event)" />

        <!-- Страховка -->
        <InsurancePage v-else-if="activeModule === 'Страховка'" :active-sub-item="activeSubItem" :filter-count="insuranceFilterCount" :filters="insuranceFilters" :filter-tags="insuranceFilterTags" @open-filter="insuranceFilterOpen = true" @open-preview="openInsurancePreview" @update:count="activeCount = $event" @reset-filters="insuranceFilterDrawerRef?.resetAll()" @create="openCreateModalForModule(activeModule)" />

        <!-- Цифровые подписи -->
        <DigitalSignaturesPage v-else-if="activeModule === 'Цифровые подписи'" :active-sub-item="activeSubItem" :filter-count="digitalSignaturesFilterCount" :filters="digitalSignaturesFilters" :filter-tags="digitalSignaturesFilterTags" @open-filter="digitalSignaturesFilterOpen = true" @open-preview="openDigitalSignaturesPreview" @update:count="activeCount = $event" @reset-filters="digitalSignaturesFilterDrawerRef?.resetAll()" @create="openCreateModalForModule(activeModule)" />

        <!-- Аналитика -->
        <AnalyticsPage v-else-if="activeModule === 'Аналитика'" />


      </div>

      <SelectionBar :count="checkedRows.size" @clear="checkedRows = new Set()" @delete="checkedRows = new Set()" />

    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, watchEffect, onMounted, onUnmounted, reactive } from 'vue'

const props = defineProps({
  mode: { type: String, default: 'all' }
})
import SearchInput from './src/components/SearchInput.vue'
import Button from './src/components/Button.vue'
import Badge from './src/components/Badge.vue'
import TableLink from './src/components/TableLink.vue'
import FilterDrawer from './src/components/FilterDrawer.vue'
import BankDrawer from './src/components/BankDrawer.vue'
import AppPreviewDrawer from './src/components/AppPreviewDrawer.vue'
import ScoringPreviewDrawer from './src/components/ScoringPreviewDrawer.vue'
import CalculatorPage from './src/components/CalculatorPage.vue'
import ScoringPage from './src/components/ScoringPage.vue'
import AnalyticsPage from './src/components/AnalyticsPage.vue'
import ScoringFilterDrawer from './src/components/ScoringFilterDrawer.vue'
import ScoringImportDrawer from './src/components/ScoringImportDrawer.vue'
import CreateApplicationModal from './src/components/CreateApplicationModal.vue'
import ExportPopover from './src/components/ExportPopover.vue'
import ToastNotification from './src/components/ToastNotification.vue'
import CatalystListbox from './src/components/CatalystListbox.vue'
import DocumentPackagesPage from './src/components/DocumentPackagesPage.vue'
import DocumentPackagesFilterDrawer from './src/components/DocumentPackagesFilterDrawer.vue'
import DocumentPackagePreviewDrawer from './src/components/DocumentPackagePreviewDrawer.vue'
import RegistrationPage from './src/components/RegistrationPage.vue'
import RegistrationFilterDrawer from './src/components/RegistrationFilterDrawer.vue'
import BatchRegistrationPage from './src/components/BatchRegistrationPage.vue'
import BatchRegistrationFilterDrawer from './src/components/BatchRegistrationFilterDrawer.vue'
import InsurancePage from './src/components/InsurancePage.vue'
import InsuranceFilterDrawer from './src/components/InsuranceFilterDrawer.vue'
import DigitalSignaturesPage from './src/components/DigitalSignaturesPage.vue'
import DigitalSignaturesFilterDrawer from './src/components/DigitalSignaturesFilterDrawer.vue'
import SelectionBar from './src/components/SelectionBar.vue'
import SortDropdown from './src/components/SortDropdown.vue'
import FilterTagsBar from './src/components/FilterTagsBar.vue'
import { Disclosure, DisclosureButton, DisclosurePanel, Popover, PopoverButton, PopoverPanel } from '@headlessui/vue'
import {
  CloudOff as CloudOffIcon,
  ChevronRight as ChevronRightIcon,
  Paperclip as PaperClipIcon,
  Copy as CopyIcon,
  Link2 as Link2Icon,
  CircleCheck as CircleCheckIcon,
  Check as CheckIcon,
  BadgeCheck as BadgeCheckIcon,
  CirclePercent as FolderOpenIcon,
  Home as HomeIcon,
  Files as DocumentDuplicateIcon,
  BriefcaseBusiness as BriefcaseIcon,
  ShieldCheck as ShieldCheckIcon,
  Signature as DocumentCheckIcon,
  PieChart as ChartPieIcon,
  X as XMarkIcon,
  Search as MagnifyingGlassIcon,
  SlidersHorizontal as FunnelIcon,
  ChevronDown as ChevronDownIcon,
  ChevronUp as ChevronUpIcon,
  Plus as PlusIcon,
  PanelLeft as PanelLeftIcon,
  Trash2 as TrashIcon,
  Copy as CopyDuplicateIcon,
  Library as LibraryIcon,
  Bell as BellIcon,
  ArrowLeftRight as ArrowLeftRightIcon,
  LogOut as LogOutIcon,
  LifeBuoy as LifeBuoyIcon,
  MessageCircleQuestion as MessageCircleQuestionIcon,
  CircleUser as CircleUserIcon,
  Ellipsis as EllipsisIcon,
  ArrowUpToLine as ArrowUpFromLineIcon,
  Layers as LayersIcon,
  House as HouseIcon,
  Building2 as Building2Icon,
  Hotel as HotelIcon,
  Car as CarIcon,
  Package as PackageIcon,
  Store as StoreIcon,
  HousePlus as HousePlusIcon,
  Fence as FenceIcon,
  FilePlusCorner as FilePlusCornerIcon,
  FileSearchCorner as FileSearchCornerIcon,
} from 'lucide-vue-next'

const propertyTypeIconMap = {
  'Квартира':                   Building2Icon,
  'Апартаменты':                HotelIcon,
  'Машиноместо':                CarIcon,
  'Кладовка':                   PackageIcon,
  'Коммерческая недвижимость':  StoreIcon,
  'Дом':                        HouseIcon,
  'Таунхаус':                   HousePlusIcon,
  'Дом с земельным участком':   FenceIcon,
}
const getPropertyIcon = (type) => propertyTypeIconMap[type] ?? Building2Icon

const isOffline = ref(true)
onMounted(() => {
  window.addEventListener('online',  () => { isOffline.value = false })
  window.addEventListener('offline', () => { isOffline.value = true  })
  window.addEventListener('resize', onResize)

  // Simulate initial page load: sidebar/header/filterbar skeleton after 500 ms delay
  const sidebarDelayTimer = setTimeout(() => { showSidebarSkeleton.value = true }, 500)
  setTimeout(() => {
    appIsInitialLoading.value = false
    showSidebarSkeleton.value = false
    clearTimeout(sidebarDelayTimer)
    mortgageIsInitialLoading.value = false
  }, 900)
})
onUnmounted(() => {
  window.removeEventListener('online',  () => { isOffline.value = false })
  window.removeEventListener('offline', () => { isOffline.value = true  })
  window.removeEventListener('resize', onResize)
})

const activeModule = ref(props.mode === 'registration' ? 'Регистрация' : 'Ипотека')
const sidebarCollapsed = ref(false)
const mobileSidebarOpen = ref(false)
const isLg = ref(window.innerWidth >= 1024)
const onResize = () => { isLg.value = window.innerWidth >= 1024 }
const mortgageSortOrder = ref('new')
const isScrolled = ref(false)
const filterDrawerOpen = ref(false)
const scoringFilterDrawerOpen = ref(false)
const scoringImportOpen = ref(false)
const docPackagesFilterOpen = ref(false)
const docPackagePreviewOpen = ref(false)
const selectedDocPackage = ref(null)
const registrationFilterOpen = ref(false)
const batchRegistrationFilterOpen = ref(false)
const registrationDrawerOpen = ref(false)
const selectedRegistration = ref(null)
const digitalSignaturesFilterOpen = ref(false)
const digitalSignaturesDrawerOpen = ref(false)
const selectedDigitalSignature = ref(null)
const openDigitalSignaturesPreview = (item) => { selectedDigitalSignature.value = item; digitalSignaturesDrawerOpen.value = true }
const digitalSignaturesDrawerRows = (item) => [
  { label: 'Центр',           value: item.center },
  { label: 'Способ выпуска',  value: item.signMethod, icon: item.signMethod === 'Офис' ? 'office' : item.signMethod === 'Курьер' ? 'courier' : 'passport' },
  { label: 'Дата активации',  value: item.status === 'Активирована' ? formatCreatedAt(item.activationDate, true) : '' },
  { label: 'Действует',       value: item.status === 'Активирована' ? item.duration : '' },
  { label: 'Менеджер',        value: item.manager, options: ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей'] },
]

const insuranceFilterOpen = ref(false)
const insuranceDrawerOpen = ref(false)
const selectedInsurance = ref(null)
const openInsurancePreview = (item) => { selectedInsurance.value = item; insuranceDrawerOpen.value = true }
const insuranceDrawerRows = (item) => [
  { label: 'Страховая',       value: item.insurer },
  { label: 'Тип страховки',   value: item.insuranceType },
  { label: 'Цена',            value: item.price },
  { label: 'Комиссия',        value: item.kv, icon: 'percent' },
  { label: 'Менеджер',        value: item.manager, options: managerSelectOptions },
]
const openRegistrationPreview = (item) => { selectedRegistration.value = item; registrationDrawerOpen.value = true }
const registrationDrawerRows = (item) => [
  { label: 'Дата создания',   value: formatCreatedAt(item.date) },
  { label: 'Тип заявления',   value: item.applicationType },
  { label: 'Компания',        value: item.company },
  { label: 'Проект',          value: item.project },
  { label: 'Тип недвижимости', value: item.propertyType },
  { label: 'Кадастровый номер', value: item.cadastral },
  { label: 'Адрес объекта',   value: item.address },
  ...(item.regDate ? [{ label: 'Дата регистрации', value: item.regDate }] : []),
  { label: 'РР',   value: 'ВС-2025-12-30-234567',  copy: 'ВС-2025-12-30-234567'  },
  { label: 'КУВД', value: 'КУВД-001/2025-234567', copy: 'КУВД-001/2025-234567' },
  { label: 'Менеджер', value: item.manager, options: managerSelectOptions },
]
const bankDrawerOpen = ref(false)
const selectedBank = ref(null)
const openBankDrawer = (bank) => { selectedBank.value = bank; bankDrawerOpen.value = true }

watch(activeModule, () => {
  bankDrawerOpen.value = false
  filterDrawerOpen.value = false
  scoringFilterDrawerOpen.value = false
  drawerOpen.value = false
  scoringDrawerOpen.value = false
  activeCount.value = activeModule.value === 'Ипотека' ? filteredApplications.value.length : null
})

const activeCount = ref(null)

const mortgageFilterDrawerRef = ref(null)
const scoringFilterDrawerRef = ref(null)
const docPackagesFilterDrawerRef = ref(null)
const registrationFilterDrawerRef = ref(null)
const batchRegistrationFilterDrawerRef = ref(null)
const insuranceFilterDrawerRef = ref(null)
const digitalSignaturesFilterDrawerRef = ref(null)

const mortgageFilterCount = ref(0)
const mortgageFilters = ref({})
const mortgageFilterTags = ref([])

const scoringFilterCount = ref(0)
const scoringFilters = ref({})
const scoringFilterTags = ref([])
const docPackagesFilterCount = ref(0)
const docPackagesFilters = ref({})
const docPackagesFilterTags = ref([])
const registrationFilterCount = ref(0)
const registrationFilterTags = ref([])
const registrationFilters = ref(null)
const batchRegistrationFilterCount = ref(0)
const batchRegistrationFilterTags = ref([])
const batchRegistrationFilters = ref(null)
const insuranceFilterCount = ref(0)
const insuranceFilterTags = ref([])
const insuranceFilters = ref(null)
const digitalSignaturesFilterCount = ref(0)
const digitalSignaturesFilters = ref(null)
const digitalSignaturesFilterTags = ref([])

const pluralCount = (n) => {
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return `${n.toLocaleString('ru-RU')} заявок`
  if (mod10 === 1) return `${n.toLocaleString('ru-RU')} заявка`
  if (mod10 >= 2 && mod10 <= 4) return `${n.toLocaleString('ru-RU')} заявки`
  return `${n.toLocaleString('ru-RU')} заявок`
}
const accountMenuOpen = ref(false)
const searchQuery = ref('')
const searchResultText = computed(() => {
  const n = filteredApplications.value.length
  const mod10 = n % 10
  const mod100 = n % 100
  if (mod100 >= 11 && mod100 <= 14) return `найдено ${n} заявок`
  if (mod10 === 1) return `найдена ${n} заявка`
  if (mod10 >= 2 && mod10 <= 4) return `найдено ${n} заявки`
  return `найдено ${n} заявок`
})
const activeSubItem = ref('Все заявки')

const createModalOpen = ref(false)
const createPopoverOpen = ref(false)
const createModalTitle = ref('Создание заявки')
const createModalModule = ref('')

const moduleCreateItemsMap = {
  'Ипотека':              { icon: HomeIcon,             title: 'Создание заявки на ипотеку' },
  'Скоринг':              { icon: ChartPieIcon,          title: 'Создание заявки на скоринг' },
  'Страховка':            { icon: ShieldCheckIcon,       title: 'Создание заявки на страховку' },
  'Регистрация':          { icon: BriefcaseIcon,         title: 'Создание заявки на регистрацию' },
  'Пакетная регистрация': { icon: LayersIcon,            title: 'Создание пакета на регистрацию' },
  'Пакеты документов':    { icon: DocumentDuplicateIcon, title: 'Создание пакета документов', label: 'Пакет документов', dividerBefore: true },
  'Цифровые подписи':     { icon: DocumentCheckIcon,     title: 'Создание заявки на выпуск ЭЦП' },
}

const createModuleItems = computed(() => {
  const namesByMode = {
    'all':          ['Ипотека', 'Скоринг', 'Страховка', 'Пакеты документов', 'Регистрация', 'Пакетная регистрация', 'Цифровые подписи'],
    'mortgage':     ['Ипотека', 'Скоринг', 'Страховка'],
    'registration': ['Регистрация', 'Пакетная регистрация', 'Цифровые подписи', 'Пакеты документов'],
  }
  return (namesByMode[props.mode] ?? []).map(name => ({ name, ...moduleCreateItemsMap[name] }))
})

const openCreateModal = (item, close) => {
  createModalTitle.value = item.title
  createModalModule.value = item.name ?? ''
  close()
  createModalOpen.value = true
}

const openCreateModalForModule = (moduleName) => {
  createModalTitle.value = moduleCreateItemsMap[moduleName]?.title ?? 'Создание заявки'
  createModalModule.value = moduleName
  createModalOpen.value = true
}

const batchExtraPackages = ref([])
const docExtraPackages = ref([])

const todayDDMM = () => {
  const d = new Date()
  const dd = String(d.getDate()).padStart(2, '0')
  const mm = String(d.getMonth() + 1).padStart(2, '0')
  return `${dd}.${mm}`
}

const handleModalCreated = ({ module, name }) => {
  if (module === 'Ипотека') {
    window.open(`/app-page.html?id=ИП-new&title=${encodeURIComponent('Страница заявки на ипотеку')}`, '_blank')
  } else if (module === 'Страховка') {
    window.open(`/app-page.html?id=СТ-new&title=${encodeURIComponent('Страница заявки на страховку')}`, '_blank')
  } else if (module === 'Скоринг') {
    window.open(`/app-page.html?id=СК-new&title=${encodeURIComponent('Страница заявки скоринга')}`, '_blank')
  } else if (module === 'Пакетная регистрация') {
    batchExtraPackages.value = [...batchExtraPackages.value, name]
    window.open(`/app-page.html?id=${encodeURIComponent(name)}&title=${encodeURIComponent('Страница пакета')}`, '_blank')
  } else if (module === 'Регистрация') {
    window.open(`/app-page.html?id=РГ-new&title=${encodeURIComponent('Страница заявки на регистрацию')}`, '_blank')
  } else if (module === 'Цифровые подписи') {
    window.open(`/app-page.html?id=ЦП-new&title=${encodeURIComponent('Страница заявки на выпуск ЭЦП')}`, '_blank')
  } else if (module === 'Пакеты документов') {
    const newItem = {
      id: `ПД ${1151 + docExtraPackages.value.length}`,
      name,
      date: todayDDMM(),
      status: 'Черновик',
      signers: 0,
      signedCount: 0,
      signersList: [],
      manager: null,
    }
    docExtraPackages.value = [...docExtraPackages.value, newItem]
    window.open(`/app-page.html?id=${encodeURIComponent(newItem.id)}&title=${encodeURIComponent('Страница пакета')}`, '_blank')
  }
}

const drawerOpen = ref(false)
const selectedApp = ref(null)
const openDrawer = (app) => { selectedApp.value = app; drawerOpen.value = true }
const openApp = (app) => { window.open(`/app-page.html?id=${app.id}`, '_blank') }

const scoringDrawerOpen = ref(false)
const selectedScoringApp = ref(null)
const openScoringDrawer = (item) => { selectedScoringApp.value = item; scoringDrawerOpen.value = true }

const handleSubAction = (action) => {
  if (action === 'scoring-import') scoringImportOpen.value = true
}


const managerSelectOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const mortgageManagerSelections = reactive({})

const months = ['января','февраля','марта','апреля','мая','июня','июля','августа','сентября','октября','ноября','декабря']
const formatCreatedAt = (d, withYear = false) => {
  if (!d) return ''
  const parts = d.split('.')
  const base = `${parseInt(parts[0])} ${months[parseInt(parts[1]) - 1]}`
  return withYear && parts[2] ? `${base} ${parts[2]}` : base
}

const drawerRows = (app) => [
  { label: 'Дата создания', value: formatCreatedAt(app.createdAt) },
  { label: 'Проект', value: app.complex },
  ...(app.complexPrice ? [{ label: 'Стоимость объекта', value: app.complexPrice, icon: 'house' }] : []),
  { label: 'Сумма кредита', value: app.amount },
  ...(app.pv ? [{ label: 'ПВ', value: app.pv }] : []),
  ...(app.term ? [{ label: 'Срок', value: app.term }] : []),
  ...(app.houseType ? [{ label: 'Тип кредита', value: app.houseType }] : []),
  ...(app.mortgageType ? [{ label: 'Тип ипотеки', value: app.mortgageType }] : []),
  { label: 'Менеджер', value: 'Смирнова Юлия', options: managerSelectOptions },
]

// Ширина колонки статуса по самому широкому значению (7px/символ + padding badge + padding ячейки)
const statusColWidth = computed(() => {
  const maxLen = applications.reduce((max, app) =>
    app.status && app.status.length > max ? app.status.length : max, 0)
  return `${maxLen * 7 + 24}px`
})

const statusFilterItems = new Set(['Ожидает решения', 'Одобрена', 'Отказано'])

const filteredApplications = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  const statusFilter = statusFilterItems.has(activeSubItem.value) ? activeSubItem.value : null
  const isStuck = activeSubItem.value === 'Зависли'
  const { statuses, managers: filterManagers, projects: filterProjects } = mortgageFilters.value

  let result = applications.filter(app => {
    if (statusFilter && app.status !== statusFilter) return false
    if (isStuck) return false
    if (statuses?.size > 0 && !statuses.has(app.status)) return false
    if (filterManagers?.size > 0 && !filterManagers.has(mortgageManagerSelections[app.id])) return false
    if (filterProjects?.size > 0 && !filterProjects.has(app.complex)) return false
    if (!q) return true
    return (
      app.id.toLowerCase().includes(q) ||
      app.fullName.toLowerCase().includes(q) ||
      app.fullPhone.replace(/\s/g, '').includes(q.replace(/\s/g, '')) ||
      app.phone.replace(/\s/g, '').includes(q.replace(/\s/g, ''))
    )
  })

  if (mortgageSortOrder.value === 'old') {
    result = [...result].sort((a, b) => {
      const [da, ma, ya] = a.createdAt.split('.')
      const [db, mb, yb] = b.createdAt.split('.')
      return new Date(ya, ma - 1, da) - new Date(yb, mb - 1, db)
    })
  }

  return result
})


const INITIAL_PAGE_SIZE = 25
const NEXT_PAGE_SIZE = 20
const PRELOAD_THRESHOLD = 3
const SKELETON_ROWS = 12

const mortgageVisibleCount = ref(INITIAL_PAGE_SIZE)
const mortgageIsInitialLoading = ref(true)
const mortgageIsLoadingMore = ref(false)
const mortgageIsReloading = ref(false)

// Initial app loading — controls sidebar / header / filterbar skeleton
const appIsInitialLoading = ref(true)
const showSidebarSkeleton = ref(false)

// Reload skeleton — delayed (500 ms) version of mortgageIsReloading
const showReloadSkeleton = ref(false)

const visibleApplications = computed(() =>
  filteredApplications.value.slice(0, mortgageVisibleCount.value)
)
const mortgageHasMore = computed(() =>
  mortgageVisibleCount.value < filteredApplications.value.length
)

const loadMoreMortgage = () => {
  if (mortgageIsLoadingMore.value || !mortgageHasMore.value) return
  mortgageIsLoadingMore.value = true
  setTimeout(() => {
    mortgageVisibleCount.value += NEXT_PAGE_SIZE
    mortgageIsLoadingMore.value = false
  }, 600)
}

const mortgageSentinelRef = ref(null)
watchEffect((onCleanup) => {
  const el = mortgageSentinelRef.value
  if (!el) return
  const observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) loadMoreMortgage()
    },
    { threshold: 0.1, rootMargin: '120px 0px' }
  )
  observer.observe(el)
  onCleanup(() => observer.disconnect())
})

// Сбрасываем при смене фильтра/поиска/сортировки/подраздела
watch(
  [activeSubItem, mortgageSortOrder, searchQuery, () => JSON.stringify(mortgageFilters.value)],
  () => {
    if (activeModule.value !== 'Ипотека') return
    mortgageVisibleCount.value = INITIAL_PAGE_SIZE
    // Reload state: old data goes opacity-50 immediately; skeleton kicks in after 500 ms
    mortgageIsReloading.value = true
    showReloadSkeleton.value = false
    const reloadDelayTimer = setTimeout(() => { showReloadSkeleton.value = true }, 500)
    setTimeout(() => {
      mortgageIsReloading.value = false
      showReloadSkeleton.value = false
      clearTimeout(reloadDelayTimer)
    }, 900)
  }
)

const checkedRows = ref(new Set())
const toggleRow = (id) => {
  const s = new Set(checkedRows.value)
  s.has(id) ? s.delete(id) : s.add(id)
  checkedRows.value = s
}
watch(activeModule, () => { checkedRows.value = new Set() })

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (['Одобрена', 'ДДУ согласован', 'ДДУ зарегистрирован', 'Кредит выдан'].includes(status))
    return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (['Ожидает решения', 'На подписании', 'Согласование ДДУ', 'Объект на согласовании'].includes(status))
    return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
  if (status === 'Банк выбран')       return `${base} bg-blue-50 text-blue-700 inset-ring-blue-700/10`
  if (status === 'Отказано')          return `${base} bg-red-50 text-red-700 inset-ring-red-600/10`
  return `${base} bg-gray-50 text-gray-600 inset-ring-gray-500/10`
}

const navigation = [
  {
    name: 'Ипотека',
    href: '#',
    icon: HomeIcon,
    current: true,
    children: [
      { name: 'Все заявки',        href: '#', current: true,  count: 12456 },
      { name: 'Мои заявки',        href: '#', current: false, count: 34    },
      { name: 'Ожидает решения',   href: '#', current: false, count: 143   },
      { name: 'Одобрена',          href: '#', current: false, count: 28    },
      { name: 'Отказано',          href: '#', current: false               },
      { name: 'Зависли',           href: '#', current: false               },
    ],
  },
  {
    name: 'Скоринг',
    href: '#',
    icon: ChartPieIcon,
    current: false,
    children: [
      { name: 'Все заявки', href: '#', current: false, count: 1247 },
      { name: 'Мои заявки', href: '#', current: false, count: 18  },
      { name: 'Импортировать', href: '#', action: 'scoring-import' },
    ],
  },
  {
    name: 'Страховка',
    href: '#',
    icon: ShieldCheckIcon,
    current: false,
    children: [
      { name: 'Все заявки',  href: '#', current: false, count: 28 },
      { name: 'Мои заявки', href: '#', current: false, count: 7  },
      { name: 'Продления',   href: '#', current: false, count: 5  },
    ],
  },
  {
    name: 'Регистрация',
    href: '#',
    icon: BriefcaseIcon,
    current: false,
    children: [
      { name: 'Все заявки', href: '#', current: false, count: 34 },
      { name: 'Мои заявки', href: '#', current: false, count: 10 },
    ],
  },
  {
    name: 'Пакетная регистрация',
    href: '#',
    icon: LayersIcon,
    current: false,
    children: [
      { name: 'Все пакеты', href: '#', current: false, count: 21 },
      { name: 'Мои пакеты', href: '#', current: false, count: 5  },
    ],
  },
  {
    name: 'Цифровые подписи',
    href: '#',
    icon: DocumentCheckIcon,
    current: false,
    children: [
      { name: 'Все заявки',  href: '#', current: false, count: 10 },
      { name: 'Мои заявки', href: '#', current: false, count: 3  },
    ],
  },
  {
    name: 'Пакеты документов',
    href: '#',
    icon: DocumentDuplicateIcon,
    current: false,
    children: [
      { name: 'Все',  href: '#', current: false, count: 8 },
      { name: 'Мои', href: '#', current: false, count: 2 },
    ],
  },
]



const mortgageModules = new Set(['Ипотека', 'Скоринг', 'Страховка'])
const mortgageOnlyModules = new Set(['Ипотека'])
const registrationModules = new Set(['Регистрация', 'Пакетная регистрация', 'Цифровые подписи', 'Пакеты документов'])
const registrationOrder = ['Регистрация', 'Пакетная регистрация', 'Цифровые подписи', 'Пакеты документов']
const filteredNavigation = computed(() => {
  if (props.mode === 'mortgage') return navigation.filter(item => mortgageModules.has(item.name))
  if (props.mode === 'mortgage-only') return navigation.filter(item => mortgageOnlyModules.has(item.name))
  if (props.mode === 'registration') return registrationOrder.map(name => navigation.find(item => item.name === name))
  return navigation
})

const secondaryNavigation = [
  { name: 'Калькулятор ипотеки', href: '#', initial: 'К' },
  { name: 'Аналитика',           href: '#', initial: 'А' },
]

const relativeTime = (dateStr) => {
  const [datePart, timePart] = dateStr.split(' ')
  const [d, m, y] = datePart.split('.')
  const [h, min] = timePart ? timePart.split(':') : ['12', '00']
  const date = new Date(+y, +m - 1, +d, +h, +min)
  const diffMs = Date.now() - date
  const diffMins = Math.floor(diffMs / 60000)
  const diffHours = Math.floor(diffMs / 3600000)
  const diffDays = Math.floor(diffMs / 86400000)
  const diffWeeks = Math.floor(diffDays / 7)
  if (diffMins < 1) return 'только что'
  if (diffMins < 60) return `${diffMins} мин назад`
  if (diffHours < 24) return `${diffHours} ч назад`
  if (diffDays < 7) return `${diffDays} д назад`
  if (diffWeeks === 1) return 'неделю назад'
  if (diffWeeks < 5) return `${diffWeeks} ${diffWeeks < 5 ? 'недели' : 'недель'} назад`
  return `${diffWeeks} недель назад`
}

const applications = [
  {
    id: 'ИП 1257847', date: '05.05',
    status: 'Новая заявка',
    initials: 'НЕ', client: 'Новикова Е.Д.', phone: '+7 925 ***-**-31',
    fullName: 'Новикова Екатерина Дмитриевна', fullPhone: '+7 925 413-87-31',
    complex: 'Самолет/Новые Ватутинки', complexPrice: '7 200 000 ₽', propertyType: 'Квартира',
    amount: '5 400 000 ₽', pv: '25%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '05.05.2026', updatedAt: '05.05.2026 10:14',
  },
  {
    id: 'ИП 1257846', date: '05.05',
    status: 'Консультация',
    initials: 'НЕ', client: 'Новикова Е.Д.', phone: '+7 925 ***-**-31',
    fullName: 'Новикова Екатерина Дмитриевна', fullPhone: '+7 925 413-87-31',
    complex: 'А101/Прокшино', complexPrice: '6 800 000 ₽', propertyType: 'Апартаменты',
    amount: '4 760 000 ₽', pv: '30%', term: '18 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '05.05.2026', updatedAt: '05.05.2026 09:02',
  },
  {
    id: 'ИП 1257845', date: '04.05',
    status: 'Подготовка',
    initials: 'ОД', client: 'Орлов Д.П.', phone: '+7 912 ***-**-07',
    fullName: 'Орлов Дмитрий Павлович', fullPhone: '+7 912 635-20-07',
    complex: 'MR Group/Савёловский Сити', complexPrice: '18 500 000 ₽', propertyType: 'Таунхаус',
    amount: '13 875 000 ₽', pv: '25%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '04.05.2026', updatedAt: '04.05.2026 17:48',
  },
  {
    id: 'ИП 1257844', date: '04.05',
    status: 'Ожидает решения',
    initials: 'ВА', client: 'Воронова А.С.', phone: '+7 926 ***-**-64',
    fullName: 'Воронова Анна Сергеевна', fullPhone: '+7 926 184-53-64',
    complex: 'ФСК/Южная Битца', complexPrice: '10 100 000 ₽', propertyType: 'Дом с земельным участком',
    amount: '7 070 000 ₽', pv: '30%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '04.05.2026', updatedAt: '04.05.2026 15:30',
  },
  {
    id: 'ИП 1257843', date: '03.05',
    status: 'Одобрена',
    initials: 'ЛИ', client: 'Лебедев И.Н.', phone: '+7 967 ***-**-19',
    fullName: 'Лебедев Игорь Николаевич', fullPhone: '+7 967 302-58-19',
    complex: 'ЛСР/Морская набережная (СПб)', complexPrice: '9 600 000 ₽', propertyType: 'Дом',
    amount: '6 720 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '03.05.2026', updatedAt: '04.05.2026 11:22',
  },
  {
    id: 'ИП 1257842', date: '03.05',
    status: 'Консультация',
    initials: 'ЛИ', client: 'Лебедев И.Н.', phone: '+7 967 ***-**-19',
    fullName: 'Лебедев Игорь Николаевич', fullPhone: '+7 967 302-58-19',
    complex: 'Группа ЛСР/Зенит (СПб)', complexPrice: '8 200 000 ₽', propertyType: 'Коммерческая недвижимость',
    amount: '5 740 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '03.05.2026', updatedAt: '03.05.2026 14:07',
  },
  {
    id: 'ИП 1257841', date: '03.05',
    status: 'Подготовка',
    initials: 'СЮ', client: 'Смирнова Ю.А.', phone: '+7 918 ***-**-56',
    fullName: 'Смирнова Юлия Андреевна', fullPhone: '+7 918 749-61-56',
    complex: 'Донстрой/Символ', complexPrice: '22 000 000 ₽', propertyType: 'Апартаменты',
    amount: '15 400 000 ₽', pv: '30%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '03.05.2026', updatedAt: '03.05.2026 12:45',
  },
  {
    id: 'ИП 1257840', date: '02.05',
    status: 'Ожидает решения',
    initials: 'МС', client: 'Морозов С.В.', phone: '+7 963 ***-**-93',
    fullName: 'Морозов Сергей Владимирович', fullPhone: '+7 963 827-44-93',
    complex: 'ПИК/Люберцы Парк', complexPrice: '6 300 000 ₽', propertyType: 'Машиноместо',
    amount: '4 410 000 ₽', pv: '30%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '02.05.2026', updatedAt: '02.05.2026 18:55',
  },
  {
    id: 'ИП 1257839', date: '02.05',
    status: 'Консультация',
    initials: 'ФИ', client: 'Фёдорова И.К.', phone: '+7 931 ***-**-27',
    fullName: 'Фёдорова Ирина Константиновна', fullPhone: '+7 931 560-39-27',
    complex: 'Самолет/Пригород Лесное', complexPrice: '5 900 000 ₽', propertyType: 'Кладовка',
    amount: '4 130 000 ₽', pv: '30%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '02.05.2026', updatedAt: '02.05.2026 13:10',
  },
  {
    id: 'ИП 1257838', date: '01.05',
    status: 'Новая заявка',
    initials: 'АР', client: 'Андреев Р.Е.', phone: '+7 909 ***-**-74',
    fullName: 'Андреев Роман Евгеньевич', fullPhone: '+7 909 213-76-74',
    complex: 'А101/Испанские кварталы', complexPrice: '8 400 000 ₽', propertyType: 'Квартира',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '01.05.2026', updatedAt: '01.05.2026 20:31',
  },
  {
    id: 'ИП 1257837', date: '01.05',
    status: 'Подготовка',
    initials: 'КМ', client: 'Козлова М.П.', phone: '+7 977 ***-**-48',
    fullName: 'Козлова Мария Петровна', fullPhone: '+7 977 094-82-48',
    complex: 'ЛСР/Цивилизация (СПб)', complexPrice: '11 200 000 ₽', propertyType: 'Таунхаус',
    amount: '7 840 000 ₽', pv: '30%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '01.05.2026', updatedAt: '01.05.2026 16:44',
  },
  {
    id: 'ИП 1257836', date: '30.04',
    status: 'Ожидает решения',
    initials: 'ВА', client: 'Васильев А.Ю.', phone: '+7 915 ***-**-62',
    fullName: 'Васильев Александр Юрьевич', fullPhone: '+7 915 371-05-62',
    complex: 'Эталон/Галактика', complexPrice: '14 600 000 ₽', propertyType: 'Коммерческая недвижимость',
    amount: '10 220 000 ₽', pv: '30%', term: '28 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '30.04.2026', updatedAt: '30.04.2026 19:20',
  },
  {
    id: 'ИП 1257835', date: '30.04',
    status: 'Консультация',
    initials: 'ПН', client: 'Павлова Н.О.', phone: '+7 936 ***-**-85',
    fullName: 'Павлова Надежда Олеговна', fullPhone: '+7 936 458-17-85',
    complex: 'Самолет/Жилой район Южный', complexPrice: '5 500 000 ₽', propertyType: 'Дом',
    amount: '3 850 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '30.04.2026', updatedAt: '30.04.2026 14:55',
  },
  {
    id: 'ИП 1257834', date: '30.04',
    status: 'Новая заявка',
    initials: 'СВ', client: 'Семёнов В.Б.', phone: '+7 921 ***-**-36',
    fullName: 'Семёнов Владимир Борисович', fullPhone: '+7 921 687-29-36',
    complex: 'ПИК/Кудрово (СПб)', complexPrice: '7 800 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '30.04.2026', updatedAt: '30.04.2026 11:08',
  },
  {
    id: 'ИП 1257833', date: '29.04',
    status: 'Подготовка',
    initials: 'ИТ', client: 'Ильина Т.В.', phone: '+7 906 ***-**-53',
    fullName: 'Ильина Татьяна Владимировна', fullPhone: '+7 906 542-33-53',
    complex: 'ФСК/Архитектор', complexPrice: '16 300 000 ₽',
    amount: '11 410 000 ₽', pv: '30%', term: '27 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '29.04.2026', updatedAt: '29.04.2026 18:37',
  },
  {
    id: 'ИП 1257832', date: '29.04',
    status: 'Консультация',
    initials: 'НА', client: 'Никитин А.С.', phone: '+7 945 ***-**-91',
    fullName: 'Никитин Артём Сергеевич', fullPhone: '+7 945 219-64-91',
    complex: 'Донстрой/Остров', complexPrice: '19 000 000 ₽',
    amount: '13 300 000 ₽', pv: '30%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '29.04.2026', updatedAt: '29.04.2026 17:14',
  },
  {
    id: 'ИП 1257831', date: '29.04',
    status: 'Консультация',
    initials: 'MM', client: 'Могиль М.В.', phone: '+7 901 ***-**-19',
    fullName: 'Могиль Михаил Валерьевич', fullPhone: '+7 901 234-56-19',
    complex: 'DOGMA/Самолет (Краснодар)', complexPrice: '5 000 000 ₽',
    amount: '3 750 000 ₽', pv: '25%', term: '16 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '29.04.2026', updatedAt: '29.04.2026 16:03',
  },
  {
    id: 'ИП 1257830', date: '28.04',
    status: 'Новая заявка',
    initials: 'MM', client: 'Могиль М.В.', phone: '+7 901 ***-**-19',
    fullName: 'Могиль Михаил Валерьевич', fullPhone: '+7 901 234-56-19',
    complex: 'Интеграционный ЖК/Химки', complexPrice: '0 ₽',
    amount: '0 ₽', pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '28.04.2026', updatedAt: '28.04.2026 11:45',
  },
  {
    id: 'ИП 1257829', date: '28.04',
    status: 'Подготовка',
    initials: 'MM', client: 'Могиль М.В.', phone: '+7 901 ***-**-19',
    fullName: 'Могиль Михаил Валерьевич', fullPhone: '+7 901 234-56-19',
    complex: 'Самолет/Митино О2', complexPrice: '15 000 000 ₽',
    amount: '8 000 000 ₽', pv: '47%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '28.04.2026', updatedAt: '28.04.2026 14:22',
  },
  {
    id: 'ИП 1257828', date: '28.04',
    status: 'Ожидает решения',
    initials: 'ЗА', client: 'Захаров А.И.', phone: '+7 916 ***-**-42',
    fullName: 'Захаров Алексей Игоревич', fullPhone: '+7 916 548-23-42',
    complex: 'ПИК/Береговой (Москва)', complexPrice: '12 400 000 ₽',
    amount: '9 300 000 ₽', pv: '25%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '28.04.2026', updatedAt: '28.04.2026 09:10',
  },
  {
    id: 'ИП 1257826', date: '28.04',
    status: 'Ожидает решения',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '28.04.2026', updatedAt: '29.04.2026 10:30',
  },
  {
    id: 'ИП 1257825', date: '27.04',
    status: 'Новая заявка',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '27.04.2026', updatedAt: '27.04.2026 17:05',
  },
  {
    id: 'ИП 1257824', date: '27.04',
    status: 'Консультация',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '27.04.2026', updatedAt: '27.04.2026 15:48',
  },
  {
    id: 'ИП 1257823', date: '27.04',
    status: 'Ожидает решения',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '27.04.2026', updatedAt: '29.04.2026 11:12',
  },
  {
    id: 'ИП 1257827', date: '27.04',
    status: 'Подготовка',
    initials: 'СП', client: 'Соколова П.В.', phone: '+7 903 ***-**-88',
    fullName: 'Соколова Полина Вячеславовна', fullPhone: '+7 903 762-14-88',
    complex: 'Эталон/Серебряный бор', complexPrice: '8 750 000 ₽',
    amount: '6 125 000 ₽', pv: '30%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '27.04.2026', updatedAt: '28.04.2026 08:55',
  },
  {
    id: 'ИП 1257822', date: '26.04',
    status: 'Новая заявка',
    initials: 'ЗА', client: 'Захаров А.И.', phone: '+7 916 ***-**-42',
    fullName: 'Захаров Алексей Игоревич', fullPhone: '+7 916 548-23-42',
    complex: 'ПИК/Мякинино', complexPrice: '7 400 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '26.04.2026', updatedAt: '26.04.2026 09:00',
  },
  {
    id: 'ИП 1257821', date: '25.04',
    status: 'Консультация',
    initials: 'ПВ', client: 'Попов В.М.', phone: '+7 499 ***-**-11',
    fullName: 'Попов Виктор Михайлович', fullPhone: '+7 499 312-67-11',
    complex: 'Самолет/Квартал Лайково', complexPrice: '6 200 000 ₽',
    amount: '4 600 000 ₽', pv: '25%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '25.04.2026', updatedAt: '25.04.2026 10:07',
  },
  {
    id: 'ИП 1257820', date: '24.04',
    status: 'Подготовка',
    initials: 'ТН', client: 'Тимофеев Н.П.', phone: '+7 922 ***-**-55',
    fullName: 'Тимофеев Николай Павлович', fullPhone: '+7 922 445-38-55',
    complex: 'А101/Новые Ватутинки-3', complexPrice: '8 900 000 ₽',
    amount: '7 100 000 ₽', pv: '30%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '24.04.2026', updatedAt: '24.04.2026 11:14',
  },
  {
    id: 'ИП 1257819', date: '23.04',
    status: 'Ожидает решения',
    initials: 'КС', client: 'Крылов С.А.', phone: '+7 911 ***-**-03',
    fullName: 'Крылов Сергей Андреевич', fullPhone: '+7 911 871-90-03',
    complex: 'ФСК/Первый Нагатинский', complexPrice: '13 500 000 ₽',
    amount: '9 400 000 ₽', pv: '35%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '23.04.2026', updatedAt: '23.04.2026 12:21',
  },
  {
    id: 'ИП 1257818', date: '22.04',
    status: 'Одобрена',
    initials: 'СП', client: 'Сергеев П.Р.', phone: '+7 933 ***-**-77',
    fullName: 'Сергеев Павел Романович', fullPhone: '+7 933 123-54-77',
    complex: 'Эталон/Медовая долина', complexPrice: '11 200 000 ₽',
    amount: '8 400 000 ₽', pv: '40%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '22.04.2026', updatedAt: '22.04.2026 13:28',
  },
  {
    id: 'ИП 1257817', date: '21.04',
    status: 'Отказано',
    initials: 'РО', client: 'Романова О.В.', phone: '+7 920 ***-**-29',
    fullName: 'Романова Ольга Викторовна', fullPhone: '+7 920 654-21-29',
    complex: 'MR Group/Метрополия', complexPrice: '21 000 000 ₽',
    amount: '16 800 000 ₽', pv: '20%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '21.04.2026', updatedAt: '21.04.2026 14:35',
  },
  {
    id: 'ИП 1257816', date: '20.04',
    status: 'Банк выбран',
    initials: 'ГН', client: 'Гусева Н.А.', phone: '+7 966 ***-**-14',
    fullName: 'Гусева Надежда Алексеевна', fullPhone: '+7 966 789-43-14',
    complex: 'Донстрой/Сердце Столицы', complexPrice: '26 500 000 ₽',
    amount: '18 500 000 ₽', pv: '25%', term: '27 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '20.04.2026', updatedAt: '20.04.2026 15:42',
  },
  {
    id: 'ИП 1257815', date: '19.04',
    status: 'Новая заявка',
    initials: 'БК', client: 'Белова К.Д.', phone: '+7 904 ***-**-68',
    fullName: 'Белова Ксения Дмитриевна', fullPhone: '+7 904 237-15-68',
    complex: 'ЛСР/Нагатино Простор', complexPrice: '9 800 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '19.04.2026', updatedAt: '19.04.2026 16:49',
  },
  {
    id: 'ИП 1257814', date: '18.04',
    status: 'Консультация',
    initials: 'ЗИ', client: 'Зубков И.Е.', phone: '+7 913 ***-**-82',
    fullName: 'Зубков Игорь Евгеньевич', fullPhone: '+7 913 560-84-82',
    complex: 'Группа ПСН/Флотилия (СПб)', complexPrice: '10 400 000 ₽',
    amount: '8 300 000 ₽', pv: '35%', term: '12 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '18.04.2026', updatedAt: '18.04.2026 17:56',
  },
  {
    id: 'ИП 1257813', date: '17.04',
    status: 'Подготовка',
    initials: 'ТМ', client: 'Туров М.О.', phone: '+7 944 ***-**-36',
    fullName: 'Туров Михаил Олегович', fullPhone: '+7 944 903-27-36',
    complex: 'LEGENDA/Лиговский (СПб)', complexPrice: '14 700 000 ₽',
    amount: '10 200 000 ₽', pv: '40%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '17.04.2026', updatedAt: '17.04.2026 18:03',
  },
  {
    id: 'ИП 1257812', date: '16.04',
    status: 'Ожидает решения',
    initials: 'ША', client: 'Широков А.В.', phone: '+7 956 ***-**-51',
    fullName: 'Широков Андрей Вячеславович', fullPhone: '+7 956 148-73-51',
    complex: 'ПИК/Аквилон Митино', complexPrice: '8 100 000 ₽',
    amount: '6 000 000 ₽', pv: '20%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '16.04.2026', updatedAt: '16.04.2026 19:10',
  },
  {
    id: 'ИП 1257811', date: '15.04',
    status: 'Одобрена',
    initials: 'РЕ', client: 'Рябова Е.С.', phone: '+7 907 ***-**-94',
    fullName: 'Рябова Елена Сергеевна', fullPhone: '+7 907 492-06-94',
    complex: 'Самолет/Некрасовка Парк', complexPrice: '5 800 000 ₽',
    amount: '4 600 000 ₽', pv: '25%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '15.04.2026', updatedAt: '15.04.2026 00:17',
  },
  {
    id: 'ИП 1257810', date: '14.04',
    status: 'Отказано',
    initials: 'ЛГ', client: 'Лунев Г.П.', phone: '+7 935 ***-**-17',
    fullName: 'Лунев Григорий Петрович', fullPhone: '+7 935 315-62-17',
    complex: 'Унистрой/Арбан (Казань)', complexPrice: '6 700 000 ₽',
    amount: '4 600 000 ₽', pv: '30%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '14.04.2026', updatedAt: '14.04.2026 01:24',
  },
  {
    id: 'ИП 1257809', date: '13.04',
    status: 'Банк выбран',
    initials: 'ПД', client: 'Пирогов Д.А.', phone: '+7 960 ***-**-43',
    fullName: 'Пирогов Денис Алексеевич', fullPhone: '+7 960 724-38-43',
    complex: 'DOGMA/Европейский (Краснодар)', complexPrice: '5 200 000 ₽',
    amount: '3 900 000 ₽', pv: '35%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '13.04.2026', updatedAt: '13.04.2026 02:31',
  },
  {
    id: 'ИП 1257808', date: '12.04',
    status: 'Новая заявка',
    initials: 'СИ', client: 'Семёнова И.Б.', phone: '+7 927 ***-**-08',
    fullName: 'Семёнова Ирина Борисовна', fullPhone: '+7 927 081-59-08',
    complex: 'Setl Group/Панорама 360 (СПб)', complexPrice: '12 300 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '12.04.2026', updatedAt: '12.04.2026 03:38',
  },
  {
    id: 'ИП 1257807', date: '11.04',
    status: 'Консультация',
    initials: 'ЗА', client: 'Захаров А.И.', phone: '+7 916 ***-**-42',
    fullName: 'Захаров Алексей Игоревич', fullPhone: '+7 916 548-23-42',
    complex: 'ПИК/Мякинино', complexPrice: '7 400 000 ₽',
    amount: '5 100 000 ₽', pv: '20%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '11.04.2026', updatedAt: '11.04.2026 04:45',
  },
  {
    id: 'ИП 1257806', date: '10.04',
    status: 'Подготовка',
    initials: 'ПВ', client: 'Попов В.М.', phone: '+7 499 ***-**-11',
    fullName: 'Попов Виктор Михайлович', fullPhone: '+7 499 312-67-11',
    complex: 'Самолет/Квартал Лайково', complexPrice: '6 200 000 ₽',
    amount: '4 600 000 ₽', pv: '25%', term: '12 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '10.04.2026', updatedAt: '10.04.2026 05:52',
  },
  {
    id: 'ИП 1257805', date: '09.04',
    status: 'Ожидает решения',
    initials: 'ТН', client: 'Тимофеев Н.П.', phone: '+7 922 ***-**-55',
    fullName: 'Тимофеев Николай Павлович', fullPhone: '+7 922 445-38-55',
    complex: 'А101/Новые Ватутинки-3', complexPrice: '8 900 000 ₽',
    amount: '7 100 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '09.04.2026', updatedAt: '09.04.2026 06:59',
  },
  {
    id: 'ИП 1257804', date: '08.04',
    status: 'Одобрена',
    initials: 'КС', client: 'Крылов С.А.', phone: '+7 911 ***-**-03',
    fullName: 'Крылов Сергей Андреевич', fullPhone: '+7 911 871-90-03',
    complex: 'ФСК/Первый Нагатинский', complexPrice: '13 500 000 ₽',
    amount: '9 400 000 ₽', pv: '35%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '08.04.2026', updatedAt: '08.04.2026 07:06',
  },
  {
    id: 'ИП 1257803', date: '07.04',
    status: 'Отказано',
    initials: 'СП', client: 'Сергеев П.Р.', phone: '+7 933 ***-**-77',
    fullName: 'Сергеев Павел Романович', fullPhone: '+7 933 123-54-77',
    complex: 'Эталон/Медовая долина', complexPrice: '11 200 000 ₽',
    amount: '8 400 000 ₽', pv: '40%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '07.04.2026', updatedAt: '07.04.2026 08:13',
  },
  {
    id: 'ИП 1257802', date: '06.04',
    status: 'Банк выбран',
    initials: 'РО', client: 'Романова О.В.', phone: '+7 920 ***-**-29',
    fullName: 'Романова Ольга Викторовна', fullPhone: '+7 920 654-21-29',
    complex: 'MR Group/Метрополия', complexPrice: '21 000 000 ₽',
    amount: '16 800 000 ₽', pv: '20%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '06.04.2026', updatedAt: '06.04.2026 09:20',
  },
  {
    id: 'ИП 1257801', date: '05.04',
    status: 'Новая заявка',
    initials: 'ГН', client: 'Гусева Н.А.', phone: '+7 966 ***-**-14',
    fullName: 'Гусева Надежда Алексеевна', fullPhone: '+7 966 789-43-14',
    complex: 'Донстрой/Сердце Столицы', complexPrice: '26 500 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '05.04.2026', updatedAt: '05.04.2026 10:27',
  },
  {
    id: 'ИП 1257800', date: '04.04',
    status: 'Консультация',
    initials: 'БК', client: 'Белова К.Д.', phone: '+7 904 ***-**-68',
    fullName: 'Белова Ксения Дмитриевна', fullPhone: '+7 904 237-15-68',
    complex: 'ЛСР/Нагатино Простор', complexPrice: '9 800 000 ₽',
    amount: '7 300 000 ₽', pv: '30%', term: '27 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '04.04.2026', updatedAt: '04.04.2026 11:34',
  },
  {
    id: 'ИП 1257799', date: '03.04',
    status: 'Подготовка',
    initials: 'ЗИ', client: 'Зубков И.Е.', phone: '+7 913 ***-**-82',
    fullName: 'Зубков Игорь Евгеньевич', fullPhone: '+7 913 560-84-82',
    complex: 'Группа ПСН/Флотилия (СПб)', complexPrice: '10 400 000 ₽',
    amount: '8 300 000 ₽', pv: '35%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '03.04.2026', updatedAt: '03.04.2026 12:41',
  },
  {
    id: 'ИП 1257798', date: '02.04',
    status: 'Ожидает решения',
    initials: 'ТМ', client: 'Туров М.О.', phone: '+7 944 ***-**-36',
    fullName: 'Туров Михаил Олегович', fullPhone: '+7 944 903-27-36',
    complex: 'LEGENDA/Лиговский (СПб)', complexPrice: '14 700 000 ₽',
    amount: '10 200 000 ₽', pv: '40%', term: '12 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '02.04.2026', updatedAt: '02.04.2026 13:48',
  },
  {
    id: 'ИП 1257797', date: '01.04',
    status: 'Одобрена',
    initials: 'ША', client: 'Широков А.В.', phone: '+7 956 ***-**-51',
    fullName: 'Широков Андрей Вячеславович', fullPhone: '+7 956 148-73-51',
    complex: 'ПИК/Аквилон Митино', complexPrice: '8 100 000 ₽',
    amount: '6 000 000 ₽', pv: '20%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '01.04.2026', updatedAt: '01.04.2026 14:55',
  },
  {
    id: 'ИП 1257796', date: '31.03',
    status: 'Отказано',
    initials: 'РЕ', client: 'Рябова Е.С.', phone: '+7 907 ***-**-94',
    fullName: 'Рябова Елена Сергеевна', fullPhone: '+7 907 492-06-94',
    complex: 'Самолет/Некрасовка Парк', complexPrice: '5 800 000 ₽',
    amount: '4 600 000 ₽', pv: '25%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '31.03.2026', updatedAt: '31.03.2026 15:02',
  },
  {
    id: 'ИП 1257795', date: '30.03',
    status: 'Банк выбран',
    initials: 'ЛГ', client: 'Лунев Г.П.', phone: '+7 935 ***-**-17',
    fullName: 'Лунев Григорий Петрович', fullPhone: '+7 935 315-62-17',
    complex: 'Унистрой/Арбан (Казань)', complexPrice: '6 700 000 ₽',
    amount: '4 600 000 ₽', pv: '30%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '30.03.2026', updatedAt: '30.03.2026 16:09',
  },
  {
    id: 'ИП 1257794', date: '29.03',
    status: 'Новая заявка',
    initials: 'ПД', client: 'Пирогов Д.А.', phone: '+7 960 ***-**-43',
    fullName: 'Пирогов Денис Алексеевич', fullPhone: '+7 960 724-38-43',
    complex: 'DOGMA/Европейский (Краснодар)', complexPrice: '5 200 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '29.03.2026', updatedAt: '29.03.2026 17:16',
  },
  {
    id: 'ИП 1257793', date: '28.03',
    status: 'Консультация',
    initials: 'СИ', client: 'Семёнова И.Б.', phone: '+7 927 ***-**-08',
    fullName: 'Семёнова Ирина Борисовна', fullPhone: '+7 927 081-59-08',
    complex: 'Setl Group/Панорама 360 (СПб)', complexPrice: '12 300 000 ₽',
    amount: '9 800 000 ₽', pv: '40%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '28.03.2026', updatedAt: '28.03.2026 18:23',
  },
  {
    id: 'ИП 1257792', date: '27.03',
    status: 'Подготовка',
    initials: 'ЗА', client: 'Захаров А.И.', phone: '+7 916 ***-**-42',
    fullName: 'Захаров Алексей Игоревич', fullPhone: '+7 916 548-23-42',
    complex: 'ПИК/Мякинино', complexPrice: '7 400 000 ₽',
    amount: '5 100 000 ₽', pv: '20%', term: '27 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '27.03.2026', updatedAt: '27.03.2026 19:30',
  },
  {
    id: 'ИП 1257791', date: '26.03',
    status: 'Ожидает решения',
    initials: 'ПВ', client: 'Попов В.М.', phone: '+7 499 ***-**-11',
    fullName: 'Попов Виктор Михайлович', fullPhone: '+7 499 312-67-11',
    complex: 'Самолет/Квартал Лайково', complexPrice: '6 200 000 ₽',
    amount: '4 600 000 ₽', pv: '25%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '26.03.2026', updatedAt: '26.03.2026 00:37',
  },
  {
    id: 'ИП 1257790', date: '25.03',
    status: 'Одобрена',
    initials: 'ТН', client: 'Тимофеев Н.П.', phone: '+7 922 ***-**-55',
    fullName: 'Тимофеев Николай Павлович', fullPhone: '+7 922 445-38-55',
    complex: 'А101/Новые Ватутинки-3', complexPrice: '8 900 000 ₽',
    amount: '7 100 000 ₽', pv: '30%', term: '12 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '25.03.2026', updatedAt: '25.03.2026 01:44',
  },
  {
    id: 'ИП 1257789', date: '24.03',
    status: 'Отказано',
    initials: 'КС', client: 'Крылов С.А.', phone: '+7 911 ***-**-03',
    fullName: 'Крылов Сергей Андреевич', fullPhone: '+7 911 871-90-03',
    complex: 'ФСК/Первый Нагатинский', complexPrice: '13 500 000 ₽',
    amount: '9 400 000 ₽', pv: '35%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '24.03.2026', updatedAt: '24.03.2026 02:51',
  },
  {
    id: 'ИП 1257788', date: '23.03',
    status: 'Банк выбран',
    initials: 'СП', client: 'Сергеев П.Р.', phone: '+7 933 ***-**-77',
    fullName: 'Сергеев Павел Романович', fullPhone: '+7 933 123-54-77',
    complex: 'Эталон/Медовая долина', complexPrice: '11 200 000 ₽',
    amount: '8 400 000 ₽', pv: '40%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '23.03.2026', updatedAt: '23.03.2026 03:58',
  },
  {
    id: 'ИП 1257787', date: '22.03',
    status: 'Новая заявка',
    initials: 'РО', client: 'Романова О.В.', phone: '+7 920 ***-**-29',
    fullName: 'Романова Ольга Викторовна', fullPhone: '+7 920 654-21-29',
    complex: 'MR Group/Метрополия', complexPrice: '21 000 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '22.03.2026', updatedAt: '22.03.2026 04:05',
  },
  {
    id: 'ИП 1257786', date: '21.03',
    status: 'Консультация',
    initials: 'ГН', client: 'Гусева Н.А.', phone: '+7 966 ***-**-14',
    fullName: 'Гусева Надежда Алексеевна', fullPhone: '+7 966 789-43-14',
    complex: 'Донстрой/Сердце Столицы', complexPrice: '26 500 000 ₽',
    amount: '18 500 000 ₽', pv: '25%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '21.03.2026', updatedAt: '21.03.2026 05:12',
  },
  {
    id: 'ИП 1257785', date: '20.03',
    status: 'Подготовка',
    initials: 'БК', client: 'Белова К.Д.', phone: '+7 904 ***-**-68',
    fullName: 'Белова Ксения Дмитриевна', fullPhone: '+7 904 237-15-68',
    complex: 'ЛСР/Нагатино Простор', complexPrice: '9 800 000 ₽',
    amount: '7 300 000 ₽', pv: '30%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '20.03.2026', updatedAt: '20.03.2026 06:19',
  },
  {
    id: 'ИП 1257784', date: '19.03',
    status: 'Ожидает решения',
    initials: 'ЗИ', client: 'Зубков И.Е.', phone: '+7 913 ***-**-82',
    fullName: 'Зубков Игорь Евгеньевич', fullPhone: '+7 913 560-84-82',
    complex: 'Группа ПСН/Флотилия (СПб)', complexPrice: '10 400 000 ₽',
    amount: '8 300 000 ₽', pv: '35%', term: '27 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '19.03.2026', updatedAt: '19.03.2026 07:26',
  },
  {
    id: 'ИП 1257783', date: '18.03',
    status: 'Одобрена',
    initials: 'ТМ', client: 'Туров М.О.', phone: '+7 944 ***-**-36',
    fullName: 'Туров Михаил Олегович', fullPhone: '+7 944 903-27-36',
    complex: 'LEGENDA/Лиговский (СПб)', complexPrice: '14 700 000 ₽',
    amount: '10 200 000 ₽', pv: '40%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '18.03.2026', updatedAt: '18.03.2026 08:33',
  },
  {
    id: 'ИП 1257782', date: '17.03',
    status: 'Отказано',
    initials: 'ША', client: 'Широков А.В.', phone: '+7 956 ***-**-51',
    fullName: 'Широков Андрей Вячеславович', fullPhone: '+7 956 148-73-51',
    complex: 'ПИК/Аквилон Митино', complexPrice: '8 100 000 ₽',
    amount: '6 000 000 ₽', pv: '20%', term: '12 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '17.03.2026', updatedAt: '17.03.2026 09:40',
  },
  {
    id: 'ИП 1257781', date: '16.03',
    status: 'Банк выбран',
    initials: 'РЕ', client: 'Рябова Е.С.', phone: '+7 907 ***-**-94',
    fullName: 'Рябова Елена Сергеевна', fullPhone: '+7 907 492-06-94',
    complex: 'Самолет/Некрасовка Парк', complexPrice: '5 800 000 ₽',
    amount: '4 600 000 ₽', pv: '25%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '16.03.2026', updatedAt: '16.03.2026 10:47',
  },
  {
    id: 'ИП 1257780', date: '15.03',
    status: 'Новая заявка',
    initials: 'ЛГ', client: 'Лунев Г.П.', phone: '+7 935 ***-**-17',
    fullName: 'Лунев Григорий Петрович', fullPhone: '+7 935 315-62-17',
    complex: 'Унистрой/Арбан (Казань)', complexPrice: '6 700 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '15.03.2026', updatedAt: '15.03.2026 11:54',
  },
  {
    id: 'ИП 1257779', date: '14.03',
    status: 'Консультация',
    initials: 'ПД', client: 'Пирогов Д.А.', phone: '+7 960 ***-**-43',
    fullName: 'Пирогов Денис Алексеевич', fullPhone: '+7 960 724-38-43',
    complex: 'DOGMA/Европейский (Краснодар)', complexPrice: '5 200 000 ₽',
    amount: '3 900 000 ₽', pv: '35%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '14.03.2026', updatedAt: '14.03.2026 12:01',
  },
  {
    id: 'ИП 1257778', date: '13.03',
    status: 'Подготовка',
    initials: 'СИ', client: 'Семёнова И.Б.', phone: '+7 927 ***-**-08',
    fullName: 'Семёнова Ирина Борисовна', fullPhone: '+7 927 081-59-08',
    complex: 'Setl Group/Панорама 360 (СПб)', complexPrice: '12 300 000 ₽',
    amount: '9 800 000 ₽', pv: '40%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '13.03.2026', updatedAt: '13.03.2026 13:08',
  },
  {
    id: 'ИП 1257777', date: '12.03',
    status: 'Ожидает решения',
    initials: 'ЗА', client: 'Захаров А.И.', phone: '+7 916 ***-**-42',
    fullName: 'Захаров Алексей Игоревич', fullPhone: '+7 916 548-23-42',
    complex: 'ПИК/Мякинино', complexPrice: '7 400 000 ₽',
    amount: '5 100 000 ₽', pv: '20%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '12.03.2026', updatedAt: '12.03.2026 14:15',
  },
  {
    id: 'ИП 1257776', date: '11.03',
    status: 'Одобрена',
    initials: 'ПВ', client: 'Попов В.М.', phone: '+7 499 ***-**-11',
    fullName: 'Попов Виктор Михайлович', fullPhone: '+7 499 312-67-11',
    complex: 'Самолет/Квартал Лайково', complexPrice: '6 200 000 ₽',
    amount: '4 600 000 ₽', pv: '25%', term: '27 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '11.03.2026', updatedAt: '11.03.2026 15:22',
  },
  {
    id: 'ИП 1257775', date: '10.03',
    status: 'Отказано',
    initials: 'ТН', client: 'Тимофеев Н.П.', phone: '+7 922 ***-**-55',
    fullName: 'Тимофеев Николай Павлович', fullPhone: '+7 922 445-38-55',
    complex: 'А101/Новые Ватутинки-3', complexPrice: '8 900 000 ₽',
    amount: '7 100 000 ₽', pv: '30%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '10.03.2026', updatedAt: '10.03.2026 16:29',
  },
  {
    id: 'ИП 1257774', date: '09.03',
    status: 'Банк выбран',
    initials: 'КС', client: 'Крылов С.А.', phone: '+7 911 ***-**-03',
    fullName: 'Крылов Сергей Андреевич', fullPhone: '+7 911 871-90-03',
    complex: 'ФСК/Первый Нагатинский', complexPrice: '13 500 000 ₽',
    amount: '9 400 000 ₽', pv: '35%', term: '12 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '09.03.2026', updatedAt: '09.03.2026 17:36',
  },
  {
    id: 'ИП 1257773', date: '08.03',
    status: 'Новая заявка',
    initials: 'СП', client: 'Сергеев П.Р.', phone: '+7 933 ***-**-77',
    fullName: 'Сергеев Павел Романович', fullPhone: '+7 933 123-54-77',
    complex: 'Эталон/Медовая долина', complexPrice: '11 200 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '08.03.2026', updatedAt: '08.03.2026 18:43',
  },
  {
    id: 'ИП 1257772', date: '07.03',
    status: 'Консультация',
    initials: 'РО', client: 'Романова О.В.', phone: '+7 920 ***-**-29',
    fullName: 'Романова Ольга Викторовна', fullPhone: '+7 920 654-21-29',
    complex: 'MR Group/Метрополия', complexPrice: '21 000 000 ₽',
    amount: '16 800 000 ₽', pv: '20%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '07.03.2026', updatedAt: '07.03.2026 19:50',
  },
  {
    id: 'ИП 1257771', date: '06.03',
    status: 'Подготовка',
    initials: 'ГН', client: 'Гусева Н.А.', phone: '+7 966 ***-**-14',
    fullName: 'Гусева Надежда Алексеевна', fullPhone: '+7 966 789-43-14',
    complex: 'Донстрой/Сердце Столицы', complexPrice: '26 500 000 ₽',
    amount: '18 500 000 ₽', pv: '25%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '06.03.2026', updatedAt: '06.03.2026 00:57',
  },
  {
    id: 'ИП 1257770', date: '05.03',
    status: 'Ожидает решения',
    initials: 'БК', client: 'Белова К.Д.', phone: '+7 904 ***-**-68',
    fullName: 'Белова Ксения Дмитриевна', fullPhone: '+7 904 237-15-68',
    complex: 'ЛСР/Нагатино Простор', complexPrice: '9 800 000 ₽',
    amount: '7 300 000 ₽', pv: '30%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная',
    createdAt: '05.03.2026', updatedAt: '05.03.2026 01:04',
  },
  {
    id: 'ИП 1257769', date: '04.03',
    status: 'Одобрена',
    initials: 'ЗИ', client: 'Зубков И.Е.', phone: '+7 913 ***-**-82',
    fullName: 'Зубков Игорь Евгеньевич', fullPhone: '+7 913 560-84-82',
    complex: 'Группа ПСН/Флотилия (СПб)', complexPrice: '10 400 000 ₽',
    amount: '8 300 000 ₽', pv: '35%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '04.03.2026', updatedAt: '04.03.2026 02:11',
  },
  {
    id: 'ИП 1257768', date: '03.03',
    status: 'Отказано',
    initials: 'ТМ', client: 'Туров М.О.', phone: '+7 944 ***-**-36',
    fullName: 'Туров Михаил Олегович', fullPhone: '+7 944 903-27-36',
    complex: 'LEGENDA/Лиговский (СПб)', complexPrice: '14 700 000 ₽',
    amount: '10 200 000 ₽', pv: '40%', term: '27 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '03.03.2026', updatedAt: '03.03.2026 03:18',
  },
  {
    id: 'ИП 1257767', date: '02.03',
    status: 'Банк выбран',
    initials: 'ША', client: 'Широков А.В.', phone: '+7 956 ***-**-51',
    fullName: 'Широков Андрей Вячеславович', fullPhone: '+7 956 148-73-51',
    complex: 'ПИК/Аквилон Митино', complexPrice: '8 100 000 ₽',
    amount: '6 000 000 ₽', pv: '20%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '02.03.2026', updatedAt: '02.03.2026 04:25',
  },
  {
    id: 'ИП 1257766', date: '01.03',
    status: 'Новая заявка',
    initials: 'РЕ', client: 'Рябова Е.С.', phone: '+7 907 ***-**-94',
    fullName: 'Рябова Елена Сергеевна', fullPhone: '+7 907 492-06-94',
    complex: 'Самолет/Некрасовка Парк', complexPrice: '5 800 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '01.03.2026', updatedAt: '01.03.2026 05:32',
  },
  {
    id: 'ИП 1257765', date: '28.02',
    status: 'Консультация',
    initials: 'ЛГ', client: 'Лунев Г.П.', phone: '+7 935 ***-**-17',
    fullName: 'Лунев Григорий Петрович', fullPhone: '+7 935 315-62-17',
    complex: 'Унистрой/Арбан (Казань)', complexPrice: '6 700 000 ₽',
    amount: '4 600 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная',
    createdAt: '28.02.2026', updatedAt: '28.02.2026 06:39',
  },
  {
    id: 'ИП 1257764', date: '27.02',
    status: 'Подготовка',
    initials: 'ПД', client: 'Пирогов Д.А.', phone: '+7 960 ***-**-43',
    fullName: 'Пирогов Денис Алексеевич', fullPhone: '+7 960 724-38-43',
    complex: 'DOGMA/Европейский (Краснодар)', complexPrice: '5 200 000 ₽',
    amount: '3 900 000 ₽', pv: '35%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная',
    createdAt: '27.02.2026', updatedAt: '27.02.2026 07:46',
  },
  {
    id: 'ИП 1257763', date: '26.02',
    status: 'Ожидает решения',
    initials: 'СИ', client: 'Семёнова И.Б.', phone: '+7 927 ***-**-08',
    fullName: 'Семёнова Ирина Борисовна', fullPhone: '+7 927 081-59-08',
    complex: 'Setl Group/Панорама 360 (СПб)', complexPrice: '12 300 000 ₽',
    amount: '9 800 000 ₽', pv: '40%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ',
    createdAt: '26.02.2026', updatedAt: '26.02.2026 08:53',
  },
]

applications.forEach(app => { mortgageManagerSelections[app.id] = 'Смирнова Юлия' })
watch(filteredApplications, v => { if (activeModule.value === 'Ипотека') activeCount.value = v.length }, { immediate: true })
</script>

<style>
svg.lucide { stroke-width: 1.5; }
</style>
