<template>
  <div :class="['flex h-screen overflow-hidden bg-gray-50 pt-2 pb-2 pr-2 gap-x-2 transition-all duration-300', sidebarCollapsed ? 'pl-2' : 'pl-4']">

    <!-- ─── Sidebar ───────────────────────────────── -->
    <div :class="['flex shrink-0 flex-col gap-y-5 overflow-hidden transition-all duration-300', sidebarCollapsed ? 'w-0' : 'w-[250px]']">

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


      <!-- Sidebar Create button (только в режиме ипотеки) -->
      <div v-if="mode === 'mortgage'" class="shrink-0 px-1">
        <Button color="indigo" class="w-full justify-center" @click="createModalOpen = true">
          <PlusIcon :size="16" class="shrink-0" />
          Создать
        </Button>
      </div>

      <!-- Nav -->
      <nav class="flex flex-1 flex-col">
        <ul role="list" class="flex flex-1 flex-col gap-y-7">
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
                <Disclosure v-else as="div" v-slot="{ open }">
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
                      <a
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
          <li>
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

          <!-- Bottom block (pinned) -->
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
    </div>

    <!-- ─── Main content ──────────────────────────── -->
    <div class="relative flex flex-1 flex-col overflow-hidden rounded-3xl bg-white shadow-sm ring-1 ring-gray-200">

      <!-- ─── Filter Drawer ────────────────────────── -->
      <FilterDrawer :open="filterDrawerOpen" :count="activeCount ?? 0" @close="filterDrawerOpen = false" />

      <!-- ─── Scoring Filter Drawer ───────────────── -->
      <ScoringFilterDrawer :open="scoringFilterDrawerOpen" :count="activeCount ?? 0" @close="scoringFilterDrawerOpen = false" />

      <!-- ─── Document Packages Filter Drawer ─────── -->
      <DocumentPackagesFilterDrawer :open="docPackagesFilterOpen" :count="activeCount ?? 0" @close="docPackagesFilterOpen = false" />

      <!-- ─── Document Package Preview Drawer ──────── -->
      <DocumentPackagePreviewDrawer :open="docPackagePreviewOpen" :item="selectedDocPackage" @close="docPackagePreviewOpen = false" />

      <!-- ─── Bank Program Drawer ──────────────────── -->
      <BankDrawer :open="bankDrawerOpen" :bank="selectedBank" @close="bankDrawerOpen = false" />

      <!-- ─── Create Application Modal ─────────────── -->
      <CreateApplicationModal :open="createModalOpen" :mode="mode" @close="createModalOpen = false" />
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
      <RegistrationFilterDrawer :open="registrationFilterOpen" :count="activeCount ?? 0" @close="registrationFilterOpen = false" />

      <!-- ─── App Preview Drawer (Регистрация) ──────────── -->
      <AppPreviewDrawer
        :open="registrationDrawerOpen"
        :app="selectedRegistration ?? {}"
        :rows="selectedRegistration ? registrationDrawerRows(selectedRegistration) : []"
        :history="registrationHistory"
        :show-documents="true"
        :show-anketa="false"
        @close="registrationDrawerOpen = false"
      />

      <!-- ─── Digital Signatures Filter Drawer ───────── -->
      <DigitalSignaturesFilterDrawer :open="digitalSignaturesFilterOpen" :count="activeCount ?? 0" @close="digitalSignaturesFilterOpen = false" />

      <!-- ─── Digital Signatures Preview Drawer ───────── -->
      <AppPreviewDrawer
        :open="digitalSignaturesDrawerOpen"
        :app="selectedDigitalSignature ?? {}"
        :rows="selectedDigitalSignature ? digitalSignaturesDrawerRows(selectedDigitalSignature) : []"
        :history="digitalSignaturesHistory"
        :show-anketa="false"
        :hide-icon-buttons="true"
        :show-history="false"
        :show-signatures="true"
        @close="digitalSignaturesDrawerOpen = false"
      />

      <!-- ─── Insurance Filter Drawer ──────────────────── -->
      <InsuranceFilterDrawer :open="insuranceFilterOpen" :count="activeCount ?? 0" @close="insuranceFilterOpen = false" />

      <!-- ─── App Preview Drawer (Страховка) ───────────── -->
      <AppPreviewDrawer
        :open="insuranceDrawerOpen"
        :app="selectedInsurance ?? {}"
        :rows="selectedInsurance ? insuranceDrawerRows(selectedInsurance) : []"
        :history="insuranceHistory"
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
      <div class="shrink-0 flex items-center gap-x-3 pl-6 pr-8 py-2.5 border-b border-[#f4f4f5]">
        <Button plain class="size-8 px-0 text-gray-400 hover:text-gray-600" @click="sidebarCollapsed = !sidebarCollapsed">
          <PanelLeftIcon :size="16" :stroke-width="1.5" class="shrink-0 aspect-square" style="width:16px;height:16px;stroke:#18181B;" aria-hidden="true" />
        </Button>
        <div class="flex items-center gap-x-4">
          <div class="flex items-center gap-x-3">
            <span class="text-[20px] leading-[32px] font-medium text-[#18181b]">{{ activeModule }}</span>
            <Badge
              v-if="activeCount !== null && activeModule !== 'Калькулятор ипотеки' && activeModule !== 'Аналитика'"
              color="zinc"
            >{{ activeCount.toLocaleString('ru-RU') }}</Badge>
          </div>

          <Button v-if="(mode === 'all' || sidebarCollapsed) && activeModule !== 'Калькулятор ипотеки' && activeModule !== 'Аналитика'" color="indigo" @click="createModalOpen = true">
            <PlusIcon :size="16" class="shrink-0" />
            Создать
          </Button>
        </div>
      </div>

      <!-- Шапка: строка 2 — поиск + фильтры (только Ипотека) -->
      <div v-if="activeModule === 'Ипотека'" class="shrink-0 flex items-center justify-between border-b border-[#f4f4f5] px-8 py-2">
        <SearchInput v-model="searchQuery" placeholder="Поиск по ID, ФИО или телефону" />
        <div class="flex items-center gap-x-3">
          <span v-if="searchQuery.trim()" class="text-[14px] font-light text-[#71717a] mr-5">
            {{ searchResultText }}
          </span>
          <Button outline @click="filterDrawerOpen = true">
            <FunnelIcon :size="16" class="shrink-0 aspect-square" style="width:16px;height:16px;" />
            Фильтры и сортировка
          </Button>
          <ExportPopover :count="filteredApplications.length" />
        </div>
      </div>

      <!-- Scrollable content -->
      <div class="flex-1 overflow-auto">

        <!-- Ипотека -->
        <div v-if="activeModule === 'Ипотека'" class="px-6 pt-6 pr-8 pb-10">
        <table class="border-separate border-spacing-0">
        <tbody>

          <tr v-for="app in filteredApplications" :key="app.id" class="group cursor-pointer" @click="openApp(app)">
            <td class="rounded-l-2xl group-hover:bg-zinc-50 align-middle py-3 pl-2 pr-2" @click.stop="toggleRow(app.id)">
              <label class="inline-flex cursor-pointer">
                <span :class="['relative flex size-4 items-center justify-center rounded-sm border', checkedRows.has(app.id) ? 'bg-indigo-600 border-transparent' : 'bg-white border-zinc-950/15 hover:border-zinc-950/30']">
                  <svg :class="['size-3 stroke-white transition-opacity', checkedRows.has(app.id) ? 'opacity-100' : 'opacity-0']" viewBox="0 0 14 14" fill="none"><path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" /></svg>
                </span>
              </label>
            </td>
            <td class="group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <TableLink @click="openDrawer(app)">{{ app.id }}</TableLink>
                <span class="text-[14px] leading-[20px] font-light text-zinc-900">от {{ app.date }}</span>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap">
              <div class="flex items-start gap-x-3">
                <span class="mt-0.5 inline-flex size-8 shrink-0 items-center justify-center rounded-full bg-[#e4e4e7] text-[14px] font-medium text-[#71717a]">{{ app.initials }}</span>
                <div class="flex flex-col gap-y-1">
                  <span class="text-[14px] leading-[20px] font-normal text-[#18181b]">{{ app.client }}</span>
                  <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ app.phone }}</span>
                </div>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap">
              <span v-if="app.status" :class="statusBadgeClass(app.status)">{{ app.status }}</span>
            </td>
            <td class="group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ app.complex }}</span>
                <span class="text-[14px] leading-[20px] font-light text-zinc-900">{{ app.complexPrice }}</span>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <span class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ app.amount }}<span v-if="app.pv" class="text-[#71717a]"> ПВ {{ app.pv }}</span></span>
                <span v-if="app.term" class="text-[14px] leading-[20px] font-light text-zinc-900">{{ app.term }}</span>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <span v-if="app.houseType" class="text-[14px] leading-[20px] font-light text-[#18181b]">{{ app.houseType }}</span>
                <span v-if="app.mortgageType" class="text-[14px] leading-[20px] font-light text-zinc-900">{{ app.mortgageType }}</span>
              </div>
            </td>
            <td class="group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap">
              <div class="flex flex-col gap-y-1">
                <span class="text-[14px] leading-[20px] font-light text-[#18181b]">Создана {{ app.createdAt.slice(0, 5) }}</span>
                <span class="text-[14px] leading-[20px] font-light text-zinc-900">Обновлена {{ app.updatedAt.replace(/\.\d{4}/, '') }}</span>
              </div>
            </td>
            <!-- Менеджер -->
            <td class="rounded-r-2xl group-hover:bg-zinc-50 align-top px-3 py-[20.5px] whitespace-nowrap" @click.stop>
              <CatalystListbox
                :options="managerSelectOptions"
                :model-value="mortgageManagerSelections[app.id]"
                @update:model-value="mortgageManagerSelections[app.id] = $event"
                placeholder="Менеджер"
              />
            </td>
          </tr>
        </tbody>
        </table>

        <!-- Empty state при поиске -->
        <div v-if="filteredApplications.length === 0 && searchQuery.trim()" class="flex flex-col items-center justify-center py-24 text-center">
          <svg class="mx-auto size-12 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true">
            <path vector-effect="non-scaling-stroke" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 13h6m-3-3v6m-9 1V7a2 2 0 012-2h6l2 2h6a2 2 0 012 2v8a2 2 0 01-2 2H5a2 2 0 01-2-2z" />
          </svg>
          <h3 class="mt-3 text-sm font-medium text-zinc-900">Заявки не найдены</h3>
          <p class="mt-1 text-sm font-light text-zinc-500">Попробуйте изменить параметры поиска или фильтрации</p>
          <div class="mt-6">
            <Button outline @click="searchQuery = ''">Сбросить поиск</Button>
          </div>
        </div>

        </div>

        <!-- Калькулятор ипотеки -->
        <CalculatorPage v-else-if="activeModule === 'Калькулятор ипотеки'" @open-bank="openBankDrawer" />

        <!-- Скоринг -->
        <ScoringPage v-else-if="activeModule === 'Скоринг'" :active-sub-item="activeSubItem" @open-filter="scoringFilterDrawerOpen = true" @open-preview="openScoringDrawer" @update:count="activeCount = $event" />

        <!-- Пакеты документов -->
        <DocumentPackagesPage v-else-if="activeModule === 'Пакеты документов'" :active-sub-item="activeSubItem" @open-filter="docPackagesFilterOpen = true" @open-preview="item => { selectedDocPackage = item; docPackagePreviewOpen = true }" @update:count="activeCount = $event" />

        <!-- Регистрация -->
        <RegistrationPage v-else-if="activeModule === 'Регистрация'" :active-sub-item="activeSubItem" @open-filter="registrationFilterOpen = true" @open-preview="openRegistrationPreview" @update:count="activeCount = $event" />

        <!-- Страховка -->
        <InsurancePage v-else-if="activeModule === 'Страховка'" :active-sub-item="activeSubItem" @open-filter="insuranceFilterOpen = true" @open-preview="openInsurancePreview" @update:count="activeCount = $event" />

        <!-- Цифровые подписи -->
        <DigitalSignaturesPage v-else-if="activeModule === 'Цифровые подписи'" :active-sub-item="activeSubItem" @open-filter="digitalSignaturesFilterOpen = true" @open-preview="openDigitalSignaturesPreview" @update:count="activeCount = $event" />

        <!-- Аналитика -->
        <AnalyticsPage v-else-if="activeModule === 'Аналитика'" />


      </div>

      <SelectionBar :count="checkedRows.size" @clear="checkedRows = new Set()" @delete="checkedRows = new Set()" />

    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted, reactive } from 'vue'

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
import CreateApplicationModal from './src/components/CreateApplicationModal.vue'
import ExportPopover from './src/components/ExportPopover.vue'
import ToastNotification from './src/components/ToastNotification.vue'
import CatalystListbox from './src/components/CatalystListbox.vue'
import DocumentPackagesPage from './src/components/DocumentPackagesPage.vue'
import DocumentPackagesFilterDrawer from './src/components/DocumentPackagesFilterDrawer.vue'
import DocumentPackagePreviewDrawer from './src/components/DocumentPackagePreviewDrawer.vue'
import RegistrationPage from './src/components/RegistrationPage.vue'
import RegistrationFilterDrawer from './src/components/RegistrationFilterDrawer.vue'
import InsurancePage from './src/components/InsurancePage.vue'
import InsuranceFilterDrawer from './src/components/InsuranceFilterDrawer.vue'
import DigitalSignaturesPage from './src/components/DigitalSignaturesPage.vue'
import DigitalSignaturesFilterDrawer from './src/components/DigitalSignaturesFilterDrawer.vue'
import SelectionBar from './src/components/SelectionBar.vue'
import { Disclosure, DisclosureButton, DisclosurePanel } from '@headlessui/vue'
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
} from 'lucide-vue-next'

const isOffline = ref(true)
onMounted(() => {
  window.addEventListener('online',  () => { isOffline.value = false })
  window.addEventListener('offline', () => { isOffline.value = true  })
})
onUnmounted(() => {
  window.removeEventListener('online',  () => { isOffline.value = false })
  window.removeEventListener('offline', () => { isOffline.value = true  })
})

const activeModule = ref('Ипотека')
const sidebarCollapsed = ref(false)
const filterDrawerOpen = ref(false)
const scoringFilterDrawerOpen = ref(false)
const docPackagesFilterOpen = ref(false)
const docPackagePreviewOpen = ref(false)
const selectedDocPackage = ref(null)
const registrationFilterOpen = ref(false)
const registrationDrawerOpen = ref(false)
const selectedRegistration = ref(null)
const digitalSignaturesFilterOpen = ref(false)
const digitalSignaturesDrawerOpen = ref(false)
const selectedDigitalSignature = ref(null)
const openDigitalSignaturesPreview = (item) => { selectedDigitalSignature.value = item; digitalSignaturesDrawerOpen.value = true }
const digitalSignaturesHistory = [
  { text: 'Заявка создана',      time: '5 дней назад' },
  { text: 'Данные проверены',    time: '4 дня назад'  },
  { text: 'Выпуск одобрен',      time: '2 дня назад'  },
  { text: 'Подпись активирована', time: '1 час назад'  },
]
const digitalSignaturesDrawerRows = (item) => [
  { label: 'ID',              value: item.id,             copy: item.id },
  { label: 'Телефон',         value: item.fullPhone,      copy: item.fullPhone },
  { label: 'Центр',           value: item.center },
  { label: 'Способ подписи',  value: item.signMethod },
  { label: 'Дата активации',  value: item.activationDate },
  { label: 'Дата истечения',  value: item.expiryDate },
  { label: 'Статус',          value: item.status },
  { label: 'Менеджер',        value: item.manager, options: ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей'] },
]

const insuranceFilterOpen = ref(false)
const insuranceDrawerOpen = ref(false)
const selectedInsurance = ref(null)
const openInsurancePreview = (item) => { selectedInsurance.value = item; insuranceDrawerOpen.value = true }
const insuranceHistory = [
  { text: 'Расчет получен',  time: '5 дней назад' },
  { text: 'Заявление готово', time: '3 дня назад' },
  { text: 'Полис оплачен',   time: '1 день назад' },
  { text: 'Полис получен',   time: '2 часа назад' },
]
const insuranceDrawerRows = (item) => [
  { label: 'ID',              value: item.id,           copy: item.id },
  { label: 'Телефон',         value: item.fullPhone,    copy: item.fullPhone },
  { label: 'Компания',        value: item.company },
  { label: 'Страховая',       value: item.insurer },
  { label: 'Тип страховки',   value: item.insuranceType },
  { label: 'Цена',            value: item.price },
  { label: 'Статус',          value: item.status },
  { label: 'Менеджер',        value: item.manager, options: managerSelectOptions },
]
const openRegistrationPreview = (item) => { selectedRegistration.value = item; registrationDrawerOpen.value = true }
const registrationHistory = [
  { text: 'Паспорт сделки готов',   time: '5 дней назад' },
  { text: 'Подписи выпущены',       time: '4 дня назад' },
  { text: 'Документы готовы',       time: '2 дня назад' },
  { text: 'Заявления подписаны',    time: '2 часа назад' },
  { text: 'Пошлина оплачена',       time: '30 минут назад' },
  { text: 'Зарегистрировано',       time: '20 минут назад' },
]
const registrationDrawerRows = (item) => [
  { label: 'ID',              value: item.id,           copy: item.id },
  { label: 'Телефон',         value: item.fullPhone, copy: item.fullPhone },
  { label: 'Компания',        value: item.company },
  { label: 'Проект',          value: item.project },
  { label: 'Тип недвижимости', value: item.propertyType },
  { label: 'Адрес объекта',   value: item.address },
  { label: 'Статус',          value: item.status },
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
const drawerOpen = ref(false)
const selectedApp = ref(null)
const openDrawer = (app) => { selectedApp.value = app; drawerOpen.value = true }
const openApp = (app) => { window.open(`/app-page.html?id=${app.id}`, '_blank') }

const scoringDrawerOpen = ref(false)
const selectedScoringApp = ref(null)
const openScoringDrawer = (item) => { selectedScoringApp.value = item; scoringDrawerOpen.value = true }


const managerSelectOptions = ['Я', 'Смирнова Юлия', 'Орлов Дмитрий', 'Лебедев Игорь', 'Воронова Анна', 'Морозов Сергей']
const mortgageManagerSelections = reactive({})

const drawerRows = (app) => [
  { label: 'ID', value: app.id, copy: app.id },
  { label: 'Телефон', value: app.fullPhone, copy: app.fullPhone },
  ...(app.mortgageType ? [{ label: 'Тип ипотеки', value: app.mortgageType }] : []),
  { label: 'Проект', value: app.complex },
  { label: 'Сумма кредита', value: app.amount },
  { label: 'Стоимость', value: app.complexPrice },
  ...(app.pv ? [{ label: 'ПВ', value: app.pv }] : []),
  ...(app.term ? [{ label: 'Срок', value: app.term }] : []),
  { label: 'Менеджер', value: 'Смирнова Юлия', options: managerSelectOptions },
]

// Ширина колонки статуса по самому широкому значению (7px/символ + padding badge + padding ячейки)
const statusColWidth = computed(() => {
  const maxLen = applications.reduce((max, app) =>
    app.status && app.status.length > max ? app.status.length : max, 0)
  return `${maxLen * 7 + 24}px`
})

const statusFilterItems = new Set(['Ожидает решения', 'Одобрено', 'Отказано', 'Кредит выдан'])

const filteredApplications = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  const statusFilter = statusFilterItems.has(activeSubItem.value) ? activeSubItem.value : null

  return applications.filter(app => {
    if (statusFilter && app.status !== statusFilter) return false
    if (!q) return true
    return (
      app.id.toLowerCase().includes(q) ||
      app.fullName.toLowerCase().includes(q) ||
      app.fullPhone.replace(/\s/g, '').includes(q.replace(/\s/g, '')) ||
      app.phone.replace(/\s/g, '').includes(q.replace(/\s/g, ''))
    )
  })
})

const checkedRows = ref(new Set())
const toggleRow = (id) => {
  const s = new Set(checkedRows.value)
  s.has(id) ? s.delete(id) : s.add(id)
  checkedRows.value = s
}
watch(activeModule, () => { checkedRows.value = new Set() })

const statusBadgeClass = (status) => {
  const base = 'inline-flex items-center rounded-md px-1.5 py-1 text-[12px] leading-[16px] font-normal inset-ring'
  if (status === 'Одобрено')          return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'Кредит выдан')      return `${base} bg-green-50 text-green-700 inset-ring-green-600/20`
  if (status === 'Ожидает решения')   return `${base} bg-amber-50 text-amber-700 inset-ring-amber-600/20`
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
      { name: 'Одобрено',          href: '#', current: false, count: 28    },
      { name: 'Отказано',          href: '#', current: false               },
      { name: 'Кредит выдан',      href: '#', current: false, count: 5     },
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
    name: 'Цифровые подписи',
    href: '#',
    icon: DocumentCheckIcon,
    current: false,
    children: [
      { name: 'Все заявки',  href: '#', current: false, count: 10 },
      { name: 'Мои заявки', href: '#', current: false, count: 3  },
    ],
  },
]



const mortgageModules = new Set(['Ипотека', 'Скоринг', 'Страховка'])
const filteredNavigation = computed(() =>
  props.mode === 'mortgage' ? navigation.filter(item => mortgageModules.has(item.name)) : navigation
)

const secondaryNavigation = [
  { name: 'Калькулятор ипотеки', href: '#', initial: 'К' },
  { name: 'Аналитика',           href: '#', initial: 'А' },
]

const applications = [
  {
    id: 'ИП 1257847', date: '05.05',
    status: 'Новая заявка',
    initials: 'НЕ', client: 'Новикова Е.Д.', phone: '+7 925 ***-**-31',
    fullName: 'Новикова Екатерина Дмитриевна', fullPhone: '+7 925 413-87-31',
    complex: 'Самолет/Новые Ватутинки', complexPrice: '7 200 000 ₽',
    amount: '5 400 000 ₽', pv: '25%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная ипотека',
    createdAt: '05.05.2026', updatedAt: '05.05.2026 10:14',
  },
  {
    id: 'ИП 1257846', date: '05.05',
    status: 'Консультация',
    initials: 'НЕ', client: 'Новикова Е.Д.', phone: '+7 925 ***-**-31',
    fullName: 'Новикова Екатерина Дмитриевна', fullPhone: '+7 925 413-87-31',
    complex: 'А101/Прокшино', complexPrice: '6 800 000 ₽',
    amount: '4 760 000 ₽', pv: '30%', term: '18 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная ипотека',
    createdAt: '05.05.2026', updatedAt: '05.05.2026 09:02',
  },
  {
    id: 'ИП 1257845', date: '04.05',
    status: 'Подготовка',
    initials: 'ОД', client: 'Орлов Д.П.', phone: '+7 912 ***-**-07',
    fullName: 'Орлов Дмитрий Павлович', fullPhone: '+7 912 635-20-07',
    complex: 'MR Group/Савёловский Сити', complexPrice: '18 500 000 ₽',
    amount: '13 875 000 ₽', pv: '25%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ-ипотека',
    createdAt: '04.05.2026', updatedAt: '04.05.2026 17:48',
  },
  {
    id: 'ИП 1257844', date: '04.05',
    status: 'Ожидает решения',
    initials: 'ВА', client: 'Воронова А.С.', phone: '+7 926 ***-**-64',
    fullName: 'Воронова Анна Сергеевна', fullPhone: '+7 926 184-53-64',
    complex: 'ФСК/Южная Битца', complexPrice: '10 100 000 ₽',
    amount: '7 070 000 ₽', pv: '30%', term: '22 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '04.05.2026', updatedAt: '04.05.2026 15:30',
  },
  {
    id: 'ИП 1257843', date: '03.05',
    status: 'Одобрено',
    initials: 'ЛИ', client: 'Лебедев И.Н.', phone: '+7 967 ***-**-19',
    fullName: 'Лебедев Игорь Николаевич', fullPhone: '+7 967 302-58-19',
    complex: 'ЛСР/Морская набережная (СПб)', complexPrice: '9 600 000 ₽',
    amount: '6 720 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная ипотека',
    createdAt: '03.05.2026', updatedAt: '04.05.2026 11:22',
  },
  {
    id: 'ИП 1257842', date: '03.05',
    status: 'Консультация',
    initials: 'ЛИ', client: 'Лебедев И.Н.', phone: '+7 967 ***-**-19',
    fullName: 'Лебедев Игорь Николаевич', fullPhone: '+7 967 302-58-19',
    complex: 'Группа ЛСР/Зенит (СПб)', complexPrice: '8 200 000 ₽',
    amount: '5 740 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная ипотека',
    createdAt: '03.05.2026', updatedAt: '03.05.2026 14:07',
  },
  {
    id: 'ИП 1257841', date: '03.05',
    status: 'Подготовка',
    initials: 'СЮ', client: 'Смирнова Ю.А.', phone: '+7 918 ***-**-56',
    fullName: 'Смирнова Юлия Андреевна', fullPhone: '+7 918 749-61-56',
    complex: 'Донстрой/Символ', complexPrice: '22 000 000 ₽',
    amount: '15 400 000 ₽', pv: '30%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ-ипотека',
    createdAt: '03.05.2026', updatedAt: '03.05.2026 12:45',
  },
  {
    id: 'ИП 1257840', date: '02.05',
    status: 'Ожидает решения',
    initials: 'МС', client: 'Морозов С.В.', phone: '+7 963 ***-**-93',
    fullName: 'Морозов Сергей Владимирович', fullPhone: '+7 963 827-44-93',
    complex: 'ПИК/Люберцы Парк', complexPrice: '6 300 000 ₽',
    amount: '4 410 000 ₽', pv: '30%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '02.05.2026', updatedAt: '02.05.2026 18:55',
  },
  {
    id: 'ИП 1257839', date: '02.05',
    status: 'Консультация',
    initials: 'ФИ', client: 'Фёдорова И.К.', phone: '+7 931 ***-**-27',
    fullName: 'Фёдорова Ирина Константиновна', fullPhone: '+7 931 560-39-27',
    complex: 'Самолет/Пригород Лесное', complexPrice: '5 900 000 ₽',
    amount: '4 130 000 ₽', pv: '30%', term: '17 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная ипотека',
    createdAt: '02.05.2026', updatedAt: '02.05.2026 13:10',
  },
  {
    id: 'ИП 1257838', date: '01.05',
    status: 'Новая заявка',
    initials: 'АР', client: 'Андреев Р.Е.', phone: '+7 909 ***-**-74',
    fullName: 'Андреев Роман Евгеньевич', fullPhone: '+7 909 213-76-74',
    complex: 'А101/Испанские кварталы', complexPrice: '8 400 000 ₽',
    amount: null, pv: null, term: null,
    houseType: null, mortgageType: null,
    createdAt: '01.05.2026', updatedAt: '01.05.2026 20:31',
  },
  {
    id: 'ИП 1257837', date: '01.05',
    status: 'Подготовка',
    initials: 'КМ', client: 'Козлова М.П.', phone: '+7 977 ***-**-48',
    fullName: 'Козлова Мария Петровна', fullPhone: '+7 977 094-82-48',
    complex: 'ЛСР/Цивилизация (СПб)', complexPrice: '11 200 000 ₽',
    amount: '7 840 000 ₽', pv: '30%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная ипотека',
    createdAt: '01.05.2026', updatedAt: '01.05.2026 16:44',
  },
  {
    id: 'ИП 1257836', date: '30.04',
    status: 'Ожидает решения',
    initials: 'ВА', client: 'Васильев А.Ю.', phone: '+7 915 ***-**-62',
    fullName: 'Васильев Александр Юрьевич', fullPhone: '+7 915 371-05-62',
    complex: 'Эталон/Галактика', complexPrice: '14 600 000 ₽',
    amount: '10 220 000 ₽', pv: '30%', term: '28 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '30.04.2026', updatedAt: '30.04.2026 19:20',
  },
  {
    id: 'ИП 1257835', date: '30.04',
    status: 'Консультация',
    initials: 'ПН', client: 'Павлова Н.О.', phone: '+7 936 ***-**-85',
    fullName: 'Павлова Надежда Олеговна', fullPhone: '+7 936 458-17-85',
    complex: 'Самолет/Жилой район Южный', complexPrice: '5 500 000 ₽',
    amount: '3 850 000 ₽', pv: '30%', term: '15 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная ипотека',
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
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '29.04.2026', updatedAt: '29.04.2026 18:37',
  },
  {
    id: 'ИП 1257832', date: '29.04',
    status: 'Консультация',
    initials: 'НА', client: 'Никитин А.С.', phone: '+7 945 ***-**-91',
    fullName: 'Никитин Артём Сергеевич', fullPhone: '+7 945 219-64-91',
    complex: 'Донстрой/Остров', complexPrice: '19 000 000 ₽',
    amount: '13 300 000 ₽', pv: '30%', term: '30 лет',
    houseType: 'Первичное жильё', mortgageType: 'ИТ-ипотека',
    createdAt: '29.04.2026', updatedAt: '29.04.2026 17:14',
  },
  {
    id: 'ИП 1257831', date: '29.04',
    status: 'Консультация',
    initials: 'MM', client: 'Могиль М.В.', phone: '+7 901 ***-**-19',
    fullName: 'Могиль Михаил Валерьевич', fullPhone: '+7 901 234-56-19',
    complex: 'DOGMA/Самолет (Краснодар)', complexPrice: '5 000 000 ₽',
    amount: '3 750 000 ₽', pv: '25%', term: '16 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
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
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '28.04.2026', updatedAt: '28.04.2026 14:22',
  },
  {
    id: 'ИП 1257828', date: '28.04',
    status: 'Ожидает решения',
    initials: 'ЗА', client: 'Захаров А.И.', phone: '+7 916 ***-**-42',
    fullName: 'Захаров Алексей Игоревич', fullPhone: '+7 916 548-23-42',
    complex: 'ПИК/Береговой (Москва)', complexPrice: '12 400 000 ₽',
    amount: '9 300 000 ₽', pv: '25%', term: '25 лет',
    houseType: 'Первичное жильё', mortgageType: 'Льготная ипотека',
    createdAt: '28.04.2026', updatedAt: '28.04.2026 09:10',
  },
  {
    id: 'ИП 1257826', date: '28.04',
    status: 'Ожидает решения',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '28.04.2026', updatedAt: '29.04.2026 10:30',
  },
  {
    id: 'ИП 1257825', date: '27.04',
    status: 'Новая заявка',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '27.04.2026', updatedAt: '27.04.2026 17:05',
  },
  {
    id: 'ИП 1257824', date: '27.04',
    status: 'Консультация',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '27.04.2026', updatedAt: '27.04.2026 15:48',
  },
  {
    id: 'ИП 1257823', date: '27.04',
    status: 'Ожидает решения',
    initials: 'КТ', client: 'Кривцова Т.С.', phone: '+7 985 ***-**-67',
    fullName: 'Кривцова Татьяна Сергеевна', fullPhone: '+7 985 321-09-67',
    complex: 'Самолет/Путилково', complexPrice: '9 000 000 ₽',
    amount: '4 000 000 ₽', pv: '56%', term: '11 лет',
    houseType: 'Первичное жильё', mortgageType: 'Стандартная ипотека',
    createdAt: '27.04.2026', updatedAt: '29.04.2026 11:12',
  },
  {
    id: 'ИП 1257827', date: '27.04',
    status: 'Подготовка',
    initials: 'СП', client: 'Соколова П.В.', phone: '+7 903 ***-**-88',
    fullName: 'Соколова Полина Вячеславовна', fullPhone: '+7 903 762-14-88',
    complex: 'Эталон/Серебряный бор', complexPrice: '8 750 000 ₽',
    amount: '6 125 000 ₽', pv: '30%', term: '20 лет',
    houseType: 'Первичное жильё', mortgageType: 'Семейная ипотека',
    createdAt: '27.04.2026', updatedAt: '28.04.2026 08:55',
  },
]

applications.forEach(app => { mortgageManagerSelections[app.id] = 'Смирнова Юлия' })
watch(filteredApplications, v => { if (activeModule.value === 'Ипотека') activeCount.value = v.length }, { immediate: true })
</script>

<style>
svg.lucide { stroke-width: 1.5; }
</style>
