<template>
  <div class="month">
    <!-- Calendar Type Tabs -->
    <div class="calendar-tabs display-flex mb-2">
      <button
        class="tab-btn flex-1 p-2 font-700 text-size-12"
        :class="{ 'tab-active': calendarType === 'gregorian' }"
        @click="calendarType = 'gregorian'"
      >
        {{ locale === 'fa-IR' ? 'میلادی' : 'Gregorian' }}
      </button>
      <button
        class="tab-btn flex-1 p-2 font-700 text-size-12"
        :class="{ 'tab-active': calendarType === 'jalali' }"
        @click="calendarType = 'jalali'"
      >
        {{ locale === 'fa-IR' ? 'شمسی' : 'Jalali' }}
      </button>
    </div>

    <div class="month-header flex-1 display-flex-column pb-2">
      <div class="month-title text-center p-2 font-700 mb-5 mt-5">
        <span class="text-primary mr-5">{{ monthName }}</span> {{ yearName }}
      </div>
      <div class="display-flex font-400 text-size-12 text-muted">
        <div v-for="dayName in dayNames" class="text-center flex-1">{{ dayName }}</div>
      </div>
    </div>

    <table class="w-100 funky-calendar">
      <tbody>
      <tr v-for="week in calendar">
        <dashboard-calendar-month-day
          v-for="day in week"
          :day="day"
          :month="month"
          :is-visible="getIsVisible(day)"
          :calendar-type="calendarType"
        />
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { capitalize, computed, ref } from 'vue'
import {
  addDays, eachDayOfInterval, endOfWeek, format,
  startOfMonth, startOfWeek,
} from 'date-fns'
import { faIR, enUS } from 'date-fns/locale'

// Jalali conversion utility (pure JS, no extra deps)
function toJalali(gy, gm, gd) {
  const g_days_in_month = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
  const j_days_in_month = [31, 31, 31, 31, 31, 31, 30, 30, 30, 30, 30, 29]

  let jy, jm, jd, i, gy2, days

  gy2 = (gm > 2) ? (gy + 1) : gy
  days = 355666
    + (365 * gy)
    + Math.floor((gy2 + 3) / 4)
    - Math.floor((gy2 + 99) / 100)
    + Math.floor((gy2 + 399) / 400)
    + gd
  for (i = 0; i < gm - 1; i++) days += g_days_in_month[i]

  jy = -1595 + (33 * Math.floor(days / 12053))
  days %= 12053
  jy += 4 * Math.floor(days / 1461)
  days %= 1461

  if (days > 365) {
    jy += Math.floor((days - 1) / 365)
    days = (days - 1) % 365
  }

  jm = 0
  for (i = 0; i < 11; i++) {
    if (days < j_days_in_month[i]) break
    days -= j_days_in_month[i]
    jm++
  }
  jd = days + 1

  return { jy, jm: jm + 1, jd }
}

const JALALI_MONTH_NAMES = [
  'فروردین','اردیبهشت','خرداد','تیر','مرداد','شهریور',
  'مهر','آبان','آذر','دی','بهمن','اسفند'
]
const JALALI_DAY_NAMES_SHORT = ['ش','ی','د','س','چ','پ','ج']
const GREGORIAN_DAY_NAMES_SHORT_FA = ['ش','ی','د','س','چ','پ','ج']

const { locale } = useI18n()

const props = defineProps({
  start: {},
  end: {},
})


const month = computed(() => startOfMonth(props.start))

const calendarLocale = computed(() => {
  return locale.value === 'fa-IR' ? faIR : enUS
})

const calendarType = ref(locale.value === 'fa-IR' ? "jalali" : "gregorian")

// Day names row
const dayNames = computed(() => {
  if (calendarType.value === 'jalali') {
    // Jalali week starts Saturday
    return ['ش', 'ی', 'د', 'س', 'چ', 'پ', 'ج']
  }
  return Array.from({ length: 7 }, (_, i) => {
    const date = addDays(startOfWeek(new Date(), { locale: calendarLocale.value }), i)
    return capitalize(format(date, 'EEEEEE', { locale: calendarLocale.value }))
  })
})

// Month name
const monthName = computed(() => {
  if (calendarType.value === 'jalali') {
    const { jm } = toJalali(props.start.getFullYear(), props.start.getMonth() + 1, props.start.getDate())
    return JALALI_MONTH_NAMES[jm - 1]
  }
  return capitalize(format(props.start, 'MMMM', { locale: calendarLocale.value }))
})

// Year name
const yearName = computed(() => {
  if (calendarType.value === 'jalali') {
    const { jy } = toJalali(props.start.getFullYear(), props.start.getMonth() + 1, props.start.getDate())
    return jy
  }
  return props.start.getFullYear()
})

// Calendar grid — for Jalali we shift week start to Saturday (weekStartsOn: 6)
const calendar = computed(() => {
  const weekOptions = calendarType.value === 'jalali' ? { weekStartsOn: 6 } : { locale: calendarLocale.value }
  const start = startOfWeek(props.start, weekOptions)
  const end = endOfWeek(props.end, weekOptions)
  const days = eachDayOfInterval({ start, end })
  return days.reduce((weeks, day, i) => {
    if (i % 7 === 0) weeks.push([])
    weeks[weeks.length - 1].push(day)
    return weeks
  }, [])
})

const getIsVisible = (day) => {
  return day >= props.start && day <= props.end
}
</script>

<style scoped>
.calendar-tabs {
  border-bottom: 2px solid var(--border-color, #e5e7eb);
}

.tab-btn {
  background: none;
  border: none;
  border-bottom: 2px solid transparent;
  margin-bottom: -2px;
  cursor: pointer;
  color: var(--text-muted, #6b7280);
  transition: color 0.2s, border-color 0.2s;
}

.tab-btn:hover {
  color: var(--text-primary, #111827);
}

.tab-active {
  color: var(--color-primary, #3b82f6) !important;
  border-bottom-color: var(--color-primary, #3b82f6) !important;
}
</style>