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
        <div class="flex min-h-full flex-col items-center pt-[20vh] px-4 pb-8">
          <TransitionChild
            as="template"
            enter="ease-out duration-200" enter-from="opacity-0 scale-95" enter-to="opacity-100 scale-100"
            leave="ease-in duration-150" leave-from="opacity-100 scale-100" leave-to="opacity-0 scale-95"
          >
          <div class="w-full max-w-lg flex flex-col">
            <DialogPanel class="relative w-full">
              <div class="rounded-2xl bg-white shadow-xl min-h-[220px]">

              <!-- Header -->
              <div class="flex items-center justify-between px-6 pt-6 pb-4">
                <DialogTitle class="text-[18px] leading-[28px] font-semibold text-zinc-900">
                  {{ title }}
                </DialogTitle>
                <button
                  @click="$emit('close')"
                  class="flex size-6 items-center justify-center text-zinc-500 hover:text-zinc-900 transition-colors"
                >
                  <XIcon :size="20" />
                </button>
              </div>

              <!-- Body: Пакетная регистрация -->
              <div v-if="module === 'Пакетная регистрация'" class="px-6 py-6 flex flex-col gap-6">
                <div class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-zinc-900">Название</label>
                  <input
                    v-model="packageName"
                    type="text"
                    placeholder=""
                    class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                    @input="selectedJkBadge = null"
                  />
                  <div class="mt-3 flex flex-wrap gap-2">
                    <button
                      v-for="jk in jkPresets"
                      :key="jk"
                      type="button"
                      @click="selectedJkBadge === jk ? (selectedJkBadge = null, packageName = '') : (selectedJkBadge = jk, packageName = jk)"
                      :class="selectedJkBadge === jk
                        ? 'rounded-full bg-indigo-50 px-2.5 py-1 text-xs font-medium text-indigo-600 shadow-xs hover:bg-indigo-50'
                        : 'rounded-full bg-white px-2.5 py-1 text-xs font-medium text-zinc-900 shadow-xs inset-ring inset-ring-gray-300 hover:bg-gray-50'"
                    >{{ jk }}</button>
                  </div>
                </div>
              </div>

              <!-- Body: Пакеты документов -->
              <div v-else-if="module === 'Пакеты документов'" class="px-6 py-6 flex flex-col gap-6">

                <!-- Название пакета документов -->
                <div>
                  <label class="block text-sm/6 font-medium text-zinc-900">
                    Название
                  </label>
                  <div class="mt-2">
                    <input
                      v-model="docPackageName"
                      type="text"
                      placeholder=""
                      class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                    />
                  </div>
                </div>

                <!-- Чекбокс юр. лица -->
                <div class="flex gap-3">
                  <div class="flex h-6 shrink-0 items-center">
                    <div class="group grid size-4 grid-cols-1">
                      <input
                        id="add-legal-entity"
                        type="checkbox"
                        v-model="addLegalEntity"
                        @change="if (!addLegalEntity) { selectedLegalEntity = ''; selectedSigner = '' }"
                        class="col-start-1 row-start-1 appearance-none rounded-sm border border-gray-300 bg-white checked:border-indigo-600 checked:bg-indigo-600 indeterminate:border-indigo-600 indeterminate:bg-indigo-600 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600 disabled:border-gray-300 disabled:bg-gray-100 disabled:checked:bg-gray-100 forced-colors:appearance-auto"
                      />
                      <svg class="pointer-events-none col-start-1 row-start-1 size-3.5 self-center justify-self-center stroke-white group-has-disabled:stroke-gray-950/25" viewBox="0 0 14 14" fill="none">
                        <path class="opacity-0 group-has-checked:opacity-100" d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                        <path class="opacity-0 group-has-indeterminate:opacity-100" d="M3 7H11" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                      </svg>
                    </div>
                  </div>
                  <div class="text-sm/6">
                    <label for="add-legal-entity" class="font-medium text-zinc-900 cursor-pointer">Добавить юридическое лицо из справочника</label>
                  </div>
                </div>

                <!-- Юридическое лицо -->
                <AppCombobox
                  v-if="addLegalEntity"
                  v-model="selectedLegalEntity"
                  :options="legalEntities"
                  label="Юридическое лицо"
                  placeholder="Выберите организацию"
                />

                <!-- Подписант -->
                <AppCombobox
                  v-if="addLegalEntity && selectedLegalEntity"
                  v-model="selectedSigner"
                  :options="signers"
                  label="Подписант"
                  placeholder="Выберите подписанта"
                />

              </div>

              <!-- Body: Цифровые подписи / Ипотека / Страховка / Скоринг -->
              <div v-else-if="['Цифровые подписи', 'Ипотека', 'Страховка', 'Скоринг', 'Регистрация'].includes(module)" class="px-6 py-6 flex flex-col gap-6">

                <!-- Тип покупателя (только Регистрация) -->
                <div v-if="module === 'Регистрация'" class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-zinc-900">Тип покупателя</label>
                  <CatalystListbox
                    :options="buyerTypes"
                    v-model="buyerType"
                    placeholder="Выберите тип"
                  />
                </div>

                <!-- Карточка юридического лица -->
                <template v-if="module === 'Регистрация' && buyerType === 'Юридическое лицо'">

                  <!-- Состояние: поиск -->
                  <div v-if="!selectedCompany" class="rounded-xl border border-zinc-200 px-4 py-4">
                    <div class="flex items-center gap-3">
                      <span class="flex size-10 shrink-0 items-center justify-center rounded-xl bg-zinc-100">
                        <Building2Icon :size="20" class="text-zinc-400" />
                      </span>
                      <span class="text-[14px] font-semibold text-zinc-900">Юридическое лицо</span>
                    </div>
                    <div class="relative mt-3">
                      <SearchIcon :size="16" class="pointer-events-none absolute left-3 top-1/2 -translate-y-1/2 text-zinc-400" />
                      <input
                        v-model="companySearch"
                        type="text"
                        placeholder="Поиск по ИНН или названию"
                        class="w-full rounded-lg border border-[#e4e4e7] bg-white pl-9 pr-3 py-[6px] text-[14px] font-normal leading-[20px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                      />
                      <div v-if="companySearch && filteredCompanies.length" class="absolute z-10 mt-1 w-full overflow-hidden rounded-xl bg-white shadow-lg ring-1 ring-zinc-200">
                        <button
                          v-for="c in filteredCompanies"
                          :key="c.inn"
                          type="button"
                          class="w-full px-4 py-3 text-left transition-colors hover:bg-zinc-50"
                          @mousedown.prevent="selectedCompany = c; companySearch = ''"
                        >
                          <p class="text-[14px] font-medium text-zinc-900">{{ c.name }}</p>
                          <p class="text-[12px] text-zinc-500">ИНН {{ c.inn }}</p>
                        </button>
                      </div>
                    </div>
                  </div>

                  <!-- Состояние: компания выбрана -->
                  <div v-else class="rounded-xl border border-zinc-200 px-4 py-4">
                    <div class="flex items-center gap-3">
                      <span class="flex size-10 shrink-0 items-center justify-center rounded-xl bg-zinc-100">
                        <Building2Icon :size="20" class="text-zinc-400" />
                      </span>
                      <span class="flex-1 text-[14px] font-semibold text-zinc-900">{{ selectedCompany.name }}</span>
                      <button
                        type="button"
                        class="relative isolate inline-flex cursor-default items-center justify-center size-10 rounded-xl text-indigo-600 hover:bg-zinc-950/5 active:bg-zinc-950/10 transition-colors"
                        @click="selectedCompany = null"
                      >
                        <Trash2Icon :size="16" />
                      </button>
                    </div>
                    <div class="mt-3">
                      <p class="text-[14px] leading-[20px] text-zinc-900">{{ selectedCompany.description }}</p>
                      <p class="mt-1 text-[12px] leading-[16px] text-zinc-500">ИНН {{ selectedCompany.inn }} • {{ selectedCompany.address }}</p>
                    </div>
                  </div>

                  <!-- Проект (юрлицо, когда компания выбрана) -->
                  <div v-if="selectedCompany" class="flex flex-col gap-[4px]">
                    <label class="text-[14px] font-medium leading-[20px] text-zinc-900">Проект</label>
                    <CatalystListbox :options="projects" v-model="project" placeholder="Выберите проект" />
                  </div>

                </template>

                <!-- Телефон + статус профиля (не юрлицо) -->
                <template v-if="module !== 'Регистрация' || buyerType !== 'Юридическое лицо'">

                  <!-- ФИО ребенка (только Ребенок до 14 лет) -->
                  <div v-if="module === 'Регистрация' && buyerType === 'Ребенок до 14 лет'" class="flex flex-col gap-[4px]">
                    <label v-if="childNameMode === 'single'" class="text-[14px] font-medium leading-[20px] text-zinc-900">ФИО ребенка</label>

                    <div v-if="childNameMode === 'single'" class="relative">
                      <input
                        v-model="childFullName"
                        type="text"
                        placeholder=""
                        class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                        @focus="showChildNameDropdown = true"
                        @blur="setTimeout(() => showChildNameDropdown = false, 150)"
                      />
                      <div v-if="showChildNameDropdown" class="absolute z-10 mt-1 w-full rounded-xl bg-white shadow-lg ring-1 ring-zinc-200">
                        <button
                          type="button"
                          class="w-full px-4 py-3 text-left text-[14px] text-zinc-900 hover:bg-zinc-50 rounded-xl transition-colors"
                          @mousedown.prevent="childNameMode = 'expanded'; showChildNameDropdown = false; childFullName = ''"
                        >Заполнить вручную</button>
                      </div>
                    </div>

                    <div v-else class="rounded-xl bg-zinc-50 px-4 pt-4 pb-5">
                      <div class="grid grid-cols-2 gap-3">
                        <div class="flex flex-col gap-[4px]">
                          <label class="text-[14px] font-normal text-zinc-900">Фамилия</label>
                          <input v-model="childLastName" type="text" class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition" />
                        </div>
                        <div class="flex flex-col gap-[4px]">
                          <label class="text-[14px] font-normal text-zinc-900">Имя</label>
                          <input v-model="childFirstName" type="text" class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition" />
                        </div>
                        <div class="flex flex-col gap-[4px]">
                          <label class="text-[14px] font-normal text-zinc-900">Отчество (при наличии)</label>
                          <input v-model="childMiddleName" type="text" class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition" />
                        </div>
                        <div class="flex items-end justify-end">
                          <button
                            type="button"
                            class="text-[14px] text-indigo-600 hover:text-indigo-700 transition-colors"
                            @click="childNameMode = 'single'; childFullName = [childLastName, childFirstName, childMiddleName].filter(Boolean).join(' ')"
                          >Свернуть</button>
                        </div>
                      </div>
                    </div>
                  </div>

                  <!-- Divider после ФИО ребенка -->
                  <hr v-if="module === 'Регистрация' && buyerType === 'Ребенок до 14 лет'" role="presentation" class="w-full border-t border-zinc-950/10" />

                  <div class="flex items-end gap-4">
                    <!-- Поле телефона с лейблом -->
                    <div class="flex flex-col gap-[4px] flex-1">
                      <label class="text-[14px] font-medium leading-[20px] text-zinc-900">{{ module === 'Регистрация' && buyerType === 'Ребенок до 14 лет' ? 'Телефон представителя' : 'Телефон' }}</label>
                      <input
                        v-model="phone"
                        type="tel"
                        placeholder="+7 (967) 890-45-34"
                        class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                        @input="phoneStatus = null; fullName = ''"
                      />
                    </div>
                    <!-- Статус профиля -->
                    <div class="flex-1">
                      <div v-if="!phone.trim() && module === 'Регистрация' && buyerType === 'Ребенок до 14 лет'">
                        <p class="text-[12px] leading-[16px] text-zinc-900">Контакты представителя</p>
                        <p class="mt-0.5 text-[12px] leading-[16px] text-zinc-500">Родитель, опекун или другой законный представитель</p>
                      </div>
                      <div v-else-if="!phone.trim()">
                        <p class="text-[12px] leading-[16px] text-zinc-900">Начните с номера</p>
                        <p class="mt-0.5 text-[12px] leading-[16px] text-zinc-500">Возможно, профайл уже есть в системе</p>
                      </div>
                      <div v-else-if="phoneStatus === 'existing'" class="flex items-center gap-3">
                        <span class="inline-flex size-9 shrink-0 items-center justify-center rounded-xl bg-indigo-50 text-[14px] font-semibold text-indigo-600">ЕА</span>
                        <div>
                          <p class="text-[12px] leading-[16px] text-zinc-500">Движ ID есть в системе</p>
                          <p class="text-[14px] leading-[20px] font-medium text-zinc-900">Е...в Е. А.</p>
                        </div>
                      </div>
                      <div v-else-if="phoneStatus === 'new'">
                        <p class="text-[12px] leading-[16px] text-zinc-900">Новый профайл</p>
                        <p class="mt-0.5 text-[12px] leading-[16px] font-normal text-zinc-500">На этот номер анкета еще не создавалась</p>
                      </div>
                    </div>
                  </div>

                  <!-- ФИО -->
                  <div v-if="phoneStatus !== null" class="flex flex-col gap-[4px]">
                    <label v-if="nameMode === 'single'" class="text-[14px] font-medium leading-[20px] text-zinc-900">{{ module === 'Регистрация' && buyerType === 'Ребенок до 14 лет' ? 'ФИО представителя' : 'ФИО' }}</label>

                    <div v-if="nameMode === 'single'" class="relative">
                      <input
                        v-model="fullName"
                        type="text"
                        placeholder=""
                        :disabled="phoneStatus === 'existing' || phoneStatus === null"
                        :class="['w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition', (phoneStatus === 'existing' || phoneStatus === null) ? 'text-[#a1a1aa] cursor-not-allowed bg-zinc-50' : 'text-zinc-900']"
                        @focus="phoneStatus === 'new' && (showNameDropdown = true)"
                        @blur="setTimeout(() => showNameDropdown = false, 150)"
                      />
                      <div v-if="showNameDropdown" class="absolute z-10 mt-1 w-full rounded-xl bg-white shadow-lg ring-1 ring-zinc-200">
                        <button
                          type="button"
                          class="w-full px-4 py-3 text-left text-[14px] text-zinc-900 hover:bg-zinc-50 rounded-xl transition-colors"
                          @mousedown.prevent="nameMode = 'expanded'; showNameDropdown = false; fullName = ''"
                        >Заполнить вручную</button>
                      </div>
                    </div>

                    <div v-else class="rounded-xl bg-zinc-50 px-4 pt-4 pb-5">
                      <div class="grid grid-cols-2 gap-3">
                        <div class="flex flex-col gap-[4px]">
                          <label class="text-[14px] font-normal text-zinc-900">Фамилия</label>
                          <input v-model="lastName" type="text" class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition" />
                        </div>
                        <div class="flex flex-col gap-[4px]">
                          <label class="text-[14px] font-normal text-zinc-900">Имя</label>
                          <input v-model="firstName" type="text" class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition" />
                        </div>
                        <div class="flex flex-col gap-[4px]">
                          <label class="text-[14px] font-normal text-zinc-900">Отчество (при наличии)</label>
                          <input v-model="middleName" type="text" class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition" />
                        </div>
                        <div class="flex items-end justify-end">
                          <button
                            type="button"
                            class="text-[14px] text-indigo-600 hover:text-indigo-700 transition-colors"
                            @click="nameMode = 'single'; fullName = [lastName, firstName, middleName].filter(Boolean).join(' ')"
                          >Свернуть</button>
                        </div>
                      </div>
                    </div>
                  </div>

                  <!-- Проект (Ипотека + Регистрация) -->
                  <div v-if="phoneStatus !== null && ['Ипотека', 'Регистрация'].includes(module)" class="flex flex-col gap-[4px]">
                    <label class="text-[14px] font-medium leading-[20px] text-zinc-900">Проект</label>
                    <CatalystListbox :options="projects" v-model="project" placeholder="Выберите проект" />
                  </div>

                </template>


              </div>

              <!-- Body: остальные модули -->
              <div v-else class="px-6 py-6 flex flex-col gap-6">
                <!-- ФИО -->
                <div class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-zinc-900">ФИО</label>
                  <input
                    v-model="fullName"
                    type="text"
                    placeholder="Иванов Иван Иванович"
                    class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                  />
                </div>

                <!-- Телефон -->
                <div class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-zinc-900">Телефон</label>
                  <input
                    v-model="phone"
                    type="tel"
                    placeholder="+7 (967) 890-45-34"
                    class="w-full rounded-lg border border-[#e4e4e7] bg-white px-3 py-[6px] text-[14px] font-normal leading-[20px] text-zinc-900 placeholder:text-[#a1a1aa] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
                  />
                </div>

                <!-- Проект -->
                <div class="flex flex-col gap-[4px]">
                  <label class="text-[14px] font-medium leading-[20px] text-zinc-900">Проект</label>
                  <div class="relative">
                    <select
                      v-model="project"
                      class="w-full appearance-none rounded-lg border border-[#e4e4e7] bg-white px-3 py-2 pr-8 text-[14px] font-normal leading-[20px] shadow-[0px_1px_2px_rgba(0,0,0,0.05)] focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition cursor-pointer"
                      :class="project ? 'text-zinc-900' : 'text-[#a1a1aa]'"
                    >
                      <option value="" disabled>Выберите проект</option>
                      <option v-for="p in projects" :key="p" :value="p">{{ p }}</option>
                    </select>
                    <ChevronDownIcon :size="16" class="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 text-zinc-900" />
                  </div>
                </div>
              </div>

              <!-- Footer -->
              <div class="flex items-center justify-end gap-3 px-6 py-4">
                <button
                  @click="$emit('close')"
                  class="inline-flex h-9 items-center justify-center rounded-lg border border-zinc-950/10 bg-white px-4 text-[14px] font-medium text-[#09090b] shadow-xs hover:bg-zinc-50 active:bg-zinc-100 transition-colors"
                >
                  Отмена
                </button>
                <button
                  :disabled="creating || (module === 'Пакетная регистрация' ? !packageName.trim() : module === 'Пакеты документов' ? (!docPackageName.trim() || (addLegalEntity && (!selectedLegalEntity || !selectedSigner))) : (module === 'Регистрация' && buyerType === 'Юридическое лицо') ? (!selectedCompany || !project) : ['Цифровые подписи', 'Ипотека', 'Страховка', 'Скоринг', 'Регистрация'].includes(module) ? (phoneStatus === null || (phoneStatus === 'new' && (nameMode === 'single' ? !fullName.trim() : (!lastName.trim() || !firstName.trim()))) || (module === 'Регистрация' && !project)) : !phone.trim())"
                  class="inline-flex h-9 items-center justify-center gap-2 rounded-lg bg-indigo-600 px-4 text-[14px] font-semibold text-white shadow-xs transition-colors disabled:opacity-40 disabled:cursor-not-allowed enabled:hover:bg-indigo-700 enabled:active:bg-indigo-800"
                  @click="handleCreate"
                >
                  <svg v-if="creating" class="size-4 animate-spin" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"/>
                  </svg>
                  Создать
                </button>
              </div>

              </div><!-- конец белой карточки -->

              <!-- Тестовые кнопки (только для ЭЦП) -->
              <div v-if="['Цифровые подписи', 'Ипотека', 'Страховка', 'Скоринг', 'Регистрация'].includes(module) && !(module === 'Регистрация' && buyerType === 'Юридическое лицо')" class="mt-3 flex gap-2">
                <button type="button" @click="setExistingProfile" class="rounded-lg bg-white hover:bg-zinc-50 px-3 py-2 text-[13px] font-medium text-zinc-900 transition-colors">Старый профиль</button>
                <button type="button" @click="setNewProfile" class="rounded-lg bg-white hover:bg-zinc-50 px-3 py-2 text-[13px] font-medium text-zinc-900 transition-colors">Новый профиль</button>
              </div>

            </DialogPanel>

          </div>
          </TransitionChild>
        </div>
      </div>

    </Dialog>
  </TransitionRoot>
</template>

<script setup>
import { ref, computed } from 'vue'
import { Dialog, DialogPanel, DialogTitle, TransitionChild, TransitionRoot } from '@headlessui/vue'
import { X as XIcon, Home as HomeIcon, ChartPie as ChartPieIcon, ShieldCheck as ShieldCheckIcon, CircleCheck as CircleCheckIcon, ChevronDown as ChevronDownIcon, Building2 as Building2Icon, Search as SearchIcon, Trash2 as Trash2Icon } from 'lucide-vue-next'
import AppCombobox from './AppCombobox.vue'
import CatalystListbox from './CatalystListbox.vue'

const props = defineProps({
  open: Boolean,
  mode: { type: String, default: 'all' },
  title: { type: String, default: 'Создание заявки' },
  module: { type: String, default: '' },
})
const emit = defineEmits(['close', 'created'])

const selectedType = ref('mortgage')
const fullName = ref('')
const phone = ref('')
const project = ref('')
const packageName = ref('')
const selectedJkBadge = ref(null)
const creating = ref(false)
const phoneStatus = ref(null) // null | 'existing' | 'new'
const buyerType = ref('Взрослый или ребенок с 14 лет')
const companySearch = ref('')
const selectedCompany = ref(null)
const childNameMode = ref('single')
const showChildNameDropdown = ref(false)
const childFullName = ref('')
const childLastName = ref('')
const childFirstName = ref('')
const childMiddleName = ref('')
const nameMode = ref('single') // 'single' | 'expanded'
const showNameDropdown = ref(false)
const lastName = ref('')
const firstName = ref('')
const middleName = ref('')

const setExistingProfile = () => {
  phone.value = '+7 913 000-00-00'
  phoneStatus.value = 'existing'
  fullName.value = 'Елисеева Елена Александровна'
}

const setNewProfile = () => {
  phone.value = '+7 913 111-22-33'
  phoneStatus.value = 'new'
  fullName.value = ''
}

// Пакеты документов
const docPackageName = ref('')
const addLegalEntity = ref(false)
const selectedLegalEntity = ref('')
const selectedSigner = ref('')

const legalEntities = [
  'Движимость (ООО)', 'СтройГрупп (АО)', 'Сбербанк (ПАО)', 'ВТБ (ПАО)',
  'Альфа-Банк (АО)', 'ДомКлик (ООО)', 'НордСтрой (ООО)', 'Эталон (ПАО)',
]

const signers = [
  'Назаров Леван Давидович', 'Смирнова Юлия Петровна', 'Орлов Дмитрий Сергеевич',
  'Лебедев Игорь Николаевич', 'Воронова Анна Михайловна', 'Морозов Сергей Иванович',
]

const jkPresets = [
  'Вектор', 'Филатов луг', 'Новые Ватутинки', 'Прокшино', 'Символ', 'Люберцы Парк',
]

const handleCreate = () => {
  creating.value = true
  setTimeout(() => {
    const name = props.module === 'Пакеты документов' ? docPackageName.value.trim() : packageName.value.trim()
    emit('created', { module: props.module, name })
    creating.value = false
    packageName.value = ''
    selectedJkBadge.value = null
    fullName.value = ''
    phone.value = ''
    project.value = ''
    docPackageName.value = ''
    addLegalEntity.value = false
    selectedLegalEntity.value = ''
    selectedSigner.value = ''
    phoneStatus.value = null
    buyerType.value = 'Взрослый или ребенок с 14 лет'
    companySearch.value = ''
    selectedCompany.value = null
    childNameMode.value = 'single'
    showChildNameDropdown.value = false
    childFullName.value = ''
    childLastName.value = ''
    childFirstName.value = ''
    childMiddleName.value = ''
    nameMode.value = 'single'
    showNameDropdown.value = false
    lastName.value = ''
    firstName.value = ''
    middleName.value = ''
    emit('close')
  }, 1200)
}

const applicationTypes = [
  { id: 'mortgage', label: 'Ипотека',  icon: HomeIcon },
  { id: 'scoring',  label: 'Скоринг',  icon: ChartPieIcon },
  { id: 'insurance',label: 'Страховка', icon: ShieldCheckIcon },
]

const buyerTypes = ['Взрослый или ребенок с 14 лет', 'Ребенок до 14 лет', 'Юридическое лицо']

const companies = [
  { name: 'ООО «Цифровые продажи»', inn: '7716899078', description: 'Разработка компьютерного программного обеспечения', address: 'г. Москва, ул. Ленинская слобода, д. 19, помещ. 21Ж-1Б' },
  { name: 'ПАО «Сбербанк»', inn: '7707083893', description: 'Банковская деятельность', address: 'г. Москва, ул. Вавилова, д. 19' },
  { name: 'ООО «ДомКлик»', inn: '7725287100', description: 'Деятельность в области информационных технологий', address: 'г. Москва, ул. Кирпичная, д. 31' },
  { name: 'АО «Движимость»', inn: '7701234567', description: 'Управление недвижимым имуществом', address: 'г. Москва, пр-т Мира, д. 45' },
  { name: 'ООО «СтройГрупп»', inn: '7709876543', description: 'Строительство жилых и нежилых зданий', address: 'г. Санкт-Петербург, Невский пр-т, д. 100' },
]

const filteredCompanies = computed(() => {
  const q = companySearch.value.toLowerCase()
  if (!q) return []
  return companies.filter(c => c.name.toLowerCase().includes(q) || c.inn.includes(q))
})

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
