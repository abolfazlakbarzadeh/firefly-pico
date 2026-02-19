<template>
  <van-cell-group inset>
    <div class="van-cell-group-title">{{ $t('dashboard.expenses_week') }}:</div>
    <div class="display-flex gap-2">
      <div class="flex-1" />

      <bar-chart-item-vertical v-for="bar in barsList" v-bind="bar" @click="onClick(bar)" class="cursor-pointer" />

      <div class="flex-1" />
    </div>
  </van-cell-group>
</template>
<script setup>
import { eachDayOfInterval, format, startOfDay, subDays } from 'date-fns'
import { capitalize, get } from 'lodash'
import RouteConstants from '~/constants/RouteConstants.js'
import Transaction from '~/models/Transaction.js'
import { getExcludedTransactionUrl } from '~/utils/DashboardUtils.js'
import { computed } from 'vue'
import { enUS, faIR } from 'date-fns/locale'

const dataStore = useDataStore()
const { locale } = useI18n()

const barsList = computed(() => {
  const amountsList = Object.values(dataStore.dashboardExpenseByDay)
  const maxAmount = Math.max(...amountsList)

  const calendarLocale = locale.value === 'fa-IR' ? faIR : enUS
  const daysList = eachDayOfInterval({
    start: subDays(new Date(), 7),
    end: startOfDay(new Date()),
  })
  return daysList.map((date) => {
    const weekdayName = capitalize(format(date, 'E', { locale: calendarLocale }))
    const amount = get(dataStore.dashboardExpenseByDay, DateUtils.dateToString(date), 0)
    const percent = (amount / maxAmount) * 100

    return {
      date: date,
      label: weekdayName,
      value: formatNumberForDashboard(amount),
      percent: percent,
    }
  })
})

const onClick = async (bar) => {
  let excludedUrl = getExcludedTransactionUrl()
  let filters = [
    TransactionFilterUtils.filters.dateAfter.toUrl(bar.date),
    TransactionFilterUtils.filters.dateBefore.toUrl(bar.date),
    TransactionFilterUtils.filters.transactionType.toUrl(Transaction.types.expense),
  ].join('&')
  await navigateTo(`${RouteConstants.ROUTE_TRANSACTION_LIST}?${filters}${excludedUrl}`)
}
</script>
