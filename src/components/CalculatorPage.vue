<template>
  <div class="relative flex-1 flex flex-col overflow-hidden">
  <div class="flex-1 overflow-auto px-8 py-6 flex flex-col gap-[52px]">

    <!-- ── Форма параметров ─────────────────────────────── -->
    <div class="flex flex-col gap-4">

      <!-- Основные поля -->
      <div class="grid grid-cols-4 gap-6">

        <!-- Жилой комплекс — Combobox -->
        <ComplexCombobox v-model="complexValue" :options="complexOptions" />

        <!-- Тип кредита -->
        <CatalystListbox label="Тип кредита" v-model="creditType" :options="creditTypeOptions" />

        <!-- Тип ипотеки -->
        <CatalystListbox label="Тип ипотеки" v-model="mortgageType" :options="mortgageTypeOptions" />

        <!-- Срок кредита -->
        <CatalystListbox label="Срок кредита" v-model="loanTerm" :options="loanTermOptions" />

        <!-- Inputs -->
        <CatalystInput label="Стоимость недвижимости" v-model="propertyPrice" suffix="₽" />
        <CatalystInput label="Личные средства"        v-model="ownFunds"      suffix="₽" />
        <DownPaymentField v-model="downPaymentPct" :property-price="propertyPrice" />
        <CatalystInput label="Сумма кредита"          v-model="loanAmount"    suffix="₽" :readonly="true" />

      </div>

      <!-- Toggle: Дополнительные параметры -->
      <div class="flex items-center gap-2">
        <button
          @click="showExtra = !showExtra"
          :class="[
            'relative flex h-5 w-8 shrink-0 items-center rounded-full p-0.5 transition-colors duration-200',
            showExtra ? 'bg-indigo-600' : 'bg-[rgba(222,222,222,0.6)]'
          ]"
        >
          <span
            :class="[
              'block h-4 w-4 rounded-full bg-white shadow transition-transform duration-200',
              showExtra ? 'translate-x-3' : 'translate-x-0'
            ]"
          />
        </button>
        <span class="text-[14px] text-[#32393e]">Дополнительные параметры</span>
      </div>

      <!-- Доп. параметры (раскрываются) -->
      <div v-if="showExtra" class="grid grid-cols-4 gap-6">
        <CatalystListbox label="Тип занятости" v-model="employmentType" :options="employmentTypeOptions" />
        <CatalystListbox label="Вид подтверждения дохода" v-model="incomeProof" :options="incomeProofOptions" />
      </div>

    </div>

    <!-- ── Предложения банков ──────────────────────────────── -->
    <div class="flex flex-col gap-6">

      <!-- Фильтры + кнопка -->
      <div class="flex items-center gap-2">
        <div class="w-44">
          <CatalystListbox v-model="activeFilter" :options="filterTabOptions" />
        </div>
        <div class="w-36">
          <CatalystListbox v-model="bankFilter" :options="bankFilterOptions" />
        </div>
        <div class="flex-1" />
        <button type="button" class="inline-flex h-9 items-center gap-x-1.5 rounded-md bg-indigo-600 px-3.5 text-sm font-semibold text-white shadow-xs hover:bg-indigo-700 active:bg-indigo-800 shrink-0">
          <PlusIcon :size="16" />
          Создать заявку
        </button>
      </div>

      <!-- Таблица -->
      <div class="relative">

      <table class="w-full">
        <thead>
          <tr class="border-b border-zinc-100">
            <th class="w-8 pb-[15px] pt-[13.5px]"></th>
            <th class="pb-[15px] pt-[13.5px] pr-6 text-left text-sm font-medium text-zinc-900">Банк</th>
            <th class="pb-[15px] pt-[13.5px] pr-6 text-left">
              <button @click="sortBy = 'payment'" class="flex items-center gap-1.5 text-sm font-medium text-zinc-900">
                Платеж
                <span class="flex items-center justify-center size-5 bg-zinc-100 rounded-[4px]">
                  <ChevronDownIcon :size="12" class="text-zinc-900" />
                </span>
              </button>
            </th>
            <th class="pb-[15px] pt-[13.5px] pr-6 text-left text-sm font-medium text-zinc-900">Ставка</th>
            <th class="pb-[15px] pt-[13.5px] pr-6 text-left text-sm font-medium text-zinc-900">ПВ</th>
            <th class="pb-[15px] pt-[13.5px] pr-6 text-left text-sm font-medium text-zinc-900">Объект</th>
            <th class="pb-[15px] pt-[13.5px] pr-6 text-left text-sm font-medium text-zinc-900">Кредит</th>
            <th class="pb-[15px] pt-[13.5px] text-left text-sm font-medium text-zinc-900">Переплата</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-zinc-100">
          <tr v-for="bank in banks" :key="bank.name + bank.payment">
            <!-- Checkbox -->
            <td class="py-4 pr-2">
              <span
                :class="[
                  'flex size-4 items-center justify-center rounded border-2 transition-colors cursor-pointer',
                  bank.checked ? 'bg-indigo-600 border-indigo-600' : 'bg-white border-[rgba(222,222,222,0.6)]'
                ]"
                @click="bank.checked = !bank.checked"
              >
                <svg v-if="bank.checked" class="size-3 stroke-white" viewBox="0 0 14 14" fill="none">
                  <path d="M3 8L6 11L11 3.5" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
              </span>
            </td>
            <td class="py-4 pr-6">
              <TableLink weight="medium" @click="emit('open-bank', bank)">{{ bank.name }}</TableLink>
            </td>
            <td class="py-4 pr-6 text-sm font-light text-zinc-900">{{ bank.payment }}</td>
            <td class="py-4 pr-6 text-sm font-light text-zinc-900">{{ bank.rate }}</td>
            <td class="py-4 pr-6 text-sm font-light text-zinc-900">{{ bank.pv }}</td>
            <td class="py-4 pr-6 text-sm font-light text-zinc-900">{{ bank.cost }}</td>
            <td class="py-4 pr-6 text-sm font-light text-zinc-900">{{ bank.credit }}</td>
            <td class="py-4 text-sm font-light text-zinc-900">{{ bank.overpayment }}</td>
          </tr>
        </tbody>
      </table>
      </div><!-- /relative wrapper -->
    </div>

  </div><!-- /overflow-auto -->

  <!-- Action bar — 24px от нижнего края контент-области, по её центру -->
  <Transition
    enter-active-class="transform transition ease-out duration-200"
    enter-from-class="translate-y-2 opacity-0"
    enter-to-class="translate-y-0 opacity-100"
    leave-active-class="transform transition ease-in duration-150"
    leave-from-class="translate-y-0 opacity-100"
    leave-to-class="translate-y-2 opacity-0"
  >
    <div
      v-if="checkedCount > 0"
      class="absolute bottom-6 left-1/2 -translate-x-1/2 z-30 flex items-center gap-x-3 rounded-2xl bg-indigo-600 px-4 py-3 shadow-xl"
    >
      <span class="text-[14px] font-medium text-white whitespace-nowrap">Выбрано {{ checkedCount }}</span>
      <button
        @click="banks.forEach(b => b.checked = false)"
        class="flex items-center gap-x-1.5 text-[14px] font-medium text-indigo-200 hover:text-white transition-colors whitespace-nowrap"
      >
        <XMarkIcon :size="14" />
        Сбросить
      </button>
      <div class="w-px h-5 bg-indigo-400 mx-1" />
      <button class="flex items-center gap-x-2 rounded-lg bg-indigo-500 hover:bg-indigo-400 active:bg-indigo-300 px-3 h-8 text-[14px] font-medium text-white transition-colors whitespace-nowrap">
        <DownloadIcon :size="15" />
        Скачать в PDF
      </button>
    </div>
  </Transition>

</div><!-- /relative outer -->
</template>

<script setup>
import { ref, computed } from 'vue'
import { ChevronDown as ChevronDownIcon, Plus as PlusIcon, X as XMarkIcon, Download as DownloadIcon } from 'lucide-vue-next'
import Button from './Button.vue'
import ComplexCombobox from './ComplexCombobox.vue'
import CatalystListbox from './CatalystListbox.vue'
import CatalystInput from './CatalystInput.vue'
import DownPaymentField from './DownPaymentField.vue'
import TableLink from './TableLink.vue'

const emit = defineEmits(['open-bank'])

const showExtra = ref(false)
const sortBy = ref('payment')
const checkedCount = computed(() => banks.value.filter(b => b.checked).length)

const activeFilter = ref('Все типы')
const filterTabOptions = ['Все типы', 'С субсидией', 'Без субсидии', 'С льготным периодом']

const bankFilter = ref('Все банки')
const bankFilterOptions = ['Все банки', 'Сбербанк', 'Росбанк Дом', 'Тинькофф', 'Альфа-Банк', 'РТС', 'ВТБ', 'МКБ']

const complexValue = ref('Красная поляна')
const complexOptions = [
  'Красная поляна',
  'Северная звезда',
  'Зелёный берег',
  'Солнечный город',
  'Речной квартал',
  'Парк Авеню',
  'Золотые ворота',
]

const creditType = ref('Первичное жильё')
const creditTypeOptions = ['Первичное жильё', 'Вторичное жильё', 'Рефинансирование', 'Строительство дома']

const mortgageType = ref('Семейная ипотека')
const mortgageTypeOptions = ['Семейная ипотека', 'Льготная ипотека', 'Стандартная', 'IT-ипотека', 'Сельская ипотека']

const loanTerm = ref('30 лет')
const loanTermOptions = ['5 лет', '10 лет', '15 лет', '20 лет', '25 лет', '30 лет']

const propertyPrice  = ref('5 000 000')
const ownFunds       = ref('1 500 000')
const loanAmount     = ref('6 500 000')
const downPaymentPct = ref('30')

const employmentType = ref('Работа по найму')
const employmentTypeOptions = ['Работа по найму', 'Самозанятый', 'ИП', 'Собственник бизнеса', 'Пенсионер']

const incomeProof = ref('2-НДФЛ')
const incomeProofOptions = ['2-НДФЛ', 'Справка по форме банка', 'Выписка из ПФР', 'Декларация']

const banks = ref([
  { name: 'Сбербанк',    initial: 'С', color: '#21A038', checked: false,
    payment: '123 000 ₽', rate: '6%', pv: '1 800 000 ₽ / 30%',
    cost: '6 000 000 ₽', credit: '4 200 000 ₽', overpayment: '2 140 830 ₽',
    psk: '12,4%—21,82%', income: '150 000 ₽/мес', insurance: 'от 120 000 ₽/год',
    programTitle: 'Семейная ипотека на 30 лет',
    programDesc: 'Льготная программа, доступная семьям с детьми до 6 лет и семьям, воспитывающим ребёнка с инвалидностью. Продлена до 2030 года.',
    programName: 'Семейная ипотека ~ Акционная стоимость: 6 150 000, процент повышения: 5%' },
  { name: 'Росбанк Дом', initial: 'Р', color: '#6B21A8', checked: false,
    payment: '167 890 ₽', rate: '20%', pv: '1 800 000 ₽ / 30%',
    cost: '6 000 000 ₽', credit: '4 200 000 ₽', overpayment: '3 890 830 ₽',
    psk: '19,1%—23,5%', income: '200 000 ₽/мес', insurance: 'от 90 000 ₽/год',
    programTitle: 'Стандартная ипотека на 30 лет',
    programDesc: 'Базовая ипотечная программа без государственного субсидирования. Доступна для широкого круга заёмщиков.',
    programName: 'Стандарт ~ Базовая ставка: 20%, срок: 30 лет' },
  { name: 'Тинькофф',    initial: 'Т', color: '#FFDD2D', lightInitial: true, checked: false,
    payment: '210 500 ₽', rate: '15%', pv: '1 500 000 ₽ / 25%',
    cost: '5 500 000 ₽', credit: '3 500 000 ₽', overpayment: '2 200 000 ₽',
    psk: '14,3%—17,8%', income: '180 000 ₽/мес', insurance: 'от 80 000 ₽/год',
    programTitle: 'Льготная ипотека на 25 лет',
    programDesc: 'Программа с государственной поддержкой для покупки жилья в новостройках. Ставка субсидируется государством.',
    programName: 'Льготная ~ Субсидированная ставка: 15%, взнос от 25%' },
  { name: 'Альфа-Банк',  initial: 'А', color: '#EF3124', checked: false,
    payment: '140 000 ₽', rate: '22%', pv: '1 600 000 ₽ / 35%',
    cost: '7 000 000 ₽', credit: '5 000 000 ₽', overpayment: '2 800 000 ₽',
    psk: '21,0%—24,9%', income: '220 000 ₽/мес', insurance: 'от 110 000 ₽/год',
    programTitle: 'Стандартная ипотека на 30 лет',
    programDesc: 'Ипотека на первичное и вторичное жильё. Повышенный первоначальный взнос снижает итоговую переплату.',
    programName: 'Стандарт Альфа ~ Ставка: 22%, ПВ от 35%' },
  { name: 'РТС',         initial: 'Р', color: '#1DAB3B', checked: false,
    payment: '185 300 ₽', rate: '17%', pv: '1 700 000 ₽ / 28%',
    cost: '6 200 000 ₽', credit: '4 000 000 ₽', overpayment: '3 210 000 ₽',
    psk: '16,2%—19,4%', income: '170 000 ₽/мес', insurance: 'от 95 000 ₽/год',
    programTitle: 'Ипотека РТС на 30 лет',
    programDesc: 'Программа для покупки жилья в аккредитованных жилых комплексах. Специальные условия для зарплатных клиентов.',
    programName: 'РТС Стандарт ~ Ставка: 17%, ПВ от 28%' },
  { name: 'ВТБ',         initial: 'В', color: '#009FDF', checked: false,
    payment: '90 750 ₽',  rate: '19%', pv: '1 900 000 ₽ / 32%',
    cost: '6 500 000 ₽', credit: '4 700 000 ₽', overpayment: '2 950 000 ₽',
    psk: '18,3%—22,1%', income: '160 000 ₽/мес', insurance: 'от 100 000 ₽/год',
    programTitle: 'Ипотека ВТБ на 30 лет',
    programDesc: 'Ипотечная программа для покупки квартиры в новостройке или на вторичном рынке. Доступна клиентам с хорошей кредитной историей.',
    programName: 'ВТБ Базовая ~ Ставка: 19%, ПВ от 32%' },
  { name: 'МКБ',         initial: 'М', color: '#034F96', checked: false,
    payment: '112 600 ₽', rate: '21%', pv: '1 750 000 ₽ / 27%',
    cost: '5 800 000 ₽', credit: '3 800 000 ₽', overpayment: '3 150 000 ₽',
    psk: '20,1%—23,8%', income: '190 000 ₽/мес', insurance: 'от 85 000 ₽/год',
    programTitle: 'Ипотека МКБ на 30 лет',
    programDesc: 'Ипотека на приобретение жилья на первичном рынке. Минимальный первоначальный взнос от 27%.',
    programName: 'МКБ Стандарт ~ Ставка: 21%, ПВ от 27%' },
])
</script>
